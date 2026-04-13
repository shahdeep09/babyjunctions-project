# APIS.md — Baby Junctions
## All API Connections — Specs, Endpoints, Auth, Rate Limits

> Every external API used in the pipeline is documented here.
> Scripts must follow the exact patterns defined in this file.
> Never hardcode credentials — always use environment variables.

---

## 1. CLAUDE API (Anthropic)

| Item | Detail |
|---|---|
| Used in | Stage 1, 3, 5, 6, 7, 8 |
| Model | claude-sonnet-4-20250514 |
| Auth | Bearer token via ANTHROPIC_API_KEY |
| Rate limit | 50 requests/minute on standard tier |
| Cost estimate | ~₹8-12 per article (4000 tokens output) |

### Connection

```python
import anthropic
client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
```

### Standard Call

```python
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=4000,
    system=SYSTEM_PROMPT,
    messages=[{"role": "user", "content": USER_MESSAGE}]
)
output = response.content[0].text
```

### Rate Limit Handling

```python
import time

def safe_claude_call(system, message, retries=3):
    for attempt in range(retries):
        try:
            return call_claude_json(system, message)
        except anthropic.RateLimitError:
            wait = 60 * (attempt + 1)
            print(f"Rate limit hit. Waiting {wait}s...")
            time.sleep(wait)
    raise Exception("Claude API rate limit exceeded after 3 retries")
```

---

## 2. AMAZON PA API (Product Advertising API 5.0)

| Item | Detail |
|---|---|
| Used in | Stage 2, Stage 5 |
| Region | India — www.amazon.in |
| Auth | AWS Signature V4 (access key + secret key) |
| Rate limit | 1 request per second max |
| Marketplace | www.amazon.in |
| Associate Tag | babyjunctions-21 (replace with actual tag after approval) |

### Setup

```python
from paapi5_python_sdk.api.default_api import DefaultApi
from paapi5_python_sdk.models.partner_type import PartnerType
from paapi5_python_sdk.rest import ApiException
import paapi5_python_sdk

# pip install paapi5-python-sdk
host = "webservices.amazon.in"
region = "eu-west-1"

client_config = paapi5_python_sdk.Configuration()
client_config.host = host

api_client = paapi5_python_sdk.ApiClient(
    access_key=os.environ["AMAZON_ACCESS_KEY"],
    secret_key=os.environ["AMAZON_SECRET_KEY"],
    host=host,
    region=region
)
api_instance = DefaultApi(api_client)
```

### SearchItems (Stage 2 — Find Products by Keyword)

```python
from paapi5_python_sdk.models.search_items_request import SearchItemsRequest
from paapi5_python_sdk.models.search_items_resource import SearchItemsResource

def search_amazon_products(keyword, max_results=10):
    resources = [
        SearchItemsResource.ITEMINFO_TITLE,
        SearchItemsResource.OFFERS_LISTINGS_PRICE,
        SearchItemsResource.CUSTOMERREVIEWS_STARRATING,
        SearchItemsResource.CUSTOMERREVIEWS_COUNT,
        SearchItemsResource.IMAGES_PRIMARY_LARGE,
        SearchItemsResource.ITEMINFO_FEATURES,
        SearchItemsResource.OFFERS_LISTINGS_AVAILABILITY_TYPE,
    ]

    request = SearchItemsRequest(
        partner_tag=os.environ["AMAZON_ASSOCIATE_TAG"],
        partner_type=PartnerType.ASSOCIATES,
        marketplace="www.amazon.in",
        keywords=keyword,
        search_index="Baby",
        item_count=max_results,
        resources=resources
    )

    try:
        response = api_instance.search_items(request)
        time.sleep(1)  # MANDATORY — 1 second between calls
        return response.search_result.items if response.search_result else []
    except ApiException as e:
        print(f"Amazon API error for '{keyword}': {e}")
        return []
```

### GetItems (Stage 5 — Refresh Prices by ASIN)

