# AI Recommendation Engine

Full-stack movie recommendation app with a React frontend and an ASP.NET Core Web API backend.

## Project Structure

- MovieApi: ASP.NET Core Web API, Entity Framework Core, PostgreSQL, pgvector package references
- movie-ui: React 19 + Vite frontend

## Stack

- Frontend: React, Vite
- Backend: ASP.NET Core (.NET 10)
- Database: PostgreSQL
- Packages configured in backend: Entity Framework Core, OpenAI SDK, pgvector

## Current App Behavior

The frontend sends a mood prompt to the backend at POST /api/recommend.

The backend loads movies from the database and returns ranked recommendations with title, genre, description, and a score. The current implementation is database-backed and returns working recommendations for the UI.

## Prerequisites

- .NET 10 SDK
- Node.js and npm
- PostgreSQL

## Local Setup

1. Create your local backend development settings:

   Copy MovieApi/appsettings.Development.example.json to MovieApi/appsettings.Development.json and fill in your own values.

2. Install frontend dependencies:

```powershell
cd movie-ui
npm install
```

## Run The App

Start the backend:

```powershell
cd MovieApi
dotnet run --urls "http://localhost:5053"
```

Start the frontend in another terminal:

```powershell
cd movie-ui
npm run dev
```

Then open the Vite URL shown in the terminal, typically:

- http://localhost:5173

## API Endpoint

- POST /api/recommend

Example request body:

```json
{
  "mood": "funny and clever"
}
```

## Notes

- The backend CORS policy allows localhost frontend ports for local development.
- The frontend is configured to try the backend on http://localhost:5053 first, then http://localhost:5000.
- Local secret values are not committed. Use the example development settings file as the template.

## Scripts

Frontend:

- npm run dev
- npm run build
- npm run preview

Backend:

- dotnet build
- dotnet run --urls "http://localhost:5053"