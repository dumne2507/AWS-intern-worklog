---
title: "Triển khai Backend AWS Lambda"
date: 2026-11-27
weight: 4
chapter: false
pre: "<b>5.4. </b>"
---

## 1. Khởi tạo hàm Lambda

1. Truy cập **AWS Lambda → Create function → Author from scratch**.
2. Cấu hình thông số cơ bản:
   * **Function name:** `RecipeSearchFunction`
   * **Runtime:** `Python 3.12`
   * **Execution role:** Chọn **Use an existing role** → `RecipesLambdaRole`.
3. Vào **Configuration → General configuration → Edit**, tăng **Timeout** lên `15 seconds`.
4. Vào **Configuration → Environment variables**, thêm 2 biến môi trường:
   * `DYNAMODB_TABLE`: `RecipesTable`
   * `BEDROCK_MODEL_ID`: `apac.amazon.nova-micro-v1:0`

---

## 2. Phân tích kiến trúc & chức năng từng phần trong `lambda_function.py`

Mã nguồn `lambda_function.py` được chia thành **6 khối chức năng chính**, đảm nhiệm toàn bộ luồng từ tiếp nhận HTTP request, xác thực dữ liệu, kết nối 3 dịch vụ lõi của AWS (**DynamoDB**, **Amazon Location Service**, **Amazon Bedrock**) đến trả phản hồi chuẩn JSON về giao diện.

### 2.1. Khởi tạo cấu hình & AWS SDK Clients
```python
import base64
import json
import logging
import math
import os
import random
import unicodedata
import urllib.error
import urllib.request
from decimal import Decimal
from urllib.parse import parse_qs, quote

import boto3
from botocore.exceptions import ClientError

logger = logging.getLogger()
logger.setLevel(logging.INFO)

REGION = os.environ.get("AWS_REGION", "ap-southeast-1")
BEDROCK_REGION = os.environ.get("BEDROCK_REGION", REGION).strip()
TABLE_NAME = os.environ.get("DYNAMODB_TABLE", "RecipesTable").strip()
BEDROCK_MODEL_ID = os.environ.get("BEDROCK_MODEL_ID", "apac.amazon.nova-micro-v1:0").strip()
BEDROCK_API_KEY = os.environ.get("BEDROCK_API_KEY", "").strip()

MAX_INPUT_LENGTH = 200
MAX_KEYWORDS = 10

dynamodb = boto3.resource("dynamodb", region_name=REGION)
table = dynamodb.Table(TABLE_NAME)
geo_places_client = boto3.client("geo-places", region_name=REGION)
bedrock_client = boto3.client("bedrock-runtime", region_name=BEDROCK_REGION)

CORS_HEADERS = {
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Headers": "Content-Type,Authorization",
    "Access-Control-Allow-Methods": "GET,POST,OPTIONS",
    "Content-Type": "application/json; charset=utf-8",
}
```
* **Chức năng chính:**
  * Đọc các biến môi trường (`AWS_REGION`, `BEDROCK_REGION`, `DYNAMODB_TABLE`, `BEDROCK_MODEL_ID`, `BEDROCK_API_KEY`) để tách biệt cấu hình khỏi mã nguồn, dễ thay đổi mà không cần sửa code.
  * Khởi tạo sẵn các kết nối (`boto3.resource` / `boto3.client`) ở phạm vi toàn cục (global scope) gồm: `table` (DynamoDB), `geo_places_client` (Location Service Places v2) và `bedrock_client` (Bedrock Runtime). Việc đặt ngoài `lambda_handler` giúp tái sử dụng kết nối qua các lần gọi **Warm Start**, giảm độ trễ phản hồi.
  * Thiết lập `CORS_HEADERS` cho phép trình duyệt từ tên miền CloudFront gọi API (`GET`, `POST`, `OPTIONS`) mà không bị chặn bởi chính sách bảo mật trình duyệt.

### 2.2. Nhóm hàm tiện ích, Chuẩn hóa & Bảo mật đầu vào
```python
def json_default(value):
    if isinstance(value, Decimal):
        return int(value) if value % 1 == 0 else float(value)
    raise TypeError(f"Unsupported JSON value: {type(value).__name__}")


def response(status_code, body):
    return {
        "statusCode": status_code,
        "headers": CORS_HEADERS,
        "body": json.dumps(body, ensure_ascii=False, default=json_default),
    }

```

  * **`json_default()` & `response()`:** Chuyển đổi kiểu số `Decimal` của DynamoDB sang `int`/`float` chuẩn JSON và đóng gói phản hồi HTTP kèm mã trạng thái (`200`, `400`, `500`) và header CORS.

