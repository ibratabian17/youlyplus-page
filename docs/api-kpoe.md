# YouLy+ / LyricsPlus API Documentation

## 🌐 Base URL

```
https://lyricsplus.prjktla.my.id
```

Source code & backend architecture: [GitHub Repository](https://github.com/ibratabian17/LyricsPlus-Backend).

---

## ⚡ Multi-Provider Architecture & Smart Race

The backend aggregates synchronized and syllable-level lyrics from **7 upstream providers and storage tiers**:

| Provider ID | Source Name | Sync Capabilities | Description |
|:------------|:------------|:------------------|:------------|
| `apple` | **Apple Music** | Word (Syllable) & Line | Native TTML parser extracting microsecond timestamps and vocal parts. |
| `spotify` | **Spotify** | Word (Syllable) & Line | Syllable-level token stream parsed from Spotify Canvas/Lyrics endpoints. |
| `qq` | **QQ Music** | Word (QRC) & Line (LRC) | Triple-DES decrypted QRC lyrics format with millisecond word tags. |
| `qaple` | **Qaple (Merged)** | Word & Line | QQ Music syllables synced against Apple Music catalog metadata. |
| `musixmatch` | **Musixmatch** | Line | Rich multi-language lyrics catalog. |
| `deezer` | **Deezer** | Line | Synchronized Deezer LRC streams. |
| `lyricsplus` | **Lyrics+ UGC** | Word & Line | Local SQLite database + Google Drive community user submissions. |

### Racing & Grade Priority

When a search request arrives, the backend executes an asynchronous race:
1. **Cache Tier**: Checks local SQLite database (`FTS5` + composite index by ISRC, Platform ID, or Title/Artist).
2. **Phase 1 (Priority Word)**: Concurrently queries word-level providers (`apple`, `spotify`, `qq`). If a word-level match is found, the race immediately finalizes.
3. **Phase 2 (Priority Line)**: If only line-level lyrics are found, background providers are queried to upgrade to word-level precision.
4. **Execution Diagnostics**: Every response returns a complete `processingTime.sourcesStatus` map detailing latency and outcomes (`OK`, `BAD`, `RTO`, `SKIP`) per provider.

---

## 🎵 Lyrics Endpoints

### 1. Get Lyrics V2 (Canonical)

Fetches the complete V2 synchronized lyrics payload including syllable timing and backend execution state.

```http
GET /v2/lyrics/get
```

#### Query Parameters:

| Parameter | Type | Required | Description |
|:----------|:-----|:---------|:------------|
| `title` | string | Conditional* | Track title. Required unless `isrc` or `platformId` is provided. |
| `artist` | string | Conditional* | Artist name. Required unless `isrc` or `platformId` is provided. |
| `album` | string | No | Album name used as a disambiguation hint. |
| `duration` | number | No | Track duration in seconds or milliseconds (e.g. `214` or `214.5`). |
| `isrc` | string | Conditional* | International Standard Recording Code for instant cache hits. |
| `platformId` | string | Conditional* | Platform track ID (e.g. Spotify ID, Apple ID, or QQ mid). |
| `source` | string | No | Comma-separated preferred providers, e.g. `apple,spotify,qq`. |
| `forceReload`| boolean | No | Set `true` to bypass cache and race upstream providers fresh. |

*\*Conditional Requirement*: Provide **either** (`title` and `artist`) **OR** (`isrc` / `platformId`).

#### Example Request:
```bash
curl "https://lyricsplus.prjktla.my.id/v2/lyrics/get?title=Mode%20Pesawat&artist=Dandy%20Barakati&source=qq,apple"
```

#### Response Shape (`200 OK`):
```json
{
  "type": "Word",
  "KpoeTools": "2.0-LPlusBcknd,lyricsplus",
  "metadata": {
    "source": "QQ Music",
    "title": "MODE PESAWAT",
    "artist": "Dandy Barakati & Dante Nababan",
    "album": "MODE PESAWAT",
    "songWriters": ["Dandy Barakati", "Dante Nababan"],
    "language": "id",
    "totalDuration": "3:24.000"
  },
  "lyrics": [
    {
      "time": 12400,
      "duration": 2600,
      "text": "Terbang tinggi di awan",
      "syllabus": [
        { "time": 12400, "duration": 500, "text": "Ter" },
        { "time": 12900, "duration": 400, "text": "bang " },
        { "time": 13300, "duration": 600, "text": "ting" },
        { "time": 13900, "duration": 400, "text": "gi " },
        { "time": 14300, "duration": 700, "text": "awan" }
      ],
      "element": { "key": "L1", "singer": "v1" }
    }
  ],
  "cached": "Database",
  "processingTime": {
    "timeElapsed": 85,
    "lastProcessed": 1791527000000,
    "totalElapsedMs": 85,
    "winnerSource": "QQ Music",
    "syncPriority": 3,
    "sourcesStatus": {
      "qq": { "status": "OK", "elapsedMs": 72 },
      "apple": { "status": "BAD", "elapsedMs": 140 }
    },
    "selectedSongMetadata": {
      "source": "QQ Music",
      "title": "MODE PESAWAT",
      "artist": "Dandy Barakati & Dante Nababan",
      "album": "MODE PESAWAT"
    }
  }
}
```

#### Cache Tiers:
- `"Database"`: Retrieved instantly from SQLite UGC / cache.
- `"GDrive"`: Retrieved from synchronized Google Drive mirror.
- `"UserJSON"`: Retrieved from community submitted JSON repository.
- `"None"`: Fetched live from upstream provider race.

#### Diagnostic Response Shape (`404 Not Found`):
When lyrics cannot be located, the backend provides detailed race diagnostics:
```json
{
  "error": {
    "message": "Lyrics not found in sources: qq",
    "status": 404,
    "details": {
      "searchedSources": ["qq"],
      "songInfo": {
        "title": "MODE PESAWAT",
        "artist": "Dandy Barakati",
        "album": ""
      }
    }
  },
  "processingTime": {
    "lastProcessed": 1791527100000,
    "sourcesStatus": {
      "qq": { "status": "BAD", "elapsedMs": 0 }
    },
    "syncPriority": null,
    "timeElapsed": 8000,
    "totalElapsedMs": 8000,
    "winnerSource": null
  }
}
```

---

### 2. Get Legacy Lyrics V1

Returns lyrics in flat token segments.

```http
GET /v1/lyrics/get
```
*Parameters match V2.*

---

### 3. Get Apple Music TTML

Returns the winner lyrics payload converted into Apple Music Timed Text Markup Language (TTML XML).

```http
GET /v1/ttml/get
```

**Response**:
```json
{
  "ttml": "<?xml version=\"1.0\" encoding=\"utf-8\"?><tt xmlns=\"http://www.w3.org/ns/ttml\"><body>...</body></tt>",
  "processingTime": { "totalElapsedMs": 120 }
}
```

---

### 4. Get Raw Upstream Payload

Fetches the unparsed, raw upstream payload directly as returned by the winner provider.

```http
GET /v1/raw/get
```

- Headers returned: `X-Lyrics-Source: <provider>` and `X-Processing-Time: <ms>`.
- Content-Type: `application/xml` for Apple/QQ, `application/json` for Spotify/Musixmatch.

---

## 📚 Catalog & Metadata

### 1. Search Song Catalog

Queries Apple Music, Spotify, QQ Music, and Musixmatch in parallel, merging and de-duplicating results by ISRC and track title.

```http
GET /v1/songlist/search?q={query}
```

**Response Item**:
```json
{
  "results": [
    {
      "id": { "spotify": "4cOdK2w...", "apple": "1658571...", "qq": "002vPwd..." },
      "sourceId": "4cOdK2w...",
      "title": "Blinding Lights",
      "artist": "The Weeknd",
      "album": "After Hours",
      "albumArtUrl": "https://...",
      "durationMs": 200040,
      "isrc": "USUG11904206",
      "songwriters": ["Abel Tesfaye", "Max Martin"],
      "availability": ["apple", "spotify", "qq"]
    }
  ]
}
```

### 2. Track Metadata

Returns rich metadata from Apple Music catalog.

```http
GET /v1/metadata/get?title={title}&artist={artist}&album={album}
```

---

## 📤 User-Generated Submission (PoW Protected)

Submissions allow community contributors to add or update synchronized lyrics stored in the SQLite database and synced to Google Drive.

### Step 1: Request Challenge

```http
GET /v1/lyricsplus/challenge
```

**Response**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "difficulty": 4
}
```

The challenge requires finding a `nonce` string such that the SHA-256 of `challenge + nonce` begins with `difficulty` zero nibbles (hex characters).

### Step 2: Submit Lyrics

```http
POST /v1/lyricsplus/submit
Content-Type: application/json
```

**Request Body**:
```json
{
  "proofOfWorkToken": "<token from step 1>",
  "nonce": "14820",
  "songTitle": "Song Title",
  "songArtist": "Artist Name",
  "songAlbum": "Album Title",
  "songDuration": "200",
  "songISRC": "USUG11904206",
  "songPlatformId": "4cOdK2wGLETKBW3PvgPWqT",
  "forceUpload": false,
  "lyricsData": {
    "type": "Word",
    "lyrics": [ ... ]
  }
}
```

**Response (`200 OK`)**:
```json
{
  "success": true,
  "message": "Lyrics submitted successfully",
  "filename": "Artist Name - Song Title.json"
}
```

---

## 🩺 System Ops & Live Health State

### 1. Health Probe (`/health`)

Returns real-time operational status, SQLite connection, upstream platform availability, and latency metrics.

```http
GET /health
```

**Response**:
```json
{
  "service": "lyricsplus-backend",
  "status": "ok",
  "version": "1.0.0",
  "commit": "gelato-7e76631",
  "buildDate": "2026-10-09T01:44:00Z",
  "uptimeSeconds": 48210,
  "database": "ok",
  "platforms": {
    "apple": { "name": "Apple Music", "configured": true },
    "spotify": { "name": "Spotify", "configured": true },
    "qq": { "name": "QQ Music", "configured": true },
    "lyricsplus": { "name": "Lyrics+", "configured": true },
    "gdrive": { "name": "Google Drive", "configured": true },
    "database": { "name": "LyricsDB", "configured": true }
  }
}
```

### 2. Readiness Probe (`/readyz`)
Returns `200 OK` if the SQLite database is healthy, or `503 Service Unavailable` if degraded.

### 3. Interactive Documentation
- OpenAPI Spec: `GET /openapi.yaml`
- Interactive Viewer: `GET /docs`