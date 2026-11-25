# Clueless – AI-Powered Smart Wardrobe and Outfit Recommendation System

> *"As if I'd leave the house without checking Clueless first."*

An intelligent personal styling platform that combines artificial intelligence, computer vision, and personalized recommendations to simplify everyday fashion decisions.

Clueless transforms a user's clothing collection into a smart digital wardrobe, analyzes personal style preferences, generates outfit combinations, enables virtual try-ons, and provides personalized fashion recommendations through AI.

**Live Demo:** [Clueless](https://clueless-drab.vercel.app)

![Clueless Landing Page](docs/screenshots/landing.jpg)

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Application Workflow](#application-workflow)
- [System Architecture](#system-architecture)
- [AI Implementation](#ai-implementation)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Database Design](#database-design)
- [Getting Started](#getting-started)
- [Environment Configuration](#environment-configuration)
- [Running the Application](#running-the-application)
- [API Reference](#api-reference)
- [Deployment](#deployment)
- [Limitations](#limitations)

---

## Overview

Clueless is a full-stack AI-powered fashion and wardrobe management application designed to provide personalized styling recommendations based on a user's existing clothing collection, preferences, and appearance-related style analysis.

The application integrates machine learning, image processing, natural language processing, and external AI services to deliver an interactive fashion experience.

Rather than relying solely on generic fashion recommendations, Clueless uses the user's digital wardrobe and styling preferences to generate relevant outfit combinations and personalized suggestions.

## Key Features

| Feature | Description |
|---|---|
| **AI-Powered Wardrobe Management** | Upload clothing images and automatically identify garment categories, colors, occasions, aesthetics, and other attributes. |
| **Digital Wardrobe** | Organize, search, filter, sort, edit, and manage clothing collections. |
| **Outfit Studio** | Manually combine clothing items to create and save personalized outfits. |
| **AI Outfit Generation** | Generate outfit combinations based on occasions, aesthetics, moods, weather, color preferences, and inspiration images. |
| **Personalized Style Analysis** | Analyze seasonal color palettes, undertones, body shapes, and clothing recommendations. |
| **AI Fashion Stylist** | Interact with an AI-powered chatbot that provides styling suggestions based on personal preferences and wardrobe information. |
| **Virtual Try-On** | Visualize selected clothing items on an uploaded photograph using an AI-based virtual try-on model. |
| **Fashion Shopping Hub** | Discover fashion products from online shopping platforms using personalized search and filtering. |
| **Wishlist Management** | Save interesting products and access them later. |
| **Fashion Moodboard** | Collect inspiration images and use them as references for outfit generation. |
| **Personalized User Profile** | Manage profile information, saved outfits, wardrobe details, and styling preferences. |

---

## Application Workflow

### 1. User Authentication and Dashboard

Users can create accounts and securely access their personalized fashion workspace.

- Account registration and authentication.
- Login using email or username.
- JWT-based authentication.
- Protected application routes.
- Centralized dashboard for accessing application features.

### 2. AI-Powered Clothing Upload

Users can upload individual or multiple clothing images.

The system processes uploaded images through an automated pipeline:

1. Image compression and preprocessing.
2. Background removal using remove.bg.
3. Garment classification using Google Gemini.
4. Visual feature extraction using CLIP.
5. Normalization of AI-generated clothing attributes.
6. Storage of clothing information and visual embeddings in PostgreSQL.

The extracted attributes include garment category, subcategory, color, occasion, aesthetic, mood, weather, location, and additional tags.

### 3. Digital Wardrobe Management

The wardrobe provides a centralized interface for managing clothing collections.

Features include:

- Clothing search and filtering.
- Category-based organization.
- Color-based filtering.
- Sorting by date and item name.
- Grid and list display modes.
- Favorite clothing items.
- Editable AI-generated attributes.
- Clothing deletion and management.

### 4. Outfit Studio

The Outfit Studio allows users to manually combine clothing items to create outfits.

Users can select combinations of:

- Tops
- Bottoms
- Shoes

Dress selections automatically adjust the outfit layout. Users can save preferred combinations to their Best Fits collection.

### 5. AI-Based Outfit Generation

The AI outfit generation module creates personalized outfit combinations using clothing already available in the user's wardrobe.

Users can customize recommendations using:

- Occasion
- Style aesthetic
- Mood
- Time
- Location
- Weather
- Specific colors
- Seasonal color palette

Users can also upload an inspiration image or select an existing moodboard image.

The system evaluates clothing attributes and visual similarity to identify suitable combinations. Previously generated combinations are excluded to provide different recommendations.

### 6. AI-Based Personal Style Analysis

Users can upload a photograph to receive personalized appearance-related styling recommendations.

The analysis includes:

- Body shape classification.
- Seasonal color analysis.
- Undertone identification.
- Recommended color palettes.
- Colors to avoid.
- Recommended clothing fits.
- Styling suggestions for different garment categories.

The analysis results are saved to the user's profile for subsequent personalization.

### 7. AI Fashion Stylist Chat

Clueless AI provides conversational fashion assistance using Google Gemini.

The chatbot incorporates:

- User styling preferences.
- Body shape and seasonal color information.
- Existing wardrobe information.
- Fashion recommendations from reference datasets.
- Previous conversation context.

This allows the stylist to provide recommendations that consider the user's existing clothing collection.

### 8. Virtual Try-On Studio

The virtual try-on feature allows users to visualize selected clothing items on their photographs.

Workflow:

1. Upload a full-body photograph.
2. Select a clothing item from the wardrobe or wishlist.
3. Submit the images for processing.
4. Generate a virtual try-on result using IDM-VTON.

The current implementation supports tops.

### 9. Fashion Shopping Hub

The Shopping Hub provides product discovery through Google Shopping search results using SerpAPI.

Features include:

- Product search.
- Men, Women, and All filters.
- Category-based filtering.
- Color filtering.
- Price limits.
- Product sorting.
- Similar product discovery.
- Wishlist integration.
- Image-based search refinement.

Shopping results are localized to India and displayed with Indian pricing.

### 10. Wishlist, Moodboard and Profile

Additional modules support personalized fashion organization.

**Wishlist**
- Save products discovered through the Shopping Hub.
- Access saved products for future reference.
- Use saved tops as virtual try-on inspiration.

**Moodboard**
- Upload and organize fashion inspiration images.
- Reuse inspiration images during outfit generation.

**User Profile**
- Manage personal information.
- Update profile photographs.
- Access saved outfits and moodboards.
- View wardrobe information.

---

## System Architecture

Clueless follows a frontend-backend architecture with external AI integrations and persistent database storage.

```mermaid
flowchart TD
    A[User] --> B[React Frontend]
    B --> C[FastAPI Backend]

    C --> D[Authentication]
    C --> E[Application Services]

    E --> F[PostgreSQL Database]
    E --> G[Google Gemini]
    E --> H[Hugging Face CLIP]
    E --> I[IDM-VTON]
    E --> J[remove.bg]
    E --> K[SerpAPI]

    G --> E
    H --> E
    I --> E
    J --> E
    K --> E

    E --> C
    C --> B
```

### Frontend Architecture

The frontend is developed as a React single-page application using TypeScript and Vite.

It communicates with the backend through REST APIs and uses JWT authentication for protected resources.

### Backend Architecture

The backend is implemented using FastAPI and separates API routes, business logic, external integrations, database models, and request validation.

Asynchronous requests and worker threads are used for external operations to improve request handling.

### Database Architecture

PostgreSQL provides persistent storage for user accounts, clothing information, wardrobe attributes, visual embeddings, and personalized user data.

SQLAlchemy handles database interactions, while Alembic manages schema migrations.

---

## AI Implementation

### 1. Clothing Classification

Google Gemini analyzes uploaded clothing images and generates structured garment attributes.

CLIP extracts visual embeddings that represent the visual characteristics of clothing items.

These attributes are used for wardrobe organization and outfit recommendations.

### 2. Visual Similarity and Outfit Generation

The outfit generation pipeline combines attribute matching and CLIP-based visual similarity.

The scoring mechanism is:

**Combined Score = 0.8 × Tag Match + 0.2 × CLIP Similarity**

The system evaluates clothing combinations and ranks them according to their compatibility with the selected styling preferences or reference image.

### 3. Appearance and Color Analysis

Google Gemini processes uploaded photographs to generate structured appearance-related styling recommendations, including body shape, seasonal color analysis, undertones, and suitable clothing styles.

### 4. Conversational Styling

Google Gemini powers the conversational stylist, using user profile information, wardrobe summaries, and bundled styling reference datasets to generate contextual recommendations.

### 5. Virtual Try-On

IDM-VTON, accessed through Hugging Face Spaces, generates virtual try-on results using a person's photograph and a selected garment image.

---

## Technology Stack

### Frontend

- React 18
- TypeScript
- Vite
- Tailwind CSS
- shadcn/ui
- Radix UI
- React Router
- TanStack Query
- Lucide React

### Backend

- Python
- FastAPI
- Uvicorn
- Gunicorn
- SQLAlchemy
- Alembic
- Pydantic
- JWT Authentication
- Passlib and bcrypt

### Artificial Intelligence and Image Processing

- Google Gemini
- Hugging Face CLIP
- IDM-VTON
- remove.bg
- Pillow
- NumPy
- Gradio Client

### Database

- PostgreSQL

### External Integrations

- Google Gemini API
- Hugging Face
- SerpAPI
- remove.bg API

### Deployment

- Vercel – Frontend hosting
- Render – Backend hosting and PostgreSQL database

---

## Project Structure

```text
Clueless/
│
├── backend/
│   ├── api/
│   ├── core/
│   ├── database/
│   ├── models/
│   ├── schemas/
│   ├── services/
│   ├── alembic/
│   ├── constants.py
│   ├── main.py
│   ├── requirements.txt
│   ├── body_analysis_dataset.csv
│   └── color_analysis_dataset.csv
│
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   ├── services/
│   │   ├── types/
│   │   └── constants/
│   ├── public/
│   ├── package.json
│   ├── vite.config.ts
│   └── vercel.json
│
├── docs/
│   └── screenshots/
│
├── render.yaml
└── README.md
```

---

## Database Design

The application primarily uses two database entities.

### Users

Stores account information and personalized styling data.

Key information includes:

- User credentials and account details.
- Profile information.
- Body shape and seasonal color analysis.
- Styling recommendations.
- Wardrobe-related collections.
- Wishlist and moodboard information.
- Saved and generated outfits.

### Clothing

Stores information about individual wardrobe items.

Key attributes include:

- User association.
- Clothing image.
- Category and subcategory.
- Color and style information.
- Occasion and aesthetic tags.
- Mood, weather, time, and location attributes.
- Color palette information.
- CLIP visual embeddings.
- Favorite status and timestamps.

Database schema changes are managed through Alembic migrations.

---

## Getting Started

### Prerequisites

Ensure the following are installed:

- Python 3.11
- Node.js 18 or later
- npm
- PostgreSQL

API credentials are required for the external AI and shopping services you intend to use.

### 1. Clone the Repository

```bash
git clone <YOUR_REPOSITORY_URL>
cd Clueless
```

### 2. Backend Setup

Navigate to the backend directory:

```bash
cd backend
```

Create and activate a virtual environment:

**Windows (PowerShell)**

```powershell
python -m venv venv
venv\Scripts\Activate.ps1
```

**macOS / Linux**

```bash
python -m venv venv
source venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

### 3. Configure the Database

Create a PostgreSQL database and configure the database connection in the backend environment file.

Example:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/clueless
```

### 4. Configure Environment Variables

Create a `.env` file inside the backend directory.

Refer to the [Environment Configuration](#environment-configuration) section.

### 5. Run Database Migrations

```bash
alembic upgrade head
```

### 6. Start the Backend

```bash
uvicorn main:app --reload
```

Backend API:

```text
http://127.0.0.1:8000
```

Interactive API documentation:

```text
http://127.0.0.1:8000/docs
```

### 7. Frontend Setup

Open another terminal:

```bash
cd frontend
npm install
npm run dev
```

The frontend development server runs at:

```text
http://localhost:8080
```

---

## Environment Configuration

### Backend Environment Variables

Create `backend/.env` and configure the required credentials.

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL database connection |
| `SECRET_KEY` | JWT token signing |
| `ALGORITHM` | JWT signing algorithm |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Authentication token expiration |
| `BACKEND_CORS_ORIGINS` | Allowed frontend origins |
| `GEMINI_API_KEY` | Gemini-powered classification and analysis |
| `GEMINI_API_KEY_ALT` | Secondary Gemini API key |
| `CHATBOT_GEMINI_API_KEY` | AI stylist chatbot |
| `HF_TOKEN` | Hugging Face authentication |
| `HF_TOKEN_ALT` | Alternative Hugging Face token |
| `REMOVE_BG_API_KEY` | Background removal service |
| `SERPAPI_KEY` | Shopping search integration |

### Frontend Environment Variables

Create `frontend/.env` when connecting to a separately hosted backend.

```env
VITE_API_URL=https://your-backend-url
```

During local development, the Vite proxy can be used without configuring this variable.

**Security:** Keep environment files and API credentials private. Never commit `.env` files or expose secret API keys in frontend code.

---

## API Reference

The backend exposes REST API endpoints for authentication, wardrobe management, styling, outfit generation, shopping, and virtual try-on.

Protected endpoints require JWT authentication.

```http
Authorization: Bearer <ACCESS_TOKEN>
```

### Authentication – `/api/auth`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/signup` | Register a new account |
| POST | `/login` | Authenticate using email or username |
| GET | `/check-email` | Check email availability |
| GET | `/check-username` | Check username availability |

### Clothing – `/api/clothing`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/upload` | Upload and process clothing images |
| GET | `/` | Retrieve wardrobe items |
| PATCH | `/{id}` | Update clothing attributes |
| PATCH | `/{id}/favorite` | Toggle favorite status |
| DELETE | `/{id}` | Delete a wardrobe item |

### User and Styling – `/api/users`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/me` | Retrieve user profile |
| GET | `/wardrobe` | Retrieve wardrobe information |
| PATCH | `/updateMe` | Update profile information |
| POST | `/analyze-appearance` | Perform appearance and color analysis |
| POST | `/generate-outfit` | Generate personalized outfit combinations |
| POST / DELETE | `/best-outfits` | Manage saved Studio outfits |
| POST / DELETE | `/saved-generated-outfits` | Manage generated outfits |
| POST / DELETE | `/wishlist` | Manage wishlist items |
| POST / DELETE | `/inspo` | Manage moodboard images |
| POST | `/search-products` | Search fashion products |

### AI Stylist and Virtual Try-On

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/chatbot/` | Interact with the AI fashion stylist |
| POST | `/api/tryon/` | Generate virtual try-on results |

Complete interactive API documentation is available at `/docs` when the backend is running.

---

## Deployment

The application uses separate frontend and backend hosting.

### Frontend – Vercel

The React frontend is deployed through Vercel.

Configuration:

- Root directory: `frontend`
- Build command: `npm run build`
- Output directory: `dist`
- Environment variable: `VITE_API_URL`

### Backend and Database – Render

The backend and PostgreSQL database are deployed using Render.

The `render.yaml` configuration defines the backend service and database resources.

The backend runs database migrations before starting the application.

External API credentials must be configured through the hosting provider's environment settings.

---

## Limitations

- Virtual try-on currently supports tops only.
- Virtual try-on depends on external Hugging Face Spaces availability.
- Shopping results are localized to India.
- AI-powered features depend on the availability, configuration, and usage limits of their respective external services.
- Image-based AI analysis and styling recommendations should be treated as suggestions rather than definitive assessments.

---

## Project Highlights

Clueless demonstrates the integration of full-stack development, artificial intelligence, computer vision, recommendation systems, image processing, and external API services into a unified fashion application.

The project brings together personalized wardrobe management, intelligent outfit generation, virtual try-on capabilities, and conversational styling to create an interactive digital fashion experience.

**Clueless — Your wardrobe, intelligently styled.**