```python
def normalize(value):
    text = str(value or "").strip().lower()
    text = text.replace("đ", "d")
    text = unicodedata.normalize("NFD", text)
    return "".join(ch for ch in text if unicodedata.category(ch) != "Mn")

```

  * **`normalize()`:** Chuẩn hóa chuỗi tiếng Việt về chữ thường không dấu (chuyển `đ -> d`, tách dấu Unicode `NFD`) giúp việc so khớp nguyên liệu và từ khóa tâm trạng chính xác ngay cả khi người dùng gõ thiếu dấu.

```python
def csv_values(value, field_name):
    if not value:
        return []
    if len(value) > MAX_INPUT_LENGTH:
        raise ValueError(f"Tham số {field_name} vượt quá {MAX_INPUT_LENGTH} ký tự.")
    values = [part.strip() for part in value.split(",") if part.strip()]
    if any(len(part) > 60 for part in values):
        raise ValueError(f"Mỗi giá trị trong {field_name} tối đa 60 ký tự.")
    return [v for v in values[:MAX_KEYWORDS] if not any(c in v for c in "<>{}$`\\;")]

```

  * **`csv_values()`:** Kiểm tra độ dài tối đa (`MAX_INPUT_LENGTH = 200`), giới hạn tối đa 10 từ khóa (`MAX_KEYWORDS = 10`) và loại bỏ các ký tự đặc biệt nguy hiểm (`<>{}$\`\\;`) để chống injection.

```python
def query_params(event):
    params = event.get("queryStringParameters") or {}
    if params:
        return params
    raw = event.get("rawQueryString", "")
    return {k: v[0] for k, v in parse_qs(raw).items() if v}


def parse_json_body(event):
    body = event.get("body") or "{}"
    if event.get("isBase64Encoded"):
        body = base64.b64decode(body).decode("utf-8")
    try:
        return json.loads(body)
    except (TypeError, json.JSONDecodeError):
        raise ValueError("Dữ liệu gửi lên không phải JSON hợp lệ.")

```

  * **`query_params()` & `parse_json_body()`:** Chuẩn hóa cách đọc tham số từ cả hai định dạng sự kiện **API Gateway REST API v1** và **HTTP API v2** (bao gồm cả giải mã Base64 nếu có).

### 2.3. Truy vấn & Lọc công thức nấu ăn (Amazon DynamoDB)

```python
def get_all_recipes():
    items = []
    kwargs = {}
    while True:
        result = table.scan(**kwargs)
        items.extend(result.get("Items", []))
        last_key = result.get("LastEvaluatedKey")
        if not last_key:
            break
        kwargs = {"ExclusiveStartKey": last_key}
    items.sort(key=lambda x: str(x.get("RecipeID", "")))
    logger.info("Loaded %s recipes from %s", len(items), TABLE_NAME)
    return items


```
  * **`get_all_recipes()`:** Quét toàn bộ dữ liệu từ bảng `RecipesTable` (tự động xử lý phân trang `LastEvaluatedKey` nếu dữ liệu lớn) và sắp xếp theo `RecipeID`.

```python
def matches(recipe, filters):
    ingredients = recipe.get("Ingredients") or []
    allergens = recipe.get("Allergens") or []
    nutrition = recipe.get("Nutrition") or {}

    if filters["ingredients"] and not has_any(ingredients, filters["ingredients"]):
        return False
    if filters["health"] and not has_tag(recipe, "HealthTags", filters["health"]):
        return False
    if filters["need"] and not (
        has_tag(recipe, "HealthTags", filters["need"])
        or has_tag(recipe, "DietTags", filters["need"])
    ):
        return False
    if filters["mood"] and not has_tag(recipe, "MoodTags", filters["mood"]):
        return False
    if filters["allergies"] and (
        has_any(ingredients, filters["allergies"])
        or has_any(allergens, filters["allergies"])
    ):
        return False
    if filters["avoid"] and has_any(ingredients, filters["avoid"]):
        return False

    numeric_rules = (
        ("maxTime", "CookingTimeMinutes"),
        ("maxBudget", "EstimatedCostVND"),
    )
    for param, field in numeric_rules:
        if filters[param] is not None:
            try:
                value = float(recipe.get(field))
            except (TypeError, ValueError):
                return False
            if value > filters[param]:
                return False

    for param, field in (("maxSodium", "SodiumMg"), ("maxCarbs", "CarbsG")):
        if filters[param] is not None:
            try:
                value = float(nutrition[field])
            except (KeyError, TypeError, ValueError):
                return False
            if value > filters[param]:
                return False
    return True


```

  * **`matches()`:** Áp dụng bộ lọc đa điều kiện lên từng công thức: nguyên liệu sẵn có (`ingredients`), dị ứng (`allergies`), nguyên liệu cần tránh (`avoid`), chế độ ăn/sức khỏe (`health`, `need`), tâm trạng (`mood`), cùng các ngưỡng số học (`maxTime`, `maxBudget`, `maxSodium`, `maxCarbs`).

