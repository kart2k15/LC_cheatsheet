# Ad-Tech Terminology
*Built: April 1, 2026 | Interview Prep — Senior DS, Ads roles (Reddit, Meta, Google, LinkedIn, TikTok etc.)*

---

## Platform Types

### First-Party Platform
A platform that owns the user relationship directly. Users are logged in, the platform collects behavioral data (clicks, upvotes, subreddits visited, comments) under its own terms of service.
- **Examples:** Reddit, Meta, TikTok, LinkedIn, Twitter/X
- **Implication for ads:** User features (demographics, interests, behavior) are stored server-side. Ranking and inference typically happen server-side since the platform owns the data and the user relationship. That said, some first-party platforms may cache certain lightweight signals (e.g., demographics, session context) on-device for latency or regional privacy reasons — this is an open architectural question that depends on the platform's specific design choices.

### Third-Party Ad Network
A company that shows ads on *someone else's* website. They don't own the user relationship — the publisher (e.g., a cooking blog) embeds their code.
- **Examples:** Google Display Network, Playwire, AppNexus, The Trade Desk
- **Implication for ads:** They can't easily send user data server-side due to privacy restrictions and lack of direct user consent. This is why on-device/browser inference (e.g., Google's Privacy Sandbox, Topics API, Protected Audience API) exists — to run targeting locally in the browser without exposing cross-site user data to servers.

### Publisher
A website or app that sells ad space on their platform. The cooking blog, a news site, a game — they're publishers. They work with third-party ad networks to monetize their inventory.

---

## Inventory & Impressions

### Ad Inventory
The total available ad slots a platform has to sell. On Reddit, this is every feed placement, sidebar slot, and sponsored post slot across all subreddits and sessions.

### Impression
The ad was **shown** to the user. It rendered on screen. No interaction required.
- Distinct from a click — showing ≠ engaging.

### Click
The user **actively clicked** on the ad. Positive engagement signal.

### View-Through
User saw the ad (impression) but didn't click — yet later converted (e.g., visited the advertiser's site). Harder to attribute but real signal.

### Frequency
How many times the same user has seen the same ad. Too high = ad fatigue, user annoyance. Frequency capping = setting a max number of times one user sees one ad.

### Fill Rate
Percentage of available ad slots that were actually filled with a paid ad. Low fill rate = unsold inventory = lost revenue.

### Sell-Through Rate
Similar to fill rate — percentage of available inventory that was sold. This was the core metric in your ad-yield project at Slalom.

---

## Pricing Models

### CPM (Cost Per Mille)
Cost per **1,000 impressions**. Advertiser pays every time their ad is shown 1,000 times, regardless of clicks.
- Common for brand awareness campaigns.

### CPC (Cost Per Click)
Advertiser pays only when a user **clicks** the ad.
- Common for performance/direct response campaigns.

### CPA (Cost Per Action / Acquisition)
Advertiser pays only when a user completes a specific **action** — purchase, sign-up, app install.
- Hardest to attribute, highest intent signal.

### CPV (Cost Per View)
Common in video ads. Advertiser pays per video view (usually defined as watching X seconds).

### eCPM (Effective CPM)
Normalizes all pricing models to a per-1,000-impressions basis so you can compare them apples-to-apples.
`eCPM = (Total Revenue / Total Impressions) × 1000`

---

## Auction Mechanics

### Second-Price Auction (Vickrey Auction)
Winner pays the price of the **second-highest bid**, not their own bid.
- **Why:** Incentivizes truthful bidding — advertisers bid their true value since they won't overpay if they win.
- Reddit runs a second-price auction.

### First-Price Auction
Winner pays their **own bid**. Requires strategic bid shading (bidding below true value). Increasingly common in programmatic.

### Reserve Price / Floor Price
The minimum bid an advertiser must meet for their ad to be eligible to win. Below this, the slot goes unfilled or goes to a default/house ad.

### Bid Shading
In first-price auctions, advertisers deliberately bid *below* their true value to avoid overpaying. Platforms and DSPs use ML to estimate the optimal shaded bid.

