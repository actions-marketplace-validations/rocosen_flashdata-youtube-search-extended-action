# YouTube Extended Search

Search YouTube with a result limit of up to 700 and optional filters. Export matching records as JSON and JSONL.

Powered by the [FlashData YouTube Extended Search API](https://flashdata.dev/docs/sources/youtube_search_extended). This is a distinct source with a separate higher rate than standard search. Requested count is not guaranteed.

## Quick start

Add a [FlashData API key](https://flashdata.dev/signup) as the repository secret `FLASHDATA_API_KEY`. API calls require FlashData Credits ([pricing](https://flashdata.dev/pricing)). Then add this workflow:

```yaml
name: YouTube Extended Search
on:
  workflow_dispatch: {}
permissions:
  contents: read
jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - name: YouTube Extended Search
        id: data
        uses: rocosen/flashdata-youtube-search-extended-action@v0.1.1
        with:
          api-key: ${{ secrets.FLASHDATA_API_KEY }}
          query: retrieval evaluation tutorials
          options-json: '{"max_results":50}'
      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: youtube-search-extended
          path: ${{ steps.data.outputs.artifact-directory }}
          retention-days: 7
```

Run it from the **Actions** tab. Download the exported files from the run’s **Artifacts**, or use the output paths in later steps. Requires an up-to-date runner with Node 24 support; no dependency installation is needed. For production, pin actions to a reviewed commit SHA.

## Inputs

| Input | Required | Description |
| --- | --- | --- |
| `api-key` | Yes | FlashData API Key. Pass secrets.FLASHDATA_API_KEY; requests use paid Credits. |
| `query` | Yes | Search topic or autocomplete prefix. |
| `options-json` | No | Optional JSON object with only source-specific options listed in the README. Never include credentials here. |

### Options

Pass these fields in `options-json`:

`max_results` (1–700, default 20); `filters`: duration (<4, 4-20, >20), sort_by (rating, relevance, view_count, upload_date), type (video, channel, playlist, movie), upload_date (today, last_hour, this_week, this_month, this_year), and boolean feature filters (360, 3d, 4k, creative_commons, hd, hdr, live, location, purchased, subtitles, vr180).

Invalid options fail before an API request. One paid query per run; no automatic pagination or retries.

## Outputs

- `response.json`: API response, including available result fields and usage.
- `records.jsonl`: one search result per line.
- `manifest.json`: source, operation, timestamps, record count and export warnings, plus query/options and reported usage.

Returned API fields: results[].results: video search records; totalResults and query; usage.credits.

Empty results produce an empty JSONL file and `result-count: 0`. Optional fields may be absent.

| Step output | Description |
| --- | --- |
| `artifact-directory` | Absolute directory with response.json, records.jsonl and manifest.json. |
| `results-file` | Absolute path to the original response JSON (sanitized lifecycle fields for download jobs). |
| `records-file` | Absolute path to JSONL records for downstream steps. Empty file for an empty result. |
| `result-count` | Number of exported records; not the requested result count. |
| `source` | The fixed FlashData source used by this action. |

## Pricing and errors

The action is free to install. FlashData API calls use paid Credits; GitHub runner/artifact costs are separate. Extended Search has a higher rate than standard YouTube Search. [Current pricing](https://flashdata.dev/pricing).

401: check the API key. 402: add Credits. 429: rate/quota limit. After a timeout, check [FlashData Jobs](https://flashdata.dev) before rerunning: a request may already have been accepted. Paid POST requests are not retried. Requests have a 65-second timeout and a 16-MiB response limit.

## Development

`npm ci` → `npm run check` → `npm test` → `npm run build`. Tests use offline fixtures. Commit the bundle with runtime changes.
