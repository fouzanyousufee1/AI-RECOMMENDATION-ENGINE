<div align="center">

# 🎬 AI Movie Recommendation Engine

**A full-stack AI-powered movie recommendation app — type a mood, get the top 3 matching films using real OpenAI embeddings and pgvector semantic search.**

[![GitHub Profile](https://img.shields.io/badge/GitHub-fouzanyousufee1-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fouzanyousufee1)
[![Repository](https://img.shields.io/badge/Repo-AI--RECOMMENDATION--ENGINE-0075ff?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fouzanyousufee1/AI-RECOMMENDATION-ENGINE)

![.NET](https://img.shields.io/badge/.NET_10-512BD4?style=flat-square&logo=dotnet&logoColor=white)
![React](https://img.shields.io/badge/React_19-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=flat-square&logo=openai&logoColor=white)

</div>

---

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [API Reference](#api-reference)
- [Database Schema](#database-schema)
- [Key Code Explained](#key-code-explained)
- [Local Setup](#local-setup)
- [Running the App](#running-the-app)
- [Frontend UI](#frontend-ui)
- [Configuration Reference](#configuration-reference)
- [Security Notes](#security-notes)

---

## Overview

This application lets a user type a mood (for example, *"something funny and clever"*) and instantly receive the three most semantically relevant movie recommendations from a curated library of 30 films. The matching is done using AI vector embeddings — not keyword search — so it can understand meaning rather than just matching words.

---

## How It Works

```
User types mood
      │
      ▼
Frontend (React)  ──POST /api/recommend──▶  Backend (.NET 10)
                                                    │
                                          OpenAI text-embedding-3-small
                                          converts mood → 1536 numbers
                                                    │
                                          pgvector <=> cosine distance
                                          compares mood vector against
                                          30 stored movie vectors in PostgreSQL
                                                    │
                                          Returns top 3 closest matches
                                                    │
      Frontend renders movie cards  ◀──────────────┘
```

**Embeddings explained:** Every piece of text (a mood phrase or a movie description) can be converted into a list of 1536 decimal numbers. Texts with similar *meaning* produce number lists that point in the same direction in that 1536-dimensional space. pgvector uses the `<=>` cosine distance operator to measure that angle and find the most similar movies — even when no exact words overlap between the query and the movie description.

---

## Architecture

```
┌─────────────────────────────┐      HTTP (JSON)     ┌──────────────────────────────────────┐
│   movie-ui  (React + Vite)  │ ──────────────────▶  │   MovieApi  (ASP.NET Core .NET 10)   │
│   localhost:5173            │                       │   localhost:5053                     │
│                             │ ◀──────────────────   │                                      │
│  • App.jsx  (UI + fetch)    │    top 3 results      │  • MoviesController.cs               │
│  • App.css  (animated UI)   │                       │    POST /api/seed                    │
└─────────────────────────────┘                       │    POST /api/recommend               │
                                                       │  • Program.cs  (DI + CORS)           │
                                                       │  • MovieContext.cs  (EF Core)        │
                                                       └──────────────┬───────────────────────┘
                                                                      │ Npgsql + pgvector
                                                                      ▼
                                                       ┌──────────────────────────────────────┐
                                                       │   PostgreSQL  (local)                │
                                                       │   Database: moviedb                  │
                                                       │   Table: movies                      │
                                                       │   Column: embedding vector(1536)     │
                                                       └──────────────────────────────────────┘
                                                                      │
                                                                      │ API call per text
                                                                      ▼
                                                       ┌──────────────────────────────────────┐
                                                       │   OpenAI API                         │
                                                       │   Model: text-embedding-3-small      │
                                                       │   Output: float[1536]                │
                                                       └──────────────────────────────────────┘
```

---

## Project Structure

```
AI-RECOMMENDATION-ENGINE/
│
├── README.md                          ← You are here (repo overview + docs)
├── .gitignore                         ← Excludes secrets, build artifacts, node_modules
│
├── MovieApi/                          ← ASP.NET Core Web API (backend)
│   ├── Controllers/
│   │   └── MoviesController.cs        ← All API logic: seed + recommend endpoints
│   ├── MovieContext.cs                ← Entity Framework Core DbContext + Movie model
│   ├── Program.cs                     ← App startup, DI registration, CORS policy
│   ├── appsettings.json               ← Base configuration (no secrets)
│   ├── appsettings.Development.json   ← Local secrets (NOT committed)
│   ├── appsettings.Development.example.json  ← Template for onboarding
│   ├── MovieApi.http                  ← HTTP scratch file for testing endpoints
│   └── MovieApi.csproj                ← .NET project file + NuGet dependencies
│
└── movie-ui/                          ← React + Vite frontend
    ├── src/
    │   ├── App.jsx                    ← Main component: mood input + movie cards
    │   ├── App.css                    ← Animated background, glass panel, card styles
    │   └── index.css                  ← Global base reset
    ├── index.html
    ├── vite.config.js
    └── package.json
```

---

## Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | React | 19 | UI components and state |
| Frontend | Vite | 8 | Dev server and build tooling |
| Backend | ASP.NET Core | .NET 10 | Web API framework |
| Backend | C# | 13 | Application language |
| Backend | OpenAI SDK | 2.10.0 | Calls `text-embedding-3-small` |
| Database | PostgreSQL | 14+ | Relational data store |
| Database | pgvector | 0.3.x | Vector similarity extension |
| ORM | Entity Framework Core | 10 | DB schema mapping |
| DB Driver | Npgsql | 10 | Raw SQL for vector queries |
| Vector Type | Pgvector.EntityFrameworkCore | 0.3.0 | `Vector` C# type |

---

## API Reference

### `POST /api/seed`

Deletes all existing movies, calls OpenAI once per movie to generate embeddings, and inserts all 30 movies with their vectors.

**Request:** No body required.

**Response:**

```json
{
  "seededCount": 30
}
```

**When to call:** Once after initial setup, or any time you want to reset the database.

---

### `POST /api/recommend`

Converts a mood string into an embedding vector and returns the three most semantically similar movies using pgvector cosine distance.

**Request:**

```json
{
  "mood": "funny and clever"
}
```

**Response:**

```json
[
  {
    "title": "Pulp Fiction",
    "genre": "CRIME / DRAMA",
    "description": "Interconnected stories of hitmen, a boxer, and a gangster's wife unfold in nonlinear fashion. Darkly funny and relentlessly cool.",
    "score": 0.357
  },
  {
    "title": "The Grand Budapest Hotel",
    "genre": "COMEDY",
    "description": "A legendary concierge and his lobby boy get entangled in a murder mystery across a fictional European republic. Whimsical and fast-paced with a sharp wit.",
    "score": 0.325
  },
  {
    "title": "Knives Out",
    "genre": "MYSTERY / COMEDY",
    "description": "A detective investigates the death of a wealthy crime novelist surrounded by scheming family members. A clever whodunit with sharp humor.",
    "score": 0.318
  }
]
```

**Score field:** A cosine similarity value between 0 and 1. Higher is more similar to the mood query.

---

## Database Schema

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE movies (
    id          SERIAL PRIMARY KEY,
    title       TEXT NOT NULL,
    genre       TEXT NOT NULL,
    description TEXT NOT NULL,
    embedding   vector(1536),
    created_at  TIMESTAMP DEFAULT NOW()
);
```

The `embedding` column stores a 1536-dimensional float vector for each movie description. The pgvector `<=>` operator computes cosine distance between two vectors directly in SQL.

---

## Key Code Explained

### How the seed endpoint generates embeddings (`MoviesController.cs`)

```csharp
// 1. Get the EmbeddingClient from the injected OpenAIClient
_embeddingClient = openAIClient.GetEmbeddingClient("text-embedding-3-small");

// 2. Generate embedding for a text string
var result = await _embeddingClient.GenerateEmbeddingAsync(text, cancellationToken: ct);
float[] floats = result.Value.ToFloats().ToArray();
var vector = new Vector(floats);  // Pgvector.Vector wraps the float[]
```

### How the recommend endpoint queries pgvector (`MoviesController.cs`)

```sql
SELECT title, genre, description,
       1 - (embedding <=> @queryVec::vector) AS score
FROM movies
ORDER BY embedding <=> @queryVec::vector
LIMIT 3;
```

- `<=>` is the pgvector cosine distance operator (0 = identical direction, 2 = opposite)
- `1 - distance` converts it into a similarity score (1 = perfect match)
- `ORDER BY ... LIMIT 3` returns the three closest movies

### How the frontend tries multiple ports (`App.jsx`)

```jsx
const API_BASES = [
  import.meta.env.VITE_API_BASE_URL,
  "http://localhost:5053",
  "http://localhost:5000"
].filter(Boolean);

// Tries each base URL in order until one succeeds
for (const base of API_BASES) {
  try {
    const res = await fetch(`${base}/api/recommend`, { ... });
    if (res.ok) { /* use this base */ break; }
  } catch { /* try next */ }
}
```

### CORS policy (`Program.cs`)

```csharp
// Allows any localhost origin regardless of port
policy.SetIsOriginAllowed(origin => {
    if (!Uri.TryCreate(origin, UriKind.Absolute, out var uri)) return false;
    return uri.Host.Equals("localhost", StringComparison.OrdinalIgnoreCase);
})
```

### Entity Framework Core movie model (`MovieContext.cs`)

```csharp
public class Movie
{
    public int Id          { get; set; }
    public string Title    { get; set; } = "";
    public string Genre    { get; set; } = "";
    public string Description { get; set; } = "";
    public Vector? Embedding  { get; set; }   // pgvector type
    public DateTime CreatedAt { get; set; }
}
```

---

## Local Setup

### Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- [Node.js 20+](https://nodejs.org/) and npm
- [PostgreSQL 14+](https://www.postgresql.org/) with the pgvector extension installed
- An [OpenAI API key](https://platform.openai.com/api-keys)

### 1. Clone the repository

```bash
git clone https://github.com/fouzanyousufee1/AI-RECOMMENDATION-ENGINE.git
cd AI-RECOMMENDATION-ENGINE
```

### 2. Configure backend secrets

Copy the example settings file and fill in your values:

```powershell
Copy-Item MovieApi\appsettings.Development.example.json MovieApi\appsettings.Development.json
```

Edit `MovieApi/appsettings.Development.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=moviedb;Username=YOUR_PG_USER;Password=YOUR_PG_PASSWORD"
  },
  "OpenAI": {
    "ApiKey": "sk-..."
  }
}
```

### 3. Create the database table

Connect to PostgreSQL and run:

```sql
CREATE DATABASE moviedb;
\c moviedb
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE movies (
    id          SERIAL PRIMARY KEY,
    title       TEXT NOT NULL,
    genre       TEXT NOT NULL,
    description TEXT NOT NULL,
    embedding   vector(1536),
    created_at  TIMESTAMP DEFAULT NOW()
);
```

### 4. Install frontend dependencies

```powershell
cd movie-ui
npm install
```

---

## Running the App

**Terminal 1 — Start the backend API:**

```powershell
cd MovieApi
dotnet run --urls "http://localhost:5053"
```

**Terminal 2 — Seed the database (run once):**

```powershell
Invoke-RestMethod -Method Post -Uri "http://localhost:5053/api/seed"
# Expected: { seededCount: 30 }
```

> Seeding calls OpenAI 30 times (once per movie description). It takes about 30–60 seconds and uses a small amount of API credit.

**Terminal 3 — Start the frontend:**

```powershell
cd movie-ui
npm run dev
```

Open **http://localhost:5173** in your browser.

---

## Frontend UI

The frontend features an animated background with drifting gradients and floating color blobs, a glass-morphism input panel, and a responsive movie card grid.

Key UI files:

| File | Purpose |
|---|---|
| `movie-ui/src/App.jsx` | Main component: mood text input, API fetch, renders movie cards |
| `movie-ui/src/App.css` | Animated background (`driftGradient`, `floatBlob` keyframes), glass panel, card styles |
| `movie-ui/src/index.css` | Global box-sizing reset and body margin reset |

**Available frontend scripts:**

```powershell
npm run dev      # Start Vite dev server (hot reload)
npm run build    # Production build → dist/
npm run preview  # Preview the production build locally
```

---

## Configuration Reference

| Key | Location | Description |
|---|---|---|
| `ConnectionStrings:Default` | `appsettings.Development.json` | PostgreSQL connection string |
| `OpenAI:ApiKey` | `appsettings.Development.json` | Your OpenAI secret key |
| `VITE_API_BASE_URL` | `movie-ui/.env` (optional) | Override backend URL for the frontend |

---

## Security Notes

- `appsettings.Development.json` is excluded from version control by `.gitignore`. Never commit your API key or database password.
- Use `appsettings.Development.example.json` as the onboarding template for other developers.
- The CORS policy restricts origins to `localhost` only — do not use `AllowAnyOrigin` in production.

---

<div align="center">

Made by [fouzanyousufee1](https://github.com/fouzanyousufee1)

[![GitHub Profile](https://img.shields.io/badge/Visit_GitHub_Profile-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fouzanyousufee1)

</div>
- dotnet run --urls "http://localhost:5053"