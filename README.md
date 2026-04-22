# Clueless AI

AI-powered personal styling platform that helps users digitize their wardrobe, analyze personal style, generate outfits, discover products, and preview virtual try-ons.

## Overview

Clueless AI is a full-stack fashion-tech application built to make wardrobe management and outfit planning more intelligent and personalized. The platform combines wardrobe digitization, profile-based styling, conversational AI, image classification, embedding-based outfit matching, and product discovery in a single experience.

### What it does

- Lets users create an account and manage a personal fashion profile
- Uploads clothing images and auto-tags garments with AI-generated metadata
- Stores wardrobe items and favorites in a searchable digital closet
- Generates outfit suggestions based on wardrobe contents, filters, and inspiration images
- Provides AI stylist guidance tailored to body shape, color palette, and wardrobe context
- Supports virtual try-on flows for garments using external inference services
- Helps users discover products and save wishlist items

### Why it exists

Choosing outfits, organizing closets, and shopping intentionally are still fragmented workflows. Clueless AI addresses that problem by turning a user’s wardrobe and style preferences into structured data that can power personalized recommendations and reduce decision fatigue.

### Who it is for

- Fashion-conscious consumers who want a smarter wardrobe assistant
- Users who want personalized styling advice without hiring a human stylist
- Teams building AI-first consumer experiences in fashion, retail, or personal commerce

## Features

- Authentication with JWT-based login and protected routes
- AI-assisted wardrobe ingestion from image uploads
- Background removal and image compression pipeline for garment photos
- Clothing classification into category, sub-category, color, mood, occasion, and aesthetic tags
- Visual embedding generation for fashion similarity matching
- AI stylist chat with wardrobe-aware and profile-aware responses
- Appearance analysis for body-shape and color-season guidance
- Outfit generation using hybrid tag matching plus CLIP-style similarity scoring
- Product search flow for shopping inspiration and wishlist curation
- Virtual try-on workflow powered by external model endpoints
- User profile persistence for saved outfits, favorites, inspo board, and wishlist

## Tech Stack

### Frontend

- React 18: client-side UI rendering and page composition
- TypeScript: type safety across components, services, and app state
- Vite: local development server and production build tooling
- React Router: page routing and protected application flows
- TanStack Query: async server-state management
- Tailwind CSS: utility-first styling
- Radix UI: accessible primitives for dialogs, tabs, popovers, and other UI patterns
- Vitest + Testing Library: frontend testing foundation

### Backend

- FastAPI: REST API framework and request validation
- SQLAlchemy: ORM for persistence and relational modeling
- Alembic: schema migrations and database versioning
- PostgreSQL: primary relational database
- Pydantic Settings: environment-based configuration management
- JWT + bcrypt: token issuance and password hashing

### AI / External Services

- Google Gemini: garment classification, appearance analysis, and stylist chat generation
- Hugging Face Inference / Spaces: image-text embeddings and virtual try-on execution
- remove.bg: garment background removal for cleaner wardrobe assets
- SerpApi: shopping/product search results
- Pillow + NumPy: image preprocessing and lightweight numerical operations

### Deployment

- Render: backend hosting and managed PostgreSQL provisioning via `render.yaml`
- Vercel config present in `frontend/vercel.json` for frontend deployment

## Architecture / Workflow

At a high level, the system follows this flow:

1. A user signs up or logs in through the React frontend.
2. The frontend stores the bearer token and calls protected FastAPI endpoints.
3. When a garment is uploaded, the backend compresses the image, optionally removes the background, classifies the item with Gemini, and generates a visual embedding through Hugging Face.
4. Structured garment metadata and embeddings are saved in PostgreSQL.
5. The outfit engine combines stored wardrobe metadata with optional reference images or text-derived embeddings to rank outfit combinations.
6. The stylist chat service uses the user profile, wardrobe summary, and dataset-backed style guidance to produce personalized responses.
7. Product discovery and wishlist flows allow users to extend wardrobe planning into shopping decisions.
8. Virtual try-on requests are sent to external inference endpoints and the generated image is returned to the frontend.

## Project Structure

```text
fitmedeploy/
├── backend/
│   ├── api/                 # FastAPI route modules for auth, users, chatbot, clothing, and try-on
│   ├── core/                # App configuration and security helpers
│   ├── database/            # SQLAlchemy session management
│   ├── models/              # ORM models for users and clothing
│   ├── schemas/             # Pydantic request/response schemas
│   ├── services/            # AI, auth, image processing, embeddings, and try-on integrations
│   ├── alembic/             # Database migration scripts
│   ├── body_analysis_dataset.csv
│   ├── color_analysis_dataset.csv
│   ├── main.py              # FastAPI application entrypoint
│   └── requirements.txt     # Python dependencies
├── frontend/
│   ├── src/
│   │   ├── components/      # Shared UI and route guard components
│   │   ├── pages/           # Page-level views for dashboard, wardrobe, styling, try-on, etc.
│   │   ├── services/        # API client helpers for auth, users, clothing, and try-on
│   │   ├── types/           # Shared TypeScript types
│   │   └── constants/       # Frontend constants
│   ├── public/              # Static assets
│   ├── vite.config.ts       # Frontend dev server and proxy config
│   └── package.json         # Node dependencies and scripts
├── render.yaml              # Render deployment definition
└── README.md
```

## Getting Started

### Prerequisites

- Python 3.11+
- Node.js 18+ and npm
- PostgreSQL
- Git
- API keys for the AI services you plan to enable

### Installation

#### 1. Clone the repository

```bash
git clone <your-repository-url>
cd fitmedeploy
```

#### 2. Set up the backend

