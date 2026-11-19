# Playwright Scrape API

This is a simple web scraping service built with Express and Playwright.

## Features

- Scrapes HTML content from specified URLs.
- Blocks requests to known ad-serving domains.
- Blocks media files to reduce bandwidth usage.
- Uses random user-agent strings to avoid detection (stock engine only).
- Optional CloakBrowser stealth Chromium engine (see below).
- Strategy to ensure the page is fully rendered.

## Install
```bash
npm install
npx playwright install
```

## RUN
```bash
npm run build
npm start
```
OR
```bash
npm run dev
```

## USE

```bash
curl -X POST http://localhost:3000/scrape \
-H "Content-Type: application/json" \
-d '{
  "url": "https://example.com",
  "wait_after_load": 1000,
  "timeout": 15000,
  "headers": {
    "Custom-Header": "value"
  },
  "check_selector": "#content"
}'
```

## USING WITH FIRECRAWL

Add `PLAYWRIGHT_MICROSERVICE_URL=http://localhost:3003/scrape` to `/apps/api/.env` to configure the API to use this Playwright microservice for scraping operations.

## BROWSER ENGINE

`BROWSER_ENGINE` picks the browser:

- `playwright` (default when running the service directly): stock Playwright Chromium.
- `cloak`: [CloakBrowser](https://github.com/CloakHQ/cloakbrowser) stealth Chromium. It sets its own consistent fingerprint, so the random user agent is not used. A `User-Agent` header in the request still overrides it.

The root `docker-compose.yaml` defaults to `cloak`. Set `BROWSER_ENGINE=playwright` in the root `.env` to switch back.

`cloakbrowser` is pinned to `0.6.5`, the last version that runs the free binary without a license key. Versions 0.7.0 and later need a key on every launch. Keep `CLOAKBROWSER_AUTO_UPDATE=false` (the Docker image sets it).

To run it outside Docker, download the binary once with `pnpm exec cloakbrowser install`, then:

```bash
BROWSER_ENGINE=cloak CLOAKBROWSER_AUTO_UPDATE=false npm start
```

CloakBrowser does not rotate proxies or solve CAPTCHAs. Sites that block by IP still need `PROXY_SERVER`.

If the browser process dies, the service relaunches it on the next request.
