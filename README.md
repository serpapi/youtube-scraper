# YouTube Search Scraper

[<img width="950" height="300" alt="YouTube Search Scraper tool" src="https://github.com/user-attachments/assets/c3bef2ad-f374-4d9b-9861-6cae4bade944" />](https://serpapi.com/youtube-search-api?utm_source=github_youtube_search_scraper)

YouTube Search Scraper retrieves YouTube search results through SerpApi. Collect video titles, links, video IDs, channel details, views, publication labels, thumbnails, and durations when available, alongside Shorts, playlists, and channel results.

Get structured JSON for applications or Markdown for LLMs and AI agents, without managing HTML parsing or proxies. This guide searches across YouTube with `engine=youtube` and `search_query`; it does not enumerate a specific channel or retrieve full video comments or transcripts. We have different APIs for that.

## How to scrape YouTube search results?

Send a GET request with a search phrase:

```text
https://serpapi.com/search?engine=youtube&search_query=coffee&gl=us&hl=en&api_key=YOUR_SERPAPI_API_KEY
```

- Register at [SerpApi to get your API key](https://serpapi.com?utm_source=github_youtube_search_scraper). Replace `YOUR_SERPAPI_API_KEY` in the examples and keep your key private; never commit it.
- Use `search_query`, not `q`. The examples search for `coffee`; the Python and JavaScript examples also set `gl=us` and `hl=en`.

## Related Scrapers
- [YouTube Video Scraper](https://github.com/serpapi/youtube-video-scraper/)
- [YouTube Video Transcript Scraper](https://github.com/serpapi/youtube-video-transcript-scraper/)
- [YouTube Channel Scraper](https://github.com/serpapi/youtube-channel-scraper/)

## Code examples

### cURL integration

JSON is the default, so no `output` parameter is needed:

```bash
curl --get https://serpapi.com/search \
 -d engine="youtube" \
 -d search_query="coffee" \
 -d api_key="YOUR_SERPAPI_API_KEY"
```

These minimal commands print the API response directly, including any error message. For values containing spaces or special characters, use `--data-urlencode` instead of `-d`. The Python and JavaScript examples below include error handling for automation.

### Output formats: JSON and Markdown

The [official YouTube Search API documentation](https://serpapi.com/youtube-search-api) supports `json` (default), `md` (Markdown), and `html` (raw source for debugging).

JSON preserves named result fields. Markdown is a text response optimized for LLMs and AI agents; its layout is not a guaranteed copy of the JSON schema. Use `https://serpapi.com/search` with `output=md`, keeping the same engine and search parameters.

#### Markdown request

```bash
curl --get https://serpapi.com/search \
 -d engine="youtube" \
 -d search_query="coffee" \
 -d output="md" \
 -d api_key="YOUR_SERPAPI_API_KEY"
```

Read Markdown as text, not with `response.json()` or `getJson`. Inspect error responses rather than treating them as search results.


### Python integration

Install [requests](https://pypi.org/project/requests/), then save the Python example as `main.py`:

```bash
python3 -m pip install requests
```

```py
import requests

SERPAPI_API_KEY = "YOUR_SERPAPI_API_KEY"
params = {
    "api_key": SERPAPI_API_KEY,
    "engine": "youtube",
    "search_query": "coffee",
    "gl": "us",
    "hl": "en",
    "output": "json",
}

response = requests.get("https://serpapi.com/search", params=params, timeout=60)
response.raise_for_status()
data = response.json()
if "error" in data:
    raise RuntimeError(data["error"])
print(data)
```

For Markdown, keep the imports and parameter definitions above and replace the request and response-handling lines with:

```py
params["output"] = "md"
response = requests.get("https://serpapi.com/search", params=params, timeout=60)
response.raise_for_status()
if "application/json" in response.headers.get("Content-Type", ""):
    raise RuntimeError(response.text)
print(response.text)
```

The content-type guard surfaces an unexpected JSON response instead of silently accepting an API error as Markdown.

### JavaScript integration

Install the [SerpApi JavaScript package](https://github.com/serpapi/serpapi-javascript). Save this CommonJS example as `index.cjs`:

```bash
npm install serpapi
```

```js
const { getJson } = require("serpapi");

async function main() {
  const data = await getJson({
    api_key: "YOUR_SERPAPI_API_KEY",
    engine: "youtube",
    search_query: "coffee",
    gl: "us",
    hl: "en",
    timeout: 60000,
  });
  if (data.error) throw new Error(data.error);
  console.log(data);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

The SDK's `timeout` is in milliseconds, and `getJson` selects JSON. Rejected requests and API errors are surfaced rather than replaced with empty results.

For Markdown, use this standalone Node.js 18+ example instead. It uses built-in `fetch`, requires no package, and reads the response as text:

```js
async function main() {
  const params = new URLSearchParams({
    api_key: "YOUR_SERPAPI_API_KEY",
    engine: "youtube",
    search_query: "coffee",
    gl: "us",
    hl: "en",
    output: "md",
  });
  const response = await fetch(`https://serpapi.com/search?${params}`, {
    signal: AbortSignal.timeout(60000),
  });
  const text = await response.text();
  if (!response.ok) throw new Error(`SerpApi HTTP ${response.status}: ${text}`);
  if ((response.headers.get("content-type") || "").includes("application/json")) {
    throw new Error(`Expected Markdown, received JSON: ${text}`);
  }
  console.log(text);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

### Other programming languages

Use a GET request from any language, or explore [SerpApi integrations](https://serpapi.com/integrations?utm_source=github_youtube_search_scraper).

## YouTube Search Scraper parameters

| Name | Description | Requirement |
|------|-------------|-------------|
| `engine` | Must be `youtube`. | Required |
| `api_key` | Your private SerpApi API key. | Required |
| `search_query` | The phrase you would enter in a regular YouTube search. | Required |
| `gl` | Two-letter country code, such as `us`, `uk`, or `fr`. | Optional |
| `hl` | Language code, such as `en`, `es`, or `fr`; regional forms such as `en-gb` and `es-419` are supported. | Optional |
| `sp` | YouTube filter/sort value, exact-spelling filter, or continuation token from the previous response. | Optional |
| `output` | `json` (default), `md`, or `html`. | Optional |
| `no_cache` | `false` (default) allows matching cached responses; `true` fetches fresh results. Cache expires after one hour. Cannot combine with `async`. | Optional |
| `async` | `false` (default) waits for results; `true` submits for later retrieval through the [Searches Archive API](https://serpapi.com/search-archive-api). Do not combine with `no_cache` or use with Ludicrous Speed enabled. | Optional |
| `zero_trace` | Enterprise-only storage opt-out; `false` (default) or `true`. | Optional |
| `json_restrictor` | Select JSON fields with the [JSON Restrictor](https://serpapi.com/json-restrictor). Retain pagination fields when collecting multiple pages. | Optional |

See the [official documentation](https://serpapi.com/youtube-search-api) for current parameters. Matching cached searches are free and do not count toward monthly searches. The examples use synchronous requests.

### Filtering and pagination

YouTube uses continuous, token-based pagination, not a `page`, `start`, or fixed-size offset:

1. Make the initial JSON request with `search_query`, localization, and any chosen `sp` filter.
2. Read `serpapi_pagination.next_page_token` or `pagination.next_page_token`.
3. Put that exact returned value in the next request's **`sp`** parameter. Keep `engine`, `search_query`, `gl`, `hl`, and authentication consistent. Replace the previous `sp`; do not concatenate filters and continuation tokens.
4. Repeat with each newly returned token. Stop when no token is returned; also guard against repeated tokens and impose a page limit to control costs.

Do not use the channel API's request parameter `next_page_token` with this engine. A page can contain different result types, so the absence of `video_results` alone is not a reliable pagination stop condition. Result counts and page sizes are not fixed.

For filters and sorting, select the desired options on YouTube and copy the `sp` parameter value from its URL. Treat tokens as opaque strings. Pass returned tokens through `--data-urlencode`, `requests` parameters, or the SDK without manually adding another encoding layer; avoid repeatedly encoding values copied from an already encoded URL.

### Follow-up requests

- For channel metadata and a channel's uploads, use YouTube Channel Scraper.
- For full video details and available comments, pass a returned `video_id` as `v` to `engine=youtube_video`. See the [Video API documentation](https://serpapi.com/youtube-video-api).
- For available transcript text, use `engine=youtube_video_transcript` with the same video ID as `v`. See the [Transcript API documentation](https://serpapi.com/youtube-video-transcript).

These are separate requests, not fields guaranteed on the search response. Search results may supply a `serpapi_link` for video details; authenticate follow-ups with your own key.

## Available data on YouTube search (JSON response)

The following is a **field guide, not a literal API response**. Descriptive strings indicate types and meaning; fields and result sections depend on the search.

```json
{
  "search_metadata": {
    "id": "String: SerpApi search ID",
    "status": "String: Processing, Success, or Error"
  },
  "search_information": {
    "total_results": "Integer: reported total when available",
    "video_results_state": "String: spelling/result state"
  },
  "video_results": [
    {
      "position_on_page": "Integer: position on this results page",
      "title": "String: video title",
      "link": "String: YouTube video URL",
      "serpapi_link": "String: video details API URL",
      "video_id": "String: YouTube video ID",
      "channel": {
        "name": "String: channel name",
        "link": "String: channel URL",
        "verified": "Boolean: verification badge when present",
        "thumbnail": "String: channel thumbnail URL"
      },
      "published_date": "String: YouTube publication label",
      "views": "Integer: parsed views when available",
      "length": "String: duration label",
      "description": "String: snippet",
      "extensions": ["String: badge such as CC or New"],
      "thumbnail": {
        "static": "String: static image URL",
        "rich": "String: preview image URL when available"
      }
    }
  ],
  "shorts_results": [
    {
      "position_on_page": "Integer: position of the Shorts section",
      "shorts": [
        {
          "title": "String: Short title",
          "link": "String: Short URL",
          "thumbnail": "String: thumbnail URL",
          "views_original": "String: displayed view count",
          "views": "Integer: parsed view count",
          "video_id": "String: video ID"
        }
      ]
    }
  ],
  "serpapi_pagination": {
    "next": "String: next SerpApi request URL",
    "next_page_token": "String: pass as sp on the next request"
  }
}
```

| JSON key | Meaning and shape |
|----------|-------------------|
| `video_results` | Main organic videos array; see the [video results schema](https://serpapi.com/youtube-video-results). |
| `channel_results` | Array of channel matches, with fields such as `title`, `link`, `handle`, `subscribers`, and `description`. Unlike `youtube_channel`, this is not a single channel object. |
| `playlist_results` | Array of playlist matches; see the [playlist results documentation](https://serpapi.com/youtube-playlist-results). Note the singular `playlist`, unlike channel `playlists_results`. |
| `shorts_results` | Array of sections containing nested `shorts` arrays; see the [Shorts schema](https://serpapi.com/youtube-shorts-results). |
| `latest_from_*` | Channel-specific video sections can use dynamic keys, such as `latest_from_mr_beast`; do not assume one fixed key. |
| `ads_results` | Sponsored results, separate from organic videos. |
| `pagination` | YouTube-side `current`, `next`, and `next_page_token` when available. |
| `serpapi_pagination` | SerpApi-side pagination links and continuation token when available. |
| `error` | API failure message; check before consuming results. |

Publication labels may be relative strings rather than timestamps. Missing fields are not zero values. Shorts thumbnails and view labels differ from regular video fields, and page positions should not be treated as global ranks across all pages.

## Use cases

- Monitor keyword visibility and competing videos using consistent country and language settings.
- Discover relevant videos, Shorts, playlists, and creators for research.
- Supply current search context to an AI agent as Markdown, then use video IDs for targeted follow-up requests.

## Blog tutorials

- [How to scrape YouTube data in 2026 ](https://serpapi.com/blog/how-to-scrape-youtube-data-with-simple-api/)
- [Competitive Video Analysis Using YouTube Search API](https://serpapi.com/blog/competitive-video-analysis-using-youtube-search-api/)

Use the current API documentation above for parameter and response details; filter URLs in tutorials may contain URL-encoded values.

## Video tutorial

- [How to Scrape YouTube Video Search Results (with Python)](https://www.youtube.com/watch?v=ZfPLN-2Xm6c)

## Contacts

Feel free to reach out via `contact@serpapi.com`.
