# Embark Advisor Solutions — Texas Life & Health prep

A personalized study tutor. Practice questions with a review-and-retest loop, plus
two live AI features that call Claude: **"explain it my way"** (plain-language or
tied to something the learner loves) and **auto-generated questions** for weak areas.

Progress, profile, and explanations are saved in the browser (localStorage), so the
learner can close the tab and pick up later on the same device.

---

## Why this needs Vercel (not GitHub Pages)

The AI features call the Anthropic API, which needs a **secret key**. A key can't
live in a file the browser downloads — anyone could read it. So the key sits in a
tiny serverless function (`api/claude.js`) that Vercel runs on the server. The
browser calls `/api/claude`; that function adds the key and calls Anthropic.

- **GitHub Pages**: hosts the quiz fine, but has no server, so the AI tutor can't
  run — every "explain it my way" tap shows the graceful fallback message.
- **Vercel**: runs the whole thing, free at this scale. **Use this.**

---

## Deploy to Vercel (about 10 minutes)

### 1. Get an Anthropic API key
Go to <https://console.anthropic.com> → **API Keys** → **Create Key**. Copy it
somewhere safe. Treat it like a password. (Tip: set a monthly spend limit under
**Billing → Limits** so costs can't run away.)

### 2. Put this folder in a GitHub repo
- Create a new repo at <https://github.com/new> (private is fine).
- Upload this entire folder (the `api/` and `public/` folders, `vercel.json`,
  `.gitignore`, and this README).
- **Do not** put your key anywhere in the files. `.gitignore` already blocks the
  common env files.

### 3. Import the repo into Vercel
- Go to <https://vercel.com> → sign up with GitHub (free **Hobby** plan).
- **Add New… → Project** → pick your repo → **Import**.
- Leave the framework preset as **Other**. Click **Deploy**. (The first deploy
  will succeed but the tutor won't work yet — that's the next step.)

### 4. Add your key as an environment variable
- In the Vercel project: **Settings → Environment Variables**.
- Add one:
  - **Name:** `ANTHROPIC_API_KEY`
  - **Value:** your key from step 1
  - Apply to **Production, Preview, and Development**.
- Save.

### 5. Redeploy so the key takes effect
- Go to **Deployments → ⋯ on the latest → Redeploy**.
- When it finishes you'll have a URL like `https://embark-prep.vercel.app`.

### 6. Send the URL to your friend
That's it. It works on her phone or laptop, anytime, whether or not your computer
is on. Her progress saves on whatever device she uses.

---

## Optional but recommended

- **Lock it down**: since the endpoint is public, anyone with the URL could use
  your API credits. For a single friend that's usually fine, but you can add a
  simple shared passphrase check in `api/claude.js` (compare a header against a
  second env var) if you want to keep it private.
- **Spend limit**: set one in the Anthropic console (step 1) as a safety net.

---

## Test locally first (optional)

```bash
npm i -g vercel        # one time
cd embark-vercel
vercel dev             # runs the site + the /api function locally
# it will prompt for env vars; paste your key when asked, or create .env.local:
#   ANTHROPIC_API_KEY=sk-ant-...
```

Open the local URL it prints. `.env.local` is gitignored, so the key stays off
GitHub.

---

## File layout

```
embark-vercel/
├─ api/
│  └─ claude.js        # secure server-side proxy (holds the key)
├─ public/
│  └─ index.html       # the full app (quiz + intro + AI tutor calls)
├─ vercel.json         # routing/config
├─ .gitignore          # keeps secrets out of git
└─ README.md           # this file
```

---

## What's saved on the device

Under one localStorage key (`embark.v1`): the learner's profile, a per-concept
memory (how often each topic was seen/answered correctly, and which interest
produced the analogy that clicked), and a cache of already-generated explanations
so the same one isn't paid for twice. Nothing is sent anywhere except the Anthropic
API calls needed to generate explanations and questions.

To reset everything, clear the site's storage in the browser, or add a reset
button (ask and it can be wired in).
