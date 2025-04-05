# YouLy+ API Documentation

## 🌐 Base URL

```
https://lyricsplus.prjktla.workers.dev
```

LyricsPlus is implemented as a serverless application. For source code and implementation details, visit:  
[GitHub Repository](https://github.com/ibratabian17/LyricsPlus-Backend)

---

---

## 🎵 Lyrics Endpoints

### 1. Get Lyrics (v1)
```http
GET /v1/lyrics/get
```

**Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| title | string | Yes | Song title |
| artist | string | Yes | Artist name |
| album | string | No | Album name |
| duration | number | No | Song duration in milliseconds |
| source | string | No | Comma-separated sources (lyricsplus, apple, musixmatch, spotify) |
| forceReload | boolean | No | Bypass cache (default: false) |

**Example Request**:
```bash
curl "https://lyricsplus.prjktla.workers.dev/v1/lyrics/get?title=Title&artist=Meghan%20Trainor&source=lyricsplus,apple&forceReload=true"
```

**Example Response**:
```json
{
  "type": "Word",
  "KpoeTools": "1.31-JSRegex",
  "metadata": {
    "source": "Apple",
    "leadingSilence": null,
    "songWriters": [
      "Chris \"Tricky\" Stewart",
      "Christina Milian",
      "Christopher Bridges",
      "Justin Bieber",
      "Terius Nash"
    ]
  },
  "lyrics": [
    {
      "time": 2432,
      "duration": 459,
      "text": "Oh, ",
      "isLineEnding": 0,
      "element": {
        "key": "L1",
        "songPart": "Intro",
        "singer": "v1"
      }
    },
    {
      "time": 2891,
      "duration": 1607,
      "text": "whoa",
      "isLineEnding": 1,
      "element": {
        "key": "L1",
        "songPart": "Intro",
        "singer": "v1"
      }
    }]
}
```

---

### 2. Get Lyrics (v2)
```http
GET /v2/lyrics/get
```

**Parameters**:  
Same as v1 endpoint

**Response Improvements**:
- Structured timing data
- Word-level synchronization
- Multiple provider metadata

**Example Response**:
```json
{
  "type": "Word",
  "KpoeTools": "1.31R2-LPlusBcknd,1.31-JSRegex",
  "metadata": {
    "source": "Apple",
    "leadingSilence": null,
    "songWriters": [
      "Chris \"Tricky\" Stewart",
      "Christina Milian",
      "Christopher Bridges",
      "Justin Bieber",
      "Terius Nash"
    ]
  },
  "lyrics": [
    {
      "time": 2432,
      "duration": 2066,
      "text": "Oh, whoa",
      "syllabus": [
        {
          "time": 2432,
          "duration": 459,
          "text": "Oh, "
        },
        {
          "time": 2891,
          "duration": 1607,
          "text": "whoa"
        }
      ],
      "element": {
        "key": "L1",
        "songPart": "Intro",
        "singer": "v1"
      }
    }]
}
```

---

### 3. Get Apple Music Format (TTML)
```http
GET /v1/ttml/get
```

**Parameters**:  
Same as v1 endpoint

**Response Format**:
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

## 📤 Publish Endpoint

### Submit Lyrics

> **Note**: This endpoint has been temporarily removed due to security concerns. A new secure submission process will be implemented in a future update.

For any lyrics submission inquiries, please contact the maintainers through the [GitHub repository](https://github.com/ibratabian17/LyricsPlus-Backend).

---

## 🔄 Response Codes

| Code | Description |
|------|-------------|
| 200 | Successful response |
| 400 | Invalid parameters |
| 404 | Lyrics not found |
| 429 | Rate limit exceeded |
| 500 | Server error |

---

## 🛠️ Client Implementation Notes

1. **Caching**:
```javascript
// Recommended cache strategy
const CACHE_TTL = 3600; // 1 hour
const cacheKey = `${title}-${artist}-${Date.now() / CACHE_TTL | 0}`;
```

2. **Error Handling**:
```javascript
try {
  const response = await fetch(url);
  if (!response.ok) {
    const error = await response.json();
    handleError(error.code, error.message);
  }
} catch (e) {
  handleNetworkError(e);
}
```