```python
from paapi5_python_sdk.models.get_items_request import GetItemsRequest
from paapi5_python_sdk.models.get_items_resource import GetItemsResource

def get_product_prices(asin_list):
    """Fetch fresh prices for up to 10 ASINs"""
    resources = [
        GetItemsResource.OFFERS_LISTINGS_PRICE,
        GetItemsResource.OFFERS_LISTINGS_AVAILABILITY_TYPE,
        GetItemsResource.CUSTOMERREVIEWS_STARRATING,
    ]

    request = GetItemsRequest(
        partner_tag=os.environ["AMAZON_ASSOCIATE_TAG"],
        partner_type=PartnerType.ASSOCIATES,
        marketplace="www.amazon.in",
        item_ids=asin_list[:10],  # Max 10 per request
        resources=resources
    )

    try:
        response = api_instance.get_items(request)
        time.sleep(1)  # MANDATORY
        return response.items_result.items if response.items_result else []
    except ApiException as e:
        print(f"Amazon GetItems error: {e}")
        return []
```

### Build Affiliate Link

```python
def build_affiliate_link(asin):
    tag = os.environ["AMAZON_ASSOCIATE_TAG"]
    return f"https://www.amazon.in/dp/{asin}?tag={tag}"
```

### Product Quality Filter

```python
def passes_quality_filter(item):
    """Returns True if product meets Baby Junctions standards"""
    try:
        rating = item.customer_reviews.star_rating.value
        reviews = item.customer_reviews.count
        availability = item.offers.listings[0].availability.type

        return (
            float(rating) >= 3.8 and
            int(reviews) >= 100 and
            availability == "Now"
        )
    except (AttributeError, TypeError, IndexError):
        return False
```

---

## 3. WORDPRESS REST API

| Item | Detail |
|---|---|
| Used in | Stage 4, Stage 5, Stage 7 |
| Auth | Basic Auth with Application Password |
| Base URL | https://babyjunctions.in/wp-json/wp/v2 |
| Format | JSON |

### Connection

```python
import requests
from requests.auth import HTTPBasicAuth

WP_AUTH = HTTPBasicAuth(
    os.environ["WP_USERNAME"],
    os.environ["WP_APP_PASSWORD"]
)
WP_BASE = os.environ["WP_SITE_URL"] + "/wp-json/wp/v2"
```

### Create Post

```python
def create_wp_post(payload):
    response = requests.post(
        f"{WP_BASE}/posts",
        json=payload,
        auth=WP_AUTH,
        headers={"Content-Type": "application/json"}
    )
    if response.status_code == 201:
        return response.json()["id"], response.json()["link"]
    else:
        raise Exception(f"WP publish failed: {response.status_code} — {response.text}")
```

### Update Post (Price/OOS)

```python
def update_wp_post(post_id, update_payload):
    response = requests.post(
        f"{WP_BASE}/posts/{post_id}",
        json=update_payload,
        auth=WP_AUTH
    )
    return response.status_code == 200
```

### Upload Image to Media Library

```python
def upload_image(image_path, filename, alt_text):
    with open(image_path, "rb") as img:
        response = requests.post(
            f"{WP_BASE}/media",
            headers={
                "Content-Disposition": f"attachment; filename={filename}",
                "Content-Type": "image/png"
            },
            data=img.read(),
            auth=WP_AUTH
        )
    if response.status_code == 201:
        media_id = response.json()["id"]
        # Set alt text
        requests.post(
            f"{WP_BASE}/media/{media_id}",
            json={"alt_text": alt_text},
            auth=WP_AUTH
        )
        return media_id
    raise Exception(f"Image upload failed: {response.status_code}")
```

### Get Post by ID

```python
def get_wp_post(post_id):
    response = requests.get(
        f"{WP_BASE}/posts/{post_id}",
        auth=WP_AUTH
    )
    return response.json()
```

---

## 4. GOOGLE SHEETS API

| Item | Detail |
|---|---|
| Used in | All stages |
| Auth | Service Account JSON |
| Library | gspread |
| Sheet Name | Baby Junctions — Master Dashboard |

### Setup

