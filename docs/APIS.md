# APIS.md — Baby Junctions
> All API connections. Scripts follow exactly. Never hardcode keys.

## 1. CLAUDE API
```python
import anthropic
client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
# Model: claude-sonnet-4-20250514 | Rate limit: 50 req/min
# Retry on RateLimitError: wait 60s × attempt number
```

## 2. AMAZON PA API 5.0
```python
# pip install paapi5-python-sdk
from paapi5_python_sdk.api.default_api import DefaultApi
import paapi5_python_sdk, time

api_client = paapi5_python_sdk.ApiClient(
    access_key=os.environ["AMAZON_ACCESS_KEY"],
    secret_key=os.environ["AMAZON_SECRET_KEY"],
    host="webservices.amazon.in",
    region="eu-west-1"
)
api = DefaultApi(api_client)
```

### SearchItems (Stage 2)
```python
from paapi5_python_sdk.models.search_items_request import SearchItemsRequest
from paapi5_python_sdk.models.search_items_resource import SearchItemsResource

def search_products(keyword):
    req = SearchItemsRequest(
        partner_tag=os.environ["AMAZON_ASSOCIATE_TAG"],
        partner_type="Associates",
        marketplace="www.amazon.in",
        keywords=keyword,
        search_index="Baby",
        item_count=10,
        resources=[
            SearchItemsResource.ITEMINFO_TITLE,
            SearchItemsResource.OFFERS_LISTINGS_PRICE,
            SearchItemsResource.CUSTOMERREVIEWS_STARRATING,
            SearchItemsResource.CUSTOMERREVIEWS_COUNT,
            SearchItemsResource.IMAGES_PRIMARY_LARGE,
            SearchItemsResource.OFFERS_LISTINGS_AVAILABILITY_TYPE,
        ]
    )
    result = api.search_items(req)
    time.sleep(1)  # MANDATORY — 1 req/sec
    return result.search_result.items if result.search_result else []

def passes_filter(item):
    try:
        return (float(item.customer_reviews.star_rating.value) >= 3.8 and
                int(item.customer_reviews.count) >= 100 and
                item.offers.listings[0].availability.type == "Now")
    except: return False

def affiliate_link(asin):
    return f"https://www.amazon.in/dp/{asin}?tag={os.environ['AMAZON_ASSOCIATE_TAG']}"
```

### GetItems (Stage 5 — price refresh)
```python
from paapi5_python_sdk.models.get_items_request import GetItemsRequest
from paapi5_python_sdk.models.get_items_resource import GetItemsResource

def refresh_prices(asin_list):
    req = GetItemsRequest(
        partner_tag=os.environ["AMAZON_ASSOCIATE_TAG"],
        partner_type="Associates",
        marketplace="www.amazon.in",
        item_ids=asin_list[:10],
        resources=[GetItemsResource.OFFERS_LISTINGS_PRICE,
                   GetItemsResource.OFFERS_LISTINGS_AVAILABILITY_TYPE]
    )
    result = api.get_items(req)
    time.sleep(1)
    return result.items_result.items if result.items_result else []
```

## 3. WORDPRESS REST API
```python
import requests
from requests.auth import HTTPBasicAuth

WP_AUTH = HTTPBasicAuth(os.environ["WP_USERNAME"], os.environ["WP_APP_PASSWORD"])
WP_BASE = os.environ["WP_SITE_URL"] + "/wp-json/wp/v2"

def create_post(payload):
    r = requests.post(f"{WP_BASE}/posts", json=payload, auth=WP_AUTH, timeout=30)
    if r.status_code == 201: return r.json()["id"], r.json()["link"]
    raise Exception(f"WP publish failed: {r.status_code}")

def update_post(post_id, payload):
    return requests.post(f"{WP_BASE}/posts/{post_id}", json=payload, auth=WP_AUTH).status_code == 200

def upload_image(path, filename, alt):
    with open(path,"rb") as f:
        r = requests.post(f"{WP_BASE}/media",
            headers={"Content-Disposition":f"attachment; filename={filename}","Content-Type":"image/png"},
            data=f.read(), auth=WP_AUTH)
    mid = r.json()["id"]
    requests.post(f"{WP_BASE}/media/{mid}", json={"alt_text":alt}, auth=WP_AUTH)
    return mid

def slug_exists(slug):
    return len(requests.get(f"{WP_BASE}/posts?slug={slug}", auth=WP_AUTH).json()) > 0
```

## 4. GOOGLE SHEETS
```python
import gspread, json
from google.oauth2.service_account import Credentials

def get_sheet(tab):
    creds = Credentials.from_service_account_info(
        json.loads(os.environ["GOOGLE_SERVICE_ACCOUNT_JSON"]),
        scopes=["https://spreadsheets.google.com/feeds","https://www.googleapis.com/auth/drive"])
    client = gspread.authorize(creds)
    return client.open(os.environ["GOOGLE_SHEETS_NAME"]).worksheet(tab)

def read_tab(tab): return get_sheet(tab).get_all_records()
def append_row(tab, row): get_sheet(tab).append_row(row)

def update_status(tab, id_col, id_val, status_col, new_status):
    sheet = get_sheet(tab)
    records = sheet.get_all_records()
    headers = sheet.row_values(1)
    for i, row in enumerate(records):
        if row[id_col] == id_val:
            sheet.update_cell(i+2, headers.index(status_col)+1, new_status)
            return True
    return False
```

