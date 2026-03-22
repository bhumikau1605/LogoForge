# LogoForge
AI LOGO GENERATOR
# ⚡ LogoForge — AI Logo Generator

A single-file web app that uses **Claude AI** to generate professional SVG logo concepts from a brand description. Pick a style, choose from preset palettes or use the custom **hexagonal color wheel picker**, and get three distinct logo variants in seconds.

---

## ✨ Features

- 🤖 **AI-generated logos** — Claude designs 3 unique SVG variants per run: Wordmark, Icon + Text, and Emblem
- 🎨 **Hex wheel color picker** — a custom-built hexagonal color wheel (rendered on Canvas) lets you pick exact colors for Primary, Secondary, and Background slots
- 🖌️ **6 logo styles** — Minimalist, Geometric, Retro, Futuristic, Playful, Luxury
- 🎭 **6 preset palettes** — Electric, Ocean, Fire, Nature, Monochrome, Royal — plus fully custom colors
- 📝 **AI design brief** — Claude explains its creative direction alongside each generation
- ⬇️ **SVG export** — download your chosen logo as a scalable `.svg` file
- 📋 **Copy SVG code** — paste directly into Figma, Illustrator, or your codebase
- 🖥️ **Zero dependencies** — pure HTML, CSS, and vanilla JS; no build tools, no npm, no frameworks

---

## 🖼️ Color Wheel Picker

The custom color picker is built entirely from scratch using the HTML5 Canvas API:

- **Hexagonal grid** — a honeycomb of colored cells covering the full spectrum
- **Hue** maps to the angle around the center; **saturation** increases toward the edges
- **3 named slots** — Primary, Secondary, Background — each tracked independently with its own ring indicator on the wheel
- **Brightness slider** — darken or lighten any picked hue without changing its color
- **Hex input** — type any `#RRGGBB` value directly; the wheel and preview update instantly
- **Click or drag** anywhere on the wheel to pick

---

## 🚀 Quick Start

> ⚠️ **For local testing only.** Calling the Anthropic API directly from the browser exposes your API key. Use the [backend proxy setup](#-backend-setup-for-production) for any public deployment.

1. Clone the repo:
   ```bash
   git clone https://github.com/your-username/logoforge.git
   cd logoforge
   ```

2. Open `logo-generator.html` in your browser — no server or install step needed.

3. The app calls `https://api.anthropic.com/v1/messages` directly. For a quick local test you can temporarily hardcode your key in the fetch call inside the `generateLogos()` function.

---

## 🔒 Backend Setup for Production

Route the Anthropic API call through your own server so your key is never exposed in client-side code.

### Option A — Node.js + Express

**1. Install dependencies**
```bash
mkdir logoforge-backend && cd logoforge-backend
npm init -y
npm install express cors dotenv
```

**2. Create `.env`**
```env
ANTHROPIC_API_KEY=sk-ant-your-key-here
PORT=3000
```

**3. Create `server.js`**
```js
import express from 'express';
import cors from 'cors';
import 'dotenv/config';

const app = express();
app.use(cors({ origin: 'http://localhost:5500' })); // set to your frontend origin
app.use(express.json());

app.post('/api/generate', async (req, res) => {
  try {
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': process.env.ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01',
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 4000,
        messages: req.body.messages,
      }),
    });
    const data = await response.json();
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.listen(process.env.PORT, () =>
  console.log(`Server running on http://localhost:${process.env.PORT}`)
);
```

**4. Start the server**
```bash
node server.js
```

**5. Update the fetch call in `logo-generator.html`**

Find the `generateLogos()` function and change the fetch URL:
```js
// Before — direct (insecure)
const res = await fetch('https://api.anthropic.com/v1/messages', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ model: '...', max_tokens: 4000, messages }),
});

// After — proxied through your backend (secure)
const res = await fetch('http://localhost:3000/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ messages, max_tokens: 4000 }),
});
```

---

### Option B — Python + FastAPI

**1. Install dependencies**
```bash
pip install fastapi uvicorn python-dotenv httpx
```

**2. Create `.env`**
```env
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

**3. Create `main.py`**
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
import httpx, os
from dotenv import load_dotenv

load_dotenv()
app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5500"],
    allow_methods=["POST"],
    allow_headers=["Content-Type"],
)

class GenerateRequest(BaseModel):
    messages: list
    max_tokens: int = 4000

@app.post("/api/generate")
async def generate(req: GenerateRequest):
    async with httpx.AsyncClient() as client:
        r = await client.post(
            "https://api.anthropic.com/v1/messages",
            headers={
                "x-api-key": os.getenv("ANTHROPIC_API_KEY"),
                "anthropic-version": "2023-06-01",
                "Content-Type": "application/json",
            },
            json={"model": "claude-sonnet-4-20250514", "max_tokens": req.max_tokens, "messages": req.messages},
            timeout=60.0,
        )
    return r.json()
```

**4. Run**
```bash
uvicorn main:app --reload --port 3000
```

**5. Update the fetch URL** in `logo-generator.html` to `http://localhost:3000/api/generate` (same change as Option A).

---

### Option C — Vercel Serverless

**1. Create `/api/generate.js`**
```js
export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).end();

  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01',
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 4000,
      ...req.body,
    }),
  });

  const data = await response.json();
  res.status(200).json(data);
}
```

**2. Add your API key in Vercel**

Go to **Project → Settings → Environment Variables**:
```
ANTHROPIC_API_KEY = sk-ant-your-key-here
```

**3. Update the fetch URL** in `logo-generator.html` to `/api/generate` (relative path works automatically on Vercel).

**4. Deploy**
```bash
vercel --prod
```

---

## 📁 Project Structure

```
logoforge/
├── logo-generator.html   # Entire app — frontend + color wheel + AI logic
├── README.md
├── .env                  # API key — never commit this
├── .gitignore
└── server.js             # Optional backend proxy (Node/Express)
```

---

## 🔐 Security Checklist

- [ ] Add `.env` to `.gitignore` — never commit your API key
- [ ] Restrict CORS to your frontend's actual domain in production
- [ ] Add rate limiting (e.g. `express-rate-limit`) to prevent abuse
- [ ] Validate and sanitize all request bodies before forwarding to the API

**Minimum `.gitignore`:**
```
.env
node_modules/
__pycache__/
.vercel/
```

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, Vanilla JS |
| Color Wheel | HTML5 Canvas API (custom-built) |
| AI Model | Claude `claude-sonnet-4-20250514` |
| Logo Format | SVG (scalable, editable, downloadable) |
| Backend (optional) | Node.js/Express, FastAPI, or Vercel |

---

## 📄 License

MIT — free to use, modify, and distribute.

---

## 🙏 Acknowledgements

Built with the [Anthropic Claude API](https://www.anthropic.com). Fonts via [Google Fonts](https://fonts.google.com).