### Clearing Price
The final price the winning advertiser pays. In second-price: the second-highest bid (or floor price, whichever is higher).

---

## Ranking & Relevance

### Ranking Score
The combined score used to decide which ad wins an auction. Typically:
`Ranking Score = Relevance Score × Bid`
Not just bid — otherwise low-quality high-bid ads always win, users disengage, platform degrades.

### Relevance Score / Quality Score
A model-predicted score representing how relevant this ad is to this user right now. Typically a predicted CTR or engagement probability.

### Click-Through Rate (CTR)
`CTR = Clicks / Impressions`
Fraction of times an ad was shown that resulted in a click. Core signal for training relevance models.

### pCTR (Predicted CTR)
The model's prediction of how likely this user is to click this ad. Used in real-time ranking.
`Ranking Score = pCTR × Bid`

### Conversion Rate (CVR)
`CVR = Conversions / Clicks`
Fraction of clicks that resulted in a conversion (purchase, sign-up etc.)

---

## Targeting

### Behavioral Targeting
Targeting based on what a user has **done** — pages visited, ads clicked, content consumed. Requires tracking user activity over time.

### Contextual Targeting
Targeting based on the **content** being consumed right now — not who the user is, but what they're reading/watching. Privacy-friendly, no user history needed.
- Growing in importance post-cookie deprecation.

### Demographic Targeting
Targeting based on user attributes — age, gender, location, income bracket. Typically slow-changing or static.

### Interest-Based Targeting
Targeting based on inferred interests from behavior. On Reddit, this is subreddit membership, upvote history, post engagement.

### Lookalike Targeting
Given a seed audience (e.g., existing customers), find users who are statistically similar. Common in cold-start scenarios for new advertisers.

### Retargeting / Remarketing
Showing ads to users who have **previously interacted** with the advertiser — visited their website, added to cart, etc. High intent signal.

---

## Privacy & Identity

### Third-Party Cookie
A tracking identifier set by a domain other than the one the user is visiting. Used historically for cross-site behavioral tracking. Being deprecated by major browsers.

### First-Party Cookie / First-Party Data
Data collected directly by the platform the user is on. Reddit's own behavioral data = first-party data. Not affected by third-party cookie deprecation.

### Privacy Sandbox (Google)
Google's initiative to replace third-party cookies with privacy-preserving APIs that run in the browser.
- **Topics API:** Browser exposes a handful of coarse interest categories without sharing browsing history to servers.
- **Protected Audience API:** Runs remarketing auctions locally in the browser — ad decision happens on-device, no user data sent to servers.

### On-Device Inference
Running the ad scoring or targeting model inside the user's browser or device rather than on a server. Relevant for third-party networks post-cookie deprecation. Less relevant for first-party platforms that own the data server-side (though not impossible).

### Identity Resolution
Connecting the same user across devices, sessions, or platforms. E.g., recognizing that the same person who browsed on mobile is now on desktop.

---

## Programmatic Ecosystem

### DSP (Demand-Side Platform)
Software used by **advertisers** to buy ad inventory programmatically. They set targeting, bids, budgets. Examples: The Trade Desk, DV360 (Google), Amazon DSP.

### SSP (Supply-Side Platform)
Software used by **publishers** to sell their ad inventory programmatically. Connects publishers to multiple DSPs/ad exchanges. Examples: Google Ad Manager, Magnite, PubMatic.

### Ad Exchange
A marketplace where DSPs and SSPs connect to transact. Real-time auctions happen here. Example: Google Ad Exchange.

### RTB (Real-Time Bidding)
The mechanism where ad auctions happen in **real time** — within milliseconds of a page load. Each impression triggers an auction, DSPs bid, winner is chosen, ad is served.

### Header Bidding
A technique where publishers offer inventory to multiple ad exchanges simultaneously *before* calling their primary ad server. Increases competition, drives up CPMs.
- Playwire uses this (via Prebid.js).

### DMP (Data Management Platform)
A system for collecting, organizing, and activating audience data. Playwire uses a built-in DMP to segment audiences for advertisers.

---

*[More sections TBD: Measurement & Attribution, Feedback Loops, ML-Specific Terms]*