```python
def ingredient_score(recipe, keywords):
    ingredients = [normalize(v) for v in (recipe.get("Ingredients") or [])]
    return sum(1 for term in keywords if any(normalize(term) in ing for ing in ingredients))

def handle_recipe_search(params, context):
    filters = {
        "ingredients": csv_values(params.get("ingredients", ""), "ingredients"),
        "allergies": csv_values(params.get("allergies", ""), "allergies"),
        "avoid": csv_values(params.get("avoid", ""), "avoid"),
        "health": (params.get("health") or "").strip(),
        "need": (params.get("need") or "").strip(),
        "mood": (params.get("mood") or "").strip(),
        "maxTime": None,
        "maxBudget": None,
        "maxSodium": None,
        "maxCarbs": None,
    }
    allowed_tags = {
        "health": {"low_sodium", "controlled_carb", "low_saturated_fat", "light", "high_protein"},
        "need": {"high_protein", "vegetarian", "vegan", "low_calorie"},
        "mood": {"comforting", "fresh", "energizing", "light"},
    }
    for key, allowed in allowed_tags.items():
        if filters[key] and filters[key] not in allowed:
            return response(400, {"error": f"Giá trị bộ lọc {key} không hợp lệ."})

    numeric_limits = {
        "maxTime": (600, 0),
        "maxBudget": (10000000, 0),
        "maxSodium": (100000, 0),
        "maxCarbs": (1000, 0),
    }
    for key, (upper, lower) in numeric_limits.items():
        raw = params.get(key)
        if raw not in (None, ""):
            try:
                value = float(raw)
            except (TypeError, ValueError):
                return response(400, {"error": f"Tham số {key} phải là số."})
            if value < lower or value > upper:
                return response(400, {"error": f"Tham số {key} nằm ngoài giới hạn cho phép."})
            filters[key] = value

    logger.info(
        "RequestId=%s filters=%s random=%s",
        getattr(context, "aws_request_id", "local"),
        filters,
        params.get("random") == "true",
    )
    recipes = get_all_recipes()
    matched = [item for item in recipes if matches(item, filters)]
    matched.sort(key=lambda item: ingredient_score(item, filters["ingredients"]), reverse=True)
    if params.get("random", "false").lower() == "true":
        random.shuffle(matched)
        matched = matched[:6]

    return response(200, {
        "recipes": matched,
        "count": len(matched),
        "filters": filters,
        "source": "dynamodb",
    })


```
  
  * **`ingredient_score()` & `handle_recipe_search()`:** Chấm điểm mức độ khớp nguyên liệu để ưu tiên đưa các món khớp nhiều nguyên liệu nhất lên đầu danh sách trả về cho tuyến đường `GET /search`.

### 2.4. Định vị & Tìm điểm mua nguyên liệu (Amazon Location Service)

