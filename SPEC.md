# Chate Sat Job Poster — Web App SPEC

> Desktop web tool for posting jobs to Telegram channels automatically.
> Notion page link paste → fetch JD → upload image → schedule → post to 2 Telegram channels.

---

## 1. Overview

**Purpose:** Replace manual Telegram job posting process with a single web tool.
- Fetch job details from Notion page link
- Edit JD content if needed
- Upload Canva image
- Set schedule time
- Post automatically to Channel 1 (image + caption + links) and Channel 2 (JD text)

**User:** Single user (personal use). No auth needed.

**Platform:** Desktop browser (Mac Safari primary). NOT a PWA — simple web app, bookmark to use.

---

## 2. Tech Stack

- **Frontend:** Vanilla HTML + CSS + JavaScript (single file: `index.html`)
- **Hosting:** Vercel (static)
- **Backend:** n8n webhooks (existing: `n8n.novamarketingmm.com`)
- **Database:** Notion (existing)
- **Telegram:** Bot API via n8n

---

## 3. File Structure

```
chatesat-poster/
├── index.html        ← entire app (HTML + CSS + JS)
├── vercel.json       ← vercel config
└── README.md
```

---

## 4. UI Layout

Two-panel layout (side by side, desktop only):

```
┌─────────────────────────────────────────────────────┐
│  TOPBAR: CS logo | "Chate Sat — Job Poster"         │
├──────────────────────┬──────────────────────────────┤
│  LEFT PANEL          │  RIGHT PANEL                 │
│  Job Details         │  Post Settings               │
│                      │                              │
│  • Notion link input │  • Image upload (drop + pick)│
│    + Fetch button    │  • Schedule date + time      │
│                      │  • Channel 2 JD link input   │
│  • Fetched info      │  • Apply link input          │
│    (position/salary/ │    (default: t.me/...62)     │
│     level/status)    │                              │
│                      │  • Preview box               │
│  • JD textarea       │  • Schedule button           │
│    (editable)        │                              │
└──────────────────────┴──────────────────────────────┘
```

