# Google Maps scraper API: what one request returns, what 25 credits buys, and which ScraperAPI plan fits (full plan table)

Three kinds of services answer a search for a google maps scraper api: hosted endpoints that turn a Maps search into JSON (ScraperAPI's Google Maps Search endpoint, SerpApi's Maps engine), dataset-style scrapers that take a keyword or a URL and return records in bulk (Bright Data), and scripts you host and maintain yourself against Google's internal Maps endpoints.

ScraperAPI belongs to the first group, and two numbers decide whether it fits your project. Google requests cost 25 credits each, and the free trial hands you 5,000 credits over 7 days, which works out to roughly 200 Maps searches before you pay anything. The smallest paid tier carries 100,000 credits a month (about 4,000 searches at that rate), and the Scaling tier is listed at $475/month, or $427.50/month on annual billing.

What follows is the endpoint's actual parameter list, every plan on the pricing page, per-plan search counts, the response branch that breaks naive parsers, and the overage rules that decide which tier you should be on.

> **Quick verdict.** Start with the trial. 5,000 credits buys about 200 Maps requests, enough to see whether the JSON contains the fields your project needs. Then choose by volume and geography: under roughly 4,000 searches a month with US or EU exit nodes, Hobby. Scraping Maps outside the US and EU, or needing analytics history past 30 days, means Business. If you want overflow billing with a spending cap rather than a hard stop when credits run dry, Scaling and above are the only tiers that offer it.
>
> 👉 [Compare every ScraperAPI plan and its current price](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## All ScraperAPI plans, with Maps search estimates

The fourth column converts monthly credits into Maps searches at the 25-credit Google rate, so you can compare tiers against your own volume instead of squinting at credit totals.

| Plan | API credits per month | Maps searches (est.) | Price and billing | Start |
| --- | --- | --- | --- | --- |
| Free trial | 5,000 total, 7 days | ~200 | $0, no credit card required | [Start the free trial](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | 100,000 | ~4,000 | Monthly and annual price on the pricing page | [Take the Hobby plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | 1,000,000 | ~40,000 | Monthly and annual price on the pricing page | [Take the Startup plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | 3,000,000 | ~120,000 | Monthly and annual price on the pricing page | [Take the Business plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling (most popular) | 5,000,000 | ~200,000 | $475/mo, or $427.50/mo billed annually ($5,130/yr, $570 saved) | [Take the Scaling plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | 10,500,000 | ~420,000 | Monthly and annual price on the pricing page | [Take the Professional plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | 21,500,000 | ~860,000 | Monthly and annual price on the pricing page | [Take the Advanced plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | 22,000,000+ | ~880,000+ | Custom quote through sales | [Request Enterprise pricing](https://www.scraperapi.com/?fp_ref=coupons) |

Concurrency, geotargeting and analytics history are what actually separate the tiers once you ignore the credit number.

| Plan | Concurrent threads | Geotargeting | Analytics history |
| --- | --- | --- | --- |
| Free trial | — | — | — |
| Hobby | 20 | US & EU | Last 30 days |
| Startup | 50 | US & EU | Last 30 days |
| Business | 100 | Country-level | Unlimited |
| Scaling | 200 | Country-level | Unlimited |
| Professional | 300 | Country-level | Unlimited |
| Advanced | 500 | Country-level | Unlimited |
| Enterprise | 500+ | Country-level | Unlimited |

A dash means the pricing page does not publish that figure for the tier.

## What one Google Maps scraper API request actually returns

ScraperAPI's endpoint is a GET to `/structured/google/mapssearch` with four parameters: `api_key`, `query`, `latitude` and `longitude`. The last three are required, and the query is a plain search string such as "vegan restaurant". The documentation returns the Maps search page transformed into JSON and includes a sample response you should read before writing a schema.

That parameter list is short, and the omissions shape how you build. ScraperApi exposes no zoom level and no result offset for this endpoint, so you cannot ask for "the same coffee search zoomed out to city level" or "page two" in the request itself. SerpApi's Maps engine handles the same problem differently, and its parameter set is worth knowing as a reference point: `q` for the query and `ll` for the origin, where `ll` follows `@latitude,longitude,zoom` with zoom running from 3z (fully zoomed out) to 21z (fully zoomed in), or a metre value such as `10410m` instead of the zoom. It also takes `start`, where 0 is the first page, 20 the second and 40 the third, plus `hl`, `gl` and `google_domain` for language, country and Google domain.

With coordinate-only requests, coverage comes from sending more requests, not from paging. A lead-gen grid over a metro area means one request per lat/lng cell, and your credit budget should be built from that cell count rather than from a page count.

## The field list, and two traps inside it

A Maps place record, as documented in SerpApi's README, can carry position, title, `place_id`, `data_id`, `data_cid` (the same identifier Google's Local API calls `ludocid`), reviews and photos links, GPS coordinates, rating, review count, a price tier such as `$` or `$$`, type and types, address, open state, hours with a weekday-keyed `operating_hours` object, phone, website, amenities, description, service options, thumbnail, and extension arrays for highlights.

Two things about that list cause real bugs.

**The first is the `place_results` branch.** SerpApi's own documentation warns that a request sometimes returns `place_results` instead of the `local_results` array, because Google decided the query matched one specific place. Code that reads `json["local_results"]` and iterates will throw or silently return nothing on those responses. Check both keys, and treat a single-place response as a valid result rather than an error.

**The second is that fields are not guaranteed.** Google returns different information depending on the place, so `price`, `phone`, `hours` and `website` will be missing for some businesses and present for others. A pipeline that assumes every record has a phone number will drop rows or crash on the first restaurant without one.

There is a version dimension too. A 2023 SerpApi guide lists extras that the current README's field table doesn't include, such as order-online URLs, menu URLs, featured reviews and a "people also search for" block, and those usually come from separate photos, reviews or place endpoints rather than the main Maps response. Treat any field list, including the one above and whatever sample response ScraperAPI's docs show, as a snapshot. Validate against your own first hundred records before you commit to a schema.

## Credit math: the number that decides your tier

ScraperAPI prices by request type. A standard page costs 1 credit, Amazon, Walmart and eBay cost 5, LinkedIn costs 30, and Google or Bing requests cost 25 credits each, subdomains included. Add-ons stack on top: JavaScript rendering (`render=true`), premium proxies (`premium=true`), bot-protection bypass and screenshots each add 10 credits, premium proxies plus rendering costs 25, and Ultra Premium runs 30 credits (75 with rendering) on paid plans. Geotargeting, sessions, device type, output format and auto parsing add nothing.

At 25 credits per Google request, the trial's 5,000 credits cover about 200 Maps searches, Hobby's 100,000 cover 4,000, Startup 40,000, Business 120,000, Scaling 200,000, Professional 420,000, and Advanced 860,000. Scaling's published $475/month works out to $2.38 per 1,000 Maps searches, dropping to $2.14 on annual billing. Higher tiers advertise a better price per credit, so the per-search figure falls as you move up.

Two things protect you from guessing wrong. Every response reports its own charge in the `sa-credit-cost` header, and the dashboard's Domain Cost Estimator prices a URL before you scrape it. You can also set `max_cost` per scrape so a single request can never blow past a ceiling. Run one Maps request on day one of the trial and read that header. If the endpoint bills at the standard Google rate, the search estimates above hold; if it bills differently, rebuild your volume math from the real number.

Billing models differ enough between providers that unit prices aren't comparable at face value. Bright Data's Google Scraper API charges per successful record, so a Maps place record is one billable unit and a search returning twenty places is twenty units. ScraperAPI charges per request. If you need twenty places per query, per-request pricing is straightforward. If you need full detail on a list of specific places, per-record billing maps more cleanly onto what you actually consume, provided you know the per-record rate. Bright Data also gives new accounts 5,000 free credits a month, no card required, which it describes as up to 5,000 Google records, so small-scale validation is free on both platforms.

## What happens when you hit the ceiling

ScraperAPI handles the two limits differently, and the difference decides whether a cheap tier is viable for production.

Exceeding your concurrency limit returns HTTP 429 and the request is turned away rather than queued. There is no extra charge for that. Running out of credits is a separate event: scraping pauses until your credits renew, unless you have turned on one of the overflow mechanisms.

Auto-upgrade moves you to the next plan when credits run out, and it's available on Hobby, Startup, Business, Scaling and Professional, opt-in. Pay-as-you-go keeps you on the same plan at a fixed per-credit rate up to a monthly cap you set, and it's available on Scaling, Professional, Advanced and Enterprise only. Annual plans can also renew early for a fresh batch of credits.

Read that distribution carefully. Hobby and Startup can climb the ladder automatically but cannot burst within a month under a cap. If your workload spikes unpredictably and you need a hard spending ceiling rather than a bigger plan, your cheapest qualifying tier is Scaling.

👉 [See which ScraperAPI tiers include pay-as-you-go billing](https://www.scraperapi.com/?fp_ref=coupons)

## Geotargeting: where Hobby and Startup stop

Hobby and Startup route through US and EU geotargeting only. Country-level targeting starts at Business and continues through Advanced, with Enterprise negotiable.

For Maps work that matters more than it looks. The endpoint takes latitude and longitude, so the coordinates control where the search appears to originate, but the exit country still comes from your plan. Scraping US restaurants on Hobby is fine. Scraping a chain in Brazil, Japan or the UAE on Hobby is not, because you cannot put the request through a local exit node.

SerpApi handles the same requirement with request parameters instead of plan tiers: `gl` takes a two-letter country code, `hl` a two-letter language code, and `google_domain` swaps the Google domain, all optional and all available on any request. Bright Data's Google endpoints mostly accept a `country` field using ISO 3166-1 alpha-2 codes. If your project is geographically scattered and price-sensitive, that difference in packaging is worth weighing before you commit to a tier.

Analytics history follows the same split: 30 days on Hobby and Startup, unlimited from Business up. If you need to look back at last quarter's success rates, that alone pushes you to Business.

## When a scraper API is the wrong purchase

If you need a few dozen records once, none of this is worth the setup. A manual export or an afternoon with a script covers it.

Self-hosted scrapers are also a legitimate option for steady, low-volume work. The open-source projects floating around this keyword hit `google.com/search?tbm=map` and the Maps preview endpoint with a long packed `pb` parameter and a browser-like TLS fingerprint. They work until Google changes the payload format or tightens fingerprint checks, and then you debug parameter packing instead of shipping your feature. That maintenance burden is the thing a hosted API removes. Decide honestly whether you'd rather pay per request or own that debugging.

Where hosted APIs clearly win is breadth: hundreds of locations, scheduled refreshes, and a schema you don't want to revisit every quarter. A single place record with a name, address, category, rating and review count is enough for a store locator or a lead list, and that's exactly what a structured endpoint returns without a parser.

## Before you pay

Match the plan to your monthly Maps search count, not to the credit figure. Under 4,000 searches with US or EU exits, Hobby covers it. Between 4,000 and 120,000 searches, work up through Startup and Business, and take Business the moment you need country-level targeting or unlimited analytics. Past that, Scaling is the popular tier for a reason: 200,000 searches at $2.38 per thousand on monthly billing, plus pay-as-you-go overflow with a cap. If you need more than 860,000 searches a month, talk to sales rather than stacking plans.

Two things to confirm before you enter card details. First, the real credit cost of the Maps endpoint on your account: send one request and read the `sa-credit-cost` header, because all the search estimates in this article assume the 25-credit Google rate. Second, the failure mode you want when credits run out: a pause, an automatic upgrade, or metered overflow. Only the last one keeps a production pipeline running through a traffic spike, and it isn't available on the two cheapest tiers.

👉 [Start the 7-day ScraperAPI trial with 5,000 credits](https://www.scraperapi.com/?fp_ref=coupons)
