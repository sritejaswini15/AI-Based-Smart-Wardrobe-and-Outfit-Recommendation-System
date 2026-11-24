# Clueless

> *"As if I'd leave the house without checking Clueless first."*

Clueless is a full-stack, AI-powered styling app. Photograph your clothes and it builds a tagged digital wardrobe, works out your body shape and seasonal colour palette, puts together outfits from what you already own, lets you virtually try tops on a photo of yourself, and finds new pieces to shop, all in one place.

**Live demo:** [clueless-drab.vercel.app](https://clueless-drab.vercel.app)

![Clueless landing page](docs/screenshots/landing.jpg)

---

## Table of Contents

- [Features at a Glance](#features-at-a-glance)
- [How It Works — A Walkthrough](#how-it-works--a-walkthrough)
  - [1. Sign In & Dashboard](#1-sign-in--dashboard)
  - [2. Add Clothes (AI Auto-Tagging)](#2-add-clothes-ai-auto-tagging)
  - [3. Wardrobe](#3-wardrobe)
  - [4. Studio — Mix & Match](#4-studio--mix--match)
  - [5. Generate Outfit — AI Curation](#5-generate-outfit--ai-curation)
  - [6. AI Stylist — Fit & Color Analysis](#6-ai-stylist--fit--color-analysis)
  - [7. AI Stylist — Chat](#7-ai-stylist--chat)
  - [8. AI Try-On Studio](#8-ai-try-on-studio)
  - [9. Shopping Hub](#9-shopping-hub)
  - [10. Wishlist, Moodboard & Profile](#10-wishlist-moodboard--profile)
- [Architecture](#architecture)
- [AI Pipelines in Detail](#ai-pipelines-in-detail)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Data Model](#data-model)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Deployment](#deployment)
- [API Reference](#api-reference)
- [Known Limitations](#known-limitations)

---

## Features at a Glance

| Feature | What it does | Powered by |
|---|---|---|
| **AI Wardrobe Upload** | Removes the background from clothing photos and auto-tags category, color, occasion, aesthetic, mood, weather and more | remove.bg · Google Gemini · CLIP |
| **Wardrobe** | Searchable, filterable, sortable digital closet with editable AI tags and favorites | FastAPI · PostgreSQL |
| **Studio** | Swipe through tops, bottoms and shoes to style looks by hand; save favorites to *Best Fits* | React |
| **Generate Outfit** | Pick an occasion, aesthetic, mood, weather and more (or upload an inspiration photo) and get a complete outfit from your own clothes | Hybrid tag + CLIP scoring |
| **Fit & Color Analysis** | Upload a photo to get your body shape, seasonal color type, best colors, colors to avoid, and fits to embrace or avoid | Google Gemini |
| **AI Stylist Chat** | A conversational stylist that knows your body shape, palette and wardrobe | Google Gemini + style datasets |
| **Virtual Try-On** | See a top from your closet or wishlist on a photo of yourself | IDM-VTON (Hugging Face Spaces) |
| **Shopping Hub** | Search real products across Indian stores (Myntra, Ajio, Amazon and others) with gender, category, color and price filters | SerpAPI Google Shopping |
| **Wishlist & Moodboard** | Save products you love and collect style inspiration | FastAPI · PostgreSQL |

---

## How It Works — A Walkthrough

### 1. Sign In & Dashboard

Create an account, then sign in with **either your email or your username**. The backend issues a JWT (HS256, 30-minute expiry by default). The frontend stores it and sends it on every API call. All feature pages are behind a `ProtectedRoute`, which sends signed-out visitors to the sign-in page.

The dashboard links to every feature, and the top bar gives quick access to the Wardrobe, Wishlist, Moodboard and Profile.

### 2. Add Clothes (AI Auto-Tagging)

Drag and drop one or more photos (or browse from your device) and click **Complete Upload**. The backend processes each image in turn:

1. **Compresses** it to at most 1024 px (PNG, so transparency is kept).
2. **Removes the background** with remove.bg, so items look clean in the wardrobe, Studio and try-on.
3. **In parallel**:
   - **Google Gemini** classifies the garment: category, a specific sub-category (e.g. *"High-Waisted Wide-Leg Jeans"*), main color, color palette, occasions, aesthetics, moods, weather, locations and extra tags.
   - **CLIP** (`openai/clip-vit-base-patch32` on Hugging Face) generates a visual embedding, the item's "Visual DNA", which outfit generation uses later.
4. Gemini's free-text tags are **normalized to a fixed vocabulary** (`backend/constants.py`) so filters stay consistent.
5. The item, its tags and its embedding are saved to PostgreSQL.

### 3. Wardrobe

Your digital closet. You can:

- **Search** by name or color.
- **Filter** by category tabs (*All, Tops, Bottoms, Shoes, Dresses, Accessories, Other*) and by color.
- **Sort** by *Newest First*, *Oldest First* or *A–Z Name*.
- Switch between **grid** and **list** views.
- **Favorite**, **edit AI tags** (occasion, aesthetic, mood, time, location, weather, color palette, specific colors, custom tags) or **delete** any item.

<p align="center">
  <img src="docs/screenshots/wardrobe-list.jpg" alt="Wardrobe list view with AI-generated item names" width="90%" />
  <br />
  <sub>Every item is named and tagged by the AI on upload, e.g. "Cropped Butterfly Graphic Ringer Tee"</sub>
</p>

### 4. Studio — Mix & Match

A hands-on styling board with three rows that loop: **top layer**, **bottoms** and **shoes**. Use the arrows to cycle through your clothes in each row. If you pick a dress, the bottoms row hides automatically. Tap the **heart** to save the look to **Best Fits**, and click a saved look to load it back into the Studio.

### 5. Generate Outfit — AI Curation

On the **Dressing Room** page you describe the outfit you want and Clueless builds it from your own clothes.

- **Curation filters** (each one optional):
  *Occasion · Style Aesthetic · Mood · Time · Location · Weather · Specific Colors · Color Palette*
- **Style inspo** (optional): upload a reference photo from your device or pick one from your Moodboard.
- Click **Generate Look**. Each click gives a different combination, because outfits you've already seen are skipped.
- **Save This Look** adds it to your curated **Archive**.

How a look is scored is explained in [AI Pipelines → Outfit generation](#outfit-generation).

<p align="center">
  <img src="docs/screenshots/generate-outfit-result.jpg" alt="Outfit generated from the wardrobe" width="90%" />
  <br />
  <sub>A complete outfit built from clothes already in the wardrobe</sub>
</p>

### 6. AI Stylist — Fit & Color Analysis

Upload a clear photo of yourself and, optionally, your height and weight, then click **Analyse**. Gemini returns a full style profile, which is saved to your account so it's there next time:

- **Seasonal color type** (e.g. *Soft Summer*), **body shape** (e.g. *Hourglass*) and **undertone**
- **Color Palette** tab: *power colors*, *neutrals* and *accents*, each with a hex swatch and a reason, plus *colors to avoid*
- **Fit & Form** tab: *fits to embrace* and *fits to avoid*, each with an explanation
- **Studio Strategy** tab: an overall style summary plus specific advice for tops, bottoms and dresses

If the analysis needs more detail, a **Refine Analysis** dialog asks about your sun reaction and natural hair color.

<p align="center">
  <img src="docs/screenshots/ai-stylist-colors.jpg" alt="Color palette analysis" width="90%" />
  <br />
  <sub>Color Palette tab: best colors and colors to avoid for a Soft Summer</sub>
</p>

### 7. AI Stylist — Chat

**Clueless AI** is a conversational stylist. Every message is sent to Gemini together with:

- your **body shape** and **color season** from the analysis,
- a **wardrobe summary** (up to 20 of your items, e.g. *"Black Short-Sleeve Pinstripe Button-Up Blouse"*),
- expert recommendations looked up in two bundled datasets (`body_analysis_dataset.csv`, `color_analysis_dataset.csv`): recommended fits, necklines, fabrics, and primary, neutral, accent and avoid colors.

Replies are formatted in Markdown and suggest specific pieces you already own. If you mention your sun reaction or natural hair color in the chat, the AI adds a hidden `[UPDATE: field=value]` tag. The backend removes the tag from the reply and saves the value to your profile.

### 8. AI Try-On Studio

1. Upload a clear, full-body photo of yourself. It is only used for this session and isn't saved.
2. Pick a top from the **Owned** tab (tops in your wardrobe) or the **Inspo** tab (tops in your wishlist).
3. The backend runs **IDM-VTON** on Hugging Face Spaces and returns your photo wearing that top.

The service tries several IDM-VTON Spaces and API tokens in turn, so a single busy server doesn't fail the request. Try-on currently supports **tops only**.

<p align="center">
  <img src="docs/screenshots/try-on-result.jpg" alt="Virtual try-on result" width="90%" />
  <br />
  <sub>The selected top from the wardrobe, tried on with IDM-VTON</sub>
</p>

### 9. Shopping Hub

Search real products from online stores:

- Free-text search (e.g. *"black top"*), a **Men / Women / All** toggle and **category** tabs
- **Filters** for colors and a **price limit**, plus **sort** options
- Hover a product to **View on Website** or **Find Similar**; tap the heart to save it to your **Wishlist**

Searches go through **SerpAPI Google Shopping**, localized to India (prices in ₹). The backend removes irrelevant results (supplements, books and so on), applies strict gender filtering and enforces the price limit. If you supply a reference image, Gemini classifies it and turns it into suggested **AI Refined Search** queries.

<p align="center">
  <img src="docs/screenshots/shopping-find-similar.jpg" alt="Shopping Hub results" width="90%" />
  <br />
  <sub>Live results for "black top" with View on Website and Find Similar</sub>
</p>

### 10. Wishlist, Moodboard & Profile

- **Wishlist**: products saved from the Shopping Hub. Tops saved here also appear in the Try-On Studio's *Inspo* tab.
- **Moodboard**: upload inspiration images. Any of them can be used as the style reference in *Generate Outfit*.
- **Profile**: your name, username, age and wardrobe count, with tabs for your Moodboard and Best Fits. Use *Edit Profile* to change your name, age or picture.

---

## Architecture

```mermaid
flowchart LR
    U["Browser<br/>React + Vite SPA<br/>(Vercel)"] -->|"REST + JWT"| API["FastAPI backend<br/>(Render)"]
    API --> DB[("PostgreSQL")]
    API --> GEM["Google Gemini<br/>classification · analysis · chat"]
    API --> CLIP["Hugging Face Inference<br/>CLIP embeddings"]
    API --> VTON["Hugging Face Spaces<br/>IDM-VTON try-on"]
    API --> RBG["remove.bg<br/>background removal"]
    API --> SERP["SerpAPI<br/>Google Shopping"]
```

- The **frontend** is a single-page app. In development, Vite proxies `/api` to `http://127.0.0.1:8000`. In production it calls the URL set in `VITE_API_URL`.
- The **backend** keeps the API layer (`api/`) separate from the external-service calls (`services/`). Slow or blocking AI calls run in worker threads (`asyncio.to_thread`) or as async HTTP requests, so they don't hold up other requests.
- **Images** are stored as base64 text in PostgreSQL alongside each item's tags and CLIP embedding.

---

## AI Pipelines in Detail

### Clothing upload

```mermaid
sequenceDiagram
    participant FE as Frontend
    participant API as POST /api/clothing/upload
    participant RB as remove.bg
    participant GM as Gemini
    participant HF as CLIP (Hugging Face)
    participant DB as PostgreSQL

    FE->>API: image (base64)
    API->>API: compress to ≤1024 px PNG
    API->>RB: remove background
    RB-->>API: transparent PNG
    par in parallel
        API->>GM: classify garment
        GM-->>API: category, sub-category, color, tags
    and
        API->>HF: image embedding
        HF-->>API: "Visual DNA" vector
    end
    API->>API: normalize tags to fixed vocabulary
    API->>DB: save item + tags + embedding
    API-->>FE: new wardrobe item
```

### Outfit generation

`POST /api/users/generate-outfit` scores every item in the wardrobe, then builds outfits from the scores:

1. **Target "vibe"**: if you uploaded a reference image, its CLIP embedding is the target. Otherwise your selected occasion, aesthetic and mood filters are turned into a text prompt (*"A professional high-end fashion outfit that is casual and minimalist…"*), and that prompt's CLIP embedding is the target.
2. **Per-item score** = `0.8 × tag match + 0.2 × CLIP similarity`. *Tag match* is the share of your selected filters that the item's tags satisfy. Items without an embedding are scored on tags alone.
3. **Combinations**: every *dress + shoes* and *top + bottom + shoes* combination is built and ranked by its total score, with a small random shuffle so results vary.
4. **No repeats**: the frontend sends the outfits you've already seen, and the highest-scoring unseen combination is returned.

### Fit & color analysis

`POST /api/users/analyze-appearance` sends your photo (compressed to 1280 px), plus height and weight if given, to Gemini. The prompt asks for structured JSON covering body proportions, skin/hair/eye colors, body shape, seasonal sub-season, undertone, contrast level, best and avoid colors (with hex codes and reasons), fits to embrace or avoid, and advice for each type of garment. The result is saved to the user's profile (`style_advice`), along with the photo used.

### Stylist chat

`POST /api/chatbot/` builds a system prompt from your profile, a summary of your wardrobe, and matching rows from the two style datasets. It then continues the conversation with Gemini using your chat history. API keys are tried in priority order: `CHATBOT_GEMINI_API_KEY` first, then the general Gemini keys.

### Virtual try-on

`POST /api/tryon/` compresses both images to 1024 px and calls IDM-VTON through `gradio_client`. It tries these Spaces in order: `yisol/IDM-VTON`, `vinesmsuic/IDM-VTON`, `Nymbo/Virtual-Try-On`. For each Space it tries `HF_TOKEN`, then `HF_TOKEN_ALT`, then no token, and returns the first image it gets back.

### Graceful fallbacks

| Missing key | Behavior |
|---|---|
| Gemini | Clothing classification and appearance analysis return **placeholder results**, so the app stays usable during local development |
| remove.bg | The original image is kept |
| Chatbot Gemini key | Falls back to the general Gemini keys; if none are set, the chat shows an "offline" message |

---

## Tech Stack

**Frontend**
- React 18 + TypeScript, built with Vite
- Tailwind CSS + shadcn/ui (Radix UI primitives), lucide-react icons
- React Router v6 (protected routes), TanStack Query
- react-markdown + remark-gfm for chat formatting
- Sonner toasts

**Backend**
- FastAPI + Uvicorn / Gunicorn
- SQLAlchemy 2 + Alembic migrations, PostgreSQL
- Pydantic v2 + pydantic-settings
- JWT authentication (python-jose) with bcrypt password hashing (passlib)
- Pillow + NumPy for image processing and vector math
- httpx (async HTTP), gradio_client (Hugging Face Spaces)

**AI & external services**
- Google Gemini (`gemini-flash-latest`): garment classification, appearance analysis, stylist chat
- Hugging Face CLIP (`openai/clip-vit-base-patch32`): image and text embeddings
- IDM-VTON on Hugging Face Spaces: virtual try-on
- remove.bg: background removal
- SerpAPI: Google Shopping product search

**Hosting**
- Frontend on **Vercel**, backend and PostgreSQL on **Render**

---

## Project Structure

```text
ai-stylist/
├── backend/
│   ├── api/                       # Route handlers
│   │   ├── auth.py                #   signup, login, availability checks
│   │   ├── users.py               #   profile, analysis, outfits, wishlist, inspo, shopping
│   │   ├── clothing.py            #   wardrobe upload & CRUD
│   │   ├── chatbot.py             #   AI stylist chat
│   │   ├── tryon.py               #   virtual try-on
│   │   └── deps.py                #   current-user dependency (JWT)
│   ├── core/                      # Settings (config.py) and JWT/password helpers
│   ├── database/                  # SQLAlchemy engine & session
│   ├── models/                    # ORM models: User, Clothing
│   ├── schemas/                   # Pydantic request/response schemas
│   ├── services/                  # External integrations
│   │   ├── ai_classifier.py       #   Gemini classification + appearance analysis
│   │   ├── chatbot_service.py     #   Gemini chat with profile context
│   │   ├── hf_service.py          #   CLIP embeddings + cosine similarity
│   │   ├── image_processing.py    #   compression + remove.bg
│   │   ├── tryon_service.py       #   IDM-VTON via gradio_client
│   │   └── auth.py                #   user lookup & authentication
│   ├── alembic/                   # Database migrations
│   ├── constants.py               # Fixed tag vocabularies (occasions, moods, …)
│   ├── body_analysis_dataset.csv  # Body-shape styling reference data
│   ├── color_analysis_dataset.csv # Seasonal color reference data
│   ├── main.py                    # FastAPI app entry point
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── pages/                 # One component per route (Wardrobe, AIStylist, AITryOn, …)
│   │   ├── components/            # UnifiedHeader, ProtectedRoute, shadcn/ui components
│   │   ├── services/              # API client helpers (auth, user, clothing, try-on)
│   │   ├── types/                 # Shared TypeScript types
│   │   └── constants/
│   ├── vite.config.ts             # Dev server on :8080, proxies /api to :8000
│   ├── vercel.json                # Sends all routes to index.html for client-side routing
│   └── package.json
├── docs/screenshots/              # README images
└── render.yaml                    # Render Blueprint (backend + PostgreSQL)
```

---

## Data Model

**`users`**: account details (`username`, `email`, `hashed_password`), profile fields (`full_name`, `age`, `profile_picture`, `height`, `weight`), styling results (`body_shape`, `color_palette`, `skin_tone`, `sun_reaction`, `hair_color`, `style_advice` JSON, `stylist_picture`), and JSON collections (`fashion_inspo_board`, `wishlist`, `best_outfits`, `saved_generated_outfits`).

**`clothing`**: `user_id`, `image_base64`, `category`, `sub_category`, `color`, `style`, tag lists stored as JSON (`occasion`, `style_aesthetic`, `mood`, `time`, `location`, `weather`, `specific_colors`, `color_palette`, `tags`), the CLIP `visual_embedding`, `is_favorite`, and timestamps.

The schema is managed with Alembic; migrations live in `backend/alembic/versions/`.

---

## Getting Started

### Prerequisites

- **Python 3.11** (the pinned dependencies, such as NumPy 1.26 and Pydantic 2.6, don't have Python 3.13 wheels)
- **Node.js 18+** and npm
- **PostgreSQL**
- API keys for the AI services you want to turn on (see [Environment Variables](#environment-variables))

### 1. Clone

```bash
git clone https://github.com/sritejaswini15/ai-stylist.git
cd ai-stylist
```

### 2. Backend

```bash
cd backend
python -m venv venv

# Windows (PowerShell)
venv\Scripts\Activate.ps1
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt
```

Create `backend/.env` (it is git-ignored, so never commit it):

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/clueless
SECRET_KEY=change-me-to-a-long-random-string
BACKEND_CORS_ORIGINS=http://localhost:8080

GEMINI_API_KEY=
CHATBOT_GEMINI_API_KEY=
HF_TOKEN=
REMOVE_BG_API_KEY=
SERPAPI_KEY=
```

Create the database tables and start the API:

```bash
alembic upgrade head
uvicorn main:app --reload
```

The API runs at **http://127.0.0.1:8000**, with interactive docs at **http://127.0.0.1:8000/docs**.

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

The app runs at **http://localhost:8080**. During development, `/api` requests go through Vite's proxy to the backend, so no frontend `.env` is needed. To point at a different backend, create `frontend/.env`:

```env
VITE_API_URL=https://your-backend.onrender.com
```

### Other scripts

```bash
npm run build     # production build to frontend/dist
npm run preview   # preview the production build
npm run lint      # ESLint
```

---

## Environment Variables

### Backend (`backend/.env`)

| Variable | Required | Default | Used for |
|---|---|---|---|
| `DATABASE_URL` | ✅ | — | PostgreSQL connection string (`postgres://` is automatically rewritten to `postgresql://`) |
| `SECRET_KEY` | ✅ | — | Signing JWTs |
| `ALGORITHM` | | `HS256` | JWT algorithm |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | | `30` | Token lifetime |
| `BACKEND_CORS_ORIGINS` | | `*` | Allowed origins, as a comma-separated list or JSON array |
| `GEMINI_API_KEY` | | — | Clothing classification and appearance analysis |
| `GEMINI_API_KEY_ALT` | | — | Backup Gemini key, used when the primary key fails |
| `CHATBOT_GEMINI_API_KEY` | | — | Dedicated key for the stylist chat |
| `HF_TOKEN` / `HF_TOKEN_ALT` | | — | CLIP embeddings and virtual try-on |
| `REMOVE_BG_API_KEY` | | — | Background removal on upload |
| `SERPAPI_KEY` | | — | Shopping Hub product search |

`OPENAI_API_KEY`, `FAL_KEY` and `COLAB_WORKER_URL` are defined in `core/config.py` but not used yet.

### Frontend (`frontend/.env`)

| Variable | Used for |
|---|---|
| `VITE_API_URL` | Backend base URL. Leave it empty in development to use the Vite proxy |

---

## Deployment

### Backend + database (Render)

`render.yaml` is a Render Blueprint that creates:

- **`fitme-backend`**, a Python 3.11.9 web service rooted at `backend/`. On each start it runs:
  ```bash
  alembic upgrade head && gunicorn -w 2 -k uvicorn.workers.UvicornWorker main:app --bind 0.0.0.0:$PORT
  ```
- **`fitme-db`**, a managed PostgreSQL database. Its connection string is linked to `DATABASE_URL` automatically.

`SECRET_KEY` is generated automatically. Enter the AI service keys in the Render dashboard.

> On Render's free plan the service goes to sleep when idle, so the first request after a while can take up to a minute.

### Frontend (Vercel)

Import the repo into Vercel with these settings:

- **Root directory:** `frontend`
- **Build command:** `npm run build`
- **Output directory:** `dist`
- **Environment variable:** `VITE_API_URL=<your Render backend URL>`

`frontend/vercel.json` sends every route to `index.html`, so deep links like `/wardrobe` work.

---

## API Reference

All endpoints except authentication require `Authorization: Bearer <token>`. Full interactive docs are at `/docs` on a running backend.

### Auth — `/api/auth`

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/signup` | Create an account |
| `POST` | `/login` | OAuth2 form login (email **or** username) → access token |
| `GET` | `/check-email?email=` | Check whether an email is available |
| `GET` | `/check-username?username=` | Check whether a username is available |

### Clothing — `/api/clothing`

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/upload` | Upload an image → background removal, AI tagging, embedding |
| `GET` | `/` | List wardrobe items |
| `PATCH` | `/{id}` | Update an item's tags or metadata |
| `PATCH` | `/{id}/favorite` | Toggle favorite |
| `DELETE` | `/{id}` | Delete an item |

### Users & styling — `/api/users`

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/me` | Full profile, including wardrobe, wishlist, moodboard and saved outfits |
| `GET` | `/wardrobe` | Wardrobe items only |
| `PATCH` | `/updateMe` | Update profile fields |
| `POST` | `/analyze-appearance` | Fit & color analysis from a photo |
| `POST` | `/generate-outfit` | AI outfit from filters and/or a reference image |
| `POST` / `DELETE` | `/best-outfits[/{id}]` | Save or remove a Studio look |
| `POST` / `DELETE` | `/saved-generated-outfits[/{id}]` | Save or remove a generated look |
| `POST` / `DELETE` | `/wishlist[/{id}]` | Add or remove a wishlist item |
| `POST` / `DELETE` | `/inspo[/{id}]` | Add or remove a moodboard image |
| `POST` | `/search-products` | Shopping Hub search via SerpAPI |
| `POST` | `/stylist-chat` | Alternate stylist chat endpoint (the app's chat page uses `/api/chatbot/`) |

### Chat & try-on

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/chatbot/` | Chat with the AI stylist (`message`, `history`) |
| `POST` | `/api/tryon/` | Virtual try-on (`person_image`, `garment_image`, `garment_description`) |

**Example: generate an outfit**

```bash
curl -X POST http://127.0.0.1:8000/api/users/generate-outfit \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"occasion":["Casual"],"aesthetic":["Minimalist"],"mood":["Chill"]}'
```

```json
{
  "status": "success",
  "data": {
    "outfit": { "topId": "12", "bottomId": "27", "shoesId": "9", "dressId": null }
  }
}
```

---

## Known Limitations

- **Try-on supports tops only**, and depends on public Hugging Face Spaces, which can be busy or rate-limited.
- **Shopping results are localized to India**.