```python
def haversine_meters(lat1, lon1, lat2, lon2):
    r = 6371000
    phi1, phi2 = math.radians(lat1), math.radians(lat2)
    dphi = math.radians(lat2 - lat1)
    dlambda = math.radians(lon2 - lon1)
    a = math.sin(dphi / 2) ** 2 + math.cos(phi1) * math.cos(phi2) * math.sin(dlambda / 2) ** 2
    return int(2 * r * math.atan2(math.sqrt(a), math.sqrt(1 - a)))


def handle_nearby_stores(params):
    ingredients = (params.get("ingredients") or "thực phẩm tươi").strip()
    if len(ingredients) > MAX_INPUT_LENGTH:
        raise ValueError(f"Tham số ingredients vượt quá {MAX_INPUT_LENGTH} ký tự.")

    tier = (params.get("tier") or "mid").strip().lower()
    if tier not in ("low", "mid", "high"):
        tier = "mid"

    address = (params.get("address") or "").strip()
    if len(address) > 200:
        raise ValueError("Địa chỉ tìm kiếm tối đa 200 ký tự.")

    try:
        lat = float(params.get("lat", 10.7769))
        lng = float(params.get("lng", 106.7009))
    except (TypeError, ValueError):
        lat, lng = 10.7769, 106.7009

    tier_config = {
        "low": "Ngân sách tiết kiệm (< 35.000đ): Ưu tiên chợ truyền thống và cửa hàng tạp hóa gần bạn.",
        "mid": "Ngân sách vừa phải (35.000đ – 70.000đ): Ưu tiên Bách Hóa Xanh, WinMart+, Co.op Food, Satrafood.",
        "high": "Ngân sách thoải mái (> 70.000đ): Ưu tiên siêu thị lớn (Co.opmart, WinMart, Lotte Mart, GO!, Tops Market).",
    }
    advice = tier_config[tier]
    resolved_address = ""

    if address:
        query_addr = (
            address
            if ("viet nam" in normalize(address) or "vietnam" in normalize(address))
            else f"{address}, Việt Nam"
        )
        geo_items = []
        try:
            geo_res = geo_places_client.geocode(
                QueryText=query_addr,
                BiasPosition=[lng, lat],
                MaxResults=1,
                Language="vi",
                Filter={"IncludeCountries": ["VNM"]},
            )
            geo_items = geo_res.get("ResultItems", [])
        except Exception:
            logger.exception("Geocode failed for address=%r, trying search_text", query_addr)

        if not geo_items:
            try:
                st_res = geo_places_client.search_text(
                    QueryText=query_addr,
                    BiasPosition=[lng, lat],
                    MaxResults=1,
                    Language="vi",
                    Filter={"IncludeCountries": ["VNM"]},
                )
                geo_items = st_res.get("ResultItems", [])
            except Exception:
                logger.exception("SearchText address lookup failed for address=%r", query_addr)

        if not geo_items or not geo_items[0].get("Position"):
            return response(200, {
                "source": "amazon-location-geo-places",
                "advice": f"Không tìm thấy tọa độ cho địa chỉ '{address}'. Hãy thử ghi rõ tên đường, phường/quận, thành phố.",
                "selectedTier": tier,
                "origin": {"lat": lat, "lng": lng},
                "stores": [],
            })

        pos = geo_items[0]["Position"]
        lng, lat = float(pos[0]), float(pos[1])
        resolved_address = (geo_items[0].get("Address") or {}).get("Label") or geo_items[0].get("Title") or address
        logger.info("Resolved manual address=%r -> (%s, %s) [%s]", address, lat, lng, resolved_address)

    excluded_name_terms = (
        "famima", "familymart", "family mart", "circle k", "gs25",
        "7-eleven", "ministop", "cheers", "shop&go", "shop and go", "fumart",
        "nha hang", "quan an", "cafe", "coffee", "tra sua", "spa", "hotel",
        "dien may", "me va be", "baby", "kids", "pet", "thuoc", "pharmacy",
    )
    excluded_category_terms = (
        "restaurant", "cafe", "coffee", "fast food", "food court",
        "bar", "pub", "gas station", "pet store", "clothing",
        "hotel", "lodging",
    )

    queries_by_tier = {
        "low": ["chợ", "cửa hàng tạp hóa", "bách hóa"],
        "mid": ["Bách Hóa Xanh", "WinMart+", "Co.op Food", "Satrafood", "cửa hàng thực phẩm"],
        "high": ["Co.opmart", "WinMart", "Lotte Mart", "GO!", "Tops Market", "Emart", "Kingfoodmart", "siêu thị"],
    }

    try:
        stores_by_key = {}
        for search_term in queries_by_tier[tier]:
            try:
                text_result = geo_places_client.search_text(
                    QueryText=search_term,
                    BiasPosition=[lng, lat],
                    MaxResults=15,
                    Filter={"IncludeCountries": ["VNM"]},
                    Language="vi",
                )
            except Exception:
                logger.exception("SearchText failed for query=%r", search_term)
                continue

            for item in text_result.get("ResultItems", []):
                title = (item.get("Title") or "").strip()
                if not title:
                    continue
                title_norm = normalize(title)
                if any(term in title_norm for term in excluded_name_terms):
                    continue

                categories = item.get("Categories") or []
                category_names = [
                    normalize(category.get("LocalizedName") or category.get("Name") or "")
                    for category in categories
                ]
                if any(
                    excluded in category
                    for category in category_names
                    for excluded in excluded_category_terms
                ):
                    continue

                pos = item.get("Position")
                if not pos or len(pos) < 2:
                    continue
                store_lat, store_lng = float(pos[1]), float(pos[0])
                dist_m = haversine_meters(lat, lng, store_lat, store_lng)
                if dist_m > 10000:
                    continue

                addr_label = (item.get("Address") or {}).get("Label") or "Gần vị trí đã chọn"
                chain_names = [
                    chain.get("Name", "")
                    for chain in (item.get("BusinessChains") or [])
                    if chain.get("Name")
                ]
                category_label = ", ".join(
                    category.get("LocalizedName") or category.get("Name", "")
                    for category in categories
                    if category.get("LocalizedName") or category.get("Name")
                )

                dedup_key = (title_norm, round(store_lat, 4), round(store_lng, 4))
                if dedup_key in stores_by_key:
                    continue

                stores_by_key[dedup_key] = {
                    "name": title,
                    "address": addr_label,
                    "distanceMeters": dist_m,
                    "lat": store_lat,
                    "lng": store_lng,
                    "category": category_label,
                    "businessChains": chain_names,
                    "budgetGroup": tier,
                }

        selected_stores = sorted(stores_by_key.values(), key=lambda x: x["distanceMeters"])

        return response(200, {
            "source": "amazon-location-geo-places",
            "advice": advice,
            "resolvedAddress": resolved_address,
            "selectedTier": tier,
            "origin": {"lat": lat, "lng": lng},
            "searchRadiusMeters": 10000,
            "stores": selected_stores[:20],
        })
    except Exception:
        logger.exception("Amazon Location nearby stores failed")
        return response(200, {
            "source": "fallback",
            "advice": advice,
            "resolvedAddress": resolved_address,
            "origin": {"lat": lat, "lng": lng},
            "stores": [],
        })

```

  * **Xác định tọa độ tâm (`lat`, `lng`):** Nếu người dùng nhập địa chỉ thủ công (`address`), hàm gọi `geo_places_client.geocode()` (và dự phòng bằng `search_text()`) để chuyển đổi chuỗi địa chỉ tiếng Việt sang tọa độ thực tế; nếu để trống thì dùng trực tiếp tọa độ GPS từ trình duyệt.
  * **Truy vấn chuỗi bán lẻ theo ngân sách (`tier`):** Ánh xạ mức ngân sách (`low`, `mid`, `high`) sang các nhóm từ khóa cửa hàng phù hợp (chợ/tạp hóa, Bách Hóa Xanh/WinMart+/Co.op Food, hoặc Co.opmart/Lotte Mart/GO!), sau đó gọi `geo_places_client.search_text()` với `BiasPosition` quanh tọa độ người dùng.
  * **Lọc nhiễu & Tính khoảng cách (`haversine_meters`):** Loại bỏ các địa điểm không phải nơi bán thực phẩm tươi sống (quán cà phê, nhà hàng, spa, trạm xăng...), loại bỏ kết quả trùng tọa độ, tính khoảng cách đường chim bay chính xác theo mét trong bán kính 10 km và sắp xếp từ gần nhất đến xa nhất cho tuyến đường `GET /nearby-stores`.

