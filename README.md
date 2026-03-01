# EZ Tiers Public API Documentation  
**Version 1.0**  
**Last Updated:** March 2026  

## Overview  

The EZ Tiers Public API provides read‑only access to the official EZ Tiers competitive ranking database. It delivers global standings, player profiles, gamemode‑specific leaderboards, and individual tier lookups. The data reflects the same rankings displayed on the EZ Tiers front‑end application and is updated in near real‑time.

**Base URL**  
```
https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api
```

All responses are returned in **JSON** format. The API supports **CORS** for cross‑origin requests from any domain.

## Authentication  

The API is **public** and requires no authentication. Simply send HTTP GET requests to the desired endpoint. Rate limiting is currently not enforced, but we ask that you be respectful and avoid excessive polling.

## Data Model  

### Player  
- `id` (integer) – Unique identifier  
- `username` (string) – Minecraft Java Edition username  
- `created_at` (ISO 8601 timestamp) – When the player was added to the database  

### Tier  
A tier is a string representing a player’s skill level in a specific gamemode. Possible values are:  
`UNRANKED`, `HT1`, `LT1`, `HT2`, `LT2`, `HT3`, `LT3`, `HT4`, `LT4`, `HT5`, `LT5`.  

Higher tiers are better: **HT1** is the highest, **LT5** the lowest.  

### Gamemodes  
The following competitive gamemodes are tracked:  
`sword`, `axe`, `crystal`, `nethpot`, `diapot`, `smp`, `uhc`, `mace`, `speedbeast`

## Endpoints  

### 1. Global Ranking  
Returns all players sorted by total points (descending). Points are calculated based on the player’s tier in every gamemode.

| Method | Endpoint   |
|--------|------------|
| GET    | `/global`  |

**Example Request**  
```bash
curl https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/global
```

**Example Response**  
```json
[
  {
    "id": 42,
    "username": "Technoblade",
    "total_points": 187,
    "best_tier": "HT1"
  },
  {
    "id": 17,
    "username": "Dream",
    "total_points": 156,
    "best_tier": "HT1"
  }
]
```

**Fields**  
- `total_points` – Sum of points awarded for each tier (see point values below).  
- `best_tier` – The highest tier the player holds in any gamemode.

---

### 2. List Players  
Retrieve a paginated list of players with optional search.

| Method | Endpoint            | Query Parameters                          |
|--------|---------------------|--------------------------------------------|
| GET    | `/players`          | `limit` (default 100), `offset` (default 0), `search` (string) |

**Example Request**  
```bash
curl "https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/players?limit=5&search=steve"
```

**Example Response**  
```json
[
  {
    "id": 3,
    "username": "Steve",
    "created_at": "2025-01-15T10:30:00Z"
  },
  {
    "id": 44,
    "username": "Steveee",
    "created_at": "2025-06-22T14:12:00Z"
  }
]
```

---

### 3. Player Details by ID  
Fetch a single player’s profile and their tier in every gamemode.

| Method | Endpoint               |
|--------|------------------------|
| GET    | `/players/{player_id}` |

**Example Request**  
```bash
curl https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/players/42
```

**Example Response**  
```json
{
  "id": 42,
  "username": "Technoblade",
  "created_at": "2024-11-01T00:00:00Z",
  "tiers": {
    "sword": "HT1",
    "axe": "HT1",
    "crystal": "HT1",
    "nethpot": "HT1",
    "diapot": "HT1",
    "smp": "HT1",
    "uhc": "HT1",
    "mace": "HT1",
    "speedbeast": "HT1"
  },
  "total_points": 187
}
```

---

### 4. List Gamemodes  
Returns the list of all competitive gamemodes.

| Method | Endpoint     |
|--------|--------------|
| GET    | `/gamemodes` |

**Example Request**  
```bash
curl https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/gamemodes
```

**Example Response**  
```json
["sword","axe","crystal","nethpot","diapot","smp","uhc","mace","speedbeast"]
```

---

### 5. Gamemode Ranking  
Returns all players ranked in a specific gamemode, sorted by tier (best first). Players with `UNRANKED` are excluded.

| Method | Endpoint                    |
|--------|-----------------------------|
| GET    | `/gamemodes/{gamemode}`     |

**Example Request**  
```bash
curl https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/gamemodes/sword
```

**Example Response**  
```json
[
  {
    "player_id": 42,
    "username": "Technoblade",
    "tier": "HT1"
  },
  {
    "player_id": 17,
    "username": "Dream",
    "tier": "HT1"
  }
]
```

---

### 6. Player Tier in a Specific Gamemode  
Retrieve a player’s tier for a given gamemode. The username is matched case‑insensitively.

| Method | Endpoint                                 |
|--------|------------------------------------------|
| GET    | `/gamemodes/{gamemode}/players/{username}` |

**Example Request**  
```bash
curl https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/gamemodes/sword/players/Technoblade
```

**Example Response (player found)**  
```json
{
  "username": "Technoblade",
  "gamemode": "sword",
  "tier": "HT1"
}
```

**Example Response (player not found)**  
```json
{
  "error": "Player not found"
}
```
Status code: `404 Not Found`

**Note:** If the player exists but is not ranked in the requested gamemode, the tier will be `"UNRANKED"`.

---

## Error Handling  

The API uses conventional HTTP response codes:

| Code | Description                     | Possible Cause                     |
|------|---------------------------------|------------------------------------|
| 200  | OK                              | Request succeeded                  |
| 404  | Not Found                       | Invalid endpoint or player not found |
| 500  | Internal Server Error           | Unexpected server issue             |

Error responses are returned as JSON with an `error` field describing the problem.

## Points Calculation (for Global Ranking)  

The points used in the global ranking are derived from the tier values as follows:

| Tier  | Points |
|-------|--------|
| HT1   | 60     |
| LT1   | 45     |
| HT2   | 30     |
| LT2   | 20     |
| HT3   | 10     |
| LT3   | 6      |
| HT4   | 4      |
| LT4   | 3      |
| HT5   | 2      |
| LT5   | 1      |
| UNRANKED | 0  |

## Rate Limiting  

Currently there are no hard rate limits. However, if you plan to make frequent automated requests, please contact us to discuss a custom integration. We reserve the right to introduce rate limiting in the future to ensure service stability.

## Code Examples  

### cURL  
```bash
# Global ranking
curl https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/global

# Player tier in sword gamemode
curl https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/gamemodes/sword/players/Technoblade
```

### JavaScript (Fetch)  
```javascript
fetch('https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/gamemodes')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

### Python (Requests)  
```python
import requests

response = requests.get('https://jdayouvytwexmpzcrnzi.supabase.co/functions/v1/api/global')
if response.status_code == 200:
    data = response.json()
    print(data)
else:
    print('Error:', response.json().get('error'))
```

## Support  

For questions, feature requests, or to report issues, please contact the EZ Tiers development team at **rayzz.office@gmail.com**

---

**© 2026 EZ Tiers. All rights reserved.**