## 5. GOOGLE DRIVE
```python
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload

def drive_service():
    creds = Credentials.from_service_account_info(
        json.loads(os.environ["GOOGLE_SERVICE_ACCOUNT_JSON"]),
        scopes=["https://www.googleapis.com/auth/drive"])
    return build("drive","v3",credentials=creds)

def create_folder(name, parent_id):
    svc = drive_service()
    # Check if exists
    res = svc.files().list(q=f"name='{name}' and '{parent_id}' in parents and mimeType='application/vnd.google-apps.folder'",fields="files(id)").execute()
    if res["files"]: return res["files"][0]["id"]
    f = svc.files().create(body={"name":name,"mimeType":"application/vnd.google-apps.folder","parents":[parent_id]},fields="id").execute()
    return f["id"]

def upload_file(local_path, filename, parent_id, mimetype="text/plain"):
    svc = drive_service()
    svc.files().create(
        body={"name":filename,"parents":[parent_id]},
        media_body=MediaFileUpload(local_path,mimetype=mimetype),
        fields="id").execute()
```

## 6. INSTAGRAM GRAPH API
```python
IG_BASE = "https://graph.facebook.com/v18.0"

def post_instagram(image_public_url, caption):
    # image must be a public URL (use WP media URL)
    r = requests.post(f"{IG_BASE}/{os.environ['INSTAGRAM_USER_ID']}/media",
        params={"image_url":image_public_url,"caption":caption,
                "access_token":os.environ["INSTAGRAM_PAGE_TOKEN"]})
    cid = r.json().get("id")
    pub = requests.post(f"{IG_BASE}/{os.environ['INSTAGRAM_USER_ID']}/media_publish",
        params={"creation_id":cid,"access_token":os.environ["INSTAGRAM_PAGE_TOKEN"]})
    return pub.json().get("id")

def get_ig_insights(post_id):
    return requests.get(f"{IG_BASE}/{post_id}/insights",
        params={"metric":"impressions,reach,profile_visits,website_clicks",
                "access_token":os.environ["INSTAGRAM_PAGE_TOKEN"]}).json().get("data",[])
```

## 7. PINTEREST API
```python
PIN_BASE = "https://api.pinterest.com/v5"

# Subcategory → Board ID mapping (fill after creating boards)
PINTEREST_BOARDS = {
    "Baby Monitors":"BOARD_ID","Baby Diapers":"BOARD_ID",
    # ... all 50 subcategories
    "default":"BOARD_ID_000"
}

def create_pin(subcategory, image_url, title, description, link):
    board_id = PINTEREST_BOARDS.get(subcategory, PINTEREST_BOARDS["default"])
    return requests.post(f"{PIN_BASE}/pins",
        headers={"Authorization":f"Bearer {os.environ['PINTEREST_ACCESS_TOKEN']}","Content-Type":"application/json"},
        json={"board_id":board_id,"title":title[:100],"description":description[:500],
              "link":link,"media_source":{"source_type":"image_url","url":image_url}}).json()
```

## 8. GOOGLE SEARCH CONSOLE
```python
from googleapiclient.discovery import build

def get_gsc_data(start_date, end_date):
    creds = Credentials.from_service_account_info(json.loads(os.environ["GOOGLE_SERVICE_ACCOUNT_JSON"]),
        scopes=["https://www.googleapis.com/auth/webmasters.readonly"])
    svc = build("searchconsole","v1",credentials=creds)
    return svc.searchanalytics().query(
        siteUrl="sc-domain:babyjunctions.in",
        body={"startDate":start_date,"endDate":end_date,"dimensions":["page","query"],"rowLimit":1000}
    ).execute().get("rows",[])

def near_miss_pages(rows):
    return sorted([{"url":r["keys"][0],"keyword":r["keys"][1],
                    "position":r["position"],"clicks":r["clicks"],"impressions":r["impressions"]}
                   for r in rows if 10 < r["position"] <= 20 and r["impressions"] > 50],
                  key=lambda x: x["impressions"], reverse=True)[:20]
```

## 9. MAILCHIMP
```python
MC_DC = os.environ["MAILCHIMP_API_KEY"].split("-")[-1]
MC_BASE = f"https://{MC_DC}.api.mailchimp.com/3.0"
MC_AUTH = ("anystring", os.environ["MAILCHIMP_API_KEY"])

def send_email(subject, preview, body_html):
    c = requests.post(f"{MC_BASE}/campaigns", auth=MC_AUTH,
        json={"type":"regular","recipients":{"list_id":os.environ["MAILCHIMP_LIST_ID"]},
              "settings":{"subject_line":subject,"preview_text":preview,
                          "from_name":"Baby Junctions","reply_to":"hello@babyjunctions.in"}}).json()
    requests.put(f"{MC_BASE}/campaigns/{c['id']}/content", auth=MC_AUTH, json={"html":body_html})
    requests.post(f"{MC_BASE}/campaigns/{c['id']}/actions/send", auth=MC_AUTH)
```

## REQUIREMENTS.TXT
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

## ENV VARIABLES
```
ANTHROPIC_API_KEY · AMAZON_ACCESS_KEY · AMAZON_SECRET_KEY · AMAZON_ASSOCIATE_TAG
WP_SITE_URL · WP_USERNAME · WP_APP_PASSWORD
GOOGLE_SERVICE_ACCOUNT_JSON · GOOGLE_SHEETS_ID · GOOGLE_SHEETS_NAME · GOOGLE_DRIVE_ROOT_FOLDER_ID
INSTAGRAM_PAGE_TOKEN · INSTAGRAM_USER_ID · PINTEREST_ACCESS_TOKEN
MAILCHIMP_API_KEY · MAILCHIMP_LIST_ID · GA4_PROPERTY_ID
```

*v1.0 | April 2026*
