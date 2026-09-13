## Load Estimation: RPS and Network Traffic by Subsystem
This section provides an initial estimate of request sizes and network traffic for the main application scenarios: posts, reactions, comments, subscriptions, location-based discovery, and user feeds.
### Data Structures
#### Post
**Metadata**
| Field | Type | Estimated Size |
|---|---|---:|
| `post_id` | UUIDv7 | 16 B |
| `owner_id` | UUIDv7 | 16 B |
| `text` | String, up to 255 Unicode characters | 1,020 B |
| `att_links` | Up to 10 × UUIDv7 | 160 B |
| `geo` | Geolocation data | 100 B |
| `created_at` | Timestamp | 32 B |
| **Total metadata** | | **1,344 B** |
The text size assumes up to 4 bytes per Unicode character to accommodate Russian text and emoji.
**Attachments**
- Maximum number of photos per post: **10**
- Average original photo size on upload: **5 MB**
- Average optimized photo size on download: **1 MB**
Estimated maximum post traffic:
| Operation | Calculation | Size |
|---|---:|---:|
| Write | 1,344 B + 10 × 5 MB | **~50.0013 MB** |
| Full read | 1,344 B + 10 × 1 MB | **~10.0013 MB** |
#### Reaction
| Field | Size |
|---|---:|
| `post_id` | 16 B |
| `user_id` | 16 B |
| `reaction` | 4 B |
| `created_at` | 32 B |
| **Total** | **68 B** |
#### Comment
| Field | Size |
|---|---:|
| `post_id` | 16 B |
| `user_id` | 16 B |
| `text` | 1,020 B |
| `created_at` | 32 B |
| **Total** | **1,084 B** |
#### Subscription
| Field | Size |
|---|---:|
| `post_id` | 16 B |
| `user_id` | 16 B |
| `owner_id` | 16 B |
| `status` | 1 B |
| `sub_time_start` | 32 B |
| **Total** | **81 B** |
#### Finder Request
| Field | Size |
|---|---:|
| `geo` | 100 B |
| `user_id` | 16 B |
| **Total** | **116 B** |
#### Posts Feed Request
| Field | Size |
|---|---:|
| `posts_owner_id` | 16 B |
| `user_id` | 16 B |
| **Total** | **32 B** |
### Protocol and Authentication Overhead
For capacity estimation, HTTP headers and authentication data are conservatively estimated as:
**HEAD = HTTP + authentication overhead ≈ 2 KB per request**
This is intentionally a conservative approximation. Actual overhead depends on the transport protocol, header compression, token format, and connection reuse.
### Photo Read Scenarios
Feed and discovery traffic depends heavily on how aggressively the client loads post attachments. Three scenarios are used:
| Scenario | Client Behavior | Traffic per 10 Posts |
|---|---|---:|
| Light | 1 preview × 300 KB per post | **~3 MB** |
| Average | 2 photos × 500 KB per post | **~10 MB** |
| Heavy / Peak | 10 photos × 1 MB per post | **~100 MB** |
The **Average** scenario is used as the baseline capacity estimate, while the **Heavy** scenario represents the upper-bound / peak case.
### Traffic by User Scenario
#### 1. Publish a Travel Post
A traveler publishes a post containing photos, a short description, and a geolocation reference.
**Write:**
`Post metadata + 10 original photos + HEAD`
`≈ 1,344 B + 50 MB + 2 KB`
**Traffic: ~50 MB**
**Read: 0**
#### 2. React to or Comment on Another Traveler's Post
Reaction and comment are treated as independent API operations.
**Reaction:**
`68 B + 2,000 B = 2,068 B`
**Traffic: ~2.07 KB**
**Comment:**
`1,084 B + 2,000 B = 3,084 B`
**Traffic: ~3.08 KB**
**Read: 0**
Post retrieval required before the interaction is accounted for separately as feed/discovery traffic.
#### 3. Subscribe to Another Traveler
A user subscribes to another traveler to follow their activity.
**Write:**
`81 B + 2,000 B = 2,081 B`
**Traffic: ~2.08 KB**
**Read: 0**
#### 4. Find Popular Travel Locations and Browse Posts
The user searches for a location and receives posts associated with that location.
**Request:**
`116 B + 2,000 B = 2,116 B`
**Traffic: ~2.12 KB**
**Response — 10 posts:**
- Light: **~3 MB**
- Average: **~10 MB**
- Heavy / Peak: **~100 MB**
**Baseline traffic per request: ~10 MB**
**Peak traffic per request: ~100 MB**
#### 5. Browse User / Subscription Feed
The user requests a reverse-chronological feed based on followed travelers.
**Request:**
`32 B + 2,000 B = 2,032 B`
**Traffic: ~2.03 KB**
**Response — 10 posts:**
- Light: **~3 MB**
- Average: **~10 MB**
- Heavy / Peak: **~100 MB**
**Baseline traffic per request: ~10 MB**
**Peak traffic per request: ~100 MB**
### Summary
| Operation | Request / Write Traffic | Response / Read Traffic | Peak Read Traffic |
|---|---:|---:|---:|
| Publish post | **~50 MB** | — | — |
| Reaction | **~2.07 KB** | — | — |
| Comment | **~3.08 KB** | — | — |
| Subscription | **~2.08 KB** | — | — |
| Location finder | **~2.12 KB** | **~10 MB** | **~100 MB** |
| Posts feed | **~2.03 KB** | **~10 MB** | **~100 MB** |
