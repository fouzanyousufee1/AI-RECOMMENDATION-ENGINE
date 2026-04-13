<div align="center">

# MovieApi — Backend Reference

**ASP.NET Core .NET 10 Web API powering the AI Movie Recommendation Engine.**

[![GitHub Profile](https://img.shields.io/badge/GitHub-fouzanyousufee1-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fouzanyousufee1)
[![Repository](https://img.shields.io/badge/Repo-AI--RECOMMENDATION--ENGINE-0075ff?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fouzanyousufee1/AI-RECOMMENDATION-ENGINE)

</div>

> See the [root README](../README.md) for full project overview, architecture diagrams, and setup instructions.

---

## What This Project Does

This is the backend API for the AI Movie Recommendation Engine. It exposes two HTTP endpoints:

1. **`POST /api/seed`** — Populates the PostgreSQL database with 30 curated movies, each with an AI-generated embedding vector from OpenAI.
2. **`POST /api/recommend`** — Accepts a mood string, embeds it, and returns the top 3 semantically similar movies using pgvector cosine distance.

---

## Files in This Project

| File | Responsibility |
|---|---|
| `Controllers/MoviesController.cs` | All API endpoint logic: seed + recommend, OpenAI calls, pgvector SQL |
| `MovieContext.cs` | Entity Framework Core `DbContext` and `Movie` entity mapping to the `movies` table |
| `Program.cs` | App startup: dependency injection, CORS policy, EF Core configuration |
| `appsettings.json` | Base app settings (no secrets) |
| `appsettings.Development.json` | Local secrets — **not committed**, must be created from the example |
| `appsettings.Development.example.json` | Safe template showing required keys without real values |
| `MovieApi.http` | HTTP scratch file for testing both endpoints manually |
| `MovieApi.csproj` | .NET project file and NuGet package references |

---

## NuGet Dependencies

| Package | Version | Purpose |
|---|---|---|
| `Microsoft.EntityFrameworkCore` | 10.x | ORM for model mapping and DB context |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | 10.x | PostgreSQL provider for EF Core |
| `OpenAI` | 2.10.0 | Official OpenAI SDK (`EmbeddingClient`) |
| `Pgvector` | 0.3.2 | `Vector` C# type for pgvector columns |
| `Pgvector.EntityFrameworkCore` | 0.3.0 | EF Core integration for vector column type |

---

## API Endpoints

### `POST /api/seed`

Clears the `movies` table, generates an OpenAI embedding for each of the 30 fixed movie descriptions, and inserts every row with its vector.

**Request body:** None required.

**Success response (`200 OK`):**
```json
{ "seededCount": 30 }
```

**Side effects:**
- Makes 30 API calls to `text-embedding-3-small`
- Wipes all existing rows before inserting
- Enables the `vector` PostgreSQL extension if not already enabled

---

### `POST /api/recommend`

Takes a mood description, converts it to a vector, and runs a pgvector cosine distance query returning the three closest movie vectors.

**Request body:**
```json
{ "mood": "something funny and clever" }
```

**Success response (`200 OK`):**
```json
[
  {
    "title": "Pulp Fiction",
    "genre": "CRIME / DRAMA",
    "description": "Interconnected stories of hitmen, a boxer, and a gangster's wife...",
    "score": 0.357
  },
  { ... },
  { ... }
]
```

**Error response (`400 Bad Request`):**
```json
"Mood is required."
```

---

## How the Vector SQL Works

```sql
SELECT title, genre, description,
       1 - (embedding <=> @queryVec::vector) AS score
FROM movies
ORDER BY embedding <=> @queryVec::vector
LIMIT 3;
```

- `<=>` is the pgvector **cosine distance** operator
- Cosine distance ranges from 0 (same direction) to 2 (opposite direction)
- `1 - distance` converts it to a **similarity score** from -1 to 1 (higher is better)
- `ORDER BY ... LIMIT 3` retrieves exactly three results

---

## Dependency Injection

Registered in `Program.cs`:

| Service | Lifetime | How Registered |
|---|---|---|
| `MovieContext` | Scoped | `AddDbContext<MovieContext>` with Npgsql + UseVector |
| `OpenAIClient` | Singleton | `AddSingleton(new OpenAIClient(apiKey))` |
| Controllers | Transient | `AddControllers()` |

`MoviesController` receives `IConfiguration` and `OpenAIClient` through constructor injection.

---

## Configuration Keys

Both keys must be present in `appsettings.Development.json` (not committed):

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=moviedb;Username=YOUR_USER;Password=YOUR_PASSWORD"
  },
  "OpenAI": {
    "ApiKey": "sk-..."
  }
}
```

Copy `appsettings.Development.example.json` to get started.

---

## Running the API

```powershell
cd MovieApi
dotnet run --urls "http://localhost:5053"
```

After the API is running, seed the database once:

```powershell
Invoke-RestMethod -Method Post -Uri "http://localhost:5053/api/seed"
```

Then test a recommendation:

```powershell
Invoke-RestMethod -Method Post -Uri "http://localhost:5053/api/recommend" `
  -ContentType "application/json" `
  -Body '{"mood":"scary and suspenseful"}'
```

---

## CORS Policy

The API allows requests from **any localhost origin** (any port):

```csharp
policy.SetIsOriginAllowed(origin => {
    if (!Uri.TryCreate(origin, UriKind.Absolute, out var uri)) return false;
    return uri.Host.Equals("localhost", StringComparison.OrdinalIgnoreCase);
})
```

This supports the Vite dev server on port 5173 or any other local frontend port without needing to update configuration.

---

<div align="center">

Made by [fouzanyousufee1](https://github.com/fouzanyousufee1)

[![GitHub Profile](https://img.shields.io/badge/Visit_GitHub_Profile-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fouzanyousufee1)

</div>
