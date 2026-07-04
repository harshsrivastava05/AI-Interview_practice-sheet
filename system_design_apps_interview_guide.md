# All-in-One System Design Interview Guide: 15 Common Apps, Websites, and Systems

**Purpose:** A memorization-oriented Markdown report for high-level and low-level system design interviews. It covers common app/system prompts, functional and non-functional requirements, architecture, production trade-offs, and interview Q&A.

**Interview rule:** do not start by naming technologies. Start with scope, requirements, consistency needs, and traffic assumptions. Then propose architecture.

> Exact numbers are illustrative. In a real interview, state assumptions and adjust after interviewer feedback.

## Index

### Core Framework
- [System Design Answer Template](#system-design-answer-template)
- [Recurring Building Blocks](#recurring-building-blocks)
- [Quick Comparison Matrix](#quick-comparison-matrix)

### App/System Deep Dives
1. [URL Shortener / TinyURL / Bitly](#url-shortener)
2. [Instagram / Photo Sharing / Social Feed](#instagram)
3. [Twitter / X / News Feed / Timeline](#twitter-x)
4. [WhatsApp / Messenger / Real-Time Chat](#whatsapp-chat)
5. [YouTube / Netflix / Video Streaming Platform](#video-streaming)
6. [Uber / Ola / Ride-Hailing Dispatch](#ride-hailing)
7. [Google Drive / Dropbox / File Storage and Sync](#file-storage-sync)
8. [Google Docs / Real-Time Collaborative Editor](#collaborative-editor)
9. [Amazon / Flipkart / E-Commerce Marketplace](#ecommerce)
10. [Payment System / Stripe / UPI-like Payments](#payment-system)
11. [Search Engine / Search Autocomplete](#search-autocomplete)
12. [Notification System / Push / Email / SMS](#notification-system)
13. [Ticketmaster / BookMyShow / Seat Booking](#ticket-booking)
14. [Swiggy / Zomato / DoorDash Food Delivery](#food-delivery)
15. [Rate Limiter / API Gateway / Throttling System](#rate-limiter-api-gateway)

---

<a id="system-design-answer-template"></a>
## System Design Answer Template

1. **Clarify scope:** Ask what is in/out: for Instagram, is chat/reels/search included?
2. **Functional requirements:** What users can do.
3. **Non-functional requirements:** Latency, availability, consistency, durability, security, cost.
4. **Scale assumptions:** DAU, QPS, read/write ratio, storage, bandwidth.
5. **API design:** Core endpoints and idempotency where needed.
6. **Data model:** Source-of-truth tables/documents/events and derived views.
7. **High-level architecture:** Services, DBs, cache, queues, object storage, CDN, search indexes.
8. **Deep dives:** 2-3 hard parts: fanout, counters, booking consistency, payments, realtime delivery.
9. **Failure modes:** Retries, dedupe, DLQ, stale cache, provider failure, regional outage.
10. **Trade-offs:** Why this design and when you would change it.

**Strong opening:** “I’ll clarify scope, define functional and non-functional requirements, estimate scale, propose APIs and data model, draw the high-level architecture, and then deep dive into the bottleneck areas like consistency, fanout, caching, and failure handling.”

---

<a id="recurring-building-blocks"></a>
## Recurring Building Blocks

| Building Block | Definition | Example | Production Use |
|---|---|---|---|
| **Load Balancer** | Distributes traffic across service instances. | Requests spread across app servers. | Availability, horizontal scaling, routing. |
| **API Gateway** | Entry point for auth, routing, validation, rate limits. | Rejects over-quota API key before backend. | Central policy and service protection. |
| **Cache** | Fast store for frequently accessed data. | Redis caches URL mappings/feed pages. | Lower latency and DB load; needs invalidation plan. |
| **CDN** | Edge network for static/media assets. | Videos/images served from nearby edge. | Global latency and bandwidth reduction. |
| **Queue/Stream** | Async buffer for events/jobs. | PostCreated triggers feed fanout. | Decoupling, retries, smoothing spikes. |
| **Object Storage** | Durable blob storage. | Photos, videos, files, thumbnails. | Cheaper and better than DB for large binaries. |
| **Search Index** | Specialized full-text/relevance index. | Product/tweet search. | Relevance, facets, typo tolerance. |
| **Sharding** | Split data by key across nodes. | Shard timelines by user_id. | Scale storage/QPS. |
| **Replication** | Multiple data copies. | Primary writes, replicas read. | Availability and read scale. |
| **Idempotency** | Repeated request has same side effect/result. | Payment retry does not double charge. | Required for retries/mobile/provider callbacks. |
| **Saga** | Distributed workflow with compensation. | Reserve inventory -> payment -> release on failure. | When services/providers cannot share a transaction. |
| **Eventual Consistency** | Derived state catches up later. | Like count/feed/search lags slightly. | Keeps hot path fast/resilient. |

---

<a id="quick-comparison-matrix"></a>
## Quick Comparison Matrix

| System | Main Bottleneck | Consistency Need | Core Storage | Must-Discuss Deep Dive |
|---|---|---|---|---|
| [URL Shortener / TinyURL / Bitly](#url-shortener) | Redirect QPS | Strong mapping; eventual analytics | KV/DB + cache | Key generation + redirect cache |
| [Instagram / Photo Sharing / Social Feed](#instagram) | Feed fanout + counters | Eventual counts/feed; strong visibility | Object storage + metadata + feed store | Dynamic likes + hybrid feed |
| [Twitter / X / News Feed / Timeline](#twitter-x) | Timeline fanout/ranking | Eventual timeline/counters | Tweet store + timeline cache | Celebrity fanout |
| [WhatsApp / Messenger / Real-Time Chat](#whatsapp-chat) | Connections + delivery | At-least-once + ordering per chat | Message store + connection registry | WebSocket + offline sync |
| [YouTube / Netflix / Video Streaming Platform](#video-streaming) | Bandwidth + transcoding | Durable assets; eventual views | Object storage + CDN + metadata | Adaptive streaming |
| [Uber / Ola / Ride-Hailing Dispatch](#ride-hailing) | Geo matching | Strong assignment/trip state | Geo cache + trip DB | Atomic assignment |
| [Google Drive / Dropbox / File Storage and Sync](#file-storage-sync) | Sync + permissions | Strong permissions; eventual sync | Object storage + metadata DB | Delta sync + ACL |
| [Google Docs / Real-Time Collaborative Editor](#collaborative-editor) | Concurrent edits | Convergence required | Op log + snapshots | OT/CRDT |
| [Amazon / Flipkart / E-Commerce Marketplace](#ecommerce) | Inventory + checkout | Strong inventory/order/payment | Catalog/search + order DB | Inventory reservation saga |
| [Payment System / Stripe / UPI-like Payments](#payment-system) | Correctness + providers | Strong ledger/audit | Payment DB + ledger | Idempotency + reconciliation |
| [Search Engine / Search Autocomplete](#search-autocomplete) | Index/query latency | Eventual freshness | Inverted index + source store | Ranking + autocomplete |
| [Notification System / Push / Email / SMS](#notification-system) | Fanout + providers | At-least-once with dedupe | Queue + jobs DB | Retries/DLQ |
| [Ticketmaster / BookMyShow / Seat Booking](#ticket-booking) | Seat contention | Strong seat booking | Seat inventory DB | Temporary holds |
| [Swiggy / Zomato / DoorDash Food Delivery](#food-delivery) | Dispatch + ETA | Strong order/payment; eventual tracking | Order DB + geo cache | Courier assignment |
| [Rate Limiter / API Gateway / Throttling System](#rate-limiter-api-gateway) | Decision QPS | Approx/strict by policy | Redis/local counters | Token bucket |

---

<a id="url-shortener"></a>
## 1. URL Shortener / TinyURL / Bitly

**What this interview tests:** unique ID generation, read-heavy redirect path, caching, analytics, abuse prevention, and sharding.

### Functional Requirements
- Create a short URL from a long URL.
- Redirect short URL to original URL.
- Support custom alias, expiration, and deletion/revocation.
- Track click analytics such as count, timestamp, referrer, geo/device approximation.
- Block malicious/spam URLs and enforce creation limits.

### Non-Functional Requirements
- Redirect path should be extremely low latency.
- Redirects should remain available even if analytics is delayed.
- Mapping correctness is strong; analytics can be eventually consistent.
- System should handle read-heavy traffic and viral links.
- Abuse prevention and revocation must propagate quickly.

### Core APIs
```http
POST /v1/urls {long_url, custom_alias?, expires_at?} -> {short_url}
GET /{short_key} -> 301/302 redirect
GET /v1/urls/{short_key}/analytics?from=&to=
DELETE /v1/urls/{short_key}
```

### Data Model / Source of Truth
- `url_mapping(short_key PK, long_url, user_id, created_at, expires_at, status)`
- `click_events(event_id, short_key, timestamp, referrer, user_agent, geo, ip_hash)`
- `analytics_rollups(short_key, day, clicks, device_counts, country_counts)`
- `abuse_flags(short_key, reason, status, reviewed_at)`

### High-Level Architecture
- API Gateway handles authentication, rate limiting, and routing.
- URL service validates long URL, generates short key, and writes mapping.
- Redirect service resolves short_key from cache/DB and returns redirect.
- Redis/edge cache stores hot short_key -> long_url mappings.
- Redirect service emits click events to Kafka/SQS/PubSub for async analytics.
- Analytics workers build rollups; abuse workers scan suspicious destinations.

### Deep-Dive Topics

#### 1.1 Short key generation
- **Definition:** Generate a compact unique identifier using Base62 random keys, counters, or Snowflake-style IDs.
- **Example:** A sequence ID 125000 can be encoded as Base62; a random 7-character Base62 key has a huge keyspace.
- **Use in production:** Use a DB unique constraint as final collision protection. Counter-based keys are simple but reveal volume; random keys hide volume but need retries.
- **Interview question:** How do you avoid key collisions?
- **Strong answer:** Use monotonic ID allocation or insert random keys with unique constraint and retry on conflict.

#### 1.2 Redirect hot path
- **Definition:** The read path that maps short_key to long_url and returns redirect.
- **Example:** GET /abc123 should do cache lookup and return 302 quickly.
- **Use in production:** Keep it stateless, cache hot mappings, avoid synchronous analytics writes, and use regional read replicas/CDN edge caching.
- **Interview question:** 301 or 302 redirect?
- **Strong answer:** Use 302 by default when target/analytics/expiration can change; use 301 only for truly permanent redirects.

#### 1.3 Caching and invalidation
- **Definition:** Fast storage for frequently accessed mappings.
- **Example:** A celebrity-shared URL gets millions of hits and should be served mostly from Redis/CDN.
- **Use in production:** Cache-aside with TTL is simple. Revocation requires invalidation or short TTL plus blocked-status checks.
- **Interview question:** What if a blocked URL is cached?
- **Strong answer:** Push invalidation for high-risk blocks and keep short TTLs; read status from source of truth if cache is stale.

#### 1.4 Analytics pipeline
- **Definition:** Asynchronous processing of click events and rollups.
- **Example:** Redirect emits {short_key, timestamp, referrer}; worker aggregates per day.
- **Use in production:** Use append-only events, partition by time/key, and roll up in OLAP storage or partitioned DB.
- **Interview question:** Why not increment count synchronously?
- **Strong answer:** It adds latency and creates hotspots for viral URLs; async aggregation is safer.

#### 1.5 Custom aliases
- **Definition:** User-selected short paths requiring uniqueness and policy checks.
- **Example:** brand.com/sale must be globally unique and not reserved.
- **Use in production:** Use unique indexes, reserved-word checks, abuse review, and conflict suggestions.
- **Interview question:** Two users request same alias?
- **Strong answer:** Transactional insert with unique constraint; first commit wins.

#### 1.6 Expiration and deletion
- **Definition:** Mapping status changes that stop redirects after expiry/delete/block.
- **Example:** Expired URL returns 410 Gone or a safe landing page.
- **Use in production:** Store status in mapping table; background expiry jobs and cache invalidation keep redirects correct.
- **Interview question:** How do you handle expired links in cache?
- **Strong answer:** Include expires_at/status in cache value or use short TTL and invalidate on expiry/delete.

#### 1.7 Sharding strategy
- **Definition:** Partitioning mapping data across DB nodes.
- **Example:** Shard by hash(short_key) because redirects always query by short_key.
- **Use in production:** Avoid user_id sharding for redirect lookup; use consistent hashing or distributed KV/DB.
- **Interview question:** Best shard key?
- **Strong answer:** short_key, because it matches the dominant read query.

#### 1.8 Abuse and rate limiting
- **Definition:** Controls for spam, phishing, and excessive link creation.
- **Example:** Anonymous user limited to 10 creates/hour; suspicious domains go to review.
- **Use in production:** Use gateway limits, domain reputation checks, malware scanning, CAPTCHA, and report workflows.
- **Interview question:** Where is rate limiting enforced?
- **Strong answer:** At gateway for coarse limits and URL service for business-specific quotas.

### Common Interview Questions
- **Q: Design TinyURL end to end.**
  - **A:** Cover create, redirect, key generation, mapping DB, cache, async analytics, abuse blocking, and sharding.
- **Q: How do you scale global redirects?**
  - **A:** Use stateless redirect services, regional caches/read replicas, CDN edge caching, and async analytics.
- **Q: What can be eventually consistent?**
  - **A:** Analytics, rollups, and dashboards. The mapping and block/revoke status must be correct.

### Common Pitfalls
- Updating analytics synchronously in redirect path.
- Ignoring cache invalidation for revoked/malicious URLs.
- Using long_url hash without collision strategy.
- Choosing user_id as primary shard key for redirect lookup.

### 60-Second Interview Summary

For **URL Shortener / TinyURL / Bitly**, first clarify scope and correctness requirements. The core architecture is: API Gateway handles authentication, rate limiting, and routing. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Short key generation**, **Redirect hot path**, **Caching and invalidation**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="instagram"></a>
## 2. Instagram / Photo Sharing / Social Feed

**What this interview tests:** media upload, CDN, feed generation, social graph, likes/comments counters, ranking, stories/reels, and dynamic content.

### Functional Requirements
- Users upload photos/videos with captions.
- Users follow/unfollow other users.
- Users view home feed and profile posts.
- Users like, unlike, comment, save, and share posts.
- Support stories/reels, notifications, and hashtag/user search if in scope.

### Non-Functional Requirements
- Media viewing should be fast globally.
- Feed reads should be low latency.
- Likes/comments counts may be eventually consistent.
- Uploads should be durable and processed asynchronously.
- Privacy and visibility checks must be correct.

### Core APIs
```http
POST /v1/media/upload-init -> signed_url
POST /v1/posts {media_id, caption, visibility}
GET /v1/feed?cursor=
POST /v1/posts/{id}/likes
DELETE /v1/posts/{id}/likes
POST /v1/posts/{id}/comments
POST /v1/users/{id}/follow
```

### Data Model / Source of Truth
- `users(user_id, username, profile, status)`
- `follows(follower_id, followee_id, created_at)`
- `media(media_id, owner_id, object_key, type, processing_status)`
- `posts(post_id, author_id, media_id, caption, visibility, created_at)`
- `likes(post_id, user_id, created_at) unique(post_id,user_id)`
- `comments(comment_id, post_id, user_id, body, status, created_at)`
- `feed_items(user_id, post_id, author_id, score, inserted_at)`
- `post_counters(post_id, like_count, comment_count, share_count)`

### High-Level Architecture
- Client uploads media directly to object storage using signed URLs.
- Media processor creates thumbnails, compressed variants, video transcodes, and CDN URLs.
- Post service writes metadata and emits PostCreated.
- Feed workers fan out posts to followers for normal users; celebrity posts are merged at read time.
- Interaction service stores likes/comments and emits counter/notification events.
- Feed read service joins feed IDs with cached post/user/media metadata and applies visibility checks.

### Deep-Dive Topics

#### 2.1 Media upload pipeline
- **Definition:** Flow for storing, validating, processing, and delivering photos/videos.
- **Example:** Upload image to S3/GCS, generate thumbnails, strip EXIF, publish CDN URL.
- **Use in production:** Do not proxy large files through app servers; use object storage, async processing, moderation, and CDN.
- **Interview question:** Why direct-to-object-storage upload?
- **Strong answer:** It saves app server bandwidth/CPU and uses storage-native multipart durability.

#### 2.2 Feed generation
- **Definition:** Selecting posts for a user’s home feed.
- **Example:** When Bob posts, insert post_id into followers’ feed_items.
- **Use in production:** Use fanout-on-write for normal accounts and fanout-on-read/hybrid for celebrities.
- **Interview question:** Fanout-on-write or read?
- **Strong answer:** Hybrid: precompute normal users for fast reads, merge celebrity/recommended posts on read.

#### 2.3 Dynamic like count
- **Definition:** A high-write aggregate changing with likes/unlikes.
- **Example:** A viral post receives 50K likes/sec; one counter row becomes a hotspot.
- **Use in production:** Store like edges as source of truth; update sharded counters/cache asynchronously.
- **Interview question:** How do you prevent counter hotspot?
- **Strong answer:** Shard counters or aggregate events async; displayed count can lag.

#### 2.4 Like idempotency
- **Definition:** Repeated like/unlike calls should not double-count.
- **Example:** Double-tap should create one like edge.
- **Use in production:** Use unique(post_id,user_id); increment counters only if insert actually happened.
- **Interview question:** How to handle duplicate like requests?
- **Strong answer:** Unique constraint plus idempotent API semantics.

#### 2.5 Comments pagination
- **Definition:** Fetching comments reliably under insert/delete churn.
- **Example:** GET comments after cursor=comment_123 returns next 20.
- **Use in production:** Use cursor pagination, moderation states, hot-comment cache, and anti-spam.
- **Interview question:** Why not offset pagination?
- **Strong answer:** Offset is slow and unstable as comments are inserted/deleted.

#### 2.6 Social graph
- **Definition:** Follower/followee relationships used for feed, privacy, and notifications.
- **Example:** follows(follower_id, followee_id) answers who I follow and who follows creator.
- **Use in production:** Maintain indexes both directions; partition celebrity follower lists.
- **Interview question:** What indexes are needed?
- **Strong answer:** By follower_id for feed reads and by followee_id for fanout.

#### 2.7 Ranking and recommendations
- **Definition:** Ordering feed posts by predicted relevance.
- **Example:** Rank = recency + relationship strength + engagement probability.
- **Use in production:** Separate candidate generation, ranking, policy filters, and fallback chronological feed.
- **Interview question:** How do you keep feed fresh?
- **Strong answer:** Use precomputed candidates plus short-lived re-ranking and event invalidation.

#### 2.8 Stories/Reels
- **Definition:** Ephemeral or video-first content with different lifecycle and ranking.
- **Example:** Stories expire after 24 hours; reels are video-heavy and ranked globally.
- **Use in production:** Use TTLs, object lifecycle, separate feed surfaces, CDN, and view-event aggregation.
- **Interview question:** How do stories differ from posts?
- **Strong answer:** They need expiration, view tracking, and different ranking/storage policies.

#### 2.9 Notification fanout
- **Definition:** Events from likes/comments/follows/mentions delivered to users.
- **Example:** Alice gets notified when Bob comments on her post.
- **Use in production:** Use event queue, dedupe, preferences, batching, and creator-specific suppression.
- **Interview question:** How avoid celebrity notification storm?
- **Strong answer:** Aggregate, sample, or digest low-value events.

### Common Interview Questions
- **Q: Design Instagram feed.**
  - **A:** Clarify upload/feed/likes/comments, then use object storage+CDN, metadata DB, social graph, feed service, counters, cache, and async events.
- **Q: How does like count update?**
  - **A:** Like edge is source of truth; counter is derived via sharded counters or stream aggregation.
- **Q: How do you handle celebrity users?**
  - **A:** Avoid fanout to every follower; merge their posts during read/ranking.

### Common Pitfalls
- Storing media bytes in relational DB.
- Updating one hot counter row per like.
- Skipping visibility checks for cached feed items.
- Using offset pagination for feeds/comments.

### 60-Second Interview Summary

For **Instagram / Photo Sharing / Social Feed**, first clarify scope and correctness requirements. The core architecture is: Client uploads media directly to object storage using signed URLs. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Media upload pipeline**, **Feed generation**, **Dynamic like count**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="twitter-x"></a>
## 3. Twitter / X / News Feed / Timeline

**What this interview tests:** timeline fanout, ranking, tweet storage, social graph, hashtags/trends, search, counters, and realtime updates.

### Functional Requirements
- Users post short text/media messages.
- Users follow/unfollow accounts.
- Users view home timeline and user profile timeline.
- Users like, reply, repost, quote, and bookmark.
- Support search, hashtags, trends, notifications, and live updates.

### Non-Functional Requirements
- Timeline reads should be low-latency.
- System must handle write spikes during events.
- Counters and search can be eventually consistent.
- Tweet storage must be durable and deletion-aware.
- Ranking failure should degrade to chronological/cached feed.

### Core APIs
```http
POST /v1/tweets {text, media_ids?}
GET /v1/timeline/home?cursor=
GET /v1/users/{id}/tweets?cursor=
POST /v1/tweets/{id}/like
POST /v1/tweets/{id}/retweet
GET /v1/search?q=&cursor=
```

### Data Model / Source of Truth
- `tweets(tweet_id, author_id, text, media_ids, created_at, status)`
- `follows(follower_id, followee_id, created_at)`
- `timeline_items(user_id, tweet_id, author_id, rank_score, inserted_at)`
- `tweet_edges(tweet_id, user_id, type, created_at)`
- `tweet_counters(tweet_id, like_count, reply_count, repost_count)`
- `hashtag_index(hashtag, tweet_id, created_at)`

### High-Level Architecture
- Tweet service writes tweets and emits TweetCreated.
- Fanout workers insert tweet IDs into followers’ home timeline cache/store for normal accounts.
- Timeline read service merges precomputed timeline, celebrity tweets, ads, recommendations, and ranks.
- Search indexer consumes tweet events to update inverted index.
- Counter and notification workers consume interaction events asynchronously.

### Deep-Dive Topics

#### 3.1 Timeline fanout
- **Definition:** Distributing new tweets to followers’ timelines.
- **Example:** Bob tweets; tweet_id is inserted into followers’ timelines.
- **Use in production:** Use fanout-on-write for normal users and hybrid fanout for celebrities.
- **Interview question:** Why not always fanout-on-write?
- **Strong answer:** Celebrity tweets can require millions of writes; hybrid avoids write explosion.

#### 3.2 Home timeline vs user timeline
- **Definition:** Home timeline aggregates followed/recommended tweets; user timeline is authored tweets.
- **Example:** GET /home merges many authors; GET /users/bob/tweets reads Bob’s tweets.
- **Use in production:** User timeline is indexed by author_id; home timeline is personalized and fanout-based.
- **Interview question:** Which is harder?
- **Strong answer:** Home timeline because it is personalized, ranked, multi-source, and high-volume.

#### 3.3 Snowflake IDs
- **Definition:** Globally unique, time-sortable IDs.
- **Example:** tweet_id encodes timestamp, worker ID, and sequence.
- **Use in production:** Avoid central DB sequence bottleneck and simplify time pagination.
- **Interview question:** Why Snowflake?
- **Strong answer:** Distributed uniqueness plus rough chronological order.

#### 3.4 Ranking
- **Definition:** Scoring candidate tweets by relevance.
- **Example:** Close friend’s reply ranks above a newer weak-connection tweet.
- **Use in production:** Use candidate generation, feature retrieval, ML scoring, policy filters, and fallback ranking.
- **Interview question:** What if ranking fails?
- **Strong answer:** Return cached/chronological feed as graceful degradation.

#### 3.5 Trending topics
- **Definition:** Detecting phrases/hashtags with unusual velocity.
- **Example:** #WorldCup spikes in a 15-minute region window.
- **Use in production:** Use streaming window aggregation, baseline comparison, region filters, and spam suppression.
- **Interview question:** Is trending just most frequent?
- **Strong answer:** No; it is velocity relative to baseline plus quality/spam filters.

#### 3.6 Search indexing
- **Definition:** Near-real-time inverted index for tweet search.
- **Example:** term redis -> recent tweet IDs containing redis.
- **Use in production:** Use fresh small index plus immutable segment merges; filter deleted/private tweets.
- **Interview question:** How do new tweets appear quickly?
- **Strong answer:** TweetCreated events update a near-real-time index.

#### 3.7 Realtime updates
- **Definition:** Pushing live signals to active clients.
- **Example:** Timeline shows “12 new posts” during a live event.
- **Use in production:** Use WebSockets/SSE/pub-sub with backpressure; push signals, not every item.
- **Interview question:** Push every tweet?
- **Strong answer:** No; push lightweight refresh signal or badge.

#### 3.8 Interaction counters
- **Definition:** Like/repost/reply counts derived from interaction edges.
- **Example:** A tweet’s repost count changes when users repost/undo repost.
- **Use in production:** Use unique edges as source of truth, async counters, hot counter cache.
- **Interview question:** How prevent double-like?
- **Strong answer:** unique(tweet_id,user_id,type) and idempotent API.

### Common Interview Questions
- **Q: Design Twitter timeline.**
  - **A:** Tweet write, follow graph, fanout, timeline cache, ranking, celebrity handling, search, counters, and notifications.
- **Q: What is the hardest part?**
  - **A:** Home timeline fanout/ranking at scale, especially celebrity accounts and breaking-news spikes.
- **Q: How handle tweet deletion?**
  - **A:** Mark deleted in source, filter at read, remove from caches/index asynchronously.

### Common Pitfalls
- Ignoring celebrity fanout explosion.
- No deletion filtering for derived timelines/search.
- Treating trends as raw hashtag frequency.
- No fallback when ranking/indexing is delayed.

### 60-Second Interview Summary

For **Twitter / X / News Feed / Timeline**, first clarify scope and correctness requirements. The core architecture is: Tweet service writes tweets and emits TweetCreated. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Timeline fanout**, **Home timeline vs user timeline**, **Snowflake IDs**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="whatsapp-chat"></a>
## 4. WhatsApp / Messenger / Real-Time Chat

**What this interview tests:** WebSockets, message delivery guarantees, ordering, offline sync, group fanout, media, receipts, presence, and encryption boundaries.

### Functional Requirements
- One-to-one and group messaging.
- Send text, media, reactions, delivery/read receipts.
- Deliver messages to online users in real time.
- Queue/sync messages for offline users.
- Show typing/presence if in scope.

### Non-Functional Requirements
- Low-latency delivery for online users.
- At-least-once delivery with deduplication.
- Ordering within conversation as much as practical.
- High availability under network reconnects.
- Message content may be end-to-end encrypted.

### Core APIs
```http
POST /v1/messages {conversation_id, client_message_id, encrypted_payload}
WebSocket: message.send / message.ack / message.delivered / message.read
GET /v1/conversations/{id}/messages?cursor=
POST /v1/media/upload-init
```

### Data Model / Source of Truth
- `users(user_id, phone_hash, public_keys)`
- `conversations(conversation_id, type, created_at)`
- `conversation_members(conversation_id, user_id, role, joined_at)`
- `messages(message_id, conversation_id, sender_id, client_message_id, payload_ref, sequence_no, created_at)`
- `message_delivery(message_id, user_id, delivered_at, read_at)`
- `connection_registry(user_id, device_id, gateway_id, connection_id, last_seen)`

### High-Level Architecture
- Clients maintain WebSocket connections to gateway servers.
- Connection registry maps users/devices to gateway instances.
- Message service validates membership, dedupes send, assigns sequence/ID, persists message.
- Delivery service routes to online users via gateway pub-sub and stores state for offline users.
- Media is uploaded to object storage; message contains encrypted media reference.
- Presence/typing uses ephemeral pub-sub with TTL.

### Deep-Dive Topics

#### 4.1 WebSocket gateways
- **Definition:** Long-lived bidirectional connections for real-time push.
- **Example:** Mobile client connects to gateway-17; messages are routed there.
- **Use in production:** Use heartbeat, connection registry, gateway pub-sub, load shedding, and reconnect handling.
- **Interview question:** Why WebSocket over polling?
- **Strong answer:** Lower latency and less repeated HTTP overhead for real-time messaging.

#### 4.2 Delivery guarantees
- **Definition:** Contract for message delivery semantics.
- **Example:** Server retries delivery; client dedupes by message_id.
- **Use in production:** Exactly-once is unrealistic; use at-least-once with idempotency and dedupe.
- **Interview question:** Exactly once?
- **Strong answer:** No. Use at-least-once plus client/server deduplication.

#### 4.3 Message ordering
- **Definition:** Consistent order within conversation.
- **Example:** Server assigns sequence_no per conversation.
- **Use in production:** Use per-conversation sequence or shard leader; strict ordering may relax for huge groups.
- **Interview question:** How order messages from multiple senders?
- **Strong answer:** Server assigns order after validation; client local order is tentative until ack.

#### 4.4 Offline sync
- **Definition:** Deliver missed messages when user reconnects.
- **Example:** Alice offline receives Bob’s message after login.
- **Use in production:** Track per-user offsets/delivery state and provide sync API for missed messages.
- **Interview question:** Store one copy per recipient?
- **Strong answer:** Payload can be stored once; delivery/read state is per recipient.

#### 4.5 Group fanout
- **Definition:** Delivering one group message to all members.
- **Example:** A 200-member group creates delivery state for 199 recipients.
- **Use in production:** Small groups can fanout-on-write; huge groups/channels use fanout-on-read or topic model.
- **Interview question:** How handle huge groups?
- **Strong answer:** Store message once and let clients fetch by group offset; avoid synchronous per-recipient fanout.

#### 4.6 Read/delivery receipts
- **Definition:** Status transitions showing sent/delivered/read.
- **Example:** Two ticks = delivered; blue ticks = read.
- **Use in production:** Batch receipt writes and accept eventual consistency.
- **Interview question:** Are receipts strongly consistent?
- **Strong answer:** Usually no; core delivery should not block on receipt writes.

#### 4.7 End-to-end encryption boundary
- **Definition:** Server routes/stores ciphertext without plaintext access.
- **Example:** Message payload is opaque to server.
- **Use in production:** Backend handles metadata, routing, keys, and device lists but cannot do plaintext search/moderation.
- **Interview question:** How does E2EE change design?
- **Strong answer:** Server cannot inspect content; search/moderation/transforms are limited or client-side.

#### 4.8 Presence and typing
- **Definition:** Transient user activity signals.
- **Example:** Alice is typing appears for a few seconds.
- **Use in production:** Use ephemeral Redis/pub-sub TTL; do not persist typing events.
- **Interview question:** Store typing in DB?
- **Strong answer:** No, it is transient and can be dropped.

### Common Interview Questions
- **Q: Design WhatsApp.**
  - **A:** Use WebSocket gateways, message service, durable store, connection registry, delivery queue, offline sync, media object storage, and receipt events.
- **Q: How do messages reach connected user?**
  - **A:** Connection registry maps user/device to gateway; delivery event routes to that gateway.
- **Q: How handle duplicate sends?**
  - **A:** client_message_id unique per sender and idempotent server send.

### Common Pitfalls
- Claiming true exactly-once delivery.
- Persisting presence/typing as durable events.
- No reconnect/catch-up protocol.
- Proxying media through chat servers.

### 60-Second Interview Summary

For **WhatsApp / Messenger / Real-Time Chat**, first clarify scope and correctness requirements. The core architecture is: Clients maintain WebSocket connections to gateway servers. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **WebSocket gateways**, **Delivery guarantees**, **Message ordering**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="video-streaming"></a>
## 5. YouTube / Netflix / Video Streaming Platform

**What this interview tests:** video upload, transcoding, object storage, CDN, adaptive bitrate streaming, metadata/search, recommendations, watch history, and analytics.

### Functional Requirements
- Upload/publish videos or ingest provider content.
- Stream videos across devices and network speeds.
- Support search, recommendations, comments, likes, subscriptions.
- Track watch history, resume position, and view counts.
- Process thumbnails, captions, moderation, and copyright checks.

### Non-Functional Requirements
- Low startup latency and low buffering.
- Massive storage and bandwidth efficiency.
- Playback should not depend on analytics availability.
- Upload/transcoding can be asynchronous.
- Global delivery through CDN.

### Core APIs
```http
POST /v1/videos/upload-init -> signed_url
POST /v1/videos/{id}/publish {title, description, visibility}
GET /v1/videos/{id} -> metadata + manifest_url
GET /v1/search?q=
POST /v1/videos/{id}/events {play,pause,watch_time}
```

### Data Model / Source of Truth
- `videos(video_id, owner_id, title, status, visibility, created_at)`
- `video_assets(video_id, rendition, codec, bitrate, object_key, duration)`
- `thumbnails(video_id, object_key, width, height)`
- `watch_history(user_id, video_id, position_sec, watched_at)`
- `video_counters(video_id, view_count, like_count, comment_count)`
- `comments(comment_id, video_id, user_id, body, status, created_at)`

### High-Level Architecture
- Upload service issues signed URL to object storage.
- Transcoding workers create multiple renditions, thumbnails, captions, and manifests.
- Playback service returns metadata and HLS/DASH manifest URL served by CDN.
- Search/recommendation services consume metadata and engagement events.
- Analytics stream processes watch events and builds views/watch-time rollups.

### Deep-Dive Topics

#### 5.1 Transcoding
- **Definition:** Converting raw video into multiple codecs/resolutions/bitrates.
- **Example:** 4K upload becomes 240p, 720p, 1080p, 4K renditions.
- **Use in production:** Use async job queues, retryable stages, CPU/GPU workers, and processing status.
- **Interview question:** Why async processing?
- **Strong answer:** Transcoding is slow and CPU/GPU-heavy; do it outside request path.

#### 5.2 Adaptive bitrate streaming
- **Definition:** Client switches quality based on bandwidth.
- **Example:** Starts at 360p, moves to 1080p, drops to 480p under congestion.
- **Use in production:** Use HLS/DASH segmented manifests cached by CDN.
- **Interview question:** Why not one MP4?
- **Strong answer:** Adaptive segments reduce startup/buffering and improve CDN caching.

#### 5.3 CDN delivery
- **Definition:** Serve video segments from edge locations.
- **Example:** Popular 720p segments cached near Mumbai.
- **Use in production:** Use origin object storage, signed URLs, cache policies, and cache-hit monitoring.
- **Interview question:** What on CDN miss?
- **Strong answer:** CDN fetches origin, serves, and caches according to policy.

#### 5.4 View count
- **Definition:** Rules deciding when playback counts as a view.
- **Example:** Count after 30 seconds watched or threshold percentage.
- **Use in production:** Use watch events, bot filtering, dedupe windows, and delayed aggregation.
- **Interview question:** Why not count page load?
- **Strong answer:** Page load is easy to fake and does not prove viewing.

#### 5.5 Watch history
- **Definition:** Per-user progress and resume state.
- **Example:** Resume at 12:43 across devices.
- **Use in production:** Throttle writes, store latest position, and process detailed events async.
- **Interview question:** Write every second?
- **Strong answer:** No; throttle to reduce write load.

#### 5.6 Recommendations
- **Definition:** Personalized candidate generation and ranking.
- **Example:** Subscriptions, similar videos, trending, collaborative filtering.
- **Use in production:** Use offline candidates plus online ranking and fallback feeds.
- **Interview question:** Useful signals?
- **Strong answer:** Watch time, completion, likes, subscriptions, freshness, search intent, negative feedback.

#### 5.7 Moderation/copyright
- **Definition:** Detect policy or copyright violations.
- **Example:** Audio fingerprint matches protected song.
- **Use in production:** Run pre/post-publish checks, human review, takedown workflows.
- **Interview question:** Where in pipeline?
- **Strong answer:** After upload before publish for obvious issues, and post-publish for reports/model updates.

#### 5.8 Comments/counters
- **Definition:** Social engagement around videos.
- **Example:** Premiere gets thousands of comments/minute.
- **Use in production:** Partition comments, cache hot pages, async moderation/counters.
- **Interview question:** How scale comments?
- **Strong answer:** Cursor pagination, partition by video/time, and hot cache.

### Common Interview Questions
- **Q: Design YouTube.**
  - **A:** Separate upload/transcoding path from playback path. Use object storage, CDN, metadata DB, search, recommendations, and analytics stream.
- **Q: How reduce buffering?**
  - **A:** CDN, adaptive bitrate, segment prefetch, good manifest/rendition choice, and client buffering.
- **Q: Source of truth for video?**
  - **A:** Object storage for assets; metadata DB for state/renditions; indexes are derived.

### Common Pitfalls
- Serving playback through app servers.
- No async transcoding pipeline.
- Counting views on page load.
- Blocking playback on analytics writes.

### 60-Second Interview Summary

For **YouTube / Netflix / Video Streaming Platform**, first clarify scope and correctness requirements. The core architecture is: Upload service issues signed URL to object storage. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Transcoding**, **Adaptive bitrate streaming**, **CDN delivery**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="ride-hailing"></a>
## 6. Uber / Ola / Ride-Hailing Dispatch

**What this interview tests:** geospatial indexing, realtime location, matching, trip state machine, ETA, pricing, atomic assignment, and payments.

### Functional Requirements
- Riders request rides from pickup to destination.
- Drivers publish location and availability.
- System matches rider to nearby suitable driver.
- Track trip lifecycle from requested to completed/cancelled.
- Show ETA, fare estimate, live tracking, and receipt.

### Non-Functional Requirements
- Low-latency matching.
- Location display can be eventually consistent.
- Driver assignment must avoid double booking.
- Regional availability and failover.
- Auditability for payments/cancellations/disputes.

### Core APIs
```http
POST /v1/rides/estimate {pickup, dropoff}
POST /v1/rides {pickup, dropoff, vehicle_type}
POST /v1/drivers/location {lat,lng,status}
POST /v1/rides/{id}/accept
POST /v1/rides/{id}/events {arrived,started,completed,cancelled}
```

### Data Model / Source of Truth
- `drivers(driver_id, status, vehicle_type, rating, current_trip_id)`
- `driver_location(driver_id, lat, lng, geohash, updated_at)`
- `rides(ride_id, rider_id, driver_id, pickup, dropoff, status, fare, created_at)`
- `ride_events(event_id, ride_id, type, actor_id, timestamp)`
- `payments(payment_id, ride_id, amount, status, provider_ref)`

### High-Level Architecture
- Driver app sends location to location service.
- Location service updates ephemeral geo index with TTL.
- Dispatch queries nearby drivers by cell and ranks candidates by ETA/status/vehicle type.
- Assignment uses atomic conditional update to reserve driver.
- Trip service records state transitions; payment service charges after completion.

### Deep-Dive Topics

#### 6.1 Geospatial indexing
- **Definition:** Index locations so nearby drivers can be found quickly.
- **Example:** Use geohash/S2/H3 cells and neighboring cell expansion.
- **Use in production:** Keep latest locations in Redis/geo store; persist sampled history async.
- **Interview question:** How find nearby drivers?
- **Strong answer:** Query pickup cell + neighbors, then compute exact distance/ETA.

#### 6.2 Location updates
- **Definition:** Frequent driver/rider position ingestion.
- **Example:** Driver sends lat/lng every 3 seconds.
- **Use in production:** Use TTL, rate limits, ephemeral latest-location store, and sampled historical pipeline.
- **Interview question:** Store every update in DB?
- **Strong answer:** No; latest in fast store, sampled history async.

#### 6.3 Driver matching
- **Definition:** Selecting best driver for a ride.
- **Example:** Rank nearby drivers by ETA, vehicle type, acceptance probability.
- **Use in production:** Use matching windows, offer timeout, retry/expansion radius.
- **Interview question:** What signals matter?
- **Strong answer:** ETA, availability, vehicle, rating, direction, acceptance/cancellation behavior.

#### 6.4 Trip state machine
- **Definition:** Controlled lifecycle of a ride.
- **Example:** requested -> offered -> accepted -> arrived -> in_progress -> completed.
- **Use in production:** Validate transitions and store events for audit/debugging.
- **Interview question:** Why state machine?
- **Strong answer:** Prevents invalid transitions and supports idempotency/disputes.

#### 6.5 Atomic assignment
- **Definition:** Ensure one driver gets one active ride.
- **Example:** Two riders try same driver; one update wins.
- **Use in production:** Use compare-and-set/transaction on driver status/current_trip_id.
- **Interview question:** Avoid double-booking?
- **Strong answer:** Atomic update where status=available; if zero rows changed, pick another driver.

#### 6.6 ETA service
- **Definition:** Predicts travel time.
- **Example:** ETA uses traffic, road network, historical speed.
- **Use in production:** Use routing service, cache common estimates, degrade to distance heuristic.
- **Interview question:** If ETA service fails?
- **Strong answer:** Use approximate ETA and continue degraded matching.

#### 6.7 Surge pricing
- **Definition:** Price adjustment based on supply/demand.
- **Example:** Airport demand high, drivers low, price multiplier rises.
- **Use in production:** Compute by cells/windows; smooth/cap changes and log decisions.
- **Interview question:** What data drives surge?
- **Strong answer:** Request rate, available drivers, acceptance, cancellations, events/weather/history.

#### 6.8 Payment flow
- **Definition:** Charge rider/pay driver after trip.
- **Example:** On completion, capture payment and generate receipt.
- **Use in production:** Use idempotency, ledger, gateway webhooks, reconciliation.
- **Interview question:** Synchronous payment?
- **Strong answer:** Finalize trip and enqueue capture; show pending if gateway slow.

### Common Interview Questions
- **Q: Design Uber.**
  - **A:** Separate location, matching, trip state, ETA/pricing, and payment services; deep dive geo index and atomic assignment.
- **Q: How handle stale driver location?**
  - **A:** TTL latest locations and ignore drivers older than threshold.
- **Q: How scale geographically?**
  - **A:** Partition by city/region/cell because matching is local.

### Common Pitfalls
- Writing every location update synchronously to relational DB.
- No atomic guard against double assignment.
- No trip state machine.
- Ignoring stale/offline drivers.

### 60-Second Interview Summary

For **Uber / Ola / Ride-Hailing Dispatch**, first clarify scope and correctness requirements. The core architecture is: Driver app sends location to location service. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Geospatial indexing**, **Location updates**, **Driver matching**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="file-storage-sync"></a>
## 7. Google Drive / Dropbox / File Storage and Sync

**What this interview tests:** large file upload/download, object storage, metadata, permissions, sharing, sync, dedupe, versioning, and conflict handling.

### Functional Requirements
- Upload/download files and folders.
- Rename, move, delete, and search files.
- Share files/folders with users/groups/public links.
- Sync changes across devices.
- Restore versions/trash if required.

### Non-Functional Requirements
- Durable storage for file bytes.
- Fast metadata operations.
- Strong permission correctness.
- Reliable resumable uploads.
- Sync propagation can be eventually consistent.

### Core APIs
```http
POST /v1/files/upload-init {name,size,checksum,parent_id}
POST /v1/files/upload-complete {upload_id, parts}
GET /v1/files/{id}/download -> signed_url
PATCH /v1/files/{id} {name,parent_id}
POST /v1/files/{id}/share {principal, role}
GET /v1/sync?cursor=
```

### Data Model / Source of Truth
- `files(file_id, owner_id, parent_id, name, mime_type, size, current_version_id, deleted_at)`
- `file_versions(version_id, file_id, object_key, checksum, size, created_at)`
- `permissions(resource_id, principal_type, principal_id, role, inherited)`
- `sync_events(event_id, type, file_id, version_id, timestamp)`
- `chunks(chunk_hash, object_key, size) if chunk-level dedupe`

### High-Level Architecture
- Client starts upload session and uploads parts directly to object storage.
- Metadata service creates file/version records after completion.
- Permission service authorizes metadata and signed download URL issuance.
- Sync service stores change events and exposes cursor-based delta sync.
- Search indexer extracts metadata/content asynchronously.

### Deep-Dive Topics

#### 7.1 Metadata vs blob storage
- **Definition:** Keep metadata in DB and file bytes in object storage.
- **Example:** files table stores name/owner; S3 stores bytes.
- **Use in production:** DB handles queries/permissions; object storage handles durability/large blobs.
- **Interview question:** Why not store files in DB?
- **Strong answer:** Large blobs bloat DB and backups; object storage is cheaper and optimized.

#### 7.2 Multipart upload
- **Definition:** Upload large files in chunks with resume.
- **Example:** 5GB file uploaded as 100MB parts; failed part retried.
- **Use in production:** Use upload sessions, part checksums, signed URLs.
- **Interview question:** Network failure?
- **Strong answer:** Retry failed parts and complete when all checksums verify.

#### 7.3 Deduplication
- **Definition:** Avoid storing duplicate content.
- **Example:** Two identical files share one blob reference.
- **Use in production:** Use content hash or chunk hash; manage privacy/reference-count/delete trade-offs.
- **Interview question:** Risks of dedupe?
- **Strong answer:** Existence leakage, deletion complexity, encryption conflicts.

#### 7.4 Permissions
- **Definition:** Who can view/edit/share.
- **Example:** Folder shared with team gives inherited access.
- **Use in production:** Check before signed URL issuance; cache ACLs carefully and invalidate.
- **Interview question:** Revoke shared link?
- **Strong answer:** Revoke ACL/link token and use short-lived signed URLs.

#### 7.5 Folder hierarchy
- **Definition:** Parent-child structure for files/folders.
- **Example:** file.parent_id points to folder_id.
- **Use in production:** Adjacency list is simple; materialized path helps fast subtree queries.
- **Interview question:** How list folder quickly?
- **Strong answer:** Index by parent_id and paginate with cursor.

#### 7.6 Delta sync
- **Definition:** Return changes since last cursor.
- **Example:** Client sends cursor=987 and receives changes 988-1020.
- **Use in production:** Use append-only change log with permission filtering.
- **Interview question:** Why not scan all files?
- **Strong answer:** Too expensive; delta sync is scalable.

#### 7.7 Conflict resolution
- **Definition:** Handle simultaneous offline edits.
- **Example:** Laptop and phone edit same file offline.
- **Use in production:** Use base-version preconditions; merge text or create conflict copy for binaries.
- **Interview question:** Offline conflict?
- **Strong answer:** Preserve both versions or merge with OT/CRDT for collaborative text.

#### 7.8 Versioning/trash
- **Definition:** Keep old versions/deleted items for restore.
- **Example:** Restore file from last Tuesday.
- **Use in production:** Immutable versions, soft delete, retention lifecycle.
- **Interview question:** Why soft delete?
- **Strong answer:** Supports restore and sync propagation before hard deletion.

### Common Interview Questions
- **Q: Design Dropbox.**
  - **A:** Object storage for bytes, metadata DB, multipart upload, permissions, sync change log, versions.
- **Q: How do clients know changes?**
  - **A:** Cursor-based delta sync over append-only change log.
- **Q: How ensure secure sharing?**
  - **A:** Authorize before signed URL issuance and use short-lived URLs plus ACL invalidation.

### Common Pitfalls
- Serving file bytes through app servers unnecessarily.
- No permission revocation story.
- No sync cursor/change log.
- Ignoring conflict copies/versioning.

### 60-Second Interview Summary

For **Google Drive / Dropbox / File Storage and Sync**, first clarify scope and correctness requirements. The core architecture is: Client starts upload session and uploads parts directly to object storage. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Metadata vs blob storage**, **Multipart upload**, **Deduplication**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="collaborative-editor"></a>
## 8. Google Docs / Real-Time Collaborative Editor

**What this interview tests:** real-time collaboration, OT/CRDT, operation logs, snapshots, WebSockets, permissions, presence, and offline editing.

### Functional Requirements
- Multiple users edit same document concurrently.
- Users see remote updates/cursors in real time.
- Support comments/suggestions if in scope.
- Support permissions and sharing.
- Support version history and reconnect catch-up.

### Non-Functional Requirements
- Low-latency edit propagation.
- Convergence: clients eventually see same document.
- Acknowledged edits are durable.
- Authorization must be strong.
- Presence can be best-effort.

### Core APIs
```http
GET /v1/docs/{id} -> snapshot + version
WebSocket op.submit {doc_id, base_version, operation}
WebSocket op.ack / op.broadcast
GET /v1/docs/{id}/ops?after_version=
POST /v1/docs/{id}/comments
```

### Data Model / Source of Truth
- `documents(doc_id, owner_id, title, current_snapshot_id, current_version)`
- `doc_snapshots(snapshot_id, doc_id, version, content_blob_ref)`
- `doc_ops(doc_id, version, actor_id, operation, created_at)`
- `doc_permissions(doc_id, principal_id, role)`
- `comments(comment_id, doc_id, anchor_range, body, status)`

### High-Level Architecture
- Client loads latest snapshot and current version.
- Client sends edit operations over WebSocket.
- Collaboration service validates permission, transforms/merges operation, assigns version, appends op log, broadcasts update.
- Snapshot service periodically compacts operation logs into snapshots.
- Presence service handles cursor/typing via ephemeral pub-sub.

### Deep-Dive Topics

#### 8.1 Operational Transform
- **Definition:** Transforms concurrent operations to preserve intent.
- **Example:** Alice inserts at pos 5 while Bob deletes at pos 3; positions shift.
- **Use in production:** Efficient but complex; requires transformation logic per operation type.
- **Interview question:** What problem does OT solve?
- **Strong answer:** Concurrent edits converge despite different arrival orders.

#### 8.2 CRDT
- **Definition:** Data type whose operations merge without central conflict transform.
- **Example:** Each inserted character has unique position ID.
- **Use in production:** Good for offline-first but may have metadata overhead and compaction needs.
- **Interview question:** OT vs CRDT?
- **Strong answer:** OT is central-transform efficient but complex; CRDT merges naturally but can be heavier.

#### 8.3 Operation log
- **Definition:** Append-only sequence of document edits.
- **Example:** version 101 = insert text at position 20.
- **Use in production:** Supports replay, sync, audit, history, and recovery.
- **Interview question:** Why store ops?
- **Strong answer:** Needed for real-time sync/history; snapshots reduce replay cost.

#### 8.4 Snapshots
- **Definition:** Periodic full document states.
- **Example:** Snapshot every 500 ops or 5 minutes.
- **Use in production:** Store immutable snapshots and replay only recent ops.
- **Interview question:** How load large old doc?
- **Strong answer:** Load latest snapshot and replay ops after it.

#### 8.5 WebSocket sessions
- **Definition:** Persistent collaboration channels.
- **Example:** Five editors receive accepted ops in real time.
- **Use in production:** Route document session to shard or use pub-sub; handle missed versions.
- **Interview question:** Reconnect catch-up?
- **Strong answer:** Client sends last version; server returns missed ops or newer snapshot.

#### 8.6 Presence/cursors
- **Definition:** Ephemeral online/cursor state.
- **Example:** Bob’s cursor appears in paragraph 3.
- **Use in production:** Use TTL pub-sub; do not persist every movement.
- **Interview question:** Durable cursors?
- **Strong answer:** No, presence is best-effort.

#### 8.7 Permissions
- **Definition:** Roles for view/comment/suggest/edit.
- **Example:** Commenter cannot submit edit ops.
- **Use in production:** Check when opening session and on ops; revoke active sessions if permission changes.
- **Interview question:** If access revoked mid-session?
- **Strong answer:** Reject future ops and close/invalidate session.

#### 8.8 Offline editing
- **Definition:** Local edits while disconnected, merged later.
- **Example:** User edits on airplane mode and syncs after reconnect.
- **Use in production:** Use local op queue and OT/CRDT merge; fall back to conflict branch/copy if needed.
- **Interview question:** Offline concurrent edits?
- **Strong answer:** Queue ops with base version and merge on reconnect.

### Common Interview Questions
- **Q: Design Google Docs.**
  - **A:** Use snapshot+op log, collaboration WebSocket service, OT/CRDT, permission checks, and presence.
- **Q: How ensure convergence?**
  - **A:** Versioned operations with OT/CRDT; all clients apply accepted operations in consistent order.
- **Q: Durable vs ephemeral?**
  - **A:** Ops/snapshots/comments durable; cursors/presence ephemeral.

### Common Pitfalls
- Using last-write-wins for collaborative text.
- No reconnect/catch-up protocol.
- Persisting every cursor movement.
- No snapshot/compaction story.

### 60-Second Interview Summary

For **Google Docs / Real-Time Collaborative Editor**, first clarify scope and correctness requirements. The core architecture is: Client loads latest snapshot and current version. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Operational Transform**, **CRDT**, **Operation log**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="ecommerce"></a>
## 9. Amazon / Flipkart / E-Commerce Marketplace

**What this interview tests:** catalog/search, inventory, cart/checkout, order state machine, payment saga, flash sale, reviews, and recommendations.

### Functional Requirements
- Browse/search products and product detail pages.
- Add items to cart and checkout.
- Validate inventory, price, address, and payment.
- Create order and handle shipping/cancel/refund.
- Support reviews, recommendations, sellers, and promotions if in scope.

### Non-Functional Requirements
- Catalog reads highly available and cacheable.
- Inventory/order/payment correctness is critical.
- Search/recommendations can be eventually consistent.
- Checkout/payment APIs must be idempotent.
- Flash sales require traffic shaping.

### Core APIs
```http
GET /v1/products/{id}
GET /v1/search?q=&filters=&cursor=
POST /v1/cart/items {sku_id, qty}
POST /v1/checkout {cart_id, address_id, payment_method}
GET /v1/orders/{id}
POST /v1/orders/{id}/cancel
```

### Data Model / Source of Truth
- `products(product_id, title, brand, description, status)`
- `skus(sku_id, product_id, attributes, price, seller_id)`
- `inventory(sku_id, available_qty, reserved_qty, version)`
- `carts(cart_id, user_id, status)`
- `cart_items(cart_id, sku_id, qty, price_snapshot)`
- `orders(order_id, user_id, status, total)`
- `order_items(order_id, sku_id, qty, price_snapshot)`
- `payments(payment_id, order_id, amount, status, provider_ref)`

### High-Level Architecture
- Catalog service serves product pages from DB/cache/search index.
- Cart service stores mutable user cart.
- Checkout orchestrator validates cart, reserves inventory, creates order, and initiates payment.
- Payment/order/inventory workflow uses saga with compensation.
- Search/recommendation/review aggregates update asynchronously.

### Deep-Dive Topics

#### 9.1 Product vs SKU
- **Definition:** Product is conceptual item; SKU is purchasable variant.
- **Example:** iPhone product has color/storage SKUs.
- **Use in production:** Separate for inventory/pricing/attributes per variant.
- **Interview question:** Why separate product and SKU?
- **Strong answer:** Because variants have independent price/inventory.

#### 9.2 Inventory reservation
- **Definition:** Temporarily holds stock during checkout.
- **Example:** Reserve one unit for 10 minutes while user pays.
- **Use in production:** Use atomic conditional update, TTL, release job, and idempotency.
- **Interview question:** Prevent overselling?
- **Strong answer:** Update only if available_qty >= requested, using lock/version/CAS.

#### 9.3 Cart vs order
- **Definition:** Cart is mutable; order is immutable/auditable.
- **Example:** Cart quantity changes; order price snapshot is fixed.
- **Use in production:** Revalidate price/inventory at checkout and store snapshots.
- **Interview question:** Trust cart price?
- **Strong answer:** No; checkout must revalidate or explicitly honor a reserved price.

#### 9.4 Order state machine
- **Definition:** Controlled order lifecycle.
- **Example:** pending -> paid -> shipped -> delivered -> refunded.
- **Use in production:** Enforce valid transitions and store events.
- **Interview question:** Why state machine?
- **Strong answer:** Prevents invalid payment/shipping/refund bugs.

#### 9.5 Payment saga
- **Definition:** Distributed workflow with compensating actions.
- **Example:** Reserve inventory -> authorize payment -> create order; release if payment fails.
- **Use in production:** Use saga orchestrator/choreography, idempotency, retries, compensation events.
- **Interview question:** Why not distributed transaction?
- **Strong answer:** Payment gateway cannot join DB transaction; saga handles distributed reality.

#### 9.6 Search/facets
- **Definition:** Full-text and filtered product retrieval.
- **Example:** Search shoes by size, price, brand.
- **Use in production:** Use search index with denormalized product docs and async updates.
- **Interview question:** Why not relational DB?
- **Strong answer:** Search relevance/facets/typo tolerance scale better in search engines.

#### 9.7 Flash sale
- **Definition:** Survive extreme demand for limited inventory.
- **Example:** 10 lakh users buy 10k phones at sale start.
- **Use in production:** Use waiting room, queue tokens, cached product pages, atomic reservation.
- **Interview question:** Design flash sale?
- **Strong answer:** Admit users gradually and protect inventory with atomic reservation.

#### 9.8 Reviews/ratings
- **Definition:** User-generated feedback and aggregate rating.
- **Example:** 4.3 rating from verified buyers.
- **Use in production:** Moderate, verify purchase, aggregate async, fight fake reviews.
- **Interview question:** Update rating aggregate?
- **Strong answer:** Reviews are source; aggregate can update async or transactionally depending on scale.

### Common Interview Questions
- **Q: Design Amazon checkout.**
  - **A:** Cart validation, inventory reservation, order state machine, payment saga, idempotency, compensation.
- **Q: Prevent overselling?**
  - **A:** Atomic inventory reservation plus admission control during spikes.
- **Q: What can be eventually consistent?**
  - **A:** Search index, recommendations, reviews, notifications; not payment/order/inventory.

### Common Pitfalls
- No inventory reservation TTL.
- Not distinguishing cart from order.
- No idempotency in checkout/payment.
- Letting all flash-sale traffic hit inventory DB.

### 60-Second Interview Summary

For **Amazon / Flipkart / E-Commerce Marketplace**, first clarify scope and correctness requirements. The core architecture is: Catalog service serves product pages from DB/cache/search index. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Product vs SKU**, **Inventory reservation**, **Cart vs order**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="payment-system"></a>
## 10. Payment System / Stripe / UPI-like Payments

**What this interview tests:** idempotency, transaction state, ledger, external providers, webhooks, reconciliation, fraud, refunds, and auditability.

### Functional Requirements
- Create payment intent/transaction.
- Authorize, capture, settle, refund depending on flow.
- Handle provider callbacks/webhooks.
- Expose payment status to merchants/users.
- Maintain ledger and reconciliation reports.

### Non-Functional Requirements
- No double charge or lost money movement.
- Correctness over raw latency.
- Immutable audit trail.
- Secure handling of payment data.
- Recover from provider timeouts/duplicate callbacks.

### Core APIs
```http
POST /v1/payment-intents {amount,currency,order_id} with Idempotency-Key
POST /v1/payment-intents/{id}/confirm
POST /v1/refunds {payment_id, amount}
POST /v1/webhooks/provider
GET /v1/payments/{id}
```

### Data Model / Source of Truth
- `payment_intents(intent_id, order_id, amount, currency, status, idempotency_key)`
- `payment_attempts(attempt_id, intent_id, provider, provider_ref, status)`
- `ledger_entries(entry_id, account_id, direction, amount, currency, transaction_ref) immutable`
- `webhook_events(provider_event_id, provider, payload_hash, processed_at)`
- `refunds(refund_id, payment_id, amount, status, provider_ref)`

### High-Level Architecture
- Client/merchant creates payment intent with idempotency key.
- Payment service validates state and calls external provider.
- Provider response/webhook updates payment attempt state.
- Ledger service writes immutable balanced entries when money movement is confirmed.
- Reconciliation jobs compare internal ledger with provider settlement reports.

### Deep-Dive Topics

#### 10.1 Idempotency keys
- **Definition:** Repeated request returns same outcome without repeated side effect.
- **Example:** Retry confirm after timeout should not charge twice.
- **Use in production:** Store key+request hash+response; reject mismatched reuse.
- **Interview question:** Prevent double charge?
- **Strong answer:** Idempotency key, unique provider refs, and state checks.

#### 10.2 Payment state machine
- **Definition:** Lifecycle of payment intent/attempt.
- **Example:** created -> authorized -> captured -> settled; failed/refunded/disputed states.
- **Use in production:** Validate transitions and handle async provider state.
- **Interview question:** Why state machine?
- **Strong answer:** Prevents illegal money movement transitions.

#### 10.3 Ledger
- **Definition:** Immutable double-entry money movement record.
- **Example:** Debit buyer, credit merchant payable.
- **Use in production:** Append-only balanced entries; corrections through reversal entries.
- **Interview question:** Why not status only?
- **Strong answer:** Ledger enables audit, settlement, refunds, reconciliation.

#### 10.4 Gateway timeout
- **Definition:** Provider call outcome is unknown.
- **Example:** Gateway times out after possibly charging card.
- **Use in production:** Mark pending/unknown; query provider or wait webhook; reconcile later.
- **Interview question:** Retry timeout blindly?
- **Strong answer:** No, retry only with provider idempotency or after status check.

#### 10.5 Webhooks
- **Definition:** Provider asynchronous status notifications.
- **Example:** payment_succeeded after 3DS authentication.
- **Use in production:** Verify signature, store raw event, dedupe provider_event_id, handle out-of-order events.
- **Interview question:** Duplicate webhook?
- **Strong answer:** Unique event ID and idempotent processing.

#### 10.6 Reconciliation
- **Definition:** Compare internal records with provider/bank files.
- **Example:** Provider settlement differs due to fees/reversals.
- **Use in production:** Daily jobs detect mismatches and create adjustments.
- **Interview question:** Why needed if webhooks exist?
- **Strong answer:** Webhooks can be missed/delayed/duplicated; settlement proves actual money movement.

#### 10.7 Fraud/risk
- **Definition:** Detect suspicious payments.
- **Example:** High velocity from same card/IP/device.
- **Use in production:** Use velocity limits, device fingerprinting, risk scoring, step-up auth, review queues.
- **Interview question:** Where run fraud checks?
- **Strong answer:** Before authorization for blocking and after for monitoring/disputes.

#### 10.8 Refunds/disputes
- **Definition:** Reverse or contest previous payments.
- **Example:** Partial refund of ₹300.
- **Use in production:** Use refund state machine, idempotency, ledger reversal entries.
- **Interview question:** Refund ledger?
- **Strong answer:** New reversal entries, never mutate original ledger.

### Common Interview Questions
- **Q: Design Stripe payments API.**
  - **A:** Payment intents, idempotency, gateway integration, webhooks, immutable ledger, refunds, reconciliation.
- **Q: What if user refreshes checkout?**
  - **A:** Reuse same order/payment intent and idempotency key; return existing status.
- **Q: What should never be mutable?**
  - **A:** Ledger entries and raw provider events.

### Common Pitfalls
- No idempotency keys.
- Mutating ledger entries.
- Trusting unsigned webhooks.
- Blindly retrying gateway timeout.

### 60-Second Interview Summary

For **Payment System / Stripe / UPI-like Payments**, first clarify scope and correctness requirements. The core architecture is: Client/merchant creates payment intent with idempotency key. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Idempotency keys**, **Payment state machine**, **Ledger**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="search-autocomplete"></a>
## 11. Search Engine / Search Autocomplete

**What this interview tests:** inverted indexes, indexing pipeline, ranking, autocomplete, freshness, typo tolerance, facets, pagination, and feedback loops.

### Functional Requirements
- Search documents/products/posts/pages.
- Return ranked results with snippets and filters.
- Suggest autocomplete while typing.
- Support typo tolerance/spell correction.
- Index new/updated/deleted documents.

### Non-Functional Requirements
- Low query and autocomplete latency.
- High relevance and acceptable freshness.
- Indexing can be eventually consistent.
- Scalable query fanout across shards.
- Graceful fallback if ranking/indexing partially fails.

### Core APIs
```http
GET /v1/search?q=&filters=&cursor=
GET /v1/suggest?prefix=
POST /v1/index/documents {id, fields}
DELETE /v1/index/documents/{id}
```

### Data Model / Source of Truth
- `documents(doc_id, fields, status, updated_at)`
- `inverted_index(term -> postings(doc_id, tf, positions))`
- `suggest_index(prefix/FST/trie -> suggestions with score)`
- `query_logs(query, user_hash, clicked_doc, timestamp)`

### High-Level Architecture
- Ingestion normalizes/tokenizes documents and sends to index builder.
- Index builder creates immutable segments and periodically merges.
- Query service parses query, fetches postings from shards, ranks candidates, and returns snippets.
- Suggest service uses precomputed trie/FST and popular query data.
- Logs feed ranking and suggestion improvements.

### Deep-Dive Topics

#### 11.1 Inverted index
- **Definition:** Map terms to documents containing them.
- **Example:** redis -> doc IDs with positions/frequencies.
- **Use in production:** Compressed postings, segments, shards, replicas.
- **Interview question:** Why inverted index?
- **Strong answer:** Avoids scanning every document for query terms.

#### 11.2 Ranking
- **Definition:** Order candidate results by relevance.
- **Example:** BM25 + freshness + popularity + personalization.
- **Use in production:** Multi-stage retrieve -> score -> rerank top N.
- **Interview question:** Ranking signals?
- **Strong answer:** Text relevance, proximity, fields, freshness, clicks, quality.

#### 11.3 Autocomplete
- **Definition:** Suggest queries/entities from prefix.
- **Example:** insta -> instagram, instagram reels.
- **Use in production:** Use trie/FST/prefix index with popularity and safety filters.
- **Interview question:** How differ from search?
- **Strong answer:** Autocomplete is prefix+low-latency per keystroke; search ranks full query results.

#### 11.4 Fresh indexing
- **Definition:** How quickly changes appear.
- **Example:** New tweet searchable within seconds.
- **Use in production:** Fresh small index plus larger immutable segment merges.
- **Interview question:** How make new content searchable?
- **Strong answer:** Near-real-time indexing path for recent docs.

#### 11.5 Spell correction
- **Definition:** Handle typos and approximate matches.
- **Example:** iphnoe -> iphone.
- **Use in production:** Use edit distance, FST, query logs, dictionaries, popularity.
- **Interview question:** How implement typo tolerance?
- **Strong answer:** Generate edit-distance candidates and score by context/popularity.

#### 11.6 Facets
- **Definition:** Filter/count results by attributes.
- **Example:** Brand/size/price filters in ecommerce.
- **Use in production:** Index structured fields and aggregate across matching results.
- **Interview question:** Why facets expensive?
- **Strong answer:** Need counts across result set/shards.

#### 11.7 Pagination
- **Definition:** Return results page by page.
- **Example:** cursor contains last score/doc_id.
- **Use in production:** Use search_after/cursor; avoid deep offset.
- **Interview question:** Why offset bad?
- **Strong answer:** Engine must score/skip many results.

#### 11.8 Query logs
- **Definition:** Capture impressions/clicks/conversions.
- **Example:** Users searching airpods often click product X.
- **Use in production:** Use anonymized logs, A/B tests, bias correction.
- **Interview question:** Evaluate search?
- **Strong answer:** Offline relevance, CTR/conversion, latency, A/B tests.

### Common Interview Questions
- **Q: Design autocomplete.**
  - **A:** Trie/FST/prefix index, hot prefix cache, popularity ranking, async updates.
- **Q: Design search engine.**
  - **A:** Ingestion, inverted index, query parser, ranking, snippets, shards, replicas, logs.
- **Q: Handle deletes?**
  - **A:** Tombstones immediately in query path; physical removal during merge.

### Common Pitfalls
- Scanning DB rows for search.
- No delete/freshness story.
- Offset pagination for deep results.
- No ranking evaluation loop.

### 60-Second Interview Summary

For **Search Engine / Search Autocomplete**, first clarify scope and correctness requirements. The core architecture is: Ingestion normalizes/tokenizes documents and sends to index builder. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Inverted index**, **Ranking**, **Autocomplete**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="notification-system"></a>
## 12. Notification System / Push / Email / SMS

**What this interview tests:** event-driven fanout, preferences, templates, scheduling, retries, dedupe, provider limits, and observability.

### Functional Requirements
- Send notifications from product events.
- Support channels: in-app, push, email, SMS.
- Respect preferences, quiet hours, unsubscribe rules.
- Support immediate, scheduled, and digest notifications.
- Track delivery/open/click status when available.

### Non-Functional Requirements
- Critical notifications reliable and timely.
- External provider failures isolated.
- At-least-once with dedupe to avoid spam.
- Provider rate limits respected.
- Auditable sending decisions.

### Core APIs
```http
POST /v1/notifications/send {user_id,type,payload,idempotency_key}
POST /v1/notifications/schedule {send_at,audience,template_id}
GET /v1/notifications/inbox?cursor=
PATCH /v1/preferences {channel,type,enabled}
POST /v1/provider-webhooks/{provider}
```

### Data Model / Source of Truth
- `notification_events(event_id, source, type, user_id, payload)`
- `notification_jobs(job_id, user_id, channel, template_id, status, priority, send_at)`
- `user_preferences(user_id, notification_type, channel, enabled, quiet_hours)`
- `templates(template_id, channel, locale, version, body)`
- `delivery_attempts(attempt_id, job_id, provider, status, error)`
- `in_app_notifications(id, user_id, title, body, read_at)`

### High-Level Architecture
- Product services emit notification events to queue/log.
- Orchestrator validates event, checks preferences, dedupes, renders templates.
- Channel workers send via push/email/SMS providers with rate limits and retries.
- Provider webhooks update delivery state.
- In-app channel writes inbox records; analytics tracks open/click.

### Deep-Dive Topics

#### 12.1 Async event fanout
- **Definition:** Product event becomes one or more notification jobs.
- **Example:** OrderShipped creates push and email jobs.
- **Use in production:** Core services emit events and do not wait for providers.
- **Interview question:** Why async?
- **Strong answer:** Provider latency/failure should not affect core transaction.

#### 12.2 Preferences
- **Definition:** User/channel/type rules.
- **Example:** Marketing email off, payment SMS on.
- **Use in production:** Check before job creation/send; transactional rules may differ legally.
- **Interview question:** Cache preferences?
- **Strong answer:** Yes with short TTL/invalidation; compliance-sensitive checks need freshness.

#### 12.3 Templates
- **Definition:** Reusable localized message formats.
- **Example:** Your order {{id}} shipped in English/Hindi.
- **Use in production:** Version templates, validate variables, preview campaigns.
- **Interview question:** Why version?
- **Strong answer:** Auditability and stable rendering for old jobs.

#### 12.4 Deduplication
- **Definition:** Avoid repeated notifications.
- **Example:** Duplicate OrderShipped should not send two SMS.
- **Use in production:** Use deterministic dedupe keys and unique constraints.
- **Interview question:** Avoid duplicates?
- **Strong answer:** Idempotency key per logical event/user/channel/window.

#### 12.5 Retries/DLQ
- **Definition:** Retry transient failures and isolate poison jobs.
- **Example:** Email provider 500 retries; invalid phone fails.
- **Use in production:** Exponential backoff, retry caps, DLQ, alerts.
- **Interview question:** Provider down?
- **Strong answer:** Queue backlog, retry, failover provider, degrade non-critical.

#### 12.6 Provider routing/rate limits
- **Definition:** Choose vendor and throttle sends.
- **Example:** SMS provider A limited to 100/sec.
- **Use in production:** Token buckets per provider/channel, health routing, cost routing.
- **Interview question:** Per-second provider limit?
- **Strong answer:** Throttle worker concurrency and queue excess by priority.

#### 12.7 Scheduling/digests
- **Definition:** Future sends or batched summaries.
- **Example:** Daily digest of 20 likes.
- **Use in production:** Time-zone aware scheduler, aggregation windows, cancellation support.
- **Interview question:** Implement digest?
- **Strong answer:** Aggregate events per user and render one job at scheduled time.

#### 12.8 Observability
- **Definition:** Track queue/send/delivery/open/failure.
- **Example:** Dashboard shows push failure spike.
- **Use in production:** Metrics: queue lag, send latency, provider errors, DLQ, 429/opt-out.
- **Interview question:** What metrics?
- **Strong answer:** Queue lag, success rate, retries, provider latency/errors, dedupe rate.

### Common Interview Questions
- **Q: Design notification service.**
  - **A:** Event queue, orchestrator, preferences, templates, dedupe, per-channel workers, retries/DLQ, provider webhooks.
- **Q: Avoid notification storms?**
  - **A:** Dedupe, user rate limits, batching/digest, suppress low-value high-volume events.
- **Q: What should be synchronous?**
  - **A:** Usually event creation/in-app record only; external sends are async.

### Common Pitfalls
- Calling providers synchronously from core services.
- No preference/unsubscribe handling.
- No dedupe/rate limits.
- No DLQ or provider visibility.

### 60-Second Interview Summary

For **Notification System / Push / Email / SMS**, first clarify scope and correctness requirements. The core architecture is: Product services emit notification events to queue/log. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Async event fanout**, **Preferences**, **Templates**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="ticket-booking"></a>
## 13. Ticketmaster / BookMyShow / Seat Booking

**What this interview tests:** seat inventory consistency, temporary holds, high concurrency, waiting room, payment timeout, real-time seat map, and fairness.

### Functional Requirements
- Browse events/shows and available seats.
- Select seats and hold temporarily.
- Complete payment before hold expires.
- Confirm booking or release seats on timeout/failure.
- Support waiting room for high-demand events.

### Non-Functional Requirements
- Never double-book same seat.
- Seat map can be slightly stale.
- Hold/booking state must be strongly consistent.
- Handle traffic spikes fairly.
- Bot prevention and purchase limits.

### Core APIs
```http
GET /v1/events/{id}/seats
POST /v1/events/{id}/holds {seat_ids} -> hold_id, expires_at
POST /v1/holds/{id}/checkout {payment_method}
GET /v1/bookings/{id}
POST /v1/holds/{id}/release
```

### Data Model / Source of Truth
- `events(event_id, venue_id, start_time, status)`
- `seats(seat_id, venue_id, section, row, number)`
- `event_seats(event_id, seat_id, status, hold_id, version)`
- `holds(hold_id, user_id, event_id, status, expires_at)`
- `bookings(booking_id, user_id, event_id, status, total)`
- `payments(payment_id, booking_id/hold_id, status, provider_ref)`

### High-Level Architecture
- Event service serves cached event/seat map metadata.
- Seat inventory service is source of truth for available/held/booked states.
- Hold API atomically changes seats from available to held with TTL.
- Checkout validates active hold, handles payment, and converts seats to booked.
- Expiration worker releases stale holds.
- Waiting room controls admission during spikes.

### Deep-Dive Topics

#### 13.1 Seat hold
- **Definition:** Temporary reservation during payment.
- **Example:** Seat A10 held for 5 minutes.
- **Use in production:** Atomic available->held update with TTL and release job.
- **Interview question:** Why hold?
- **Strong answer:** Prevents two users paying for same seat.

#### 13.2 Double-book prevention
- **Definition:** Guarantee one confirmed booking per seat.
- **Example:** Two users click A10; only one hold succeeds.
- **Use in production:** Use row lock, version CAS, unique constraint, or per-event partition.
- **Interview question:** How prevent double booking?
- **Strong answer:** Conditional update status=held where status=available.

#### 13.3 Hold expiration
- **Definition:** Release seats if payment not completed.
- **Example:** Hold expires at 7:05 PM.
- **Use in production:** Scheduled expiry queue/scanner; checkout verifies hold active.
- **Interview question:** Payment succeeds after expiry?
- **Strong answer:** Fail booking and refund/void; prefer auth before capture.

#### 13.4 Waiting room
- **Definition:** Admission control under high demand.
- **Example:** 100K users queue, 2K admitted/min.
- **Use in production:** Signed queue tokens, bot checks, gradual admission.
- **Interview question:** Why waiting room?
- **Strong answer:** Protects inventory/payment and improves fairness.

#### 13.5 Realtime seat map
- **Definition:** Show availability changes.
- **Example:** Seat turns grey when held.
- **Use in production:** Use WebSocket/SSE/polling for deltas; final hold checks source of truth.
- **Interview question:** Can seat map be stale?
- **Strong answer:** Yes; hold transaction is authoritative.

#### 13.6 Payment integration
- **Definition:** Charge user and confirm booking.
- **Example:** Authorize payment, convert hold to booked, capture.
- **Use in production:** Use idempotency, timeouts, refund/void compensation.
- **Interview question:** Payment failure?
- **Strong answer:** Do not book; release/expire hold and mark payment failed.

#### 13.7 Event partitioning
- **Definition:** Shard inventory operations by event/show.
- **Example:** Concert X routes to partition X.
- **Use in production:** Natural shard key is event_id; hot sections may subdivide.
- **Interview question:** Shard key?
- **Strong answer:** event_id because seat operations are scoped to event.

#### 13.8 Anti-bot/fairness
- **Definition:** Prevent bulk automated hoarding.
- **Example:** Limit holds per account/device/card.
- **Use in production:** Rate limits, CAPTCHA, device fingerprint, queue token, purchase limits.
- **Interview question:** Stop many holds?
- **Strong answer:** Per-user/session/payment limits and fast expiry.

### Common Interview Questions
- **Q: Design BookMyShow.**
  - **A:** Event metadata, seat map, atomic holds, TTL release, checkout/payment, waiting room, realtime updates.
- **Q: What must be strongly consistent?**
  - **A:** Hold and booking transitions; display can be stale.
- **Q: Handle flash sale load?**
  - **A:** Waiting room, cached reads, atomic inventory operations, and admission control.

### Common Pitfalls
- No temporary hold before payment.
- Trusting stale seat map for booking.
- No timeout/release flow.
- No queue/admission control for hot events.

### 60-Second Interview Summary

For **Ticketmaster / BookMyShow / Seat Booking**, first clarify scope and correctness requirements. The core architecture is: Event service serves cached event/seat map metadata. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Seat hold**, **Double-book prevention**, **Hold expiration**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="food-delivery"></a>
## 14. Swiggy / Zomato / DoorDash Food Delivery

**What this interview tests:** restaurant discovery, menu validation, order state, dispatch, courier tracking, batching, ETA, payment/refund, and notifications.

### Functional Requirements
- Browse nearby restaurants and menus.
- Place order and pay.
- Restaurant accepts/rejects and prepares order.
- Delivery partner assigned and tracked.
- Support cancellation/refund, ratings, and notifications.

### Non-Functional Requirements
- Low-latency local discovery.
- Correct order/payment state.
- Tracking can be eventually consistent.
- Handle lunch/dinner spikes.
- Marketplace balancing between users/restaurants/couriers.

### Core APIs
```http
GET /v1/restaurants?lat=&lng=&filters=
GET /v1/restaurants/{id}/menu
POST /v1/orders {restaurant_id, items, address, payment_method}
POST /v1/orders/{id}/restaurant-action {accept/reject}
POST /v1/delivery/location
GET /v1/orders/{id}/track
```

### Data Model / Source of Truth
- `restaurants(restaurant_id, location, status, cuisine, rating)`
- `menu_items(item_id, restaurant_id, name, price, availability)`
- `orders(order_id, user_id, restaurant_id, status, total)`
- `order_items(order_id, item_id, qty, price_snapshot)`
- `delivery_partners(partner_id, status, current_location, current_order_id)`
- `assignments(order_id, partner_id, status)`
- `order_events(event_id, order_id, type, actor, timestamp)`

### High-Level Architecture
- Discovery service uses geo index and caches nearby restaurants/menus.
- Order service validates menu price/availability and creates order/payment.
- Restaurant app receives order and returns accept/reject/prep status.
- Dispatch service assigns courier based on location, readiness, ETA, and batching.
- Tracking service streams courier location; notification service sends updates.

### Deep-Dive Topics

#### 14.1 Restaurant discovery
- **Definition:** Find restaurants serviceable near user.
- **Example:** Show open restaurants within delivery radius.
- **Use in production:** Use geospatial index, serviceability polygons, availability cache.
- **Interview question:** How find nearby restaurants?
- **Strong answer:** Geo cells/spatial query, then filter by open/radius/capacity.

#### 14.2 Menu validation
- **Definition:** Ensure ordered items/prices are valid.
- **Example:** Item price changes after browsing.
- **Use in production:** Cache menus for reads but revalidate at order creation; store price snapshot.
- **Interview question:** Trust cached menu?
- **Strong answer:** No, checkout must revalidate.

#### 14.3 Order state machine
- **Definition:** Lifecycle from placed to delivered/cancelled.
- **Example:** placed -> accepted -> preparing -> picked_up -> delivered.
- **Use in production:** Store order_events and enforce valid actor transitions.
- **Interview question:** Why events?
- **Strong answer:** Audit, support, notifications, rebuildable status.

#### 14.4 Restaurant timeout
- **Definition:** Handle no accept/reject response.
- **Example:** No accept in 2 min -> cancel/refund.
- **Use in production:** Timers/scheduled jobs and compensation.
- **Interview question:** Reject after payment?
- **Strong answer:** Cancel, refund/wallet credit, notify user.

#### 14.5 Courier assignment
- **Definition:** Match delivery partner to order.
- **Example:** Partner near restaurant and available near ready time.
- **Use in production:** Use geo, ETA, capacity, batching, acceptance probability.
- **Interview question:** Difference from Uber?
- **Strong answer:** Food has pickup+drop, prep timing, batching/freshness constraints.

#### 14.6 Batching
- **Definition:** One courier handles multiple compatible orders.
- **Example:** Two same-restaurant nearby drops.
- **Use in production:** Constrain by route detour, freshness, SLA, ready time.
- **Interview question:** When batch?
- **Strong answer:** When detour and timing do not violate SLA/freshness.

#### 14.7 Realtime tracking
- **Definition:** Show courier location/ETA.
- **Example:** User sees rider on map after pickup.
- **Use in production:** Latest location in ephemeral store, throttled stream to active users.
- **Interview question:** Push every update?
- **Strong answer:** No; throttle/smooth and push only active tracking.

#### 14.8 ETA prediction
- **Definition:** Estimate prep + pickup + delivery time.
- **Example:** ETA = prep + courier-to-restaurant + drop route + buffer.
- **Use in production:** Use historical prep, restaurant load, traffic, courier availability.
- **Interview question:** Why hard?
- **Strong answer:** Depends on restaurant operations, traffic, batching, and courier availability.

### Common Interview Questions
- **Q: Design Swiggy/Zomato.**
  - **A:** Local discovery, menu validation, order/payment state, restaurant acceptance, dispatch, tracking, ETA, notifications.
- **Q: Source of truth for order status?**
  - **A:** Order service/order_events, not client UI.
- **Q: Handle meal-time spikes?**
  - **A:** Cache reads, regional scaling, queues for dispatch/notifications, protect checkout.

### Common Pitfalls
- Trusting cached menu at checkout.
- No restaurant rejection/refund flow.
- Treating courier assignment like nearest-driver only.
- Pushing every location update.

### 60-Second Interview Summary

For **Swiggy / Zomato / DoorDash Food Delivery**, first clarify scope and correctness requirements. The core architecture is: Discovery service uses geo index and caches nearby restaurants/menus. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Restaurant discovery**, **Menu validation**, **Order state machine**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

<a id="rate-limiter-api-gateway"></a>
## 15. Rate Limiter / API Gateway / Throttling System

**What this interview tests:** rate-limit algorithms, distributed counters, policy management, fail-open/closed trade-offs, low-latency decisions, and observability.

### Functional Requirements
- Limit requests by user, IP, API key, endpoint, tenant, or region.
- Support per-second/minute/day quotas and burst limits.
- Return allow/deny with remaining quota and Retry-After.
- Work across multiple gateway nodes.
- Expose metrics and policy configuration.

### Non-Functional Requirements
- Very low decision latency.
- High availability; limiter should not be global bottleneck.
- Accuracy appropriate to endpoint risk.
- Policy changes propagate quickly.
- Clear fail-open/fail-closed behavior.

### Core APIs
```http
POST /v1/limit/check {key, endpoint, cost} -> {allowed, remaining, retry_after}
PUT /v1/policies/{id}
GET /v1/usage/{key}
Gateway middleware enforces before backend routing
```

### Data Model / Source of Truth
- `rate_limit_policies(policy_id, key_pattern, algorithm, limit, window, burst)`
- `usage_counters(key, window_start, count) in Redis`
- `token_buckets(key, tokens, last_refill_at)`
- `decision_logs(request_id, key, policy_id, allowed, timestamp) sampled`

### High-Level Architecture
- API Gateway extracts identity key and endpoint.
- Policy service resolves applicable rate-limit policy.
- Limiter checks Redis/local counter atomically and returns decision.
- Allowed requests route to backend; denied requests return 429 Retry-After.
- Metrics/log pipeline records decisions and violations.

### Deep-Dive Topics

#### 15.1 Token bucket
- **Definition:** Tokens refill at fixed rate; requests consume tokens and bursts are allowed.
- **Example:** Bucket 100, refill 10/sec allows burst 100 then sustained 10/sec.
- **Use in production:** Good API default; implement with Redis atomic script or local buckets.
- **Interview question:** Why token bucket?
- **Strong answer:** Allows bursts while enforcing average rate.

#### 15.2 Fixed window
- **Definition:** Counts requests in discrete windows.
- **Example:** 100 requests from 10:00:00-10:00:59.
- **Use in production:** Simple but boundary burst problem.
- **Interview question:** Weakness?
- **Strong answer:** User can send 100 at end and 100 at start of next window.

#### 15.3 Sliding window
- **Definition:** Rate over rolling interval.
- **Example:** 100 requests in any last 60 seconds.
- **Use in production:** More accurate, costlier; approximate with buckets.
- **Interview question:** Log vs counter?
- **Strong answer:** Log accurate but memory-heavy; counter approximate and cheaper.

#### 15.4 Distributed counters
- **Definition:** Shared counts across gateway nodes.
- **Example:** API key X hits gateway A and B.
- **Use in production:** Redis atomic ops, sharded Redis, or local approximate quotas.
- **Interview question:** How share state?
- **Strong answer:** Central fast store or local quotas with eventual global enforcement.

#### 15.5 Key selection
- **Definition:** Dimension used for limiting.
- **Example:** Login limit by IP+username; API by API key.
- **Use in production:** Composite keys by abuse model and endpoint.
- **Interview question:** What key?
- **Strong answer:** User/API key for auth APIs; IP/device for anonymous; composite for login/OTP.

#### 15.6 Fail-open/closed
- **Definition:** Behavior when limiter dependency fails.
- **Example:** Product read may fail open; login may fail closed.
- **Use in production:** Choose by endpoint risk; use local fallback and alert.
- **Interview question:** Redis down?
- **Strong answer:** Local emergency limits and endpoint-specific fail-open/closed policy.

#### 15.7 Policy config
- **Definition:** Versioned rules for limits and bursts.
- **Example:** Free tier 1000/day; paid 100/sec.
- **Use in production:** Cache at gateways with TTL and pub/sub invalidation.
- **Interview question:** Policy updates?
- **Strong answer:** Gateways receive invalidation/update events.

#### 15.8 Observability
- **Definition:** Metrics/logs for rate limit behavior.
- **Example:** Top limited API keys and 429 rate by endpoint.
- **Use in production:** Monitor latency, allow/deny, Redis errors, top keys, policy misses.
- **Interview question:** What metrics?
- **Strong answer:** Decision latency, 429 rate, Redis latency/errors, top keys, backend reduction.

### Common Interview Questions
- **Q: Design distributed rate limiter.**
  - **A:** Clarify keys/policies, algorithm, Redis/local storage, gateway integration, failure behavior, metrics.
- **Q: Best algorithm?**
  - **A:** Token bucket is strong default; sliding window is more accurate but costlier.
- **Q: Where should it live?**
  - **A:** Gateway for early rejection plus service-level domain-specific limits.

### Common Pitfalls
- Only fixed window without boundary-burst discussion.
- No Redis/central-store failure plan.
- Only IP limiting for authenticated APIs.
- No Retry-After or metrics.

### 60-Second Interview Summary

For **Rate Limiter / API Gateway / Throttling System**, first clarify scope and correctness requirements. The core architecture is: API Gateway extracts identity key and endpoint. Then separate source-of-truth data from derived caches/indexes/rollups. The most valuable deep dives are **Token bucket**, **Fixed window**, **Sliding window**. Always explain what is strongly consistent, what is eventually consistent, and how retries, dedupe, cache staleness, and failures are handled.

[Back to Index](#index)

---

## Final Revision Checklist

- [ ] Can you state functional and non-functional requirements before designing?
- [ ] Can you identify source of truth vs cache/index/derived view?
- [ ] Can you explain strong vs eventual consistency for the system?
- [ ] Can you identify the hot path and remove slow work from it?
- [ ] Can you use queues/events for async side effects?
- [ ] Can you make retries idempotent?
- [ ] Can you handle cache invalidation/stale reads?
- [ ] Can you justify the shard key from query pattern?
- [ ] Can you define failure modes and degradation behavior?
- [ ] Can you defend trade-offs instead of only listing technologies?

## Suggested Memorization Order

1. URL Shortener — APIs, DB, cache, async analytics.
2. Rate Limiter — algorithms, distributed counters, gateway thinking.
3. Notification System — queues, retries, provider failures, preferences.
4. Instagram/Twitter — feed fanout, counters, social graph.
5. WhatsApp/Google Docs — realtime systems, ordering, convergence.
6. E-commerce/Ticket Booking/Payments — consistency, sagas, idempotency.
7. Uber/Food Delivery — geospatial matching and state machines.
8. YouTube/Drive/Search — media/files/indexing at scale.

## References and Further Practice

- Public system design prep resources commonly include prompts like URL shortener, Twitter/news feed, Uber, Ticketmaster, chat, notification service, video streaming, search autocomplete, payment processing, and file storage.
- Treat this report as an interview script. Real interviews reward scoped trade-offs more than memorized diagrams.