### 2.5. Chatbot Đầu bếp AI Tâm lý (Amazon Bedrock)

```python
def find_recipes_by_mood_weather(msg, items):
    norm_msg = normalize(msg)

    # 1. Ưu tiên cao nhất: Khớp trực tiếp tên món ăn (chính xác hoặc gần đúng)
    exact_name_matches = []
    partial_name_matches = []
    for r in items:
        r_name = normalize(r.get("Name", ""))
        if not r_name:
            continue
        if r_name == norm_msg or r_name in norm_msg:
            exact_name_matches.append(r)
        elif len(norm_msg) >= 4 and norm_msg in r_name:
            partial_name_matches.append(r)

    if exact_name_matches:
        # Ưu tiên tên món dài hơn (khớp chi tiết hơn) lên đầu, không shuffle
        exact_name_matches.sort(key=lambda r: len(normalize(r.get("Name", ""))), reverse=True)
        return exact_name_matches, True

    if partial_name_matches:
        partial_name_matches.sort(key=lambda r: len(normalize(r.get("Name", ""))))
        return partial_name_matches, True

    # 2. Ưu tiên thứ hai: Khớp theo nguyên liệu xuất hiện trong câu nhắn (xếp theo số nguyên liệu khớp)
    ing_scored = []
    for r in items:
        ings = [normalize(ing) for ing in (r.get("Ingredients") or []) if normalize(ing)]
        matched_ings = [ing for ing in ings if ing in norm_msg]
        if matched_ings:
            # Ưu tiên món khớp nhiều nguyên liệu hơn
            ing_scored.append((len(matched_ings), r))

    if ing_scored:
        ing_scored.sort(key=lambda x: x[0], reverse=True)
        top_count = ing_scored[0][0]
        top_ing_recipes = [r for cnt, r in ing_scored if cnt == top_count]
        random.shuffle(top_ing_recipes)
        rest_ing_recipes = [r for cnt, r in ing_scored if cnt < top_count]
        return top_ing_recipes + rest_ing_recipes, False

    # 3. Ưu tiên thứ ba: Khớp theo tâm trạng / thời tiết / chế độ ăn
    target_moods = []
    target_cats = []

    if any(k in norm_msg for k in ["mua", "lanh", "se lanh", "buon", "stress", "ap luc", "am bung", "co don"]):
        target_moods.append("comforting")
        target_cats.extend(["soup", "main"])
    if any(k in norm_msg for k in ["nong", "oi buc", "nang", "ngan", "nhe bung", "giam can", "thanh dam", "mat"]):
        target_moods.extend(["fresh", "light"])
        target_cats.extend(["salad", "side", "soup"])
    if any(k in norm_msg for k in ["met", "duoi", "ue oai", "can nang luong", "doi", "tap gym", "vui"]):
        target_moods.append("energizing")
        target_cats.append("main")

    scored = []
    for r in items:
        score = 0
        r_moods = r.get("MoodTags") or []
        if any(m in r_moods for m in target_moods):
            score += 2
        if r.get("Category") in target_cats:
            score += 1
        if ("chay" in norm_msg or "thanh dam" in norm_msg) and "vegetarian" in (r.get("DietTags") or []):
            score += 3
        if score > 0:
            scored.append((score, r))

    scored.sort(key=lambda x: x[0], reverse=True)
    if not scored:
        return [], False

    top_score = scored[0][0]
    top_group = [r for score, r in scored if score >= max(1, top_score - 1)]
    random.shuffle(top_group)
    rest_group = [r for score, r in scored if score < max(1, top_score - 1)]
    return top_group + rest_group, False

```

  * **`find_recipes_by_mood_weather()`:** Phân tích ngữ nghĩa câu nhắn của người dùng (ví dụ: *mưa lạnh, buồn, stress, nắng nóng, mệt mỏi*) để ánh xạ sang các nhóm `MoodTags` (`comforting`, `fresh`, `light`, `energizing`) và loại món ăn tương ứng, đồng thời loại trừ các món đã gợi ý ở lượt trước (`excludeIds`) để tránh lặp lại.