### Design
- Clean, flat, minimal
- Color: white panels, green (#1a7a42) accent for logo + button
- Font: system sans-serif
- No gradients, no shadows

---

## 5. Left Panel — Job Details

### 5.1 Notion Link Input + Fetch

```
[Notion page URL input field          ] [Fetch button]
```

- User pastes Notion job page URL
- Clicks "Fetch" button
- JS sends POST to n8n webhook: `POST /webhook/chatesat-fetch`
- Body: `{ "notion_url": "https://www.notion.so/..." }`
- n8n fetches Notion page → returns JSON:

```json
{
  "job_title": "Site Engineer",
  "salary": "5 to 6 Lakhs",
  "level": "Mid",
  "status": "Urgent",
  "region": "YGN",
  "jd_content": "Site Engineer\n💵 Salary...\n\nRole Requirements👇\n..."
}
```

- On success: populate fetched info block + JD textarea
- Show loading state on Fetch button while waiting
- Show error message if fetch fails

### 5.2 Fetched Info Block

Display after successful fetch (read-only):
- Position name (bold)
- Status badge (Urgent = red, Normal = orange, Replacement = yellow)
- Region badge (blue)
- Salary
- Level

### 5.3 JD Textarea

- Pre-filled with `jd_content` from fetch response
- Fully editable — user can delete/edit any part
- Min height: 300px
- Resizable vertically
- Monospace-friendly font for emoji alignment

---

## 6. Right Panel — Post Settings

### 6.1 Image Upload

- Drag & drop zone (large, dashed border)
- Click to open file picker (accept: image/png, image/jpeg)
- On file select: show image preview inside the drop zone
- Store file in memory (will be sent as base64 or multipart to n8n)

### 6.2 Schedule

Two inputs side by side:
- Date picker (default: tomorrow's date)
- Time picker (default: 19:00)

### 6.3 Links

**Channel 2 JD link** (required):
```
Label: "Channel 2 — JD post link"
Placeholder: "https://t.me/chatesatjobs2/191"
Hint: "Channel 2 မှာ တင်မည့် post ရဲ့ link ထည့်ပါ"
```
- User manually enters the link for Channel 2's upcoming JD post

**Apply link** (editable, pre-filled):
```
Label: "အလုပ်လျှောက်ရန် link"
Default value: "https://t.me/chatesatjobs/62"
Hint: "Default link — လိုအပ်ရင် ပြင်နိုင်သည်"
```

### 6.4 Preview Box

Shows summary before posting:
| Field | Value |
|-------|-------|
| Channel 1 | Image + caption + links |
| Channel 2 | JD full text |
| Schedule | [date] · [time] |
| JD link | [channel 2 link value] |
| Apply link | [apply link value] |

### 6.5 Schedule Button

- Green button, full width: "Schedule လုပ်မည်"
- Disabled state if: no image uploaded, no JD content, no channel 2 link, no schedule time
- On click: send to n8n (see Section 7)
- Loading state while sending
- Success state: show green confirmation message
- Error state: show error message with retry option

---

## 7. n8n Webhooks

### 7.1 Fetch Webhook

**Endpoint:** `POST https://n8n.novamarketingmm.com/webhook/chatesat-fetch`

**Request:**
```json
{
  "notion_url": "https://www.notion.so/Site-Engineer-367788..."
}
```

**Response:**
```json
{
  "job_title": "Site Engineer",
  "salary": "5 to 6 Lakhs",
  "level": "Mid",
  "status": "Urgent",
  "region": "YGN",
  "jd_content": "full JD text here..."
}
```

n8n workflow:
```
Webhook → HTTP Request (Notion page fetch) → Code node (parse properties + body) → Respond to Webhook
```

### 7.2 Post Webhook

**Endpoint:** `POST https://n8n.novamarketingmm.com/webhook/chatesat-post`

**Request (multipart/form-data):**
```
image: [file binary]
job_title: "Site Engineer"
jd_content: "full edited JD text"
channel2_link: "https://t.me/chatesatjobs2/191"
apply_link: "https://t.me/chatesatjobs/62"
schedule_time: "2026-05-23T19:00:00+06:30"
```

n8n workflow:
```
Webhook
→ Schedule Channel 2: send JD text to @chatesatjobs2 at schedule_time
→ Get Channel 2 message link (from channel2_link field — user provided)
→ Build Channel 1 caption:
    "Position - [job_title]

    [အသေးစိတ်ကြည့်မယ် — hyperlink to channel2_link]
    [အလုပ်လျှောက်မယ် — hyperlink to apply_link]"
    (both links formatted as Telegram quote/entity style)
→ Schedule Channel 1: send image + caption to @chatesatjobs at schedule_time
→ Respond to Webhook: { "success": true }
```

---

## 8. Telegram Caption Format (Channel 1)

```
Position - Site Engineer

| အသေးစိတ်ကြည့်မယ် 🔍 "
| အလုပ်လျှောက်မယ် ↗ "
```

- "အသေးစိတ်ကြည့်မယ်" → text link to `channel2_link`
- "အလုပ်လျှောက်မယ်" → text link to `apply_link`
- Both lines formatted as Telegram blockquote entities
- Use Telegram Bot API `parse_mode: "HTML"` or entities array

---

## 9. States & Validation

### Required before posting:
- [ ] Notion fetch completed (job_title exists)
- [ ] JD content not empty
- [ ] Image uploaded
- [ ] Schedule date + time set
- [ ] Channel 2 link not empty

### UI States:
- **Idle:** empty form
- **Fetching:** Fetch button shows spinner, input disabled
- **Fetched:** info block visible, textarea populated
- **Ready:** all fields filled, Schedule button enabled (green)
- **Posting:** button shows spinner, all inputs disabled
- **Success:** green banner "Schedule လုပ်ပြီးပြီ! [date] [time] မှာ တက်မည်"
- **Error:** red banner with error message + retry button

---

## 10. Deployment

```bash
# Local path
/Users/yekyawthuwin/Desktop/DATA/Claude Project/ChateSat/chatesat-poster

# GitHub
github.com/yairkyaw/chatesat-poster

# Vercel
chatesat-poster.vercel.app (auto deploy on git push)
```

### vercel.json
```json
{
  "version": 2,
  "builds": [{ "src": "index.html", "use": "@vercel/static" }]
}
```

### Git workflow
```bash
cd "/Users/yekyawthuwin/Desktop/DATA/Claude Project/ChateSat/chatesat-poster"
git add .
git commit -m "description"
git push
# Vercel auto deploy
```

---

## 11. Implementation Order

1. `index.html` shell — two panel layout, all sections
2. CSS — design system, colors, layout
3. Image upload — drag & drop + file picker + preview
4. Fetch logic — Notion link → POST to n8n → populate fields
5. Form validation — enable/disable Schedule button
6. Post logic — send multipart to n8n → handle response
7. States — loading, success, error UI
8. Deploy: `vercel --prod`

---

## 12. Placeholder Values to Replace After Build

| Placeholder | Replace with |
|------------|--------------|
| `FETCH_WEBHOOK_URL` | n8n fetch webhook URL |
| `POST_WEBHOOK_URL` | n8n post webhook URL |
| `https://t.me/chatesatjobs/62` | actual apply link (already known) |

---

## Notes

- No auth, no login — personal tool
- No localStorage needed — stateless per session
- n8n handles all Telegram scheduling via Bot API
- Image sent as multipart form data to n8n (n8n handles Telegram media upload)
- Timezone: Asia/Yangon (UTC+6:30) — pass ISO string with offset