```python
import gspread
from google.oauth2.service_account import Credentials
import json

def get_sheets_client():
    scopes = [
        "https://spreadsheets.google.com/feeds",
        "https://www.googleapis.com/auth/drive"
    ]
    creds = Credentials.from_service_account_info(
        json.loads(os.environ["GOOGLE_SERVICE_ACCOUNT_JSON"]),
        scopes=scopes
    )
    return gspread.authorize(creds)

def get_sheet(tab_name):
    client = get_sheets_client()
    spreadsheet = client.open(os.environ["GOOGLE_SHEETS_NAME"])
    return spreadsheet.worksheet(tab_name)
```

### Read All Records

```python
def read_tab(tab_name):
    sheet = get_sheet(tab_name)
    return sheet.get_all_records()
```

### Append Row

```python
def append_to_tab(tab_name, row_data):
    sheet = get_sheet(tab_name)
    sheet.append_row(row_data)
```

### Update Cell

```python
def update_cell(tab_name, row_index, col_name, value):
    sheet = get_sheet(tab_name)
    headers = sheet.row_values(1)
    col_index = headers.index(col_name) + 1
    sheet.update_cell(row_index + 2, col_index, value)  # +2 for header offset
```

### Find Row and Update Status

```python
def update_topic_status(topic_id, new_status):
    sheet = get_sheet("Daily Topics")
    records = sheet.get_all_records()
    for i, row in enumerate(records):
        if row["topic_id"] == topic_id:
            # Find status column
            headers = sheet.row_values(1)
            status_col = headers.index("status") + 1
            sheet.update_cell(i + 2, status_col, new_status)
            return True
    return False
```

---

## 5. GOOGLE DRIVE API

| Item | Detail |
|---|---|
| Used in | Stage 1, 3, 4, 7 |
| Auth | Same Service Account as Sheets |
| Library | google-api-python-client |

### Setup

```python
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload
from google.oauth2.service_account import Credentials

def get_drive_service():
    scopes = ["https://www.googleapis.com/auth/drive"]
    creds = Credentials.from_service_account_info(
        json.loads(os.environ["GOOGLE_SERVICE_ACCOUNT_JSON"]),
        scopes=scopes
    )
    return build("drive", "v3", credentials=creds)
```

### Create Date Folder

```python
def create_daily_folder(parent_folder_id):
    service = get_drive_service()
    today = datetime.now().strftime("%Y-%m-%d")

    # Check if folder already exists
    results = service.files().list(
        q=f"name='{today}' and '{parent_folder_id}' in parents and mimeType='application/vnd.google-apps.folder'",
        fields="files(id, name)"
    ).execute()

    if results["files"]:
        return results["files"][0]["id"]

    # Create new folder
    folder_metadata = {
        "name": today,
        "mimeType": "application/vnd.google-apps.folder",
        "parents": [parent_folder_id]
    }
    folder = service.files().create(body=folder_metadata, fields="id").execute()
    return folder["id"]
```

### Upload Text File

```python
def upload_text_file(content, filename, parent_folder_id):
    service = get_drive_service()

    # Write to temp file first
    temp_path = f"/tmp/{filename}"
    with open(temp_path, "w", encoding="utf-8") as f:
        f.write(content)

    file_metadata = {
        "name": filename,
        "parents": [parent_folder_id]
    }
    media = MediaFileUpload(temp_path, mimetype="text/plain")
    service.files().create(body=file_metadata, media_body=media, fields="id").execute()
```

### Upload Word Doc (.docx)

```python
def upload_docx(docx_path, filename, parent_folder_id):
    service = get_drive_service()
    file_metadata = {
        "name": filename,
        "parents": [parent_folder_id]
    }
    media = MediaFileUpload(
        docx_path,
        mimetype="application/vnd.openxmlformats-officedocument.wordprocessingml.document"
    )
    service.files().create(body=file_metadata, media_body=media, fields="id").execute()
```

---

## 6. INSTAGRAM GRAPH API

| Item | Detail |
|---|---|
| Used in | Stage 6 |
| Auth | Page Access Token (long-lived) |
| Version | v18.0 |
| Limit | 25 posts per 24 hours |

### Post Image to Instagram

