# YouLy+ API Documentation

## 🌐 Base URL

```
https://lyricsplus.prjktla.workers.dev
```

LyricsPlus is implemented as a serverless application. For source code and implementation details, visit the [GitHub Repository](https://github.com/ibratabian17/LyricsPlus).

---

## 🧠 Search Logic & Priority

When retrieving lyrics, the API uses a "Smart Priority" system to ensure the highest accuracy.

1.  **Full Search Scope**: The system queries all configured sources (including Apple Music, Musixmatch, and Spotify).
2.  **Cache Lookup**:
    *   If an `isrc` or `platformId` is provided, the system checks the cache for an exact match first.
3.  **External Disambiguation**:
    *   If no cache hit occurs, the system queries external providers using `title` and `artist`.
    *   If `isrc` or `platformId` were provided, they are used to filter and select the specific version of the song from the external results.

> **Tip**: For the most accurate results, provide `isrc` or `platformId` alongside `title` and `artist`.

---

## 🎵 Lyrics Endpoints

### 1. Get Lyrics (v1)
Fetches synchronized lyrics in the legacy format.

```http
GET /v1/lyrics/get
```

**Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Conditional* | Song title. |
| `artist` | string | Conditional* | Artist name. |
| `isrc` | string | Conditional* | International Standard Recording Code. |
| `platformId` | string | Conditional* | Platform-specific ID (Apple, Spotify, etc.). |
| `album` | string | No | Album name. |
| `duration` | number | No | Song duration in milliseconds. |
| `source` | string | No | Preferred sources (e.g., `musixmatch,spotify`). |
| `forceReload` | boolean | No | Set `true` to bypass cache. |

> **\*Conditional Requirement**: You must provide **either** (`title` AND `artist`) **OR** (`isrc` / `platformId`).

**Example Request**:
```bash
curl "https://lyricsplus.prjktla.workers.dev/v1/lyrics/get?title=Title&artist=Meghan%20Trainor&isrc=US123456789&source=apple"
```

**Example Response**:
```json
{
  "type": "Word",
  "metadata": {
    "source": "Apple",
    "songWriters": ["Writer 1", "Writer 2"]
  },
  "lyrics": [
    {
      "time": 2432,
      "duration": 459,
      "text": "Oh, ",
      "isLineEnding": 0
    },
    {
      "time": 2891,
      "duration": 1607,
      "text": "whoa",
      "isLineEnding": 1
    }
  ]
}
```

---

### 2. Get Lyrics (v2)
Fetches synchronized lyrics with improved structure (syllable grouping).

```http
GET /v2/lyrics/get
```

**Parameters**: Same as **v1**.

**Example Response**:
```json
{
  "type": "Word",
  "metadata": { "source": "Apple" },
  "lyrics": [
    {
      "time": 2432,
      "duration": 2066,
      "text": "Oh, whoa",
      "syllabus": [
        { "time": 2432, "duration": 459, "text": "Oh, " },
        { "time": 2891, "duration": 1607, "text": "whoa" }
      ]
    }
  ]
}
```

---

### 3. Get Apple Music Format (TTML)
Fetches lyrics in Timed Text Markup Language (TTML).

```http
GET /v1/ttml/get
```

**Parameters**: Same as **v1**.

**Example Response**:
```xml
<tt xmlns="http://www.w3.org/ns/ttml">
  <body>
    <div begin="00:00:01.200" end="00:00:04.500">
      <p>I'm loving the shape of you</p>
    </div>
  </body>
</tt>
```

---

## 📚 Catalog & Metadata

### 1. Search Song Catalog
Search for songs within the catalog.

```http
GET /v1/songlist/search
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | Yes | Search query (Title, Artist, etc.). |

### 2. Get Song Metadata
Obtains comprehensive metadata (Cover art, release date, etc.).

```http
GET /v1/metadata/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Song title. |
| `artist` | string | Yes | Artist name. |
| `album` | string | No | Album name. |
| `duration` | number | No | Duration in ms. |

---

## 📤 Submission Endpoints

The submission process is secured via a Proof-of-Work (PoW) challenge to prevent spam.

### Step 1: Request Challenge
Get a token and difficulty level for the PoW calculation.

```http
GET /v1/lyricsplus/challenge
```

**Response**:
```json
{
  "token": "eyJhbGciOiJIUz...",
  "difficulty": 4
}
```

### Step 2: Submit Lyrics
Submit the lyrics along with the solved PoW.

```http
POST /v1/lyricsplus/submit
```

**Body Parameters (JSON)**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `proofOfWorkToken` | string | Yes | Token from Step 1. |
| `nonce` | string | Yes | The calculated solution to the PoW. |
| `songTitle` | string | Yes | Song title. |
| `songArtist` | string | Yes | Artist name. |
| `songAlbum` | string | No | Album name. |
| `songDuration` | number | Yes | Duration in ms. |
| `lyricsData` | object | Yes | Synchronized lyrics content. |
| `forceUpload` | boolean | No | Force overwrite (requires privileges). |

---

## 🔄 Response Codes

| Code | Description |
|------|-------------|
| `200` | Successful request. |
| `400` | Bad Request (Invalid parameters/body). |
| `404` | Not Found (Lyrics or Song not found). |
| `429` | Rate limit exceeded. |
| `500` | Internal Server Error. |

---

## 🛠️ Client Implementation Notes

### Caching Strategy
To reduce latency and API load, implement client-side caching.
```javascript
// Recommended cache strategy
const CACHE_TTL = 3600; // 1 hour
// Include IDs in cache key for precision
const cacheKey = `${title}-${artist}-${isrc}-${Date.now() / CACHE_TTL | 0}`;
```

### Handling Errors
```javascript
try {
  const response = await fetch(url);
  if (!response.ok) {
    const error = await response.json();
    console.error(`Error ${response.status}:`, error.message);
  }
} catch (e) {
  console.error("Network Error", e);
}
```