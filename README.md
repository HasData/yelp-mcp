# Yelp MCP Server

<!-- mcp-name: com.hasdata/yelp -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client three read-only Yelp tools. Search businesses by keyword and location, read one business in full, and page through its complete review feed, all as structured JSON, with no Yelp Fusion key and nothing to host.

It reads public Yelp pages that a signed-out visitor can see, on any of the 41 regional domains.

**1,000 free credits every month, no card required**, which is 100 Yelp calls at the 10-credit rate.

```
https://mcp.hasdata.com/api/mcp?apis=yelp
```

[![Glama score](https://glama.ai/mcp/servers/HasData/yelp-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/yelp-mcp)
[![tool contract](https://github.com/HasData/yelp-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/yelp-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://mcp.hasdata.com/api/mcp?apis=yelp)
[![Tools](https://img.shields.io/badge/tools-3-10b981?style=flat-square)](#tools)
[![npm](https://img.shields.io/npm/v/@hasdata/yelp-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/yelp-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-yelp-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-yelp-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [Tool selection](#tool-selection)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp), free to create with no card, and the free tier covers about 100 calls a month at the 10-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/yelp-mcp` on npm and `hasdata-yelp-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/api/mcp?apis=yelp` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http yelp "https://mcp.hasdata.com/api/mcp?apis=yelp" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Settings, then Connectors, then Add custom connector, then paste `https://mcp.hasdata.com/api/mcp?apis=yelp` and sign in.

For the config-file route, Claude Desktop loads only local (stdio) servers, so it reaches a remote server through a stdio launcher. The `@hasdata/yelp-mcp` package is that launcher, and it reads the key from the environment. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "yelp": {
      "command": "npx",
      "args": ["-y", "@hasdata/yelp-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

For Python instead of Node, swap the launcher for the PyPI package, which `uvx` runs without a manual install:

```json
{
  "mcpServers": {
    "yelp": {
      "command": "uvx",
      "args": ["hasdata-yelp-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

`~/.cursor/mcp.json` for every project, or `.cursor/mcp.json` for one:

```json
{
  "mcpServers": {
    "yelp": {
      "url": "https://mcp.hasdata.com/api/mcp?apis=yelp",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Windsurf</b></summary>

`~/.codeium/windsurf/mcp_config.json`. Windsurf calls the field `serverUrl`, not `url`:

```json
{
  "mcpServers": {
    "yelp": {
      "serverUrl": "https://mcp.hasdata.com/api/mcp?apis=yelp",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

`.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "yelp": {
      "type": "http",
      "url": "https://mcp.hasdata.com/api/mcp?apis=yelp",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Each of these lands on one tool, or on two in sequence when the second needs an identifier the first returns.

- Find coffee roasteries in Austin, TX and rank them by rating against review count.
- Pull the hours, phone and website for the Yelp business `desnudo-coffee-austin`.
- Read every one-star and two-star review of this business and group the complaints by theme.
- Which dishes does Yelp list as popular for this place, and what do reviewers say about them?
- Compare the rating distribution of these three competitors in the same neighborhood.
- Show me the reviews Yelp does not recommend for this business and how they differ from the recommended ones.

A prompt that names a business rather than a Yelp ID takes two calls, one search to resolve the ID and one place or reviews lookup to read it. The search tool returns both `placeId` and `placeAlias`, and either one works as the `placeId` argument to the place tool.

## Tools

Three tools, 10 credits per successful call. Every tool accepts `domain` to switch country, one of 41 values, from `www.yelp.com` through the European, Asian and Latin American sites, including the language-specific variants such as `fr.yelp.ca` and `zh.yelp.com.hk`.

### Get Yelp search results

[`hasdata_yelp_search_getSearchResults`](https://docs.hasdata.com/apis/yelp/search?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp)

A page of businesses for a keyword in a place.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `keyword` | string | yes | What to search for, such as `coffee` |
| `location` | string | yes | Where to search, such as `Austin, TX` |
| `l` | string | | Map bounding box instead of a radius, as `g:lon1,lat1,lon2,lat2` |
| `domain` | string | | Yelp site, defaults to `www.yelp.com` |
| `start` | number | | Result offset, stepping by 10 |

Returns `searchInformation` with the echoed `keyword`, `location` and `totalResults`, an `ads` array of paid placements, an `organicResults` array, and `pagination` with `currentPage`, `perPage`, `totalPages`, `nextPageUrl` and `otherPagesUrls`.

The two result arrays are not the same shape. An organic result carries `position` and `streetAddress`, while an ad carries neither and adds `phone` and a `highlights` array of the badges Yelp shows advertisers. Merging the arrays without checking which one a business came from turns paid placement into rank.

```json
{
  "position": 2,
  "placeId": "oiJ7QuhhpsEpe9zFTZF0bA",
  "placeAlias": "desnudo-coffee-austin",
  "url": "https://www.yelp.com/biz/desnudo-coffee-austin",
  "title": "Desnudo Coffee",
  "streetAddress": "2505 Webberville Rd, Austin",
  "price": "$$",
  "categories": [{ "title": "Coffee Roasteries", "url": "https://www.yelp.com/search?find_desc=Coffee+Roasteries&amp;find_loc=Austin%2C+TX" }],
  "snippet": "My favorite coffee shop to go to ever! Everyone must go. [[HIGHLIGHT]]Great coffee[[ENDHIGHLIGHT]], great vibes,great customer...",
  "rating": 4.7,
  "reviews": 347,
  "thumbnail": "https://s3-media0.fl.yelpcdn.com/bphoto/0nE_IcbRiyxrw3kk2z6owg/ls.jpg",
  "allImagesUrl": "https://www.yelp.com/biz_photos/oiJ7QuhhpsEpe9zFTZF0bA"
}
```

### Get Yelp place details

[`hasdata_yelp_place_getPlaceDetails`](https://docs.hasdata.com/apis/yelp/place?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp)

One business in full.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `placeId` | string | yes | A Yelp ID such as `oiJ7QuhhpsEpe9zFTZF0bA`, or an alias such as `desnudo-coffee-austin` |
| `domain` | string | | Yelp site, defaults to `www.yelp.com` |

Returns a `placeResult` object with `name`, `url`, `address`, `neighborhoods`, `country`, `phone`, `website`, `price`, `categories`, `rating`, `reviews`, `isClaimed` and `isClaimable`, an `operationHours` object holding a week of `hours` plus `today`, a `features` array, a `menu` object with `popularDishes`, a `faqs` array of questions Yelp readers asked and answered, a `reviewHighlights` array of the phrases Yelp pins to the top of the page, an `images` array, and `businessMap`, a static map image URL.

`features` is the amenities block, and each entry carries a `title` and an `isActive` flag, so a false entry is Yelp stating the amenity is absent rather than unknown. That difference matters when you filter, because dropping the false entries and dropping the missing ones are not the same query.

```json
{
  "name": "Desnudo Coffee",
  "url": "https://www.yelp.com/biz/desnudo-coffee-austin",
  "address": "2505 Webberville Rd Austin, TX 78702",
  "neighborhoods": "East Austin",
  "country": "US",
  "phone": "(424) 400-1857",
  "website": "http://www.desnudocoffee.com",
  "price": "$$",
  "categories": ["Coffee Roasteries"],
  "rating": 4.7,
  "reviews": 347,
  "isClaimed": true,
  "isClaimable": false,
  "operationHours": { "hours": [{ "day": "Mon", "hours": ["7:00 AM - 2:00 PM"] }] },
  "features": [
    { "title": "Offers delivery", "isActive": true },
    { "title": "ADA-compliant restroom", "isActive": false }
  ],
  "menu": {
    "section": "Popular Drinks",
    "popularDishes": [{ "name": "Brown Sugar Miso Latte", "rating": 4.7, "reviews": 113, "photos": 71 }]
  }
}
```

### Get Yelp place reviews

[`hasdata_yelp_reviews_getPlaceReviews`](https://docs.hasdata.com/apis/yelp/reviews?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp)

The review feed of one business, in full text, with sorting, filtering and pagination.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `placeId` | string | yes | The Yelp ID of the business |
| `domain` | string | | Yelp site, defaults to `www.yelp.com` |
| `sortBy` | string | | `relevanceDesc` (default), `dateDesc`, `dateAsc`, `ratingDesc`, `ratingAsc` or `elitesDesc` |
| `rating` | string | | Keep only these star ratings, such as `5` or `1,2` |
| `query` | string | | Free-text search inside the reviews |
| `languageCode` | string | | Two-letter language of the reviews, defaults to `en` |
| `notRecommended` | boolean | | Return the feed Yelp filters out instead of the recommended one |
| `start` | number | | Offset, stepping by `num` |
| `num` | number | | Page size, 49 at most, and 49 by default |
| `nextPageToken` | string | | Cursor taken verbatim from the previous response |

Returns `searchInformation` with the business name, alias, URL, `totalResults`, `rating`, a `reviewCountsByRating` array and a `reviewCountsByLanguage` breakdown, a `pagination` object, and a `reviews` array.

Each review carries `position`, `id`, `link`, a `user` object, a `comment` object holding `text` and its detected `language`, `date`, `rating`, and, when the reviewer attached them, `photos`, `videos` and `reactions`. The `user` object reports `name`, `userId`, `address`, lifetime `reviews`, `friends` and `photos` counts, and `eliteYear` for a Yelp Elite member.

A review the author later rewrote also carries `previousReviews`, holding the earlier version with its own text, rating and date. That is the field to read when the question is whether a rating moved, because the current review alone cannot answer it.

```json
{
  "position": 1,
  "id": "7zLIm3c2v2hRBVmgaKkR7w",
  "link": "https://www.yelp.com/biz/desnudo-coffee-austin?hrid=7zLIm3c2v2hRBVmgaKkR7w",
  "user": {
    "name": "Karson S.",
    "userId": "3LxSs_dQ37-LBRz07EDbyg",
    "address": "Austin, TX",
    "reviews": 374,
    "friends": 61,
    "photos": 875,
    "eliteYear": "26"
  },
  "comment": { "text": "Coming back to Desnudo to update my old review...", "language": "en" },
  "date": "2026-08-20T17:33:47-05:00",
  "rating": 5,
  "photos": [{ "link": "https://s3-media0.fl.yelpcdn.com/bphoto/YbjFKmRf7j0ejQQSaqtqVw/o.jpg", "caption": "Matcha latte", "width": 1126, "height": 2000 }],
  "reactions": [{ "type": "HELPFUL", "label": "Helpful", "count": 1 }],
  "previousReviews": [{ "id": "0WNI2IG7K1_Dg9zdXm03SA", "rating": 4, "comment": { "text": "..." } }]
}
```

## Errors and failure paths

Plan for these rather than assuming a happy path.

**A search with no matches returns a successful result with an empty `organicResults` array**, not an error. `requestMetadata.status` is still `ok`. Test the array length before iterating.

**`snippet` is marked-up text, not clean text.** Yelp wraps the matched words in `[[HIGHLIGHT]]` and `[[ENDHIGHLIGHT]]`, and those markers arrive verbatim. Strip them before you index, embed or display the snippet.

**Category URLs arrive HTML-escaped.** The `url` inside a `categories` entry contains `&amp;` rather than a bare ampersand, because that is how it sits in the page. Unescape it before following the link.

**`rating` and `query` do not combine on the reviews tool.** Yelp ignores the star filter while a free-text query is running, so a filtered search comes back with reviews of every rating. Filter the result yourself when you need both.

**`start` and `nextPageToken` are two different ways to page, and they do not mix.** Pass one or the other. The token carries both the offset and the page size, so resending it alone continues the feed, while `start` needs `num` to stay put across calls.

**The not-recommended feed is a different feed with different limits.** Setting `notRecommended` returns reviews Yelp filtered out of the main list, and those come ten at a time rather than 49, with no photos, videos or reactions attached.

**A business can be unclaimed, and an unclaimed page is thin.** `isClaimed` false usually means no website, no hours and no amenities, because nobody filled them in. Read the flag before you treat a missing field as a scraping failure.

Results that carry data also carry a `requestMetadata.id` worth quoting in support.

## Pricing, free tier and limits

Each Yelp tool costs **10 credits per successful call**. Response size does not change the price, so a 49-review page and a 5-review page cost the same, which makes the largest page the cheapest way to read a feed.

The free tier is **1,000 credits every month with no card**, which is 100 Yelp calls at the base rate. It renews with the billing cycle, so a low-volume agent runs on the free tier indefinitely.

Paid plans start at **$49 a month** for 200,000 credits, which is 20,000 calls. The unit price falls with volume, from **$2.45 per 1,000 calls** on the entry plan to **$1.00** on Business, **$0.84** on Growth and **$0.74** on the largest [high-volume plans](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp).

Your plan also sets concurrency. The free tier allows 1 request at a time, Startup 15, Business 30, Growth 50, and the high-volume plans run from 200 to 1,500. Retry on the 429 with a backoff in anything unattended, because an agent that fans out across a list of businesses will reach the ceiling before you do.

A request that comes back non-200 is not billed. A successful call that finds nothing is still a call.

## Tool selection

Start from what the prompt gives you. A keyword and a place go to the search tool, a Yelp ID or alias goes straight to the place tool, and a question about what customers said goes to the reviews tool. Spending a search call to reach an ID you already have is the most common waste.

Then pick by what the question is about. The place tool answers questions about the business itself, its hours, amenities, price tier and headline rating. The reviews tool answers questions about its customers, and it is the only one that returns review text, authors and the rating distribution. The `reviewHighlights` block on the place tool is a sample Yelp curates, not a substitute for the feed.

Read the feed with the largest page. `num` defaults to its maximum of 49 already, so leave it alone unless you are deliberately sampling.

## How it compares

Yelp's own Fusion API is the official route to this data, and it is a different instrument.

| | Yelp Fusion API | This server |
| :--- | :--- | :--- |
| Eligibility | An approved developer app | An API key |
| Review text | Up to three per business, truncated | The full feed, full text, paged |
| Review authors | Name and photo | Name, location, lifetime counts, Elite year |
| Rating distribution | Not returned | `reviewCountsByRating` on every call |
| Filtered reviews | Not returned | The not-recommended feed |
| Edit history | Not returned | `previousReviews` when a review was rewritten |
| Amenities and hours | A limited attribute set | The amenities block as the page shows it |

The row that decides it is review text. Fusion returns three excerpts per business, which answers a display question on a storefront and cannot answer an analysis question about sentiment, complaints or how a rating moved. When three excerpts and an official contract are what you need, Fusion is the better fit.

## FAQ

### Is there an official Yelp MCP server?

Yelp does not publish one. This one is maintained by HasData and reads public Yelp pages.

### What is a Yelp MCP server?

An MCP server exposes tools an AI client can call. This one turns Yelp search results, business pages and review feeds into JSON an agent can reason over, without a browser or a scraping library in your stack.

### Do I need a Yelp account or a Fusion key?

No. The only credential is your HasData key.

### Which Yelp sites are covered?

All 41 domains the API accepts, from `www.yelp.com` through the European, Asian and Latin American sites. Several countries have more than one, split by language, such as `fr.yelp.ca` next to `www.yelp.ca`. Pass `domain` to switch.

### Can I get every review of a business?

Yes, by paging. The feed returns 49 at a time, and `pagination.hasNextPage` tells you when to stop. Reading a business with 347 reviews takes eight calls.

### What is the difference between recommended and not-recommended reviews?

Yelp runs software that hides some reviews from the main feed. The default response is the recommended feed, the one a visitor sees. Setting `notRecommended` returns the hidden one instead, which is smaller, paged ten at a time, and stripped of photos and reactions.

### Why did my rating filter return every rating?

Because a `query` was set at the same time. Yelp drops the star filter when it runs a text search, so the two cannot be combined server-side.

### Can I use this together with other HasData APIs?

Yes. One key covers everything, and one endpoint serves them all through the `apis` parameter. Point a client at `?apis=yelp,google_maps` to get both tool sets in one connection, or at [`mcp.hasdata.com/api/mcp`](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp) for the full catalogue.

### Is HasData affiliated with Yelp?

No. HasData is an independent service and is not affiliated with, endorsed by, or sponsored by Yelp. Yelp is a trademark of its respective owner. The tools work with publicly available data only, and you are responsible for using the results in line with Yelp's terms and the law that applies to you.

### Compliance and personal data

The review tools return personal data. A review carries the author's display name, profile photo, stated location, user ID and a link to their profile, and reviewers are private individuals rather than businesses. That puts the response in scope of the GDPR and the CCPA in a way a business listing is not. Decide what you need before you store it, keep it no longer than the purpose requires, and check your own obligations. Aggregate analysis rarely needs the author fields at all.

## HasData links

- [Yelp Scraper API](https://hasdata.com/apis/yelp-api?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp), the REST endpoints behind these tools
- [API documentation](https://docs.hasdata.com/apis/yelp/search?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp)
- [MCP server documentation](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp)
- [Pricing](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp)
- [Dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=yelp-mcp)

Other HasData MCP servers: [Google Search](https://github.com/HasData/google-search-mcp), [Google Maps](https://github.com/HasData/google-maps-mcp), [Google Trends](https://github.com/HasData/google-trends-mcp), [Google Flights](https://github.com/HasData/google-flights-mcp), [DuckDuckGo](https://github.com/HasData/duckduckgo-mcp), [YouTube](https://github.com/HasData/youtube-mcp), [TikTok](https://github.com/HasData/tiktok-mcp), [Instagram](https://github.com/HasData/instagram-mcp), [Amazon](https://github.com/HasData/amazon-mcp), [Zillow](https://github.com/HasData/zillow-mcp), [Airbnb](https://github.com/HasData/airbnb-mcp), [Booking.com](https://github.com/HasData/booking-mcp), [Indeed](https://github.com/HasData/indeed-mcp).

## Development

The launcher is a thin stdio bridge to the remote server, so there is nothing to build.

```bash
npm install
HASDATA_API_KEY=your_key_here npm test
```

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=yelp` returns the expected tool count, that no name changed, that every tool still declares its required parameters and carries a description, and that the key in use is actually accepted. That last check calls a tool for real and costs 10 credits, which is the price of a canary that can fail for the right reason.

The contract suite also runs weekly on a schedule, because the upstream tool list can change without anyone touching this repository.

## Contributing

A tool table, a response sample or a documented behaviour that does not match reality is worth an issue. There is a template for exactly that. Pull requests are welcome for the same, and for anything in the launcher.

## License

MIT, see [LICENSE](LICENSE).