```python
IG_USER_ID = os.environ["INSTAGRAM_USER_ID"]
IG_TOKEN = os.environ["INSTAGRAM_PAGE_TOKEN"]
IG_API = "https://graph.facebook.com/v18.0"

def post_to_instagram(image_public_url, caption):
    # Step 1: Create media container
    create_response = requests.post(
        f"{IG_API}/{IG_USER_ID}/media",
        params={
            "image_url": image_public_url,
            "caption": caption,
            "access_token": IG_TOKEN
        }
    )
    creation_id = create_response.json().get("id")
    if not creation_id:
        raise Exception(f"Instagram media creation failed: {create_response.text}")

    # Step 2: Publish
    publish_response = requests.post(
        f"{IG_API}/{IG_USER_ID}/media_publish",
        params={
            "creation_id": creation_id,
            "access_token": IG_TOKEN
        }
    )
    return publish_response.json().get("id")
```

### Fetch Instagram Insights (Stage 7)

```python
def get_instagram_post_insights(post_id):
    metrics = "impressions,reach,profile_visits,website_clicks"
    response = requests.get(
        f"{IG_API}/{post_id}/insights",
        params={
            "metric": metrics,
            "access_token": IG_TOKEN
        }
    )
    return response.json().get("data", [])
```

---

## 7. PINTEREST API

| Item | Detail |
|---|---|
| Used in | Stage 6 |
| Auth | OAuth 2.0 Bearer Token |
| Version | v5 |
| Limit | 100 requests per minute |

### Create Pinterest Pin

```python
PIN_TOKEN = os.environ["PINTEREST_ACCESS_TOKEN"]
PIN_API = "https://api.pinterest.com/v5"

def create_pinterest_pin(board_id, image_url, title, description, link):
    response = requests.post(
        f"{PIN_API}/pins",
        headers={
            "Authorization": f"Bearer {PIN_TOKEN}",
            "Content-Type": "application/json"
        },
        json={
            "board_id": board_id,
            "title": title[:100],
            "description": description[:500],
            "link": link,
            "media_source": {
                "source_type": "image_url",
                "url": image_url
            }
        }
    )
    return response.json()
```

### Pinterest Board Mapping (Subcategory → Board ID)

```python
# Map each subcategory to its Pinterest board
# Replace BOARD_ID values with actual board IDs after creating boards
PINTEREST_BOARDS = {
    "Baby Monitors":       "BOARD_ID_001",
    "Baby Diapers":        "BOARD_ID_002",
    "Baby Strollers":      "BOARD_ID_003",
    "Baby Carriers":       "BOARD_ID_004",
    "Baby Skincare":       "BOARD_ID_005",
    "Baby Bottles":        "BOARD_ID_006",
    "Baby Food":           "BOARD_ID_007",
    "Baby Cribs":          "BOARD_ID_008",
    "Baby Car Seats":      "BOARD_ID_009",
    "Baby Safety Gates":   "BOARD_ID_010",
    # ... add all 50 subcategories
    "default":             "BOARD_ID_000"  # fallback board
}

def get_pinterest_board(subcategory):
    return PINTEREST_BOARDS.get(subcategory, PINTEREST_BOARDS["default"])
```

---

## 8. GOOGLE SEARCH CONSOLE API

| Item | Detail |
|---|---|
| Used in | Stage 7 |
| Auth | Service Account (same as Sheets/Drive) |
| Library | google-api-python-client |
| Property | sc-domain:babyjunctions.in |

### Fetch Performance Data

```python
from googleapiclient.discovery import build

def get_gsc_data(start_date, end_date, row_limit=1000):
    service = build("searchconsole", "v1", credentials=creds)

    request = {
        "startDate": start_date,  # "2026-03-01"
        "endDate": end_date,      # "2026-03-28"
        "dimensions": ["page", "query"],
        "rowLimit": row_limit,
        "startRow": 0
    }

    response = service.searchanalytics().query(
        siteUrl="sc-domain:babyjunctions.in",
        body=request
    ).execute()

    return response.get("rows", [])

def find_near_miss_pages(gsc_data):
    """Find pages ranking position 11-20 with decent impressions"""
    near_miss = []
    for row in gsc_data:
        avg_pos = row.get("position", 0)
        impressions = row.get("impressions", 0)
        if 10 < avg_pos <= 20 and impressions > 50:
            near_miss.append({
                "url": row["keys"][0],
                "keyword": row["keys"][1],
                "position": avg_pos,
                "clicks": row.get("clicks", 0),
                "impressions": impressions,
                "ctr": row.get("ctr", 0)
            })
    return sorted(near_miss, key=lambda x: x["impressions"], reverse=True)[:20]
```