```python
def handle_chat(event):
    payload = parse_json_body(event)
    user_message = str(payload.get("message") or "").strip()
    if not user_message:
        raise ValueError("Vui lòng nhập tâm trạng, thời tiết hoặc câu hỏi của bạn.")
    if len(user_message) > 500:
        raise ValueError("Câu hỏi tối đa 500 ký tự.")

    items = get_all_recipes()
    shuffled_items = list(items)
    random.shuffle(shuffled_items)

    exclude_ids = set(payload.get("excludeIds") or [])

    matched_recipes, is_explicit_dish = find_recipes_by_mood_weather(user_message, items)
    if is_explicit_dish:
        candidate_recipes = matched_recipes
    else:
        fresh_matches = [r for r in matched_recipes if r.get("RecipeID") not in exclude_ids]
        candidate_recipes = fresh_matches if fresh_matches else matched_recipes

    suggested_id = candidate_recipes[0].get("RecipeID") if candidate_recipes else None
    suggested_name = candidate_recipes[0].get("Name") if candidate_recipes else None

    menu_context = "\n".join([
        f"- [{r.get('RecipeID')}] {r.get('Name')}: loại {r.get('Category')}, "
        f"tâm trạng phù hợp: {', '.join(r.get('MoodTags') or [])}, "
        f"chi phí ~{r.get('EstimatedCostVND')}đ, nấu {r.get('TotalTimeMinutes') or r.get('CookingTimeMinutes', 20)} phút, "
        f"nguyên liệu: {', '.join(r.get('Ingredients') or [])}"
        for r in shuffled_items
    ])

    avoid_note = ""
    if exclude_ids:
        avoid_names = [r.get("Name") for r in items if r.get("RecipeID") in exclude_ids]
        if avoid_names:
            avoid_note = f"\n4. KHÔNG gợi ý lại các món người dùng vừa xem ở lượt trước: {', '.join(avoid_names)}. Hãy chọn món khác trong danh sách."

    primary_hint = (
        f"2. Gợi ý món **{suggested_name}** (hoặc 1–2 món phù hợp khác TỪ DANH SÁCH BÊN DƯỚI, ghi rõ tên món chính xác, chi phí và thời gian nấu), giải thích ngắn gọn vì sao món đó hợp với tâm trạng/thời tiết hiện tại.\n"
        if suggested_name
        else "2. Gợi ý 1–2 món phù hợp nhất TỪ DANH SÁCH BÊN DƯỚI (ghi rõ tên món chính xác, chi phí và thời gian nấu), giải thích ngắn gọn vì sao món đó hợp với tâm trạng/thời tiết hiện tại.\n"
    )

    system_prompt = (
        "Bạn là Đầu Bếp AI Tâm Lý trên website hướng dẫn dạy nấu ăn Việt Nam. "
        "Người dùng có thể hỏi gợi ý món ăn theo TÂM TRẠNG (vui, buồn, stress, mệt mỏi, cần năng lượng, ăn thanh đạm), "
        "THỜI TIẾT (trời mưa, trời lạnh, nắng nóng, oi bức), ngân sách hoặc nguyên liệu sẵn có.\n"
        "Quy tắc trả lời:\n"
        "1. Đồng cảm nhẹ nhàng với tâm trạng hoặc thời tiết của người dùng (1 câu ngắn).\n"
        f"{primary_hint}"
        "3. Khích lệ người dùng bấm nút xem hướng dẫn từng bước bên dưới."
        f"{avoid_note}\n"
        "Trả lời bằng tiếng Việt tự nhiên, ấm áp, đa dạng cách diễn đạt, ngắn gọn (tối đa 5 câu).\n\n"
        f"Danh sách công thức hiện có:\n{menu_context}"
    )

    try:
        if BEDROCK_API_KEY:
            encoded_model = quote(BEDROCK_MODEL_ID, safe="")
            endpoint = f"https://bedrock-runtime.{BEDROCK_REGION}.amazonaws.com/model/{encoded_model}/converse"
            req_body = json.dumps({
                "messages": [{"role": "user", "content": [{"text": user_message}]}],
                "system": [{"text": system_prompt}],
                "inferenceConfig": {"maxTokens": 450, "temperature": 0.9},
            }).encode("utf-8")
            req = urllib.request.Request(
                endpoint,
                data=req_body,
                headers={
                    "Authorization": f"Bearer {BEDROCK_API_KEY}",
                    "Content-Type": "application/json",
                },
                method="POST",
            )
            with urllib.request.urlopen(req, timeout=12) as resp:
                res = json.loads(resp.read().decode("utf-8"))
        else:
            res = bedrock_client.converse(
                modelId=BEDROCK_MODEL_ID,
                messages=[{"role": "user", "content": [{"text": user_message}]}],
                system=[{"text": system_prompt}],
                inferenceConfig={"maxTokens": 450, "temperature": 0.9},
            )

        reply_text = res["output"]["message"]["content"][0]["text"]
        return response(200, {
            "reply": reply_text,
            "suggestedRecipeId": suggested_id,
            "suggestedRecipeName": suggested_name,
            "source": "amazon-bedrock",
        })
    except urllib.error.HTTPError as http_exc:
        err_detail = http_exc.read().decode("utf-8", errors="ignore")
        logger.warning("Amazon Bedrock HTTP %s: %s", http_exc.code, err_detail)
    except Exception as exc:
        logger.warning("Amazon Bedrock fallback triggered: %s", exc)

    if candidate_recipes:
        top = candidate_recipes[0]
        second = candidate_recipes[1] if len(candidate_recipes) > 1 else None
        extra = f" hoặc <strong>{second.get('Name')}</strong> (~{second.get('EstimatedCostVND')}đ)" if second else ""
        reply = (
            f"Với thời tiết và tâm trạng hiện tại của bạn, một bữa ăn tự nấu sẽ giúp tinh thần dễ chịu hơn nhiều! "
            f"Mình gợi ý món <strong>{top.get('Name')}</strong> (~{top.get('EstimatedCostVND')}đ, "
            f"{top.get('TotalTimeMinutes') or top.get('CookingTimeMinutes', 20)} phút){extra}. "
            f"Bạn bấm vào nút bên dưới để xem hướng dẫn chi tiết từng bước nhé!"
        )
    else:
        reply = (
            "Dù tâm trạng hay thời tiết hôm nay thế nào, tự vào bếp nấu một món ngon cũng giúp thư giãn rất tốt! "
            "Bạn hãy cho mình biết bạn đang thích ăn món nóng hổi (canh/kho) hay món thanh mát (salad/xào) để mình chọn công thức chuẩn nhất nhé."
        )
    return response(200, {
        "reply": reply,
        "suggestedRecipeId": suggested_id,
        "suggestedRecipeName": suggested_name,
        "source": "fallback",
    })

```

  * **`handle_chat()`:** Đóng gói danh sách thực đơn hiện có từ DynamoDB vào `system_prompt`, gọi mô hình **Amazon Nova Micro** (`apac.amazon.nova-micro-v1:0`) qua `bedrock_client.converse()` (hoặc REST endpoint nếu dùng `BEDROCK_API_KEY`) để sinh câu trả lời đồng cảm, tự nhiên kèm mã món gợi ý (`suggestedRecipeId`) cho tuyến đường `POST /chat`. Có sẵn cơ chế **Fallback** trả về gợi ý định dạng chuẩn nếu Bedrock gặp sự cố mạng hoặc giới hạn quota.

