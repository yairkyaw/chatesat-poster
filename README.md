# Chate Sat — Job Poster

Desktop web tool for posting jobs to Telegram channels automatically.

## How to use

1. Paste a Notion job page URL → click **Fetch**
2. Edit JD content if needed
3. Upload Canva image (drag & drop or click)
4. Set schedule date + time
5. Enter Channel 2 JD post link
6. Click **Schedule လုပ်မည်**

## Stack

- Frontend: Vanilla HTML/CSS/JS (single file)
- Hosting: Vercel
- Backend: n8n webhooks (`n8n.novamarketingmm.com`)

## Deploy

```bash
git add .
git commit -m "update"
git push
# Vercel auto-deploys on push
```