---

## 9. MAILCHIMP API

| Item | Detail |
|---|---|
| Used in | Stage 8 |
| Auth | API Key in header |
| Version | v3 |
| Region | Determined by API key suffix (us1, us2, etc.) |

### Send Weekly Campaign

```python
MAILCHIMP_KEY = os.environ["MAILCHIMP_API_KEY"]
MAILCHIMP_LIST = os.environ["MAILCHIMP_LIST_ID"]
MAILCHIMP_DC = MAILCHIMP_KEY.split("-")[-1]  # e.g. "us1"
MAILCHIMP_BASE = f"https://{MAILCHIMP_DC}.api.mailchimp.com/3.0"

MC_AUTH = ("anystring", MAILCHIMP_KEY)

def send_weekly_email(subject, preview_text, body_html):
    # Create campaign
    campaign = requests.post(
        f"{MAILCHIMP_BASE}/campaigns",
        auth=MC_AUTH,
        json={
            "type": "regular",
            "recipients": {"list_id": MAILCHIMP_LIST},
            "settings": {
                "subject_line": subject,
                "preview_text": preview_text,
                "title": f"Baby Junctions Weekly — {datetime.now().strftime('%d %b %Y')}",
                "from_name": "Baby Junctions",
                "reply_to": "hello@babyjunctions.in"
            }
        }
    ).json()

    campaign_id = campaign["id"]

    # Set content
    requests.put(
        f"{MAILCHIMP_BASE}/campaigns/{campaign_id}/content",
        auth=MC_AUTH,
        json={"html": body_html}
    )

    # Send
    requests.post(
        f"{MAILCHIMP_BASE}/campaigns/{campaign_id}/actions/send",
        auth=MC_AUTH
    )
    return campaign_id
```

---

## 10. REQUIREMENTS.TXT

Save this as `requirements.txt` in the GitHub repository root.

```
anthropic>=0.25.0
requests>=2.31.0
gspread>=6.0.0
google-auth>=2.29.0
google-api-python-client>=2.127.0
paapi5-python-sdk>=1.0.0
Pillow>=10.3.0
python-docx>=1.1.0
mailchimp-marketing>=3.0.80
python-dotenv>=1.0.1
```

---

## 11. ENVIRONMENT VARIABLES REFERENCE

All stored in GitHub Actions Secrets. Never in code.

```
ANTHROPIC_API_KEY            Claude API key
AMAZON_ACCESS_KEY            Amazon PA API access key
AMAZON_SECRET_KEY            Amazon PA API secret key
AMAZON_ASSOCIATE_TAG         e.g. babyjunctions-21
WP_SITE_URL                  https://babyjunctions.in
WP_USERNAME                  WordPress admin username
WP_APP_PASSWORD              WordPress Application Password
GOOGLE_SERVICE_ACCOUNT_JSON  Full JSON of service account key
GOOGLE_SHEETS_ID             Google Sheet ID from URL
GOOGLE_SHEETS_NAME           Baby Junctions — Master Dashboard
GOOGLE_DRIVE_ROOT_FOLDER_ID  Root folder ID for content archive
INSTAGRAM_PAGE_TOKEN         Long-lived Instagram Graph API token
INSTAGRAM_USER_ID            Instagram Business Account ID
PINTEREST_ACCESS_TOKEN       Pinterest OAuth token
MAILCHIMP_API_KEY            Mailchimp API key
MAILCHIMP_LIST_ID            Mailchimp audience/list ID
GA4_PROPERTY_ID              Google Analytics 4 property ID
```

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
