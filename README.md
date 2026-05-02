# 💄 นายหัว 2497 · Ads Report Automation

Pulls Facebook Ads data for 5 clinic clients each night, writes it into their Google Sheet "Daily Log" tab, screenshots the Dashboard, and pushes a summary to LINE OA.

Plus a **glam Pink/Rose Streamlit dashboard** you can open anytime.

---

## 🚀 Quick start (mock mode — works immediately)

```bash
cd ~/Desktop/Code/ads-report
bash INSTALL.sh
source .venv/bin/activate
streamlit run dashboard.py
```

Open <http://localhost:8501>. Dashboard runs with realistic fake data so you see the layout before plugging real credentials in.

---

## 🔑 Switching to live data

### 1. Facebook Marketing API

1. <https://developers.facebook.com/apps/> → **Create app** → type **Business**
2. Add product **Marketing API**
3. Tools → **Graph API Explorer** → request permissions: `ads_read`, `business_management`
4. Generate User Access Token → copy
5. Make it long-lived:
   ```
   curl -G "https://graph.facebook.com/v20.0/oauth/access_token" \
     -d "grant_type=fb_exchange_token" \
     -d "client_id=YOUR_APP_ID" \
     -d "client_secret=YOUR_APP_SECRET" \
     -d "fb_exchange_token=YOUR_SHORT_TOKEN"
   ```
6. Paste into `.env` → `FB_ACCESS_TOKEN=...`
7. For each clinic that needs an Ad Account ID: Ads Manager → top-left dropdown → 16-digit number after `act_`. Fill in the `FB_ACCOUNT_*` lines.

### 2. Google Sheets

1. <https://console.cloud.google.com/> → New project "ads-report"
2. Enable **Google Sheets API** and **Google Drive API**
3. APIs & Services → Credentials → **Create OAuth client ID** → type **Desktop**
4. Download JSON → save as `credentials/google_oauth.json`
5. First time you run `daily_report.py`, browser opens once for consent → token saved to `credentials/google_token.json`

### 3. LINE OA

1. <https://developers.line.biz/console/> → create **Messaging API channel**
2. Channel access token → paste into `.env` → `LINE_CHANNEL_ACCESS_TOKEN`
3. Add the bot to your team's LINE group, send any message → get `groupId` from webhook (or temporary: send a `!whoami` to bot — see LINE docs)
4. Paste into `LINE_GROUP_ID`

> Optional: To send screenshots to LINE you need a public HTTPS URL. Drop pngs from `outputs/` into Cloudflare Pages or Imgur, set `PUBLIC_IMAGE_BASE_URL` in `.env`.

### 4. Flip the switch

```bash
# in .env
MOCK_MODE=false
```

```bash
python daily_report.py    # test once
```

---

## ⏰ Scheduling (macOS cron)

```bash
crontab -e
```

Add:
```
59 23 * * * cd /Users/fadiion/Desktop/Code/ads-report && /Users/fadiion/Desktop/Code/ads-report/.venv/bin/python daily_report.py >> logs/$(date +\%Y-\%m).log 2>&1
```

> macOS may ask you to grant cron "Full Disk Access" the first time. System Settings → Privacy & Security → Full Disk Access → add `/usr/sbin/cron`.

---

## 📁 Files

| File | Purpose |
|---|---|
| `daily_report.py` | Cron entry point. Pulls FB → writes Sheets → screenshots → LINE |
| `dashboard.py` | Streamlit glam dashboard (`streamlit run dashboard.py`) |
| `config.py` | Client list + theme colors |
| `lib/fb_ads.py` | Facebook Marketing API client (real + mock) |
| `lib/sheets.py` | Google Sheets writer (Daily Log tab, day-N row math) |
| `lib/notify.py` | LINE push + Gmail fallback |
| `lib/screenshot.py` | Playwright screenshot of Dashboard tab |
| `outputs/` | Cached JSON + PNG per client per day |
| `logs/` | Cron output |

---

## 🎨 Theme

Burgundy `#6B1A35` · Rose `#D9899C` · Gold `#C9A961` · Cream `#FBF6F0` · Sarabun + Playfair Display.

---

## ⚠️ Constraints (do not violate)

- ✋ **Don't** edit April or other months' sheets
- ✋ **Don't** delete old data — script only writes 3 ad rows per day; daily-total formula row is left untouched
- ✋ **Don't** touch cancelled clients: Vogue / Nobel / Genovita / Genitive Biocare
- 🎯 Target ROAS = 10× per client
- 💡 Glow doesn't track Purchase Value → uses Cost-per-Inbox (configured in `config.py`)

---

## 🛟 Troubleshooting

| Symptom | Fix |
|---|---|
| `ModuleNotFoundError` | `source .venv/bin/activate` first |
| Streamlit shows mock banner forever | `.env` not loaded — check it's in project root + `MOCK_MODE=false` |
| Sheets write fails with 403 | Re-run consent: delete `credentials/google_token.json`, run again |
| LINE 401 | Token rotated — generate new channel access token |
| FB API rate limit | The 5-client × 1×/day load is far below limits; if it ever hits, add `time.sleep(2)` between clients |
