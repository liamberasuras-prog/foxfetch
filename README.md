# FoxFetch Seekr

A privacy-conscious search app starter built with Next.js, TypeScript, PostgreSQL, and Prisma.

## What’s here

```text
app/
  api/
    search/route.ts    Bang handling, Mwmbl search, and community-index blending
    sites/route.ts     Public-site validation, metadata extraction, and indexing
    suggest/route.ts   Search suggestions and community-site suggestions
  globals.css          Responsive themes and app styles
  layout.tsx           App metadata and root layout
  page.tsx              Search home, results, instant answers, and site modal
lib/
  prisma.ts             Reusable Prisma client
generated/prisma/       Prisma Client output (generated locally)
prisma/
  schema.prisma         PostgreSQL community-site schema
prisma.config.ts        Prisma 7 database configuration
.env.example            Search-provider and database configuration
```

## Run locally

1. Install Node.js 20.19 or newer and PostgreSQL.
2. Copy `.env.example` to `.env.local` and set `DATABASE_URL`.
3. Live web search and suggestions use the public Mwmbl API; image results use Wikimedia Commons; video results use Internet Archive; news uses Google News RSS. None require an API key or credit card.
4. Install and initialize:

   ```sh
   npm install
   npm run db:generate
   npm run db:push
   npm run dev
   ```

5. Open [http://localhost:3000](http://localhost:3000).

The app does not save search queries or IP addresses, set tracking cookies, or keep a search history. The selected appearance is stored in browser local storage. Search queries and autocomplete are sent to Mwmbl’s public API; Mwmbl is a nonprofit search engine, and its service receives the requests. Image searches go to Wikimedia Commons, with creator/license metadata and links to file pages. Video searches go to Internet Archive and only return MP4 files under 100 MB; results link to the original archive record. Check individual rights before reusing any video. News results are retrieved from Google News RSS. Its feed is provided for personal, non-commercial feed-reader use; do not use that news integration for a public or commercial deployment without checking the provider’s terms. Instant-answer lookups use Wikipedia, Open-Meteo, Frankfurter, or Dictionary API as needed.

## Search behavior

- `GET /api/search?q=...&type=all|images|videos|news&safe=moderate&country=US` uses Mwmbl for web results, Wikimedia Commons for images, Internet Archive for videos, and Google News RSS for articles. PostgreSQL community results are blended into web searches.
- Search results have shareable URLs at `/search?q=...`; the selected vertical is preserved with `&type=images|videos|news`. Opening or refreshing a search URL runs that search again.
- `!w`, `!wiki`, `!yt`, `!youtube`, `!gh`, and `!r` redirect to their respective search pages.
- `POST /api/sites` checks a public HTTP(S) page, extracts its title/description/Open Graph image/favicon, and upserts it into PostgreSQL. The site title and description provided in the form take precedence over scraped metadata.
- `GET /api/suggest?q=...` returns Mwmbl autocomplete suggestions and matching community-site titles.

The local index stores metadata and links, not copies of submitted websites. The site submit endpoint declines redirects and non-HTML pages and applies response time/size limits. For a public production deployment, add rate limiting, a maintained public-IP/DNS validation layer for crawler requests, database backups, and provider-specific privacy disclosures before accepting submissions at scale.
