# 🔍 TikTok Ad Library Scraper  Export unlimited TikTok ads by keyword, advertiser, or URL

> **Formula:** `Scrapes TikTok ads` + `from the TikTok Ad Library (library.tiktok.com)` + `as clean, structured JSON`

**[TikTok Ad Library Scraper](https://apify.com/dz_omar/tiktok-ad-library-scraper?fpr=smcx63)** extracts ads from the official TikTok Ad Library  search by keyword, by advertiser name, or paste a library URL directly, and get back advertiser, creative type, shown dates, estimated audience, video and image creatives, and (optionally) full targeting data. No login, no cookies, no browser automation. **Collect unlimited results** — set Maximum Results to `0` and pull every ad a search returns, with no 1,000-ad cap.

Perfect for **competitive ad researchers**, **performance marketers**, and **agencies** who need TikTok ad creative and targeting intelligence without manually clicking through the Ad Library.

<!-- ✅ EMBED VIDEO: replace this dummy URL with your real demo video once recorded. Paste the raw YouTube URL on its own line  Apify auto-renders it as a player. -->
https://www.youtube.com/watch?v=DUMMY_VIDEO_ID

---

## Why scrape the TikTok Ad Library?

The TikTok Ad Library is TikTok's public DSA transparency portal, holding roughly one year of ads served across the EU/EEA. It exposes the actual creatives, the advertiser behind them, how long each ad ran, and  per ad  the age, gender, and regional impression breakdown. That is competitor intelligence you would otherwise pay a SaaS subscription for.

Here are some of the most common use cases:

- **Competitor ad monitoring**: Track every ad a rival advertiser is running, with first/last shown dates.
- **Creative research**: Pull the actual video and image creatives competitors use to model your own.
- **Targeting intelligence**: See which countries, age bands, and genders an advertiser targets (with Fetch Ad Detail).
- **Market & trend analysis**: Search a keyword across all regions to map who is advertising in a category.
- **Ad compliance & due diligence**: Capture ad copy, audit status, and rejection reasons for a brand.

---

## Can I scrape more than 1,000 TikTok ads? (Unlimited results)

**Yes — there is no 1,000-ad cap.** Set **Maximum Results** to `0` and the scraper paginates until the TikTok Ad Library has no more ads to return, collecting **every ad in your search**. Need tens of thousands? Each URL and each keyword runs as its own independent search, so you scale results by adding more keywords, more regions, or splitting a broad query into narrower date ranges.

- ♾️ **`maxResults: 0` → unlimited** ads per search — no artificial limit.
- 🌍 **Stack searches** — combine regions, advertisers, and date windows in a single run to pull far beyond 1,000 ads.
- ⚡ **Real-time output** — each ad is written to your dataset the moment it is scraped, so even a long unlimited run is never lost if it is aborted.

---

## What data can the TikTok Ad Library Scraper extract?

### 🏷️ Ad Identity
- `id`  Unique TikTok ad identifier
- `name`  Advertiser display name
- `type`  Creative format (`video` or `image`)
- `audit_status`  TikTok moderation status

### 📅 Timeline
- `first_shown_date`  First date the ad ran (Unix ms, UTC)
- `last_shown_date`  Most recent date the ad ran (Unix ms, UTC)

### 📊 Reach
- `estimated_audience`  Bucketed audience size (e.g. `0-1K`, `1K-10K`)

### 🎬 Creatives
- `videos`  Playable signed video URL + cover image
- `image_urls`  Image creative URLs
- `video_download_url` / `cover_download_url` / `image_download_urls`  permanent re-hosted links, present only when **Download Media** is enabled (originals are always kept alongside)

### 🎯 Full Targeting *(only when Fetch Ad Detail is enabled)*
- Creative title & advertising objective
- Advertiser registry location & business ID
- Target audience size and targeted countries
- Per-region impressions with age (13-17 … 55+) and gender breakdowns

### 🔖 Provenance
- `region`, `sort`, `source_url`, `_source`  which search produced the record

---

## ⚙️ How to use the TikTok Ad Library Scraper

You provide **URLs**, **keyword filters**, or both — **at least one is required**. Each URL and each keyword counts as one input item. (If you run with nothing filled in, the Actor finishes successfully and writes a single dataset item explaining what to add — it does not error out.)

> **Total results = (number of URLs + number of keywords) × Maximum Results.**
> Example: 5 URLs + 6 keywords = 11 inputs. At 16 Maximum Results → 11 × 16 = **176 ads** maximum.

### Input Options

#### Start URLs (Array)

Paste full `library.tiktok.com/ads` URLs copied from your browser. Region, dates, sort order, and advertiser filters are read directly from the URL  no extra config.

| Input value | What it extracts |
|---|---|
| `https://library.tiktok.com/ads?region=DE&adv_name=nike` | Ads matching "nike" in Germany |
| `https://library.tiktok.com/ads?region=all&adv_name=...&query_type=2&adv_biz_ids=...` | A specific advertiser's ads, all regions |

```json
{
    "startUrls": [
        { "url": "https://library.tiktok.com/ads?region=DE&adv_name=nike&sort_type=last_shown_date,desc" }
    ]
}
```

#### Keyword Filters (Array of objects)

Build searches manually. Each entry runs independently with its own region, date range, and sort order.

- **`query`**  The keyword or advertiser name.
- **`searchAsAdvertiser`** *(boolean, default `false`)*  Off: match the text against ad content (TikTok's "Search this exact phrase"). On: treat the text as an advertiser name and return that advertiser's ads (TikTok's "Advertiser name").
- **`region`**  One target country (or `all` for EU/EEA). The API processes one region per search.
- **`dateFrom` / `dateTo`**  Date range. See the date window note below.
- **`sortOrder`**  `newest`, `oldest`, `created`, `created-asc`, `popular`, `unpopular`.

```json
{
    "keywords": [
        { "query": "summer sale", "region": "DE", "sortOrder": "newest" },
        { "query": "Apify Technologies s.r.o.", "searchAsAdvertiser": true, "region": "AT" }
    ],
    "maxResults": 10
}
```

#### `maxResults` (Integer)
- **Default**: `10`
- Maximum ads collected **per URL and per keyword**. Set to `0` for **unlimited** — collect every ad a search returns, with **no 1,000-result cap**. Start with `10` to test, then switch to `0` for a full pull.

#### `fetchAdDetail` (Boolean)
- **Default**: `false`
- When on, the scraper makes one extra request per ad to collect full advertiser, creative, and targeting detail. **This is slower**  one additional request per ad.

#### `downloadMedia` (Boolean)
- **Default**: `false`
- When on, each ad's video and images are downloaded to permanent storage and the links are added to the output (`video_download_url`, `cover_download_url`, `image_download_urls`). TikTok's own media links are signed and expire after a few hours; this re-hosts them so they stay accessible.
- Media is downloaded by the **[Universal Downloader](https://apify.com/dz_omar/universal-downloader?fpr=smcx63)** actor and saved to **your** account's storage (free accounts: files expire in ~7 days; paid accounts: retained).

> ⏱️ **Performance note  both options add time.**
> `fetchAdDetail` adds one request per ad. `downloadMedia` adds a separate download per ad (a few seconds each). Each one alone makes a run noticeably slower; **enabling both together compounds the cost**  for large result counts (e.g. 1,000 ads) expect a substantially longer run. If you only need metadata, leave both off. Turn them on when you specifically need the full detail and/or permanent media files.

### 📆 The date window (important)

The TikTok Ad Library only retains about **one year** of "last shown date" history, and the earliest selectable day slides forward daily. This scraper handles that automatically:

- **No dates given** → defaults to roughly the last year (today back to ~364 days).
- **Start date too old** → clamped up to the earliest available day, with a warning. The run continues.
- **`dateFrom` after `dateTo`** → the entry is rejected with a clear message (dates are reversed).
- **Entire range older than ~1 year** → the entry is rejected (nothing to scrape there).

---

## 💰 Pricing

> 🎉 **This Actor is currently FREE to use.** You only pay Apify's standard platform usage for the run. There are no per-result charges at this time.

Pricing may change in the future. Any change will be shown transparently here and in the Apify Store before you run.

<!--
═══════════════════════════════════════════════════════════════════
  💵 MONETIZATION SNIPPET  when you switch to Pay Per Event:
  1. In Apify Console, configure the two events with your tier prices
     (these must match .actor/pay_per_event.json exactly).
  2. Delete the "currently FREE" block above.
  3. Paste the block below in its place.
═══════════════════════════════════════════════════════════════════

How much does it cost to scrape the TikTok Ad Library?

This Actor uses **Pay Per Event** pricing  you are charged only for results actually delivered, nothing else. Compute, proxy, and runtime are absorbed by the Actor; if a run produces one result, you pay for one result.

| Event | FREE | BRONZE | SILVER | GOLD |
|---|---|---|---|---|
| Ad result (no detail) | $0.0009 | $0.00055 | $0.0005 | $0.0005 |
| Ad result (with detail) | $0.0018 | $0.0011 | $0.0010 | $0.0010 |

Because pricing is per result, cost is simply **results × per-result price**. Collecting 1,000 ads without detail on the FREE tier ≈ **$0.90**; with full detail ≈ **$1.80**. `fetchAdDetail: true` bills at the "with detail" rate.

> 💡 Tip: Set `maxResults: 10` first to test your setup before a full run.
-->

---

## 📊 Sample Output

**Without `fetchAdDetail` (`details` is null):**

```json
{
    "id": "1866984139925698",
    "name": "nexxtstyyle",
    "type": "video",
    "audit_status": "1",
    "first_shown_date": 1780495093000,
    "last_shown_date": 1780495093000,
    "estimated_audience": "0-1K",
    "videos": [
        { "video_url": "https://library.tiktok.com/api/v1/cdn/.../video.mp4", "cover_img": "https://p19-common-sign.tiktokcdn.com/.../cover.jpeg" }
    ],
    "image_urls": [],
    "details": null,
    "region": "DE",
    "sort": "last_shown_date,desc",
    "source_url": "URL: https://library.tiktok.com/ads?region=DE&adv_name=nike&sort_type=last_shown_date,desc",
    "_source": "tiktok_ad_library_scraper"
}
```

**With `fetchAdDetail` (`details` populated  truncated):**

```json
{
    "id": "1866984139925698",
    "name": "nexxtstyyle",
    "type": "video",
    "estimated_audience": "0-1K",
    "videos": [ { "video_url": "...", "cover_img": "..." } ],
    "details": {
        "ad": { "title": "Nike☀️🔝 #nike #summer", "advertising_objective": "Reach", "estimated_audience": "1K-10K" },
        "advertiser": { "name": "Nextstyleee", "adv_biz_ids": "7555084503564304385", "registry_location": "Emirates" },
        "targeting": {
            "target_audience_size": "23.6M-28.8M",
            "countries": ["ES", "FR", "IT", "DE", "BE"],
            "location": {
                "total_region": 5,
                "data": [
                    { "region": "ES", "impressions": "2K", "breakdowns": [ { "age": "18-24", "gender": "FEMALE", "impressions": "0-1K" } ] }
                ]
            }
        }
    },
    "region": "DE",
    "source_url": "URL: https://library.tiktok.com/ads?region=DE&adv_name=nike",
    "_source": "tiktok_ad_library_scraper"
}
```

**With `downloadMedia` (permanent links added alongside originals):**

```json
{
    "id": "1866996966152193",
    "name": "nikesk87",
    "type": "video",
    "videos": [
        {
            "video_url": "https://library.tiktok.com/api/v1/cdn/.../video.mp4?...",
            "cover_img": "https://p16-common-sign.tiktokcdn.com/.../cover.image?...",
            "video_download_url": "https://api.apify.com/v2/key-value-stores/<store>/records/video-...mp4?signature=...",
            "cover_download_url": "https://api.apify.com/v2/key-value-stores/<store>/records/image-...image?signature=..."
        }
    ],
    "image_urls": [],
    "image_download_urls": [],
    "region": "DE",
    "_source": "tiktok_ad_library_scraper"
}
```

The originals (`video_url`, `cover_img`, `image_urls`) are always kept; the `*_download_url` fields are added only when Download Media is on.

---

## ❓ Frequently Asked Questions

**Do I need a TikTok account to use this Actor?**
No. The TikTok Ad Library is a public transparency portal. This scraper requires no login, cookies, or credentials.

**Is there a limit on how many ads I can scrape? Can I get more than 1,000 results?**
No fixed limit. Set **Maximum Results** to `0` for unlimited — the scraper keeps paginating until the TikTok Ad Library returns no more ads for your search, with no 1,000-result cap. For very large pulls, split a broad search into multiple keywords, regions, or date ranges; each runs independently and streams to your dataset in real time.

**How far back can I search?**
About one year. The TikTok Ad Library retains roughly 365 days of "last shown date" history, and the window slides forward daily. Dates older than that are automatically clamped or skipped with a clear message.

**Can I scrape multiple keywords and URLs in one run?**
Yes. Provide any mix of URLs and keyword filters  they are processed in the same run (URLs first, then keywords), each capped at `maxResults`.

**What's the difference between keyword search and advertiser search?**
A keyword search matches your text against ad content. Turning on `searchAsAdvertiser` treats your text as an advertiser name and returns that advertiser's ads  the same split TikTok shows as "Search this exact phrase" vs "Advertiser name."

**Why is the run slow when I enable Download Media or Fetch Ad Detail?**
Both add per-ad work. `fetchAdDetail` makes one extra request per ad; `downloadMedia` downloads each ad's video and images (a few seconds per ad) via the [Universal Downloader](https://apify.com/dz_omar/universal-downloader?fpr=smcx63). Enabling both together compounds the time, so a large run (e.g. 1,000 ads) can take substantially longer. Leave them off if you only need metadata and the original (short-lived) media URLs.

**Where are downloaded media files stored, and do they expire?**
They're saved to your own Apify account's key-value store by the Universal Downloader. On free accounts, unnamed storage expires in about 7 days; on paid accounts it's retained. The original TikTok URLs expire after a few hours regardless  that's why Download Media exists.

**What happens if the run is aborted or hits a block mid-way?**
Results are written in real time, one ad at a time, as they are scraped. Anything already collected is saved. A failing source is skipped and the next one continues  one bad input never loses the whole run.

**Why did one of my keyword/URL entries get skipped?**
Most likely its date range was reversed (`dateFrom` after `dateTo`) or fell entirely outside TikTok's ~1-year retention. The run log states exactly which entry and why.

---

## ⚖️ Legal & Ethical Use

This Actor extracts **publicly visible data** from the TikTok Ad Library  the same information anyone can see in their browser on library.tiktok.com.

**Please use this tool responsibly:**
- Only use the data for lawful purposes such as research, competitive analysis, and compliance.
- Comply with [TikTok's Terms of Service](https://www.tiktok.com/legal/terms-of-service) and applicable data protection regulations (GDPR, CCPA, etc.).
- Do not use extracted data for spam, harassment, or unsolicited outreach.

For questions about a specific use case, contact us at [flowextractapi@outlook.com](mailto:flowextractapi@outlook.com).

---

## 🔄 How results are saved

Results stream into your dataset **in real time**  each ad is pushed the moment it is scraped, in order (URLs first, then keywords). There is no end-of-run batch dump:

| Trigger | Behavior |
|---|---|
| Each ad scraped | Pushed to the dataset immediately |
| Source fails or is blocked | That source is skipped; already-pushed ads are retained |
| Budget/charge limit reached | Run stops cleanly; everything collected so far is kept |
| Run aborted or migrated | Already-pushed ads remain; the run resumes from its checkpoint when restarted (see below) |

---

## 🔄 Does the scraper resume if it crashes? (Resume & crash recovery)

**Yes — it picks up where it left off.** The Actor checkpoints its progress to the run's storage, so if it is **migrated** to another server, **aborted**, hits its **timeout**, or runs **out of memory**, the next start continues instead of re-scraping from zero. Finished sources are skipped and ads already saved are de-duplicated, so you never get duplicate rows or pay twice for the same ad.

| Trigger | What happens |
|---|---|
| Server migration | Progress is saved instantly and the Actor reboots onto the new server, then resumes |
| Periodic save | A checkpoint is written about every 60 seconds and every few ads |
| Abort / timeout | The checkpoint is kept; re-running the **same input** resumes from there |
| Clean completion | The checkpoint is cleared, so a fresh run of the same input starts from scratch |

The checkpoint is **encrypted at rest** (AES-256-GCM) on production runs, so nothing operational is exposed in the Storage tab. Resume is automatic — there is nothing to enable.

---

## 🌐 Proxy Support

| User tier | Proxy used |
|---|---|
| 💎 Paying (dedicated proxy configured) | Your dedicated proxy — with automatic **one-way fallback to Apify Proxy** if it fails, so a proxy outage never sinks the run |
| 🆓 Free | Apify Proxy (RESIDENTIAL) — built in, automatic |
| ⚠️ None available | Direct connection |

Proxy selection is automatic and tier-aware. A blocked IP is rotated to a fresh one mid-run, and the proxy is never logged or written to the dataset.

---

## 🚫 Error Handling

| Situation | What you see | What to do |
|---|---|---|
| Reversed dates | `Date From … is after Date To …` | Swap the dates so the start comes first |
| Range too old | `range … is entirely older than TikTok's ~1-year retention` | Choose dates within the last year |
| Date out of range at runtime | `TikTok rejected the date range as out of its ~1-year retention window` | Narrow the range to the last year |
| Initial block | `TikTok blocked the initial request` | Try again later, or enable a proxy |
| No input at all | Run **succeeds** with a `no_input` guidance item in the dataset | Add at least one Start URL or Keyword Filter |
| No valid input | Run **succeeds** with a `no_valid_sources` guidance item explaining why | Fix reversed/too-old dates or invalid URLs/keywords |

---

## 💬 Support & Contact

### Get Help

- 🌐 **Website**: [flowextractapi.com](https://flowextractapi.com)
- 📧 **Email**: [flowextractapi@outlook.com](mailto:flowextractapi@outlook.com)
- 🙋 **Apify Profile**: [FlowExtract API](https://apify.com/dz_omar?fpr=smcx63)
- 💬 **GitHub Issues**: [FlowExtractAPI](https://github.com/FlowExtractAPI)

### Social Media

- 💼 **LinkedIn**: [flowextract-api](https://www.linkedin.com/in/flowextract-api/)
- 🐦 **Twitter**: [@FlowExtractAPI](https://x.com/@FlowExtractAPI)
- 📱 **Facebook**: [flowextractapi](https://www.facebook.com/flowextractapi)

---

## 🌟 Related Actors by FlowExtract API

### 🏠 Real Estate Data

**[domain.com.au Scraper](https://apify.com/dz_omar/domain-scraper?fpr=smcx63)**
Extract Australian property listings from domain.com.au with full enrichment, agent contacts, schools, and market insights.

**[Realtor.com Scraper](https://apify.com/dz_omar/realtor-scraper?fpr=smcx63)**
Extract US property listings from Realtor.com. Pricing, agent contacts, tax history, AVM estimates, nearby schools, flood and noise data, and full listing history.

**[PropertyFinder Scraper](https://apify.com/dz_omar/propertyfinder-scraper?fpr=smcx63)**
Extract property listings from PropertyFinder across UAE, Saudi Arabia, Bahrain, Egypt, and Qatar.

**[Idealista Scraper API](https://apify.com/dz_omar/idealista-scraper-api?fpr=smcx63)**
Advanced Idealista property data extraction with API access.

**[Idealista Scraper](https://apify.com/dz_omar/idealista-scraper?fpr=smcx63)**
Extract Spanish real estate listings from Idealista.

**[Leboncoin.fr Scraper](https://apify.com/dz_omar/leboncoin-scraper?fpr=smcx63)**
Extract French classified listings from Leboncoin  real estate, vehicles, jobs, and more.

**[AI Contact Intelligence Extractor](https://apify.com/dz_omar/ai-contact-intelligence?fpr=smcx63)**
Extract emails, phones, contacts & custom data using AI.

### 🍔 Food & Delivery

**[DoorDash Scraper](https://apify.com/dz_omar/doordash-scraper?fpr=smcx63)**
Scrape store listings, full menus, featured items, and customer reviews from DoorDash.

### 🎬 Video & Media Tools

**[YouTube Transcript & Metadata Extractor](https://apify.com/dz_omar/youtube-transcript-metadata?fpr=smcx63)**
Extract complete video transcripts with timestamps and comprehensive metadata.

**[YouTube Full Channel, Playlists, Shorts, Live](https://apify.com/dz_omar/Youtube-Scraper-Pro?fpr=smcx63)**
Extract complete playlist information with all video details from any YouTube playlist.

**[Zoom Scraper | Downloader & Transcript](https://apify.com/dz_omar/zoom-scraper?fpr=smcx63)**
Extract Zoom meeting recordings, transcripts, and metadata.

**[Loom Scraper | Downloader & Transcript](https://apify.com/dz_omar/loom-video-scraper?fpr=smcx63)**
Download Loom videos and extract transcripts.

### 🛠️ Developer & Security Tools

**[Screenshot](https://apify.com/dz_omar/screenshot?fpr=smcx63)**
Fast, reliable webpage screenshots with customizable options.

**[Ultimate Screenshot](https://apify.com/dz_omar/ultimate-screenshot?fpr=smcx63)**
Advanced screenshot tool with full-page capture, custom viewports, and quality controls.

### 📱 Social Media Tools

**[Facebook Ads Scraper Pro](https://apify.com/dz_omar/facebook-ads-scraper-pro?fpr=smcx63)**
Extract Facebook ads data for competitor analysis and market research.

**[LinkedIn Ad Library Scraper](https://apify.com/dz_omar/linkedin-ads-scraper?fpr=smcx63)**
Extract comprehensive advertising data from LinkedIn's Ad Library.


---

**Ready to extract TikTok ad data?** [Start using TikTok Ad Library Scraper now!](https://apify.com/dz_omar/tiktok-ad-library-scraper?fpr=smcx63)