```bash
cd backend
python -m venv .venv
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

macOS / Linux:

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

#### 3. Set up the frontend

```bash
cd ../frontend
npm install
```

### Environment Setup

Create a `backend/.env` file and configure the required settings. Do not commit this file.

Required backend variables:

```env
SECRET_KEY=
DATABASE_URL=
```

Common backend variables:

```env
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
BACKEND_CORS_ORIGINS=http://localhost:8080
GEMINI_API_KEY=
GEMINI_API_KEY_ALT=
CHATBOT_GEMINI_API_KEY=
HF_TOKEN=
HF_TOKEN_ALT=
SERPAPI_KEY=
REMOVE_BG_API_KEY=
OPENAI_API_KEY=
COLAB_WORKER_URL=
```

Create a `frontend/.env` file when you want the frontend to target a non-proxied backend:

```env
VITE_API_URL=http://127.0.0.1:8000
```

If `VITE_API_URL` is omitted, the Vite dev server proxies `/api` requests to `http://127.0.0.1:8000`.

### Database Setup

From the `backend/` directory:

```bash
alembic upgrade head
```

## Usage

### Run the backend

```bash
cd backend
uvicorn main:app --reload
```

The API will be available at:

```text
http://127.0.0.1:8000
```

### Run the frontend

```bash
cd frontend
npm run dev
```

The frontend will be available at:

```text
http://127.0.0.1:8080
```

### Example Commands

Run frontend tests:

```bash
cd frontend
npm test
```

Create a production frontend build:

```bash
cd frontend
npm run build
```

### Sample API Input

Upload a clothing item:

```bash
curl -X POST http://127.0.0.1:8000/api/clothing/upload \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -H "Content-Type: application/json" \
  -d "{\"image_base64\":\"data:image/png;base64,<BASE64_IMAGE>\"}"
```

Generate an outfit:

```bash
curl -X POST http://127.0.0.1:8000/api/users/generate-outfit \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -H "Content-Type: application/json" \
  -d "{\"occasion\":[\"Casual\"],\"aesthetic\":[\"Minimalist\"],\"mood\":[\"Chill\"]}"
```

### Sample API Output

Successful outfit-generation response:

```json
{
  "status": "success",
  "data": {
    "outfit": {
      "topId": "12",
      "bottomId": "27",
      "shoesId": "9",
      "dressId": null
    }
  }
}
```

## API Endpoints

### Authentication

- `POST /api/auth/signup` - Register a new user
- `POST /api/auth/login` - Issue a bearer token
- `GET /api/auth/check-email` - Check email availability
- `GET /api/auth/check-username` - Check username availability

### Users / Styling

- `GET /api/users/me` - Fetch the authenticated user profile
- `PATCH /api/users/updateMe` - Update user profile data
- `POST /api/users/stylist-chat` - Chat with the AI stylist
- `POST /api/users/analyze-appearance` - Analyze body shape and color profile
- `GET /api/users/wardrobe` - Get wardrobe items via user profile response
- `POST /api/users/generate-outfit` - Generate outfit combinations from wardrobe data
- `POST /api/users/search-products` - Search products for discovery and wishlist flows
- `POST /api/users/inspo` - Add an inspiration board item
- `DELETE /api/users/inspo/{item_id}` - Remove an inspiration board item
- `POST /api/users/wishlist` - Add a wishlist item
- `DELETE /api/users/wishlist/{item_id}` - Remove a wishlist item
- `POST /api/users/best-outfits` - Save a favorite curated outfit
- `DELETE /api/users/best-outfits/{outfit_id}` - Remove a saved curated outfit
- `POST /api/users/saved-generated-outfits` - Save a generated outfit
- `DELETE /api/users/saved-generated-outfits/{outfit_id}` - Remove a saved generated outfit

### Clothing

- `POST /api/clothing/upload` - Upload and classify a wardrobe item
- `GET /api/clothing/` - List wardrobe items
- `PATCH /api/clothing/{clothing_id}` - Update clothing metadata
- `PATCH /api/clothing/{clothing_id}/favorite` - Toggle favorite status
- `DELETE /api/clothing/{clothing_id}` - Delete a wardrobe item

### Chatbot

- `POST /api/chatbot/` - General wardrobe-aware stylist chat

### Try-On

- `POST /api/tryon/` - Run virtual garment try-on

## Model Details

This project is not training custom models inside the repository, but it orchestrates multiple external AI services:

- Gemini Flash: used for garment classification, stylist chat, and appearance analysis
- Hugging Face CLIP (`openai/clip-vit-base-patch32`): used for image/text embeddings and wardrobe similarity scoring
- Hugging Face Spaces / IDM-VTON-compatible endpoints: used for virtual try-on image generation
- Dataset-assisted logic: `body_analysis_dataset.csv` and `color_analysis_dataset.csv` are used to enrich styling recommendations and profile interpretation

### Current ML Workflow

- Input images are compressed before inference
- Garment images can be background-cleaned before classification
- AI-generated tags are normalized into controlled vocabularies
- Visual embeddings are persisted with clothing records
- Outfit ranking blends semantic tag overlap with visual similarity

## Contributing Guidelines

1. Fork the repository and create a feature branch from `main`.
2. Keep frontend and backend changes scoped, documented, and tested where applicable.
3. Never commit secrets, `.env` files, generated tokens, or private API credentials.
4. Run migrations when database models change and include the Alembic revision in your pull request.
5. Validate local changes before opening a PR:

```bash
cd frontend
npm test
npm run build
```

For backend changes, make sure the API starts cleanly and migrations apply successfully:

```bash
cd backend
alembic upgrade head
uvicorn main:app --reload
```

6. Open a pull request with:
   - a concise problem statement
   - implementation notes
   - screenshots or request/response examples for UI/API changes
   - any migration or deployment considerations