### 2.6. Bộ điều phối trung tâm (`lambda_handler`)

```python
def lambda_handler(event, context):
    method = (
        event.get("httpMethod")
        or (event.get("requestContext", {}).get("http") or {}).get("method")
        or "GET"
    ).upper()
    raw_path = event.get("rawPath") or event.get("path") or ""
    route_key = event.get("routeKey") or ""

    if method == "OPTIONS":
        return response(200, {"message": "CORS OK"})

    try:
        if method == "POST" and ("chat" in route_key or raw_path.endswith("/chat")):
            return handle_chat(event)

        if method == "GET" and ("nearby-stores" in route_key or raw_path.endswith("/nearby-stores")):
            return handle_nearby_stores(query_params(event))

        if method == "GET":
            return handle_recipe_search(query_params(event), context)

        return response(405, {"error": "Phương thức HTTP không được hỗ trợ."})
    except ValueError as exc:
        return response(400, {"error": str(exc)})
    except ClientError:
        logger.exception("AWS service request failed")
        return response(500, {"error": f"Không truy cập được dịch vụ AWS. Kiểm tra tên bảng {TABLE_NAME} và IAM Role."})
    except Exception:
        logger.exception("Unhandled Lambda error")
        return response(500, {"error": "Lỗi nội bộ. Xem CloudWatch Logs để biết chi tiết."})
```

  * Là điểm vào (Entry Point) duy nhất của hàm Lambda, nhận mọi HTTP request từ API Gateway.
  * Xử lý nhanh yêu cầu preflight `OPTIONS` cho CORS.
  * Phân tuyến (routing) dựa trên phương thức HTTP và đường dẫn:
    * `POST .../chat` → `handle_chat()`
    * `GET .../nearby-stores` → `handle_nearby_stores()`
    * `GET .../search` → `handle_recipe_search()`
  * Bắt lỗi tập trung (`ValueError` trả về `400 Bad Request`, `ClientError` và ngoại lệ hệ thống ghi log ra CloudWatch và trả về `500 Internal Server Error` an toàn).

---

## 3. Bảng tóm tắt luồng xử lý

| Hàm / Khối chức năng | Dịch vụ AWS tương tác | Tuyến đường API | Đầu ra chính |
| :--- | :--- | :--- | :--- |
| `handle_recipe_search` | Amazon DynamoDB (`Scan`) | `GET /search` | Danh sách món ăn đã lọc & xếp hạng khớp nguyên liệu |
| `handle_nearby_stores` | Amazon Location (`Geocode`, `SearchText`) | `GET /nearby-stores` | Tọa độ tâm & danh sách cửa hàng trong 10 km theo ngân sách |
| `handle_chat` | DynamoDB + Amazon Bedrock (`Converse`) | `POST /chat` | Lời tư vấn AI tự nhiên kèm ID món gợi ý mở nhanh |
| `lambda_handler` | API Gateway + CloudWatch Logs | Tất cả routes | Điều phối request, xử lý CORS và bắt lỗi tập trung |
