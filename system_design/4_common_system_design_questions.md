# 4 Most Common System Design Interview Questions

Based on research from top tech companies and interview platforms, these are the most frequently asked system design questions. Each includes a complete solution approach.

---

## Table of Contents
1. [Question 1: Design Twitter/X](#question-1-design-twitterx)
2. [Question 2: Design Instagram](#question-2-design-instagram)
3. [Question 3: Design YouTube](#question-3-design-youtube)
4. [Question 4: Design TinyURL (URL Shortener)](#question-4-design-tinyurl-url-shortener)
5. [Question 5: Design Uber (Ride-Hailing)](#question-5-design-uber-ride-hailing)
6. [Question 6: Design Netflix (Streaming)](#question-6-design-netflix-streaming)
7. [Question 7: Design WhatsApp (Messaging)](#question-7-design-whatsapp-messaging)
8. [Question 8: Design Slack (Team Chat)](#question-8-design-slack-team-chat)
9. [Question 9: Design Airbnb](#question-9-design-airbnb)
10. [Question 10: Design Rate Limiter](#question-10-design-rate-limiter)

---

# Question 1: Design Twitter/X

## Problem Statement
Design a social media platform like Twitter where users can:
- Post tweets (text, images, videos)
- Follow other users
- Like and retweet posts
- See a personalized feed of tweets from people they follow
- Search tweets
- Get trending topics

### Scale Requirements
```
Assumptions:
- 300 million monthly active users (MAU)
- 50 million daily active users (DAU)
- Each user posts ~2 tweets per day (average)
- Heavy read: Light write ratio (200:1)

Calculations:
Tweets per day = 50M DAU × 2 = 100M tweets/day
Tweets per second (TPS) = 100M / 86,400 ≈ 1,200 TPS

Feed reads per user per day = 50 (browsing activity)
Feed reads per day = 50M × 50 = 2.5B reads/day
Feed reads per second = 2.5B / 86,400 ≈ 29,000 RPS

Peak traffic (3x): ~87,000 RPS
```

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENTS                              │
│  Web  │  Mobile App  │  API Clients                     │
└────────────────┬────────────────────────────────────────┘
                 │
                 ↓
┌─────────────────────────────────────────────────────────┐
│         API GATEWAY / LOAD BALANCER                     │
│  • Rate limiting (max tweets/hour)                      │
│  • Authentication                                       │
│  • Request routing                                      │
└────────────────┬────────────────────────────────────────┘
                 │
        ┌────────┼────────┐
        ↓        ↓        ↓
┌──────────────────────────────────────────────────────────┐
│              MICROSERVICES                              │
│  ┌─────────────────────────────────────────────────────┐│
│  │ • Tweet Service       • User Service               ││
│  │ • Feed Service        • Search Service             ││
│  │ • Like Service        • Trending Service           ││
│  └─────────────────────────────────────────────────────┘│
└──────────────┬────────────────────────────────────────────┘
               │
    ┌──────────┼──────────┐
    ↓          ↓          ↓
┌────────────────────────────────────────────────────────┐
│  CACHE (Redis)      MESSAGE QUEUE (Kafka)            │
│  • Tweet cache      • Tweet events                   │
│  • User cache       • Like events                    │
│  • Feed cache       • Follow events                  │
│  • Session cache    • Notification events            │
└────────────────────────────────────────────────────────┘
    │
    ↓
┌────────────────────────────────────────────────────────┐
│  DATABASE LAYER                                       │
│  • PostgreSQL: Users, Follows, Likes                 │
│  • Cassandra: Tweets (write-heavy, immutable)        │
│  • ElasticSearch: Tweet search & indexing            │
│  • HBase: User timelines (high write volume)         │
└────────────────────────────────────────────────────────┘
    │
    ↓
┌────────────────────────────────────────────────────────┐
│  EXTERNAL SERVICES                                    │
│  • CDN (CloudFront): Images/videos                    │
│  • S3: Media storage                                  │
│  • Analytics: BigQuery                                │
└────────────────────────────────────────────────────────┘
```

---

## Database Schema

### Users Table (PostgreSQL)
```sql
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255),
    display_name VARCHAR(100),
    bio TEXT,
    profile_image_url VARCHAR(500),
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    is_active BOOLEAN,
    followers_count INT DEFAULT 0,
    following_count INT DEFAULT 0,
    INDEX (username),
    INDEX (created_at)
);

CREATE TABLE follows (
    follower_id BIGINT NOT NULL,
    following_id BIGINT NOT NULL,
    created_at TIMESTAMP,
    PRIMARY KEY (follower_id, following_id),
    FOREIGN KEY (follower_id) REFERENCES users(user_id),
    FOREIGN KEY (following_id) REFERENCES users(user_id),
    INDEX (following_id)
);
```

### Tweets Table (Cassandra - Write-Optimized)
```
Why Cassandra?
✓ High write throughput (tweets are immutable)
✓ Distributed architecture
✓ Time-series data (tweets have timestamps)
✓ Easy horizontal scaling

Schema:
CREATE TABLE tweets (
    tweet_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    text TEXT,
    media_urls LIST<VARCHAR>,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE user_tweets (
    user_id BIGINT,
    tweet_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, created_at, tweet_id)
) WITH CLUSTERING ORDER BY (created_at DESC);
```

### Likes & Retweets (Cassandra)
```
CREATE TABLE likes (
    user_id BIGINT,
    tweet_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, tweet_id)
);

CREATE TABLE likes_by_tweet (
    tweet_id BIGINT,
    user_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (tweet_id, created_at, user_id)
) WITH CLUSTERING ORDER BY (created_at DESC);
```

---

## Core Services

### 1. Tweet Service

**Responsibilities:**
- Create tweets
- Delete tweets
- Get tweet details
- Update tweet counts (likes, retweets)

**Write Flow (Creating a Tweet):**
```
1. Receive tweet creation request
   └─ Validate text length (max 280 chars)

2. Generate tweet_id (distributed ID generator)
   └─ Use Snowflake ID format

3. Store in Cassandra
   └─ user_tweets table (indexed by user_id, created_at)
   └─ tweets table (indexed by tweet_id)

4. Update user stats
   └─ Increment tweets_count in PostgreSQL

5. Invalidate cache
   └─ User's timeline cache

6. Publish event to Kafka
   └─ tweet_created event for followers' feeds

7. Return tweet_id to client
```

**Implementation:**
```javascript
async function createTweet(userId, tweetData) {
    // 1. Validate
    if (tweetData.text.length > 280) {
        throw new Error('Tweet too long');
    }

    // 2. Generate unique ID
    const tweetId = snowflakeIdGenerator.generate();

    // 3. Store in Cassandra
    const tweet = {
        tweet_id: tweetId,
        user_id: userId,
        text: tweetData.text,
        media_urls: tweetData.mediaUrls || [],
        created_at: new Date(),
        likes_count: 0,
        retweets_count: 0
    };

    await cassandra.insert('tweets', tweet);
    await cassandra.insert('user_tweets', {
        user_id: userId,
        tweet_id: tweetId,
        created_at: tweet.created_at
    });

    // 4. Update user stats
    await postgres.query(
        'UPDATE users SET tweets_count = tweets_count + 1 WHERE id = ?',
        [userId]
    );

    // 5. Invalidate cache
    await redis.delete(`user:${userId}:timeline`);

    // 6. Publish event
    await kafka.publish('tweet-events', {
        event: 'tweet_created',
        tweet_id: tweetId,
        user_id: userId,
        followers: await getFollowers(userId)
    });

    return tweet;
}
```

---

### 2. Feed Service (Most Critical)

**Challenge:** Generating feeds in real-time for 50M users is expensive.

**Solution: Push vs Pull Model**

```
PULL Model (Traditional):
User requests feed
├─ Query all tweets from followed users
├─ Sort by timestamp
└─ Return top 50

Problem: Too many queries at scale

PUSH Model (Fanout):
Tweet is created
├─ Get all followers (expensive if celebrity has 10M followers)
├─ For each follower: Write tweet to their feed cache
└─ User reads from cache (fast)

Problem: Expensive for celebrities

HYBRID Model (Best):
├─ For normal users (< 10K followers):
│  └─ Push tweets to followers' feeds immediately
│
└─ For celebrities (> 10K followers):
   ├─ Push to top 1000 followers
   └─ On feed request: Fetch last 100 tweets from followed celebrities
```

**Feed Service Implementation:**
```javascript
// When user requests feed
async function getFeed(userId, page = 1, limit = 20) {
    const cacheKey = `feed:${userId}:page:${page}`;

    // 1. Try cache first
    let feed = await redis.get(cacheKey);
    if (feed) {
        return JSON.parse(feed);
    }

    // 2. Cache miss: Build feed
    const followingIds = await getFollowing(userId);

    // Split into regular and celebrity accounts
    const { regular, celebrities } = await categorizeAccounts(followingIds);

    // Get tweets from regular accounts (pushed to cache)
    const regularTweets = await getRegularFeed(userId, page, limit);

    // Get tweets from celebrities (pull model)
    const celebrityTweets = await getPulledTweets(celebrities, limit);

    // Merge and sort
    const feed = [...regularTweets, ...celebrityTweets]
        .sort((a, b) => b.created_at - a.created_at)
        .slice(0, limit);

    // 3. Cache for 1 minute
    await redis.setex(cacheKey, 60, JSON.stringify(feed));

    return feed;
}

// Fanout on tweet creation (push model)
async function fanoutTweet(tweetId, userId) {
    const followers = await getFollowers(userId);

    // Batch process followers
    const batchSize = 1000;
    for (let i = 0; i < followers.length; i += batchSize) {
        const batch = followers.slice(i, i + batchSize);

        // Push to Redis for each follower's feed
        await Promise.all(batch.map(followerId =>
            redis.lpush(`feed:${followerId}`, JSON.stringify({
                tweet_id: tweetId,
                user_id: userId,
                created_at: Date.now()
            }))
        ));
    }

    // Trim feeds to keep only recent 1000 tweets
    await Promise.all(batch.map(followerId =>
        redis.ltrim(`feed:${followerId}`, 0, 999)
    ));
}
```

---

### 3. Search Service

**Using Elasticsearch:**

```javascript
// Index tweets in Elasticsearch
async function indexTweet(tweet) {
    await elasticsearch.index({
        index: 'tweets',
        id: tweet.tweet_id,
        body: {
            tweet_id: tweet.tweet_id,
            user_id: tweet.user_id,
            text: tweet.text,
            username: tweet.username,
            created_at: tweet.created_at,
            likes_count: tweet.likes_count
        }
    });
}

// Search tweets
async function searchTweets(query, page = 1, limit = 20) {
    const from = (page - 1) * limit;

    const results = await elasticsearch.search({
        index: 'tweets',
        body: {
            query: {
                bool: {
                    must: [
                        {
                            multi_match: {
                                query: query,
                                fields: ['text^2', 'username'],
                                fuzziness: 'AUTO'
                            }
                        }
                    ],
                    filter: [
                        {
                            range: {
                                created_at: {
                                    gte: 'now-30d'  // Last 30 days
                                }
                            }
                        }
                    ]
                }
            },
            sort: [
                { likes_count: { order: 'desc' } },
                { created_at: { order: 'desc' } }
            ],
            from: from,
            size: limit
        }
    });

    return results.hits.hits.map(hit => hit._source);
}
```

---

## API Endpoints

```
POST /tweets
  Create a new tweet
  Body: { text, media_urls }
  Response: { tweet_id, created_at }

GET /tweets/{tweet_id}
  Get tweet details
  Response: { tweet_id, user_id, text, likes_count, ... }

DELETE /tweets/{tweet_id}
  Delete a tweet

POST /tweets/{tweet_id}/like
  Like a tweet

DELETE /tweets/{tweet_id}/like
  Unlike a tweet

GET /feed
  Get personalized feed
  Query: page, limit
  Response: { tweets: [...], next_page_token }

GET /search?q=query&page=1
  Search tweets

POST /users/{user_id}/follow
  Follow a user

DELETE /users/{user_id}/follow
  Unfollow a user

GET /trending
  Get trending topics
```

---

## Handling Scale Challenges

### Challenge 1: Feed Generation for Heavy Users
```
User with 1M followers posts tweet:
- Fanout to feed of 1M users (expensive!)

Solution: Hybrid approach
├─ For < 10K followers: Immediate fanout
└─ For > 10K followers: Lazy load on user request
```

### Challenge 2: Hotspot Data (Celebrity Tweets)
```
A celebrity's tweet gets 1M likes:
- Database becomes bottleneck for write

Solution:
├─ Use Cassandra (write-optimized)
├─ Batch writes (accumulate, then write)
├─ Cache like counts
└─ Update asynchronously
```

### Challenge 3: Consistency vs Availability
```
Trade-off: Eventual consistency
├─ Feed may be slightly stale (ok)
├─ Like counts update asynchronously (ok)
├─ User actions (follow) are immediate (critical)
```

---

## Scaling Strategy

```
1. Vertical Scaling First
   └─ Use efficient algorithms
   └─ Optimize database queries
   └─ Add caching

2. Horizontal Scaling
   ├─ Multiple application servers
   ├─ Database replication (read replicas)
   ├─ Cassandra node scaling
   └─ CDN for media

3. Database Sharding
   ├─ Shard by user_id
   ├─ Shard key: user_id % num_shards
   └─ Range sharding for time-series data

4. Regional Distribution
   ├─ Deploy in multiple regions
   ├─ Use CDN for images/videos
   ├─ Database replication across regions
   └─ Eventually consistent feeds
```

---

---

# Question 2: Design Instagram

## Problem Statement
Design a social media platform like Instagram where users can:
- Upload and share photos
- Follow other users
- Like and comment on photos
- See a feed of photos from followed users
- Explore/discover new content
- View user profiles and followers

### Scale Requirements
```
Assumptions:
- 2 billion monthly active users
- 500 million daily active users
- Each user uploads ~10 photos per month (~0.3 per day)
- Heavy read: Light write ratio (300:1)

Calculations:
Photos per day = 500M DAU × 0.3 = 150M photos/day
Photos per second = 150M / 86,400 ≈ 1,700 TPS

Photo views per day = 500M DAU × 100 = 50B views/day
Photo views per second = 50B / 86,400 ≈ 578,000 RPS

Storage:
- Each photo: ~200KB average
- 150M photos/day × 200KB = 30TB/day
- Yearly: ~11PB (before replication)
```

---

## High-Level Architecture

```
┌──────────────────────────────────────────────┐
│          CLIENT LAYER                        │
│  Mobile App (iOS/Android)  │  Web Browser   │
└────────────┬─────────────────────────────────┘
             │ HTTPS
             ↓
┌──────────────────────────────────────────────┐
│      API GATEWAY / LOAD BALANCER             │
│  • Rate limiting                             │
│  • Authentication (JWT)                      │
│  • Request routing                           │
└────────────┬─────────────────────────────────┘
             │
     ┌───────┼───────┐
     ↓       ↓       ↓
┌──────────────────────────────────────────────┐
│         MICROSERVICES                        │
│  User Service  │  Photo Service             │
│  Feed Service  │  Search Service            │
│  Like Service  │  Comment Service           │
│  Story Service │  Notification Service      │
└────────┬───────────────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌──────────────────────────────────────────────┐
│  CACHE (Redis)    │  MESSAGE QUEUE (Kafka)  │
│  • Photo metadata │  • Photo upload events  │
│  • User profiles  │  • Like/comment events  │
│  • Feed cache     │  • Follow events        │
│  • Explore cache  │  • Story expiry         │
└────────┬──────────────────────────────────────┘
         │
    ┌────┼───────────────────┐
    ↓    ↓                   ↓
┌──────────────────────────────────────────────┐
│  DATABASE LAYER                              │
│  PostgreSQL: Users, Metadata                │
│  NoSQL: Photo data, Comments, Likes         │
│  ElasticSearch: Search & indexing           │
│  TimescaleDB: User activity/metrics         │
└────────┬──────────────────────────────────────┘
         │
    ┌────┼────────────────────────┐
    ↓    ↓                        ↓
┌──────────────────────────────────────────────┐
│  STORAGE & EXTERNAL SERVICES                 │
│  • AWS S3: Photo storage                     │
│  • CloudFront CDN: Photo delivery            │
│  • Rekognition: Image ML                     │
│  • Analytics: BigQuery                       │
└──────────────────────────────────────────────┘
```

---

## Database Design

### Photo Metadata (PostgreSQL)
```sql
CREATE TABLE photos (
    photo_id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    caption TEXT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    likes_count INT DEFAULT 0,
    comments_count INT DEFAULT 0,
    is_deleted BOOLEAN DEFAULT FALSE,
    photo_url VARCHAR(500),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX (user_id, created_at),
    INDEX (created_at)
);

CREATE TABLE photo_comments (
    comment_id BIGINT PRIMARY KEY,
    photo_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    text VARCHAR(1000),
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    FOREIGN KEY (photo_id) REFERENCES photos(photo_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX (photo_id, created_at)
);

CREATE TABLE photo_likes (
    photo_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    created_at TIMESTAMP,
    PRIMARY KEY (photo_id, user_id),
    INDEX (photo_id),
    INDEX (user_id, created_at)
);
```

### Photo Content Storage
```
S3 Bucket Structure:
s3://instagram-photos/
├── {user_id}/{photo_id}/
│   ├── original.jpg (original upload)
│   ├── 640x640.jpg (square)
│   ├── 1080x1080.jpg (feed view)
│   └─ 150x150.jpg (thumbnail)
```

---

## Key Features

### 1. Photo Upload Pipeline

```javascript
async function uploadPhoto(userId, photoFile, caption) {
    // 1. Generate unique photo ID
    const photoId = snowflakeIdGenerator.generate();

    // 2. Validate file
    if (!isValidImage(photoFile)) {
        throw new Error('Invalid image format');
    }

    // 3. Create database record (initially PROCESSING)
    const photo = {
        photo_id: photoId,
        user_id: userId,
        caption: caption,
        status: 'PROCESSING',
        created_at: new Date()
    };
    await postgres.insert('photos', photo);

    // 4. Upload to S3 (asynchronous)
    await queue.enqueue({
        task: 'process_photo',
        photo_id: photoId,
        user_id: userId,
        file: photoFile
    });

    // 5. Return immediately
    return { photo_id: photoId, status: 'PROCESSING' };
}

// Background job to process photo
async function processPhoto(photoId, userId, photoFile) {
    try {
        // 1. Upload original to S3
        const originalKey = `${userId}/${photoId}/original.jpg`;
        await s3.upload(originalKey, photoFile);

        // 2. Generate thumbnails
        const thumbnails = [
            { name: '640x640.jpg', size: [640, 640] },
            { name: '1080x1080.jpg', size: [1080, 1080] },
            { name: '150x150.jpg', size: [150, 150] }
        ];

        for (let thumb of thumbnails) {
            const resized = await resizeImage(photoFile, thumb.size);
            await s3.upload(`${userId}/${photoId}/${thumb.name}`, resized);
        }

        // 3. Generate metadata (ML)
        const labels = await detectLabels(photoFile);
        const colors = await analyzeColors(photoFile);

        // 4. Update database
        await postgres.query(
            'UPDATE photos SET status = ?, labels = ?, colors = ? WHERE id = ?',
            ['READY', labels, colors, photoId]
        );

        // 5. Invalidate user's profile cache
        await redis.delete(`user:${userId}:photos`);

        // 6. Publish event
        await kafka.publish('photo-events', {
            event: 'photo_uploaded',
            photo_id: photoId,
            user_id: userId
        });
    } catch (error) {
        // Mark as failed
        await postgres.query(
            'UPDATE photos SET status = ? WHERE id = ?',
            ['FAILED', photoId]
        );
        throw error;
    }
}
```

### 2. Feed Generation

```javascript
async function getFeed(userId, page = 1, limit = 20) {
    const offset = (page - 1) * limit;
    const cacheKey = `feed:${userId}:${page}`;

    // 1. Try cache
    let feed = await redis.get(cacheKey);
    if (feed) {
        return JSON.parse(feed);
    }

    // 2. Get followed users
    const followingIds = await getFollowingUsers(userId);

    // 3. Query photos from followed users
    const photos = await postgres.query(`
        SELECT p.*, u.username, u.profile_pic_url,
               (SELECT COUNT(*) FROM photo_likes WHERE photo_id = p.photo_id) as likes_count,
               (SELECT COUNT(*) FROM photo_comments WHERE photo_id = p.photo_id) as comments_count
        FROM photos p
        JOIN users u ON p.user_id = u.user_id
        WHERE p.user_id IN (${followingIds.join(',')})
            AND p.status = 'READY'
        ORDER BY p.created_at DESC
        LIMIT ? OFFSET ?
    `, [limit + 1, offset]);

    // 4. Check if user liked each photo
    for (let photo of photos) {
        const liked = await postgres.query(
            'SELECT 1 FROM photo_likes WHERE photo_id = ? AND user_id = ? LIMIT 1',
            [photo.photo_id, userId]
        );
        photo.user_liked = liked.length > 0;
    }

    // 5. Cache for 5 minutes
    await redis.setex(cacheKey, 300, JSON.stringify(photos.slice(0, limit)));

    // 6. Include next_page_token if more results
    return {
        photos: photos.slice(0, limit),
        next_page_token: photos.length > limit ? btoa(offset + limit) : null
    };
}
```

### 3. Explore/Discovery Feed

```javascript
async function getExplore(userId, page = 1, limit = 20) {
    const cacheKey = `explore:${userId}:${page}`;

    // 1. Check cache (popular content cached longer)
    let explore = await redis.get(cacheKey);
    if (explore) {
        return JSON.parse(explore);
    }

    // 2. Query trending/popular photos
    const photos = await postgres.query(`
        SELECT p.*, u.username,
               COUNT(DISTINCT pl.user_id) as likes_count,
               COUNT(DISTINCT pc.comment_id) as comments_count
        FROM photos p
        JOIN users u ON p.user_id = u.user_id
        LEFT JOIN photo_likes pl ON p.photo_id = pl.photo_id
        LEFT JOIN photo_comments pc ON p.photo_id = pc.photo_id
        WHERE p.user_id NOT IN (
            SELECT following_id FROM follows WHERE follower_id = ?
        )
        AND p.status = 'READY'
        AND p.created_at > NOW() - INTERVAL 7 DAY
        GROUP BY p.photo_id
        ORDER BY (likes_count + comments_count * 2) DESC
        LIMIT ? OFFSET ?
    `, [userId, limit + 1, (page - 1) * limit]);

    // 3. Cache for longer (less frequently updated)
    await redis.setex(cacheKey, 3600, JSON.stringify(photos));

    return photos.slice(0, limit);
}
```

---

## Handling Scale

### Likes Counter (High Write Volume)
```
Problem: Photo gets 10M likes
- Direct DB writes: Too slow
- Race conditions: Lost updates

Solution: Counter Cache
├─ Cache stores current count
├─ On like: Increment cache (fast)
├─ Batch write to DB every minute
└─ On failure: Restore from DB

Implementation:
async function likePhoto(photoId, userId) {
    // 1. Check if already liked
    const already = await redis.scard(`photo_likes:${photoId}:${userId}`);
    if (already) {
        throw new Error('Already liked');
    }

    // 2. Add to Redis set (fast, atomic)
    await redis.sadd(`photo_likes:${photoId}:${userId}`, userId);

    // 3. Increment counter in cache
    await redis.incr(`photo_likes_count:${photoId}`);

    // 4. Publish event (for async DB update)
    await kafka.publish('like-events', {
        event: 'photo_liked',
        photo_id: photoId,
        user_id: userId,
        timestamp: Date.now()
    });

    return { success: true };
}

// Background job to persist likes to DB
async function syncLikesToDatabase() {
    const photoLikes = await redis.keys('photo_likes_count:*');

    for (let key of photoLikes) {
        const photoId = key.replace('photo_likes_count:', '');
        const count = await redis.get(key);

        // Update DB
        await postgres.query(
            'UPDATE photos SET likes_count = ? WHERE photo_id = ?',
            [count, photoId]
        );

        // Reset cache (ready for next batch)
        await redis.delete(key);
    }
}

// Run every 60 seconds
setInterval(syncLikesToDatabase, 60000);
```

---

---

# Question 3: Design YouTube

## Problem Statement
Design a video streaming platform like YouTube where users can:
- Upload videos
- Watch videos with adaptive bitrate streaming
- Like and comment on videos
- Subscribe to channels
- Search for videos
- Recommend related videos
- Support live streaming

### Scale Requirements
```
Assumptions:
- 2 billion users
- 500 million daily active users
- 500 hours of video uploaded per minute
- 1 billion videos watched per day
- Peak concurrent viewers: 1M simultaneous streams

Calculations:
Videos uploaded per day = 500 hours/min × 60 × 24 = 720,000 hours
Video watch QPS = 1B watches / 86,400 = ~11,600 QPS

Bandwidth (HD 5Mbps avg):
11,600 QPS × 5Mbps = 58,000 Mbps = 58 Gbps required

Storage:
- 500 hours/min × 60 × 24 × 365 = 262.8M hours/year
- At 1GB per hour: 262.8 Exabytes/year!
- Solution: Tier content (keep popular content local, archive others)
```

---

## High-Level Architecture

```
┌──────────────────────────────────────────────┐
│      CLIENTS                                 │
│  Web │ Mobile │ Smart TV │ Desktop          │
└────────────┬─────────────────────────────────┘
             │ HTTPS
             ↓
┌──────────────────────────────────────────────┐
│      API GATEWAY / LOAD BALANCER             │
└────────────┬─────────────────────────────────┘
             │
    ┌────────┼────────┐
    ↓        ↓        ↓
┌──────────────────────────────────────────────┐
│      MICROSERVICES                           │
│  Video Service   │  Search Service          │
│  Stream Service  │  Recommendation Service  │
│  User Service    │  Analytics Service       │
│  Comment Service │  Live Service            │
└────────┬────────────────────────────────────┘
         │
    ┌────┼─────┐
    ↓    ↓     ↓
┌──────────────────────────────────────────────┐
│  Cache (Redis) │ MESSAGE QUEUE (Kafka)      │
│  Video metadata│ Upload events              │
│  Search cache  │ Watch events               │
│  User sessions │ Comment events             │
│  Stats cache   │ Analytics                  │
└────────┬──────────────────────────────────────┘
         │
    ┌────┼────────────────────┐
    ↓    ↓                    ↓
┌──────────────────────────────────────────────┐
│  DATABASE LAYER                              │
│  PostgreSQL: Metadata, Users, Comments      │
│  MongoDB: Video details, Comments           │
│  Cassandra: User watch history              │
│  ElasticSearch: Full-text search            │
└────────┬──────────────────────────────────────┘
         │
    ┌────┼──────────────────────┐
    ↓    ↓                      ↓
┌──────────────────────────────────────────────┐
│  MEDIA & STORAGE LAYER                       │
│  Video Encoding Service                      │
│  • Transcode to multiple bitrates            │
│  • Generate thumbnails                       │
│  • Distributed Storage                       │
│  ├─ S3 (upload area)                        │
│  ├─ CDN (CloudFront) - Global               │
│  └─ Regional cache servers                   │
└──────────────────────────────────────────────┘
```

---

## Database Design

### Video Metadata
```sql
CREATE TABLE videos (
    video_id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    thumbnail_url VARCHAR(500),
    duration INT,  -- seconds
    resolution VARCHAR(10),  -- 1080p, 720p, etc
    size_bytes BIGINT,
    view_count BIGINT DEFAULT 0,
    like_count INT DEFAULT 0,
    comment_count INT DEFAULT 0,
    status ENUM('UPLOADING', 'PROCESSING', 'PUBLISHED', 'DELETED'),
    visibility ENUM('PUBLIC', 'PRIVATE', 'UNLISTED'),
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    published_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX (user_id),
    INDEX (published_at),
    FULLTEXT INDEX (title, description)
);

CREATE TABLE video_files (
    file_id BIGINT PRIMARY KEY,
    video_id BIGINT NOT NULL,
    bitrate INT,  -- 500, 1000, 2500, 5000 (kbps)
    resolution VARCHAR(10),  -- 240p, 360p, 720p, 1080p
    format VARCHAR(10),  -- mp4, webm, mkv
    size_bytes BIGINT,
    duration INT,
    file_path VARCHAR(500),  -- s3://bucket/video_id/bitrate_resolution.mp4
    created_at TIMESTAMP,
    FOREIGN KEY (video_id) REFERENCES videos(video_id),
    INDEX (video_id, bitrate)
);
```

### Watch History (Cassandra)
```
Why Cassandra?
✓ High write volume (every watch is recorded)
✓ Time-series data
✓ Easy to query "videos watched by user"
✓ Distributed, scalable

Schema:
CREATE TABLE watch_history (
    user_id BIGINT,
    video_id BIGINT,
    watched_at TIMESTAMP,
    watch_duration INT,
    watched_percentage FLOAT,
    PRIMARY KEY (user_id, watched_at, video_id)
) WITH CLUSTERING ORDER BY (watched_at DESC);

CREATE TABLE video_watch_events (
    video_id BIGINT,
    user_id BIGINT,
    watched_at TIMESTAMP,
    watch_duration INT,
    PRIMARY KEY (video_id, watched_at, user_id)
) WITH CLUSTERING ORDER BY (watched_at DESC);
```

---

## Key Features

### 1. Video Upload & Encoding

```javascript
// Upload endpoint
async function uploadVideo(userId, videoFile, metadata) {
    // 1. Generate video ID
    const videoId = snowflakeIdGenerator.generate();

    // 2. Store metadata
    const video = {
        video_id: videoId,
        user_id: userId,
        title: metadata.title,
        description: metadata.description,
        status: 'UPLOADING',
        created_at: new Date()
    };
    await postgres.insert('videos', video);

    // 3. Upload to S3
    const uploadKey = `uploads/${userId}/${videoId}/original.${getExtension(videoFile)}`;
    const uploadUrl = await s3.getPresignedUrl(uploadKey, 3600);  // 1 hour expiry

    return {
        video_id: videoId,
        upload_url: uploadUrl,
        upload_id: videoId
    };
}

// After upload completes, trigger encoding
async function onVideoUploadComplete(videoId, userId) {
    // 1. Update status
    await postgres.query(
        'UPDATE videos SET status = ? WHERE video_id = ?',
        ['PROCESSING', videoId]
    );

    // 2. Queue encoding job
    const encodingJob = {
        video_id: videoId,
        user_id: userId,
        source: `s3://bucket/uploads/${userId}/${videoId}/original.mp4`,
        target_bitrates: [500, 1000, 2500, 5000],  // kbps
        target_resolutions: ['240p', '360p', '720p', '1080p']
    };

    await encodingQueue.enqueue(encodingJob);

    // 3. Publish event
    await kafka.publish('video-events', {
        event: 'video_upload_started',
        video_id: videoId
    });
}

// Encoding service (runs on powerful servers)
async function encodeVideo(job) {
    const { video_id, user_id, source, target_bitrates, target_resolutions } = job;

    try {
        // 1. Download from S3
        const sourceFile = await s3.download(source);

        // 2. Encode to multiple bitrates
        for (let bitrate of target_bitrates) {
            for (let resolution of target_resolutions) {
                const outputFile = await ffmpeg.encode(sourceFile, {
                    bitrate: bitrate,
                    resolution: resolution,
                    codec: 'h264'
                });

                // 3. Upload encoded file to S3
                const outputPath = `s3://bucket/videos/${video_id}/${bitrate}_${resolution}.mp4`;
                await s3.upload(outputPath, outputFile);

                // 4. Store file metadata
                await postgres.insert('video_files', {
                    file_id: snowflakeIdGenerator.generate(),
                    video_id: video_id,
                    bitrate: bitrate,
                    resolution: resolution,
                    size_bytes: outputFile.size,
                    file_path: outputPath
                });
            }
        }

        // 5. Generate thumbnail
        const thumbnail = await ffmpeg.extractFrame(sourceFile, 5);  // At 5 seconds
        await s3.upload(`s3://bucket/thumbnails/${video_id}.jpg`, thumbnail);

        // 6. Update video status
        await postgres.query(
            'UPDATE videos SET status = ?, thumbnail_url = ? WHERE video_id = ?',
            ['PUBLISHED', `https://cdn.example.com/thumbnails/${video_id}.jpg`, video_id]
        );

        // 7. Clear cache
        await redis.delete(`video:${video_id}`);

        // 8. Publish event
        await kafka.publish('video-events', {
            event: 'video_ready',
            video_id: video_id,
            user_id: user_id
        });
    } catch (error) {
        // Mark as failed
        await postgres.query(
            'UPDATE videos SET status = ? WHERE video_id = ?',
            ['FAILED', video_id]
        );
        throw error;
    }
}
```

### 2. Adaptive Bitrate Streaming (HLS/DASH)

```javascript
// Generate HLS playlist
async function generateHLSPlaylist(videoId) {
    // Get available video files
    const files = await postgres.query(
        'SELECT bitrate, resolution, file_path FROM video_files WHERE video_id = ? ORDER BY bitrate ASC',
        [videoId]
    );

    // Generate M3U8 playlist
    let playlist = `#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:0
`;

    for (let file of files) {
        const cdnUrl = `https://cdn.example.com/videos/${videoId}/${file.bitrate}_${file.resolution}.m3u8`;
        playlist += `#EXT-X-STREAM-INF:BANDWIDTH=${file.bitrate * 1000},RESOLUTION=${file.resolution}
${cdnUrl}
`;
    }

    playlist += '#EXT-X-ENDLIST';

    return playlist;
}

// Stream endpoint
app.get('/stream/:videoId/playlist.m3u8', async (req, res) => {
    const { videoId } = req.params;

    // 1. Check if user has access (privacy settings)
    const video = await postgres.query(
        'SELECT * FROM videos WHERE video_id = ?',
        [videoId]
    );

    if (video.visibility === 'PRIVATE' && video.user_id !== req.user.id) {
        return res.status(403).json({ error: 'Access denied' });
    }

    // 2. Generate playlist
    const playlist = await generateHLSPlaylist(videoId);

    // 3. Return playlist
    res.set('Content-Type', 'application/vnd.apple.mpegurl');
    res.send(playlist);
});

// CDN serves actual video segments
// Client (VLC, browser) automatically selects best bitrate:
// 1. Start with lowest bitrate (240p)
// 2. Monitor network speed
// 3. Switch up if bandwidth available
// 4. Switch down if buffering occurs
```

### 3. Watch History & Recommendations

```javascript
async function recordWatch(videoId, userId, watchDuration, watchPercentage) {
    // 1. Record in Cassandra (async, high throughput)
    await cassandra.insert('watch_history', {
        user_id: userId,
        video_id: videoId,
        watched_at: new Date(),
        watch_duration: watchDuration,
        watched_percentage: watchPercentage
    });

    // 2. Update view count (cache)
    await redis.incr(`video:${videoId}:views`);

    // 3. Publish event for analytics
    await kafka.publish('watch-events', {
        event: 'video_watched',
        video_id: videoId,
        user_id: userId,
        watch_duration: watchDuration,
        percentage: watchPercentage
    });
}

// Get recommendations
async function getRecommendations(userId, limit = 20) {
    const cacheKey = `recommendations:${userId}`;

    // 1. Check cache
    let recommendations = await redis.get(cacheKey);
    if (recommendations) {
        return JSON.parse(recommendations);
    }

    // 2. Get user's watch history
    const watchHistory = await cassandra.query(
        'SELECT video_id FROM watch_history WHERE user_id = ? LIMIT 100',
        [userId]
    );

    // 3. Find similar videos using ML model
    // ML model trained on video features, user preferences
    const similarVideos = await mlService.getRecommendations(
        watchHistory,
        userId,
        limit
    );

    // 4. Filter out already watched
    const alreadyWatched = watchHistory.map(w => w.video_id);
    const recommended = similarVideos.filter(v => !alreadyWatched.includes(v.video_id));

    // 5. Cache for 1 hour
    await redis.setex(cacheKey, 3600, JSON.stringify(recommended));

    return recommended;
}
```

---

---

# Question 4: Design TinyURL (URL Shortener)

## Problem Statement
Design a URL shortening service like TinyURL or bit.ly where users can:
- Convert long URLs into short, unique URLs
- Redirect short URLs to original URLs
- Track clicks and analytics
- Customize short URL (if not taken)
- Expire URLs after a time period

### Scale Requirements
```
Assumptions:
- 100 million URLs created per month
- 1 billion redirects per month
- 10-year retention period

Calculations:
URLs created per second = 100M / (30 × 86,400) ≈ 39 TPS
Redirects per second = 1B / (30 × 86,400) ≈ 386 RPS
Peak redirect RPS = 386 × 3 = ~1,200 RPS

Storage:
- Each URL entry: ~500 bytes
- 100M URLs/month × 12 months × 10 years = 12B URLs
- 12B × 500 bytes = 6 TB metadata
- Analytics: 6 TB additional
```

---

## High-Level Architecture

```
┌──────────────────────────────────┐
│   Client Apps                    │
│   Web Browser │ Mobile App       │
└────────────┬──────────────────────┘
             │
             ↓
┌──────────────────────────────────┐
│   API Gateway / Load Balancer    │
│   • Rate limiting                │
│   • Request routing              │
└────────────┬──────────────────────┘
             │
      ┌──────┴──────┐
      ↓             ↓
┌──────────────────────────────────┐
│   SERVICES                       │
│   URL Service     │  Analytics   │
│   Redirect Service│  Cleanup     │
└────────┬─────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌──────────────────────────────────┐
│  Cache (Redis)    │  Message     │
│  • Short→Long URL │  Queue       │
│  • Popular URLs   │ (Kafka)      │
│  • Counters       │              │
└────────┬─────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌──────────────────────────────────┐
│  DATABASE LAYER                  │
│  PostgreSQL: URL mappings        │
│  Cassandra: Analytics            │
│  ElasticSearch: Search           │
└──────────────────────────────────┘
```

---

## Database Design

### URL Mapping (PostgreSQL)

```sql
CREATE TABLE url_mappings (
    id BIGINT PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    original_url VARCHAR(2048) NOT NULL,
    user_id BIGINT,
    created_at TIMESTAMP,
    expires_at TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    created_by VARCHAR(50),  -- username or 'anonymous'
    custom_short_code BOOLEAN DEFAULT FALSE,
    click_count INT DEFAULT 0,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX (short_code),
    INDEX (user_id, created_at),
    INDEX (expires_at),
    INDEX (created_at)
);

CREATE TABLE url_analytics (
    id BIGINT PRIMARY KEY,
    short_code VARCHAR(10) NOT NULL,
    user_id BIGINT,
    ip_address VARCHAR(45),
    user_agent VARCHAR(500),
    referrer VARCHAR(2048),
    country VARCHAR(100),
    device_type VARCHAR(50),  -- desktop, mobile, tablet
    clicked_at TIMESTAMP,
    FOREIGN KEY (short_code) REFERENCES url_mappings(short_code),
    INDEX (short_code, clicked_at),
    INDEX (user_id, clicked_at)
);

CREATE TABLE reserved_codes (
    short_code VARCHAR(10) PRIMARY KEY,
    reserved_at TIMESTAMP,
    reserved_by VARCHAR(100)
);
```

---

## Core Implementation

### 1. URL Shortening Service

```javascript
async function shortenUrl(originalUrl, userId = null, customCode = null, expiryDays = 365) {
    // 1. Validate URL
    if (!isValidUrl(originalUrl)) {
        throw new Error('Invalid URL');
    }

    // 2. Check if URL already shortened by user
    if (userId) {
        const existing = await postgres.query(
            'SELECT short_code FROM url_mappings WHERE original_url = ? AND user_id = ? LIMIT 1',
            [originalUrl, userId]
        );

        if (existing.length > 0) {
            return { short_code: existing[0].short_code };
        }
    }

    // 3. Generate or validate short code
    let shortCode;
    if (customCode) {
        // Validate custom code
        if (!isValidShortCode(customCode)) {
            throw new Error('Invalid custom code format');
        }

        // Check if already taken
        const taken = await postgres.query(
            'SELECT id FROM url_mappings WHERE short_code = ? LIMIT 1',
            [customCode]
        );

        if (taken.length > 0) {
            throw new Error('Code already taken');
        }

        shortCode = customCode;
    } else {
        // Generate random short code
        shortCode = await generateUniqueShortCode();
    }

    // 4. Store in database
    const urlMapping = {
        id: snowflakeIdGenerator.generate(),
        short_code: shortCode,
        original_url: originalUrl,
        user_id: userId,
        created_at: new Date(),
        expires_at: new Date(Date.now() + expiryDays * 24 * 60 * 60 * 1000),
        custom_short_code: customCode ? true : false
    };

    await postgres.insert('url_mappings', urlMapping);

    // 5. Cache in Redis for fast lookup
    const cacheKey = `short:${shortCode}`;
    await redis.setex(cacheKey, 24 * 60 * 60, originalUrl);  // 24-hour cache

    // 6. Return short URL
    return {
        short_code: shortCode,
        short_url: `https://tinyurl.example.com/${shortCode}`,
        original_url: originalUrl,
        expires_at: urlMapping.expires_at
    };
}

// Generate unique short code
async function generateUniqueShortCode(length = 7) {
    const characters = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    let maxRetries = 3;

    while (maxRetries > 0) {
        let code = '';
        for (let i = 0; i < length; i++) {
            code += characters.charAt(Math.floor(Math.random() * characters.length));
        }

        // Check if code already exists
        const exists = await redis.exists(`short:${code}`);
        if (!exists) {
            const dbExists = await postgres.query(
                'SELECT id FROM url_mappings WHERE short_code = ? LIMIT 1',
                [code]
            );
            if (dbExists.length === 0) {
                return code;
            }
        }

        maxRetries--;
    }

    throw new Error('Unable to generate unique code');
}
```

### 2. Redirect Service (Critical Path)

```javascript
// Main redirect endpoint
app.get('/:shortCode', async (req, res) => {
    const { shortCode } = req.params;
    const startTime = Date.now();

    try {
        // 1. Check Redis cache first (fastest)
        let originalUrl = await redis.get(`short:${shortCode}`);

        if (!originalUrl) {
            // 2. Cache miss: Query database
            const urlMapping = await postgres.query(
                'SELECT original_url, is_active, expires_at FROM url_mappings WHERE short_code = ? LIMIT 1',
                [shortCode]
            );

            if (urlMapping.length === 0) {
                return res.status(404).json({ error: 'URL not found' });
            }

            const mapping = urlMapping[0];

            // 3. Check if expired
            if (mapping.expires_at < new Date()) {
                return res.status(410).json({ error: 'URL expired' });
            }

            // 4. Check if active
            if (!mapping.is_active) {
                return res.status(410).json({ error: 'URL disabled' });
            }

            originalUrl = mapping.original_url;

            // 5. Update cache
            const ttl = Math.floor((mapping.expires_at - Date.now()) / 1000);
            await redis.setex(`short:${shortCode}`, ttl, originalUrl);
        }

        // 6. Record analytics asynchronously (don't block redirect)
        recordAnalytics(shortCode, req);

        // 7. Increment click counter in cache
        await redis.incr(`clicks:${shortCode}`);

        // 8. Redirect (302 Found - temporary)
        res.redirect(302, originalUrl);
    } catch (error) {
        console.error('Redirect error:', error);
        res.status(500).json({ error: 'Server error' });
    }
});

// Async analytics recording
async function recordAnalytics(shortCode, request) {
    const geoLocation = await getGeoLocation(request.ip);

    const analytics = {
        id: snowflakeIdGenerator.generate(),
        short_code: shortCode,
        ip_address: request.ip,
        user_agent: request.get('User-Agent'),
        referrer: request.get('Referer'),
        country: geoLocation.country,
        device_type: getDeviceType(request.get('User-Agent')),
        clicked_at: new Date()
    };

    // Queue for async processing (don't block response)
    await kafka.publish('analytics-events', {
        event: 'url_clicked',
        ...analytics
    });
}

// Consumer to persist analytics
kafka.subscribe('analytics-events', async (message) => {
    const analytics = JSON.parse(message.value);

    // Batch insert every 1000 events or every 10 seconds
    analyticsBatch.push(analytics);

    if (analyticsBatch.length >= 1000) {
        await flushAnalytics();
    }
});

async function flushAnalytics() {
    if (analyticsBatch.length === 0) return;

    // Batch insert to Cassandra
    await cassandra.batchInsert('url_analytics', analyticsBatch);
    analyticsBatch = [];
}

// Flush every 10 seconds
setInterval(flushAnalytics, 10000);
```

### 3. Analytics Service

```javascript
// Get click statistics
async function getStats(shortCode, userId) {
    // 1. Verify ownership
    const urlMapping = await postgres.query(
        'SELECT user_id FROM url_mappings WHERE short_code = ?',
        [shortCode]
    );

    if (!urlMapping || urlMapping[0].user_id !== userId) {
        throw new Error('Unauthorized');
    }

    // 2. Get click count from cache or DB
    let totalClicks = await redis.get(`clicks:${shortCode}`);
    if (!totalClicks) {
        const result = await postgres.query(
            'SELECT click_count FROM url_mappings WHERE short_code = ?',
            [shortCode]
        );
        totalClicks = result[0].click_count;
    }

    // 3. Get detailed analytics
    const analytics = await cassandra.query(`
        SELECT
            country,
            device_type,
            COUNT(*) as count,
            clicked_at
        FROM url_analytics
        WHERE short_code = ?
        GROUP BY country, device_type, clicked_at
    `, [shortCode]);

    // 4. Aggregate by country
    const byCountry = {};
    for (let row of analytics) {
        if (!byCountry[row.country]) {
            byCountry[row.country] = 0;
        }
        byCountry[row.country] += row.count;
    }

    // 5. Aggregate by device
    const byDevice = {};
    for (let row of analytics) {
        if (!byDevice[row.device_type]) {
            byDevice[row.device_type] = 0;
        }
        byDevice[row.device_type] += row.count;
    }

    // 6. Get time series (clicks over time)
    const timeSeries = await cassandra.query(`
        SELECT
            DATE(clicked_at) as date,
            COUNT(*) as count
        FROM url_analytics
        WHERE short_code = ?
        GROUP BY DATE(clicked_at)
        ORDER BY date DESC
        LIMIT 30
    `, [shortCode]);

    return {
        short_code: shortCode,
        total_clicks: totalClicks,
        by_country: byCountry,
        by_device: byDevice,
        last_30_days: timeSeries
    };
}
```

### 4. Cleanup Service (Expiration)

```javascript
// Background job to clean up expired URLs
async function cleanupExpiredUrls() {
    // Find expired URLs
    const expiredUrls = await postgres.query(`
        SELECT short_code FROM url_mappings
        WHERE expires_at < NOW()
        AND is_active = TRUE
        LIMIT 10000
    `);

    for (let url of expiredUrls) {
        try {
            // 1. Mark as inactive
            await postgres.query(
                'UPDATE url_mappings SET is_active = FALSE WHERE short_code = ?',
                [url.short_code]
            );

            // 2. Remove from cache
            await redis.delete(`short:${url.short_code}`);
            await redis.delete(`clicks:${url.short_code}`);

            // 3. Optionally archive analytics
            // await archiveAnalytics(url.short_code);
        } catch (error) {
            console.error(`Failed to expire ${url.short_code}:`, error);
        }
    }

    console.log(`Cleaned up ${expiredUrls.length} expired URLs`);
}

// Run every 6 hours
setInterval(cleanupExpiredUrls, 6 * 60 * 60 * 1000);
```

---

## Performance Optimization

### Caching Strategy

```javascript
// Multi-level caching
// L1: In-memory cache in application (milliseconds)
// L2: Redis distributed cache (milliseconds)
// L3: Database (milliseconds - milliseconds)

const inMemoryCache = new Map();
const MAX_MEMORY_CACHE = 100000;

async function getUrl(shortCode) {
    // 1. Check in-memory cache (fastest)
    if (inMemoryCache.has(shortCode)) {
        return inMemoryCache.get(shortCode);
    }

    // 2. Check Redis (fast)
    let url = await redis.get(`short:${shortCode}`);
    if (url) {
        // Store in memory cache
        if (inMemoryCache.size < MAX_MEMORY_CACHE) {
            inMemoryCache.set(shortCode, url);
        }
        return url;
    }

    // 3. Query database (slower)
    const result = await postgres.query(
        'SELECT original_url FROM url_mappings WHERE short_code = ?',
        [shortCode]
    );

    if (result.length > 0) {
        url = result[0].original_url;
        // Update all cache levels
        inMemoryCache.set(shortCode, url);
        await redis.setex(`short:${shortCode}`, 86400, url);
        return url;
    }

    return null;
}
```

### Database Optimization

```sql
-- Add indexes for fast lookups
CREATE INDEX idx_short_code ON url_mappings(short_code);
CREATE INDEX idx_user_id ON url_mappings(user_id, created_at);
CREATE INDEX idx_expires ON url_mappings(expires_at);

-- Partitioning for analytics (time-based)
-- Partition by month for easier cleanup
ALTER TABLE url_analytics PARTITION BY RANGE (YEAR_MONTH(clicked_at));
```

---

## API Endpoints

```
POST /api/shorten
  Create short URL
  Body: { original_url, custom_code?, expires_days? }
  Response: { short_code, short_url, original_url }

GET /api/:short_code
  Get URL details (for logged-in users)
  Response: { original_url, created_at, expires_at }

GET /api/:short_code/stats
  Get analytics
  Response: { total_clicks, by_country, by_device, time_series }

DELETE /api/:short_code
  Delete short URL

GET /:short_code
  Redirect to original URL
  Response: 302 Found + Location header
```

---

## Summary Table

| Feature | Complexity | Solution |
|---------|-----------|----------|
| Unique short codes | Medium | Random generation with DB check or hash function |
| Fast redirects | High | Redis cache + CDN for static HTML |
| Analytics at scale | High | Kafka + Cassandra for time-series data |
| Custom codes | Low | Check availability before allocation |
| URL expiration | Medium | Background cleanup job |
| Hot URLs | Medium | In-memory cache for popular links |
| Global distribution | High | CDN + regional databases |

---

---

## Interview Tips for All 4 Questions

### 1. **Start with Clarification (First 5 minutes)**
```
Don't assume scale, ask:
✓ How many daily active users?
✓ What's the read-to-write ratio?
✓ What regions do we serve?
✓ What's the availability requirement?
✓ Do we need real-time consistency?
```

### 2. **Identify the Core Challenge**
```
Twitter: Feed generation at scale
Instagram: Image storage and CDN strategy
YouTube: Video encoding and adaptive streaming
TinyURL: High-throughput reads with caching
```

### 3. **Start Simple, Then Optimize**
```
Don't jump to sharding and microservices immediately.
Build monolithic first, then:
├─ Identify bottleneck (database? cache? network?)
└─ Add the appropriate layer
```

### 4. **Discuss Trade-offs**
```
"I chose MongoDB over PostgreSQL because:
✓ Schema flexibility
✓ Horizontal scaling
BUT trade-off:
✗ No ACID transactions
✗ Eventual consistency
```

### 5. **Mention Scalability at the End**
```
"To handle 10x traffic:
1. Horizontal scaling: Add more servers
2. Database sharding: By user_id
3. Caching: Redis for hot data
4. CDN: For static content
5. Async processing: Kafka for events
```

### 6. **Consider Monitoring & Operations**
```
Don't forget:
✓ How do you detect failures?
✓ How do you monitor performance?
✓ How do you handle deployments?
✓ What's your logging strategy?
```

---

**Remember:** The goal isn't to design a perfect system, but to show you can think through trade-offs, communicate clearly, and make informed decisions based on requirements.

---

---

# Question 5: Design Uber (Ride-Hailing)

## Problem Statement
Design a ride-hailing service like Uber where:
- Drivers can register and go online/offline
- Users can request rides
- System matches riders with nearby drivers
- Real-time location tracking
- Pricing calculation
- Payment processing
- Ratings and reviews

### Scale Requirements
```
Assumptions:
- 100 million monthly active users
- 1 million drivers online at peak
- 500K requests per minute (peak)
- Real-time location updates every 3 seconds

Calculations:
Requests per second = 500K / 60 ≈ 8,300 RPS
Location updates per second = 1M drivers × (1 update / 3 sec) ≈ 333K RPS
Matching algorithm calls per second = 8,300 RPS

Critical Requirements:
✓ Low latency (< 10 seconds to find driver)
✓ High availability (users can always request)
✓ Geographical distribution (nearest driver)
```

---

## High-Level Architecture

```
┌────────────────────────────────────────────┐
│         CLIENT LAYER                       │
│  Rider App │ Driver App │ Admin Dashboard  │
└────────────┬─────────────────────────────────┘
             │ WebSocket (real-time updates)
             ↓
┌────────────────────────────────────────────┐
│      API GATEWAY / LOAD BALANCER           │
│  • Rate limiting per user                  │
│  • WebSocket management                    │
└────────────┬────────────────────────────────┘
             │
    ┌────────┼────────┐
    ↓        ↓        ↓
┌────────────────────────────────────────────┐
│         MICROSERVICES                      │
│  User Service    │  Booking Service       │
│  Driver Service  │  Payment Service       │
│  Location Service│  Rating Service        │
│  Matching Service│  Analytics Service     │
└────────┬────────────────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌────────────────────────────────────────────┐
│  Real-Time Cache (Redis)                  │
│  • Active driver locations                │
│  • Active ride requests                   │
│  • Driver availability status              │
│  • Rate limits                             │
│  • WebSocket connections                  │
└────────┬─────────────────────────────────┘
         │
    ┌────┼──────────┐
    ↓    ↓          ↓
┌────────────────────────────────────────────┐
│  MESSAGE QUEUE (Kafka)                    │
│  • Location updates                       │
│  • Booking events                         │
│  • Payment events                         │
│  • Analytics events                       │
└────────┬─────────────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌────────────────────────────────────────────┐
│  DATABASES                                 │
│  PostgreSQL: Users, Bookings, Payments   │
│  Cassandra: Location history, Events     │
│  Redis Geo: Spatial indexing             │
│  ElasticSearch: Search                   │
└────────────────────────────────────────────┘
```

---

## Core Services

### 1. Location Service (Real-Time)

**Challenge:** Tracking 1 million drivers' locations in real-time

```javascript
// Driver sends location update every 3 seconds
async function updateDriverLocation(driverId, latitude, longitude) {
    // 1. Store in Redis Geo Hash (spatial index)
    await redis.geoadd(
        'driver_locations',
        longitude,
        latitude,
        driverId
    );

    // 2. Publish to Kafka (for analytics, notifications)
    await kafka.publish('location-events', {
        event: 'driver_location_updated',
        driver_id: driverId,
        latitude: latitude,
        longitude: longitude,
        timestamp: Date.now()
    });

    // 3. Update location history in Cassandra (async)
    await cassandra.insert('driver_location_history', {
        driver_id: driverId,
        latitude: latitude,
        longitude: longitude,
        timestamp: Date.now()
    });
}

// Find nearby drivers
async function findNearbyDrivers(latitude, longitude, radiusKm = 5) {
    // GEORADIUS: Find all drivers within 5km
    const nearbyDrivers = await redis.georadius(
        'driver_locations',
        longitude,
        latitude,
        radiusKm,
        'km',
        'WITHDIST'  // Include distance
    );

    return nearbyDrivers
        .sort((a, b) => a[1] - b[1])  // Sort by distance
        .slice(0, 10);  // Return top 10 nearest
}
```

### 2. Matching Service (Most Critical)

```javascript
async function matchRiderWithDriver(riderId, pickupLat, pickupLng, dropLat, dropLng) {
    try {
        // 1. Find nearby available drivers
        const nearbyDrivers = await findNearbyDrivers(pickupLat, pickupLng, 5);

        if (nearbyDrivers.length === 0) {
            return {
                success: false,
                message: 'No drivers available in your area',
                wait_time_estimate: '5-10 minutes'
            };
        }

        // 2. Score drivers based on multiple factors
        const scoredDrivers = await scoreDrivers(
            nearbyDrivers,
            pickupLat,
            pickupLng,
            riderId
        );

        // 3. Select top driver
        const selectedDriver = scoredDrivers[0];

        // 4. Send offer to driver
        const offerAccepted = await sendOfferToDriver(
            selectedDriver.driver_id,
            riderId,
            pickupLat,
            pickupLng,
            30  // Offer expires in 30 seconds
        );

        if (!offerAccepted) {
            // Try next driver
            return await matchRiderWithDriver(riderId, pickupLat, pickupLng, dropLat, dropLng);
        }

        // 5. Create booking
        const booking = {
            booking_id: generateId(),
            rider_id: riderId,
            driver_id: selectedDriver.driver_id,
            pickup_location: { lat: pickupLat, lng: pickupLng },
            dropoff_location: { lat: dropLat, lng: dropLng },
            status: 'CONFIRMED',
            created_at: Date.now(),
            estimated_fare: calculateFare(pickupLat, pickupLng, dropLat, dropLng),
            estimated_duration: calculateDuration(pickupLat, pickupLng, dropLat, dropLng)
        };

        await postgres.insert('bookings', booking);

        // 6. Notify both parties via WebSocket
        notifyDriver(selectedDriver.driver_id, { booking: booking });
        notifyRider(riderId, { booking: booking });

        // 7. Publish event
        await kafka.publish('booking-events', {
            event: 'ride_matched',
            booking_id: booking.booking_id,
            rider_id: riderId,
            driver_id: selectedDriver.driver_id
        });

        return booking;
    } catch (error) {
        console.error('Matching error:', error);
        throw error;
    }
}

// Score drivers
async function scoreDrivers(drivers, pickupLat, pickupLng, riderId) {
    const scores = [];

    for (let driver of drivers) {
        let score = 100;

        // Factor 1: Distance (closer is better)
        score -= driver.distance * 2;  // Each km = -2 points

        // Factor 2: Driver rating (higher is better)
        const rating = await getDriverRating(driver.driver_id);
        score += rating * 10;

        // Factor 3: Driver's current direction
        const heading = await getDriverHeading(driver.driver_id);
        const directionToPickup = calculateHeading(driver.location, { lat: pickupLat, lng: pickupLng });
        const headingDifference = Math.abs(heading - directionToPickup);
        if (headingDifference < 45) {
            score += 20;  // Bonus if going same direction
        }

        // Factor 4: Recent cancellations (lower is better)
        const recentCancellations = await getRecentCancellations(driver.driver_id);
        score -= recentCancellations * 5;

        scores.push({
            driver_id: driver.driver_id,
            score: score
        });
    }

    return scores.sort((a, b) => b.score - a.score);
}

// Send offer to driver (with timeout)
async function sendOfferToDriver(driverId, riderId, pickupLat, pickupLng, timeoutSeconds) {
    return new Promise((resolve) => {
        const offerId = generateId();

        // Send via WebSocket
        webSocketManager.sendToDriver(driverId, {
            event: 'ride_offer',
            offer_id: offerId,
            rider_id: riderId,
            pickup: { lat: pickupLat, lng: pickupLng },
            estimated_fare: 15.50
        });

        // Set timeout - if no response, driver didn't accept
        const timeout = setTimeout(() => {
            resolve(false);
        }, timeoutSeconds * 1000);

        // Listen for acceptance
        webSocketManager.once(`offer_response:${offerId}`, (response) => {
            clearTimeout(timeout);
            resolve(response.accepted);
        });
    });
}
```

### 3. Payment Service

```javascript
async function processPayment(bookingId, riderId, amount) {
    try {
        // 1. Create payment record
        const payment = {
            payment_id: generateId(),
            booking_id: bookingId,
            rider_id: riderId,
            amount: amount,
            status: 'PROCESSING',
            created_at: Date.now()
        };

        await postgres.insert('payments', payment);

        // 2. Process with payment gateway
        const result = await stripeAPI.charge({
            customer_id: riderId,
            amount: amount * 100,  // Convert to cents
            idempotency_key: bookingId  // Prevent duplicate charges
        });

        if (result.status === 'succeeded') {
            // 3. Update payment status
            await postgres.query(
                'UPDATE payments SET status = ?, transaction_id = ? WHERE payment_id = ?',
                ['COMPLETED', result.transaction_id, payment.payment_id]
            );

            // 4. Publish event
            await kafka.publish('payment-events', {
                event: 'payment_completed',
                booking_id: bookingId,
                amount: amount
            });

            return { success: true };
        } else {
            throw new Error('Payment declined');
        }
    } catch (error) {
        console.error('Payment error:', error);
        // Update as failed
        await postgres.query(
            'UPDATE payments SET status = ? WHERE booking_id = ?',
            ['FAILED', bookingId]
        );
        throw error;
    }
}
```

---

## Handling Scale Challenges

### Challenge 1: Location Updates (333K updates/second)

```
Solution: Redis Geo + Kafka

┌─────────────────────────────────┐
│ Driver Location Update (333K/s) │
└────────────┬────────────────────┘
             │
             ↓
        ┌────────────┐
        │ Redis Geo  │ (Spatial index)
        └────┬───────┘ (In-memory, super fast)
             │
             ↓
        ┌────────────┐
        │   Kafka    │ (Async persistence)
        └────┬───────┘ (Batch write to Cassandra)
             │
             ↓
        ┌────────────┐
        │ Cassandra  │ (Historical data)
        └────────────┘
```

### Challenge 2: Surge Pricing

```javascript
// Calculate fare based on demand
async function calculateFare(pickupLat, pickupLng, dropLat, dropLng) {
    // 1. Get base fare from distance
    const distance = calculateDistance(pickupLat, pickupLng, dropLat, dropLng);
    const baseFare = distance * RATE_PER_KM + BASE_FARE;

    // 2. Get surge multiplier from cache
    const area = getAreaCode(pickupLat, pickupLng);
    const surgeMultiplier = await redis.get(`surge:${area}`);

    // 3. Calculate final fare
    const finalFare = baseFare * (surgeMultiplier || 1.0);

    return {
        base_fare: baseFare,
        surge_multiplier: surgeMultiplier || 1.0,
        final_fare: finalFare
    };
}

// Update surge pricing (every minute)
async function updateSurgePricing() {
    // Get request count by area (last 5 minutes)
    const requestCounts = await getRequestCountsByArea('5m');

    for (let [area, count] of Object.entries(requestCounts)) {
        const availableDrivers = await getAvailableDriversInArea(area);

        // Supply/Demand ratio
        const ratio = count / availableDrivers;

        let multiplier = 1.0;
        if (ratio > 5) multiplier = 2.0;      // Very high demand
        else if (ratio > 3) multiplier = 1.5; // High demand
        else if (ratio > 1.5) multiplier = 1.2; // Moderate demand

        await redis.setex(`surge:${area}`, 300, multiplier);  // 5 min cache
    }
}
```

---

---

# Question 6: Design Netflix (Streaming)

## Problem Statement
Design a video streaming service like Netflix where users can:
- Browse and search movies/shows
- Stream videos at optimal quality
- Resume from where they left off
- Get personalized recommendations
- Create multiple profiles
- View download options
- Support various devices

### Scale Requirements
```
Assumptions:
- 250 million subscribers
- 100 million daily active users
- 300+ million concurrent streams during peak
- 500 petabytes of content

Calculations:
Concurrent streams at peak = 300M
Average bitrate = 5 Mbps
Peak bandwidth = 300M × 5 Mbps = 1.5 Exabits/second = 1.5 Ebit/s = 150+ Petabytes/hour

Critical Requirements:
✓ Ultra-low latency start (< 2 seconds to play)
✓ 99.99% availability
✓ Quality adaptation based on bandwidth
✓ Global CDN distribution
```

---

## High-Level Architecture

```
┌────────────────────────────────────────────┐
│         CLIENT TIER                        │
│  Web │ iOS │ Android │ Smart TV │ Desktop  │
└────────────┬─────────────────────────────────┘
             │ HTTPS
             ↓
┌────────────────────────────────────────────┐
│      API GATEWAY / LOAD BALANCER           │
│  • User authentication                     │
│  • Device management                       │
│  • Concurrent streams limit                │
└────────────┬────────────────────────────────┘
             │
    ┌────────┼────────┐
    ↓        ↓        ↓
┌────────────────────────────────────────────┐
│         MICROSERVICES                      │
│  Content Service   │  Playback Service    │
│  User Service      │  Recommendation Svc  │
│  Search Service    │  Analytics Service   │
│  Download Service  │  Payment Service     │
└────────┬────────────────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌────────────────────────────────────────────┐
│  Cache (Redis)                             │
│  • User preferences                        │
│  • Watch history (recent)                  │
│  • Recommendation results                  │
│  • Metadata cache                          │
│  • Session tokens                          │
└────────┬─────────────────────────────────┘
         │
    ┌────┼────────────┐
    ↓    ↓            ↓
┌────────────────────────────────────────────┐
│  DATABASE LAYER                            │
│  PostgreSQL: Users, Subscriptions, Billing │
│  MongoDB: Metadata, Watchlist, Profiles   │
│  Cassandra: Watch history, Events         │
│  ElasticSearch: Full-text search          │
└────────┬────────────────────────────────────┘
         │
    ┌────┼──────────────────┐
    ↓    ↓                  ↓
┌────────────────────────────────────────────┐
│  CDN & MEDIA LAYER                        │
│  • OpenConnect: Netflix's private CDN     │
│  • AWS CloudFront: Fallback               │
│  • Regional edge servers                  │
│  • Multiple encoded versions (HEVC, H264) │
│  • Adaptive bitrate (HLS/DASH)           │
└────────────────────────────────────────────┘
```

---

## Core Features

### 1. Adaptive Bitrate Streaming

```javascript
// Generate HLS manifest with multiple bitrates
async function getStreamingManifest(contentId, userId) {
    // 1. Get user's subscription tier
    const tier = await getUserSubscriptionTier(userId);
    const maxResolution = getMaxResolution(tier);  // 4K, 1080p, 720p, 480p

    // 2. Get available encodings
    const encodings = await getEncodingsForContent(contentId, maxResolution);

    // 3. Generate manifest
    const manifest = `#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:0
`;

    for (let encoding of encodings) {
        const bandwidth = encoding.bitrate * 1000;
        const resolution = encoding.resolution;
        const cdnUrl = `${encoding.cdn_url}/manifest.m3u8`;

        manifest += `#EXT-X-STREAM-INF:BANDWIDTH=${bandwidth},RESOLUTION=${resolution}
${cdnUrl}
`;
    }

    manifest += '#EXT-X-ENDLIST';

    // 4. Cache manifest
    await redis.setex(`manifest:${contentId}:${userId}`, 3600, manifest);

    return manifest;
}

// Client automatically adapts bitrate
// Netflix algorithm:
// 1. Start at lowest bitrate (480p)
// 2. Monitor buffer level
// 3. Monitor network speed
// 4. Switch up if: buffer > threshold AND speed sufficient
// 5. Switch down if: buffer < low threshold OR speed drops
```

### 2. Resume Watching / Continue

```javascript
async function saveWatchProgress(userId, contentId, timestamp, duration) {
    try {
        // 1. Save to Cassandra (high write volume)
        await cassandra.insert('watch_progress', {
            user_id: userId,
            content_id: contentId,
            watch_timestamp: timestamp,
            total_duration: duration,
            updated_at: Date.now()
        });

        // 2. Update Redis cache for quick access
        const key = `watch:${userId}:${contentId}`;
        await redis.setex(key, 7 * 24 * 60 * 60, JSON.stringify({
            timestamp: timestamp,
            duration: duration,
            watched_at: Date.now()
        }));

        // 3. Publish event for analytics
        await kafka.publish('playback-events', {
            event: 'playback_progress',
            user_id: userId,
            content_id: contentId,
            progress_percentage: (timestamp / duration) * 100
        });
    } catch (error) {
        console.error('Error saving progress:', error);
        // Don't block playback on error
    }
}

// Get continue watching
async function getContinueWatching(userId) {
    const cacheKey = `continue:${userId}`;

    // 1. Check cache first
    let result = await redis.get(cacheKey);
    if (result) {
        return JSON.parse(result);
    }

    // 2. Query Cassandra for recent watches
    const recentWatches = await cassandra.query(`
        SELECT content_id, watch_timestamp, total_duration
        FROM watch_progress
        WHERE user_id = ?
        ORDER BY updated_at DESC
        LIMIT 20
    `, [userId]);

    // 3. Filter out completed videos
    const continueWatching = recentWatches
        .filter(w => w.watch_timestamp < w.total_duration * 0.95)  // Not 95% complete
        .slice(0, 10);

    // 4. Enrich with content metadata
    const enriched = await Promise.all(
        continueWatching.map(async (w) => {
            const metadata = await getContentMetadata(w.content_id);
            return {
                ...metadata,
                resume_from: w.watch_timestamp
            };
        })
    );

    // 5. Cache for 1 hour
    await redis.setex(cacheKey, 3600, JSON.stringify(enriched));

    return enriched;
}
```

### 3. Recommendation Engine

```javascript
async function getRecommendations(userId, limit = 20) {
    const cacheKey = `recommendations:${userId}`;

    // 1. Check cache
    let recommendations = await redis.get(cacheKey);
    if (recommendations) {
        return JSON.parse(recommendations);
    }

    // 2. Get user's watch history and preferences
    const watchHistory = await cassandra.query(`
        SELECT content_id FROM watch_progress
        WHERE user_id = ?
        ORDER BY updated_at DESC
        LIMIT 100
    `, [userId]);

    const ratings = await postgres.query(
        'SELECT content_id, rating FROM ratings WHERE user_id = ? LIMIT 100',
        [userId]
    );

    // 3. Call ML recommendation service
    const recommended = await mlService.getRecommendations({
        user_id: userId,
        watch_history: watchHistory,
        ratings: ratings,
        subscription_tier: await getUserTier(userId)
    }, limit);

    // 4. Filter out already watched
    const alreadyWatched = watchHistory.map(w => w.content_id);
    const filtered = recommended.filter(r => !alreadyWatched.includes(r.content_id));

    // 5. Cache for 24 hours (recommendations don't change often)
    await redis.setex(cacheKey, 24 * 3600, JSON.stringify(filtered));

    return filtered;
}

// ML Model: Collaborative Filtering + Content-Based
// Training data:
// ├─ User-content matrix (ratings)
// ├─ Content metadata (genre, cast, director)
// ├─ User demographics
// └─ Watch history

// Real-time updates via Kafka
// When user watches content:
// ├─ Update user profile
// ├─ Invalidate recommendation cache
// └─ Retrain model (offline)
```

---

---

# Question 7: Design WhatsApp (Messaging)

## Problem Statement
Design a messaging service like WhatsApp where users can:
- Send and receive text messages
- Send media (images, videos, audio)
- Send messages to groups
- Delivery and read receipts
- End-to-end encryption
- Message search
- Last seen status
- Online/offline status

### Scale Requirements
```
Assumptions:
- 2 billion users
- 500 million daily active users
- 100 million messages per second at peak
- Group chats: up to 256 members per group
- Message size: 1-2KB average

Calculations:
Messages per second = 100M
At 1-2KB per message = 100M - 200M KB/s = 100-200 GB/s throughput

Peak concurrent connections = 500M users
Connection overhead per user = ~10KB = 5TB RAM needed just for connections

Critical Requirements:
✓ Ultra-low latency (< 200ms)
✓ 99.99% availability
✓ End-to-end encryption
✓ Message persistence
✓ Horizontal scaling (handle millions per server)
```

---

## High-Level Architecture

```
┌────────────────────────────────────────────┐
│         CLIENT TIER                        │
│  iOS │ Android │ Web │ Desktop             │
└────────────┬─────────────────────────────────┘
             │ WebSocket (persistent connection)
             ↓
┌────────────────────────────────────────────┐
│      API GATEWAY / LOAD BALANCER           │
│  • WebSocket routing                       │
│  • Connection pooling                      │
│  • SSL/TLS termination                     │
└────────────┬────────────────────────────────┘
             │
    ┌────────┼────────┐
    ↓        ↓        ↓
┌────────────────────────────────────────────┐
│         MESSAGE SERVICES                   │
│  Chat Service      │ Group Service        │
│  Status Service    │ Encryption Service   │
│  Media Service     │ Search Service       │
│  Notification Svc  │ Delivery Service     │
└────────┬────────────────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌────────────────────────────────────────────┐
│  Cache & Message Queue                    │
│  • Redis: Online users, Active chats      │
│  • Kafka: Message stream                  │
│  • RabbitMQ: Delivery queue                │
└────────┬─────────────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌────────────────────────────────────────────┐
│  DATABASE LAYER                            │
│  MySQL/PostgreSQL: Users, Groups          │
│  Cassandra: Messages (write-optimized)    │
│  Cassandra: Conversations (indexed)       │
│  ElasticSearch: Message search            │
│  HBase: Media metadata                    │
└────────────────────────────────────────────┘
        │
        ↓
┌────────────────────────────────────────────┐
│  MEDIA & STORAGE                           │
│  • S3: Message media                       │
│  • CDN: Media delivery                     │
│  • Regional caches                         │
└────────────────────────────────────────────┘
```

---

## Core Features

### 1. Message Sending & Delivery

```javascript
async function sendMessage(senderId, recipientId, messageText, attachments = []) {
    const messageId = generateId();
    const timestamp = Date.now();

    try {
        // 1. Create message object
        const message = {
            message_id: messageId,
            sender_id: senderId,
            recipient_id: recipientId,
            text: messageText,
            attachments: attachments,
            created_at: timestamp,
            status: 'SENT',  // SENT, DELIVERED, READ
            encrypted_text: encryptMessage(messageText)  // E2E encryption
        };

        // 2. Store in Cassandra (immutable, write-optimized)
        await cassandra.insert('messages', message);

        // 3. Store in conversation thread
        await cassandra.insert('conversations', {
            conversation_id: getConversationId(senderId, recipientId),
            participant_1: senderId,
            participant_2: recipientId,
            last_message_id: messageId,
            last_message_text: messageText,
            updated_at: timestamp
        });

        // 4. Publish to Kafka (for delivery)
        await kafka.publish('messages', {
            message_id: messageId,
            sender_id: senderId,
            recipient_id: recipientId,
            timestamp: timestamp
        });

        // 5. Check if recipient is online
        const recipientOnline = await redis.hget('online_users', recipientId);

        if (recipientOnline) {
            // 6a. Recipient is online: Send via WebSocket
            await webSocketManager.sendMessage(recipientId, {
                type: 'MESSAGE',
                message_id: messageId,
                sender_id: senderId,
                text: messageText,
                timestamp: timestamp
            });

            // 7a. Update status to DELIVERED immediately
            await updateMessageStatus(messageId, 'DELIVERED', timestamp);
        } else {
            // 6b. Recipient offline: Queue for delivery when online
            await redis.lpush(`queue:${recipientId}`, JSON.stringify(message));
        }

        // 8. Return ACK to sender
        return {
            message_id: messageId,
            status: 'SENT',
            timestamp: timestamp
        };
    } catch (error) {
        console.error('Message send error:', error);
        // Update message status to FAILED
        await updateMessageStatus(messageId, 'FAILED');
        throw error;
    }
}

// Update message delivery status
async function updateMessageStatus(messageId, status, timestamp = Date.now()) {
    // 1. Update in Cassandra
    await cassandra.insert('message_status', {
        message_id: messageId,
        status: status,
        updated_at: timestamp
    });

    // 2. Notify sender
    const message = await cassandra.query(
        'SELECT sender_id FROM messages WHERE message_id = ?',
        [messageId]
    );

    if (message && message[0]) {
        await webSocketManager.sendToUser(message[0].sender_id, {
            type: 'DELIVERY_ACK',
            message_id: messageId,
            status: status,
            timestamp: timestamp
        });
    }
}

// When recipient comes online: deliver queued messages
async function onUserOnline(userId) {
    // 1. Add to online users
    await redis.hset('online_users', userId, Date.now());

    // 2. Get queued messages
    const queueKey = `queue:${userId}`;
    const queuedMessages = await redis.lrange(queueKey, 0, -1);

    for (let msgStr of queuedMessages) {
        const message = JSON.parse(msgStr);

        // 3. Send via WebSocket
        await webSocketManager.sendMessage(userId, {
            type: 'MESSAGE',
            ...message
        });

        // 4. Update status to DELIVERED
        await updateMessageStatus(message.message_id, 'DELIVERED');
    }

    // 5. Clear queue
    await redis.delete(queueKey);
}
```

### 2. Group Messaging

```javascript
async function sendGroupMessage(senderId, groupId, messageText, attachments = []) {
    const messageId = generateId();
    const timestamp = Date.now();

    try {
        // 1. Verify user is member of group
        const isMember = await postgres.query(
            'SELECT 1 FROM group_members WHERE group_id = ? AND user_id = ?',
            [groupId, senderId]
        );

        if (!isMember) {
            throw new Error('Not a member of this group');
        }

        // 2. Create group message
        const message = {
            message_id: messageId,
            group_id: groupId,
            sender_id: senderId,
            text: messageText,
            attachments: attachments,
            created_at: timestamp,
            status: 'SENT'
        };

        // 3. Store message
        await cassandra.insert('group_messages', message);

        // 4. Get all group members
        const members = await postgres.query(
            'SELECT user_id FROM group_members WHERE group_id = ?',
            [groupId]
        );

        // 5. Publish to Kafka
        await kafka.publish('group_messages', {
            message_id: messageId,
            group_id: groupId,
            sender_id: senderId,
            member_ids: members.map(m => m.user_id),
            timestamp: timestamp
        });

        // 6. Send to each member
        for (let member of members) {
            if (member.user_id === senderId) continue;  // Don't send to self

            const online = await redis.hget('online_users', member.user_id);

            if (online) {
                // Send immediately
                await webSocketManager.sendMessage(member.user_id, {
                    type: 'GROUP_MESSAGE',
                    message_id: messageId,
                    group_id: groupId,
                    sender_id: senderId,
                    text: messageText,
                    timestamp: timestamp
                });
            } else {
                // Queue for later
                await redis.lpush(
                    `queue:group:${member.user_id}:${groupId}`,
                    JSON.stringify(message)
                );
            }
        }

        return { message_id: messageId, status: 'SENT' };
    } catch (error) {
        console.error('Group message error:', error);
        throw error;
    }
}
```

### 3. Read Receipts

```javascript
async function markAsRead(userId, messageId) {
    const timestamp = Date.now();

    try {
        // 1. Store read receipt
        await cassandra.insert('read_receipts', {
            message_id: messageId,
            reader_id: userId,
            read_at: timestamp
        });

        // 2. Get sender of message
        const message = await cassandra.query(
            'SELECT sender_id FROM messages WHERE message_id = ?',
            [messageId]
        );

        if (message && message[0]) {
            const senderId = message[0].sender_id;

            // 3. Notify sender
            await webSocketManager.sendToUser(senderId, {
                type: 'READ_RECEIPT',
                message_id: messageId,
                read_by: userId,
                read_at: timestamp
            });
        }

        return { success: true };
    } catch (error) {
        console.error('Read receipt error:', error);
    }
}
```

---

---

# Question 8: Design Slack (Team Chat)

## Problem Statement
Design a team collaboration platform like Slack where:
- Users send messages in channels and DMs
- Messages are searchable and persistent
- Threads for organized conversations
- File sharing and uploads
- Reactions and emoji
- User presence status
- Message history
- Integration with external tools

### Scale Requirements
```
Assumptions:
- 20 million organizations
- 500 million daily active users
- 10 million concurrent users
- 1 billion messages per day
- 50 million files shared per day

Calculations:
Messages per second = 1B / 86,400 ≈ 11,600 TPS
Concurrent messages = ~200 RPS at any time
File uploads per second = 50M / 86,400 ≈ 580 per second

Critical Requirements:
✓ Full message history (searchable)
✓ Low latency (< 500ms for delivery)
✓ File sharing (up to 2GB)
✓ Horizontal scaling
✓ Data isolation per workspace
```

---

## Key Differences from WhatsApp

| Feature | WhatsApp | Slack |
|---------|----------|-------|
| **Messages** | Point-to-point, Groups | Channels, Threads, DMs |
| **History** | Recent only | Full searchable history |
| **Integration** | No | Hundreds of integrations |
| **File Limit** | Small (< 100MB) | Large (2GB per file) |
| **Data Retention** | Minimal | Full history |
| **Search** | Limited | Full-text search on all messages |

---

## Architecture

```
┌────────────────────────────────────────────┐
│         CLIENT TIER                        │
│  Web │ Desktop │ Mobile │ API Clients      │
└────────────┬─────────────────────────────────┘
             │ WebSocket
             ↓
┌────────────────────────────────────────────┐
│      API GATEWAY / LOAD BALANCER           │
│  • WebSocket routing per workspace         │
│  • Authentication                          │
│  • Rate limiting                           │
└────────────┬────────────────────────────────┘
             │
    ┌────────┼────────┐
    ↓        ↓        ↓
┌────────────────────────────────────────────┐
│         SERVICES                           │
│  Message Service   │  Channel Service      │
│  Search Service    │  File Service         │
│  Presence Service  │ Integration Service   │
│  Thread Service    │ Notification Service  │
└────────┬────────────────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌────────────────────────────────────────────┐
│  Cache & Message Queue                    │
│  • Redis: Sessions, Presence              │
│  • Kafka: Message stream                  │
│  • Redis Streams: Timeline                │
└────────┬─────────────────────────────────┘
         │
    ┌────┼──────────┐
    ↓    ↓          ↓
┌────────────────────────────────────────────┐
│  DATABASE LAYER                            │
│  PostgreSQL: Workspaces, Users, Channels  │
│  Cassandra: Messages (massive scale)      │
│  ElasticSearch: Full-text search          │
│  Clickhouse: Analytics                    │
└────────────────────────────────────────────┘
        │
        ↓
┌────────────────────────────────────────────┐
│  FILE STORAGE & CDN                        │
│  • S3: File storage                        │
│  • CDN: File delivery                      │
└────────────────────────────────────────────┘
```

---

## Core Implementation

### 1. Message Service with Threads

```javascript
async function sendChannelMessage(userId, channelId, messageText, attachments = []) {
    const messageId = generateId();
    const timestamp = Date.now();

    try {
        // 1. Verify user has access to channel
        const hasAccess = await postgres.query(`
            SELECT 1 FROM channel_members
            WHERE channel_id = ? AND user_id = ?
        `, [channelId, userId]);

        if (!hasAccess) {
            throw new Error('No access to channel');
        }

        // 2. Create message
        const message = {
            message_id: messageId,
            channel_id: channelId,
            user_id: userId,
            text: messageText,
            attachments: attachments,
            created_at: timestamp,
            reactions: {},  // emoji: [user_ids]
            thread_reply_count: 0
        };

        // 3. Store in Cassandra (indexed by channel_id and created_at)
        await cassandra.insert('channel_messages', message);

        // 4. Index in Elasticsearch
        await elasticsearch.index({
            index: `messages_${getWorkspaceId(channelId)}`,
            id: messageId,
            body: {
                channel_id: channelId,
                user_id: userId,
                text: messageText,
                created_at: timestamp
            }
        });

        // 5. Update channel metadata
        await postgres.query(`
            UPDATE channels SET
            last_message_id = ?,
            message_count = message_count + 1,
            updated_at = ?
            WHERE channel_id = ?
        `, [messageId, timestamp, channelId]);

        // 6. Publish to Kafka
        await kafka.publish('messages', {
            event: 'message_sent',
            message_id: messageId,
            channel_id: channelId,
            user_id: userId,
            timestamp: timestamp
        });

        // 7. Send to all connected users in channel
        const connectedUsers = await getConnectedUsersInChannel(channelId);
        for (let userId of connectedUsers) {
            await webSocketManager.sendToUser(userId, {
                type: 'MESSAGE',
                message: message
            });
        }

        return message;
    } catch (error) {
        console.error('Send message error:', error);
        throw error;
    }
}

// Reply to a message (thread)
async function replyToMessage(userId, channelId, parentMessageId, replyText) {
    const replyId = generateId();
    const timestamp = Date.now();

    try {
        // 1. Create reply
        const reply = {
            message_id: replyId,
            channel_id: channelId,
            parent_message_id: parentMessageId,
            user_id: userId,
            text: replyText,
            created_at: timestamp
        };

        // 2. Store in Cassandra
        await cassandra.insert('message_replies', reply);

        // 3. Update parent message reply count
        await cassandra.insert('message_thread_count', {
            parent_message_id: parentMessageId,
            reply_count: (await getThreadReplyCount(parentMessageId)) + 1
        });

        // 4. Index reply
        await elasticsearch.index({
            index: `messages_${getWorkspaceId(channelId)}`,
            id: replyId,
            body: {
                channel_id: channelId,
                parent_message_id: parentMessageId,
                user_id: userId,
                text: replyText,
                created_at: timestamp
            }
        });

        // 5. Notify users in thread
        const threadUsers = await getThreadParticipants(parentMessageId);
        for (let threadUserId of threadUsers) {
            await webSocketManager.sendToUser(threadUserId, {
                type: 'THREAD_REPLY',
                parent_message_id: parentMessageId,
                reply: reply
            });
        }

        return reply;
    } catch (error) {
        console.error('Reply error:', error);
        throw error;
    }
}
```

### 2. Full-Text Search

```javascript
async function searchMessages(workspaceId, query, filters = {}) {
    const {
        channel_ids,
        user_ids,
        from_date,
        to_date,
        has_attachments,
        page = 1,
        limit = 20
    } = filters;

    const from = (page - 1) * limit;

    const searchBody = {
        query: {
            bool: {
                must: [
                    {
                        multi_match: {
                            query: query,
                            fields: ['text^3', 'attachments.filename'],
                            fuzziness: 'AUTO',
                            operator: 'or'
                        }
                    }
                ],
                filter: []
            }
        },
        from: from,
        size: limit,
        highlight: {
            fields: {
                text: {}
            }
        }
    };

    // Add optional filters
    if (channel_ids && channel_ids.length > 0) {
        searchBody.query.bool.filter.push({
            terms: { channel_id: channel_ids }
        });
    }

    if (user_ids && user_ids.length > 0) {
        searchBody.query.bool.filter.push({
            terms: { user_id: user_ids }
        });
    }

    if (from_date || to_date) {
        const dateRange = {};
        if (from_date) dateRange.gte = from_date;
        if (to_date) dateRange.lte = to_date;
        searchBody.query.bool.filter.push({
            range: { created_at: dateRange }
        });
    }

    if (has_attachments) {
        searchBody.query.bool.filter.push({
            exists: { field: 'attachments' }
        });
    }

    // Execute search
    const results = await elasticsearch.search({
        index: `messages_${workspaceId}`,
        body: searchBody
    });

    return {
        results: results.hits.hits.map(hit => ({
            ...hit._source,
            highlight: hit.highlight
        })),
        total: results.hits.total.value,
        page: page,
        pages: Math.ceil(results.hits.total.value / limit)
    };
}
```

---

---

# Question 9: Design Airbnb

## Problem Statement
Design a property rental platform like Airbnb where:
- Hosts can list properties
- Guests can search and book properties
- Booking with date availability
- Payment processing
- Reviews and ratings
- Messaging between hosts and guests
- Map-based search with location
- Availability calendar

### Scale Requirements
```
Assumptions:
- 7 million listings worldwide
- 500 million monthly active users
- 100 million monthly bookings
- Peak search QPS: 100,000

Calculations:
Bookings per second = 100M / (30 × 86,400) ≈ 39 TPS
Search QPS = 100,000
List updates per second = ≈ 500 (hosts updating calendars)

Critical Requirements:
✓ Real-time availability
✓ Geospatial search (nearby properties)
✓ Calendar accuracy (no double bookings)
✓ Global scale (multiple currencies, languages)
```

---

## High-Level Architecture

```
┌────────────────────────────────────────────┐
│         CLIENT TIER                        │
│  Web │ Mobile (iOS/Android)                │
└────────────┬─────────────────────────────────┘
             │
             ↓
┌────────────────────────────────────────────┐
│      API GATEWAY / LOAD BALANCER           │
│  • Rate limiting                           │
│  • Authentication                          │
│  • Request routing                         │
└────────────┬────────────────────────────────┘
             │
    ┌────────┼────────┐
    ↓        ↓        ↓
┌────────────────────────────────────────────┐
│         SERVICES                           │
│  Listing Service   │ Booking Service      │
│  Search Service    │ Payment Service      │
│  Availability Svc  │ Review Service       │
│  Messaging Service │ Analytics Service    │
└────────┬────────────────────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
┌────────────────────────────────────────────┐
│  Cache & Message Queue                    │
│  • Redis: Listings, Availability          │
│  • Kafka: Booking events                  │
│  • Redis Geo: Location index               │
└────────┬─────────────────────────────────┘
         │
    ┌────┼────────────┐
    ↓    ↓            ↓
┌────────────────────────────────────────────┐
│  DATABASE LAYER                            │
│  PostgreSQL: Users, Listings, Bookings    │
│  MongoDB: Reviews, Messages               │
│  Cassandra: Availability calendars        │
│  Elasticsearch: Search & filters          │
│  PostGIS: Geospatial queries              │
└────────────────────────────────────────────┘
        │
        ↓
┌────────────────────────────────────────────┐
│  FILE STORAGE & CDN                        │
│  • S3: Property images                     │
│  • CDN: Image delivery                     │
└────────────────────────────────────────────┘
```

---

## Core Implementation

### 1. Geospatial Search

```javascript
// Index property with geo location
async function indexProperty(propertyId, latitude, longitude, propertyData) {
    // 1. Store geo location in Redis (for fast geo queries)
    await redis.geoadd(
        'properties',
        longitude,
        latitude,
        propertyId
    );

    // 2. Store full property data in PostgreSQL with PostGIS
    await postgres.query(`
        INSERT INTO listings (listing_id, host_id, name, location, geom, price, ...)
        VALUES (?, ?, ?, ?, ST_PointFromText('POINT(? ?)'), ?, ...)
    `, [propertyId, propertyData.host_id, propertyData.name, ..., longitude, latitude, ...]);

    // 3. Index for full-text search
    await elasticsearch.index({
        index: 'listings',
        id: propertyId,
        body: {
            listing_id: propertyId,
            name: propertyData.name,
            description: propertyData.description,
            address: propertyData.address,
            latitude: latitude,
            longitude: longitude,
            price: propertyData.price,
            amenities: propertyData.amenities,
            bedrooms: propertyData.bedrooms
        }
    });
}

// Search nearby properties
async function searchNearbyListings(latitude, longitude, radiusKm = 5, filters = {}) {
    // 1. Get nearby properties using Redis Geo
    const nearbyProperties = await redis.georadius(
        'properties',
        longitude,
        latitude,
        radiusKm,
        'km',
        'WITHDIST'
    );

    if (nearbyProperties.length === 0) {
        return [];
    }

    const listingIds = nearbyProperties.map(p => p[0]);

    // 2. Apply filters using Elasticsearch
    const searchQuery = {
        bool: {
            must: [
                {
                    terms: { listing_id: listingIds }
                }
            ],
            filter: []
        }
    };

    // Add filters
    if (filters.min_price || filters.max_price) {
        searchQuery.bool.filter.push({
            range: {
                price: {
                    gte: filters.min_price,
                    lte: filters.max_price
                }
            }
        });
    }

    if (filters.bedrooms) {
        searchQuery.bool.filter.push({
            term: { bedrooms: filters.bedrooms }
        });
    }

    if (filters.amenities && filters.amenities.length > 0) {
        searchQuery.bool.filter.push({
            terms: { amenities: filters.amenities }
        });
    }

    if (filters.check_in && filters.check_out) {
        // Check availability (will do in next section)
        // Add to filter
    }

    // 3. Search
    const results = await elasticsearch.search({
        index: 'listings',
        body: {
            query: searchQuery,
            sort: [
                {
                    _geo_distance: {
                        location: { lat: latitude, lon: longitude },
                        order: 'asc',
                        unit: 'km'
                    }
                }
            ],
            size: 50
        }
    });

    return results.hits.hits.map(hit => ({
        ...hit._source,
        distance_km: nearbyProperties.find(p => p[0] === hit._source.listing_id)[1]
    }));
}
```

### 2. Availability Management

```javascript
// Check availability for date range
async function checkAvailability(listingId, checkInDate, checkOutDate) {
    const dateKey = `availability:${listingId}`;

    // 1. Get blocked dates from Redis (cache)
    const blockedDates = await redis.get(dateKey);

    if (blockedDates) {
        const blocked = JSON.parse(blockedDates);
        const checkInTime = new Date(checkInDate).getTime();
        const checkOutTime = new Date(checkOutDate).getTime();

        for (let date of blocked) {
            const blockTime = new Date(date).getTime();
            if (blockTime >= checkInTime && blockTime < checkOutTime) {
                return false;  // Not available
            }
        }
        return true;
    }

    // 2. Query Cassandra (source of truth)
    const bookings = await cassandra.query(`
        SELECT check_in_date, check_out_date
        FROM listings_availability
        WHERE listing_id = ? AND status = 'BOOKED'
    `, [listingId]);

    for (let booking of bookings) {
        const bookingStart = new Date(booking.check_in_date).getTime();
        const bookingEnd = new Date(booking.check_out_date).getTime();
        const checkInTime = new Date(checkInDate).getTime();
        const checkOutTime = new Date(checkOutDate).getTime();

        // Check for overlap
        if (checkInTime < bookingEnd && checkOutTime > bookingStart) {
            return false;  // Overlaps with existing booking
        }
    }

    // 3. Cache result
    await redis.setex(dateKey, 3600, JSON.stringify(blockedDates));

    return true;
}

// Block dates (when booking confirmed)
async function blockAvailability(listingId, checkInDate, checkOutDate, bookingId) {
    try {
        // 1. Store in Cassandra
        const booking = {
            listing_id: listingId,
            booking_id: bookingId,
            check_in_date: checkInDate,
            check_out_date: checkOutDate,
            status: 'BOOKED',
            created_at: Date.now()
        };

        await cassandra.insert('listings_availability', booking);

        // 2. Invalidate cache
        await redis.delete(`availability:${listingId}`);

        // 3. Update PostgreSQL (for quick lookup)
        await postgres.query(`
            UPDATE listings SET
            available_from = ?,
            available_to = ?
            WHERE listing_id = ?
        `, [checkOutDate, getNextAvailableDate(listingId), listingId]);

        return { success: true };
    } catch (error) {
        console.error('Block availability error:', error);
        throw error;
    }
}
```

### 3. Booking Service

```javascript
async function createBooking(guestId, listingId, checkInDate, checkOutDate) {
    const bookingId = generateId();

    try {
        // 1. Check availability (with lock)
        const available = await checkAndLockAvailability(listingId, checkInDate, checkOutDate);

        if (!available) {
            throw new Error('Property not available for selected dates');
        }

        // 2. Calculate price
        const nights = (new Date(checkOutDate) - new Date(checkInDate)) / (1000 * 60 * 60 * 24);
        const property = await postgres.query(
            'SELECT price_per_night, cleaning_fee, service_fee FROM listings WHERE listing_id = ?',
            [listingId]
        );

        const totalPrice = (property[0].price_per_night * nights) +
                          property[0].cleaning_fee +
                          property[0].service_fee;

        // 3. Create booking record
        const booking = {
            booking_id: bookingId,
            guest_id: guestId,
            listing_id: listingId,
            check_in_date: checkInDate,
            check_out_date: checkOutDate,
            total_price: totalPrice,
            currency: property[0].currency,
            status: 'PENDING_PAYMENT',
            created_at: Date.now()
        };

        await postgres.insert('bookings', booking);

        // 4. Process payment
        const paymentResult = await processPayment(guestId, totalPrice, bookingId);

        if (!paymentResult.success) {
            throw new Error('Payment failed');
        }

        // 5. Confirm booking
        await postgres.query(
            'UPDATE bookings SET status = ? WHERE booking_id = ?',
            ['CONFIRMED', bookingId]
        );

        // 6. Block availability
        await blockAvailability(listingId, checkInDate, checkOutDate, bookingId);

        // 7. Send notifications
        const host = await getListingHost(listingId);
        await notifyHost(host.user_id, { booking: booking });
        await notifyGuest(guestId, { booking: booking });

        // 8. Publish event
        await kafka.publish('booking-events', {
            event: 'booking_created',
            booking_id: bookingId,
            guest_id: guestId,
            host_id: host.user_id,
            listing_id: listingId
        });

        return booking;
    } catch (error) {
        console.error('Booking error:', error);
        // Release lock on availability
        await releaseAvailabilityLock(listingId, checkInDate, checkOutDate);
        throw error;
    }
}
```

---

---

# Question 10: Design Rate Limiter

## Problem Statement
Design a rate limiter that:
- Limits requests per user/API key
- Supports different limit strategies (per minute, hour, day)
- Works across distributed systems
- Returns 429 Too Many Requests when limit exceeded
- Provides clear rate limit headers in response

### Requirements
```
Assumptions:
- 1 million API keys
- 1 million requests per second
- Need per-user and global limits
- Different limits for different API tiers

Challenges:
✓ Distributed: Multiple servers, same limits
✓ Fast: No round-trip delays
✓ Accurate: No lost or duplicate counts
✓ Scalable: Handle spike traffic
```

---

## Implementation Approaches

### Approach 1: Token Bucket Algorithm (Recommended)

```javascript
class TokenBucket {
    constructor(capacity, refillRate) {
        this.capacity = capacity;  // Max tokens
        this.tokens = capacity;    // Current tokens
        this.refillRate = refillRate;  // Tokens per second
        this.lastRefillTime = Date.now();
    }

    // Add tokens based on time elapsed
    refill() {
        const now = Date.now();
        const timePassed = (now - this.lastRefillTime) / 1000;  // seconds
        const tokensToAdd = timePassed * this.refillRate;

        this.tokens = Math.min(
            this.capacity,
            this.tokens + tokensToAdd
        );

        this.lastRefillTime = now;
    }

    // Try to consume a token
    tryConsume(tokens = 1) {
        this.refill();

        if (this.tokens >= tokens) {
            this.tokens -= tokens;
            return true;
        }

        return false;
    }
}

// Distributed Token Bucket using Redis
async function rateLimitTokenBucket(userId, limit = 100, windowSeconds = 60) {
    const key = `rate_limit:${userId}`;
    const now = Date.now() / 1000;  // Current time in seconds

    try {
        // 1. Get or create bucket
        let bucket = await redis.get(key);

        if (!bucket) {
            bucket = {
                tokens: limit,
                lastRefillTime: now
            };
        } else {
            bucket = JSON.parse(bucket);

            // Refill tokens based on time elapsed
            const timePassed = now - bucket.lastRefillTime;
            const tokensToAdd = timePassed * (limit / windowSeconds);

            bucket.tokens = Math.min(limit, bucket.tokens + tokensToAdd);
            bucket.lastRefillTime = now;
        }

        // 2. Try to consume 1 token
        if (bucket.tokens >= 1) {
            bucket.tokens -= 1;

            // 3. Update in Redis with TTL
            await redis.setex(key, windowSeconds * 2, JSON.stringify(bucket));

            return {
                allowed: true,
                remaining: Math.floor(bucket.tokens),
                resetTime: bucket.lastRefillTime + windowSeconds
            };
        } else {
            // Rate limit exceeded
            const resetTime = bucket.lastRefillTime + windowSeconds;
            const waitTime = Math.ceil(resetTime - now);

            return {
                allowed: false,
                remaining: 0,
                retryAfter: waitTime,
                resetTime: resetTime
            };
        }
    } catch (error) {
        console.error('Rate limit check error:', error);
        // Fail open (allow request if we can't check)
        return { allowed: true };
    }
}

// Middleware for Express
app.use(async (req, res, next) => {
    const userId = req.user.id || req.ip;
    const tier = req.user.tier || 'free';

    // Different limits for different tiers
    const limits = {
        'free': { requests: 100, window: 3600 },      // 100/hour
        'pro': { requests: 10000, window: 3600 },     // 10K/hour
        'enterprise': { requests: Infinity, window: 3600 }  // Unlimited
    };

    const limit = limits[tier];
    const result = await rateLimitTokenBucket(userId, limit.requests, limit.window);

    // Set rate limit headers
    res.set('X-RateLimit-Limit', limit.requests.toString());
    res.set('X-RateLimit-Remaining', result.remaining.toString());
    res.set('X-RateLimit-Reset', new Date(result.resetTime * 1000).toISOString());

    if (!result.allowed) {
        res.set('Retry-After', result.retryAfter.toString());
        return res.status(429).json({
            error: 'Too Many Requests',
            message: `Rate limit exceeded. Retry after ${result.retryAfter} seconds`,
            retry_after: result.retryAfter
        });
    }

    next();
});
```

### Approach 2: Sliding Window Log

```javascript
// Log every request, check if within limit
async function rateLimitSlidingWindow(userId, limit = 100, windowSeconds = 60) {
    const key = `rate_limit:${userId}`;
    const now = Date.now();
    const windowStart = now - (windowSeconds * 1000);

    try {
        // 1. Count requests in the last window
        const count = await redis.zcount(
            key,
            windowStart,
            now
        );

        if (count < limit) {
            // 2. Add current request to sorted set
            await redis.zadd(key, now, `${now}_${Math.random()}`);

            // 3. Clean up old entries
            await redis.zremrangebyscore(key, 0, windowStart);

            // 4. Set expiration
            await redis.expire(key, windowSeconds * 2);

            return {
                allowed: true,
                remaining: limit - count - 1,
                resetTime: new Date(now + (windowSeconds * 1000))
            };
        } else {
            return {
                allowed: false,
                remaining: 0,
                retryAfter: Math.ceil((windowSeconds * 1000) / 1000)
            };
        }
    } catch (error) {
        console.error('Rate limit error:', error);
        return { allowed: true };
    }
}
```

### Approach 3: Fixed Window Counter (Simplest)

```javascript
async function rateLimitFixedWindow(userId, limit = 100, windowSeconds = 60) {
    const now = Math.floor(Date.now() / 1000);
    const windowKey = Math.floor(now / windowSeconds);
    const key = `rate_limit:${userId}:${windowKey}`;

    try {
        // 1. Increment counter
        const count = await redis.incr(key);

        // 2. Set expiration on first request
        if (count === 1) {
            await redis.expire(key, windowSeconds);
        }

        if (count <= limit) {
            return {
                allowed: true,
                remaining: limit - count,
                resetTime: new Date((windowKey + 1) * windowSeconds * 1000)
            };
        } else {
            return {
                allowed: false,
                remaining: 0,
                resetTime: new Date((windowKey + 1) * windowSeconds * 1000)
            };
        }
    } catch (error) {
        console.error('Rate limit error:', error);
        return { allowed: true };
    }
}
```

---

## Distributed Rate Limiting

```javascript
// Lua script for atomic operations across distributed systems
const luaScript = `
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

-- Count requests in window
local count = redis.call('ZCOUNT', key, now - (window * 1000), now)

if count < limit then
    -- Add current request
    redis.call('ZADD', key, now, now .. '_' .. math.random())

    -- Clean old entries
    redis.call('ZREMRANGEBYSCORE', key, 0, now - (window * 1000))

    -- Set expiration
    redis.call('EXPIRE', key, window * 2)

    return {1, limit - count - 1}  -- Allowed, remaining
else
    return {0, 0}  -- Not allowed
end
`;

async function rateLimitAtomic(userId, limit = 100, windowSeconds = 60) {
    const key = `rate_limit:${userId}`;
    const now = Date.now();

    try {
        // Execute Lua script atomically
        const result = await redis.eval(
            luaScript,
            1,            // Number of keys
            key,          // Key
            limit,        // Limit
            windowSeconds,// Window
            now           // Current time
        );

        const [allowed, remaining] = result;

        return {
            allowed: allowed === 1,
            remaining: remaining,
            resetTime: new Date(now + (windowSeconds * 1000))
        };
    } catch (error) {
        console.error('Rate limit error:', error);
        return { allowed: true };  // Fail open
    }
}
```

---

## Comparison of Approaches

| Approach | Pros | Cons | Best For |
|----------|------|------|----------|
| **Token Bucket** | Allows bursts, smooth | Complexity | API rate limiting |
| **Sliding Window** | Accurate, fair | Memory intensive | Precise limiting |
| **Fixed Window** | Simple, fast | Boundary issues | Basic limiting |

---

**Remember:** Choose token bucket for most APIs because it:
- Allows brief bursts (good UX)
- Is simple to understand
- Works well at scale
- Matches user expectations

---

---

# Question 11: Design Google Docs (Collaborative Editing)

## Problem Statement
Design a collaborative document editing platform like Google Docs where:
- Multiple users can edit the same document simultaneously
- Real-time synchronization across clients
- Conflict resolution for concurrent edits
- Version history and rollback capability
- Comments and suggestions
- Auto-save functionality
- Offline editing with sync

### Scale Requirements
```
Assumptions:
- 1.5 billion monthly active users
- 100 million documents edited daily
- 10 million concurrent editing sessions
- Average document: 50KB

Calculations:
Concurrent editors per document = 5-10 average
Edit operations per second = ~100K (globally)
Sync operations per second = ~500K

Critical Requirements:
✓ Real-time sync (< 100ms latency)
✓ Conflict-free collaborative editing (CRDT)
✓ Full edit history
✓ 99.9% availability
```

---

## Key Challenges

### Challenge 1: Conflict Resolution
```
User A                          User B
├─ Selects "Hello"             ├─ Selects "World"
├─ Types at position 0         ├─ Types at position 5
└─ Result: "Hi Hello World"     └─ Result: "HelloWorld Hi"

Which is correct? Need CRDT (Conflict-free Replicated Data Type)
```

### Challenge 2: Real-Time Synchronization

```javascript
// Using Operational Transformation (OT) or CRDT
async function handleDocumentEdit(userId, docId, operation) {
    // 1. Receive operation from client
    // Example: Insert "text" at position 10

    // 2. Transform against concurrent operations
    const transformedOp = await transformOperation(operation, concurrentOps);

    // 3. Apply to document
    const newDoc = applyOperation(currentDoc, transformedOp);

    // 4. Store in database
    await postgres.insert('document_operations', {
        doc_id: docId,
        user_id: userId,
        operation: transformedOp,
        timestamp: Date.now(),
        version: currentVersion + 1
    });

    // 5. Broadcast to all clients
    for (let client of getConnectedClients(docId)) {
        if (client.userId !== userId) {
            webSocket.sendToClient(client, {
                type: 'OPERATION',
                operation: transformedOp,
                version: currentVersion + 1
            });
        }
    }

    return { success: true, version: currentVersion + 1 };
}
```

### Implementation Approach

```
CRDT (Conflict-free Replicated Data Type):
├─ Each character has unique ID (timestamp + userId)
├─ Operations are commutative (order doesn't matter)
├─ No coordination needed
└─ Example: Yjs, Automerge libraries

Operational Transformation (OT):
├─ Transform operations against concurrent changes
├─ Requires central server coordination
├─ Google Docs uses this approach
└─ More complex but mature
```

### Database Design for Version History

```sql
CREATE TABLE documents (
    doc_id BIGINT PRIMARY KEY,
    title VARCHAR(255),
    owner_id BIGINT,
    content TEXT,
    current_version INT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    FOREIGN KEY (owner_id) REFERENCES users(user_id)
);

CREATE TABLE document_operations (
    operation_id BIGINT PRIMARY KEY,
    doc_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    operation_type VARCHAR(50),  -- INSERT, DELETE, REPLACE
    position INT,
    content TEXT,
    version INT,
    timestamp TIMESTAMP,
    FOREIGN KEY (doc_id) REFERENCES documents(doc_id),
    INDEX (doc_id, version)
);

CREATE TABLE document_versions (
    version_id BIGINT PRIMARY KEY,
    doc_id BIGINT NOT NULL,
    content TEXT,
    version_number INT,
    created_at TIMESTAMP,
    created_by BIGINT,
    FOREIGN KEY (doc_id) REFERENCES documents(doc_id),
    INDEX (doc_id, version_number)
);
```

---

---

# Question 12: Design Dropbox (File Storage & Sync)

## Problem Statement
Design a file storage and sync service like Dropbox where:
- Users can upload and store files
- Automatic sync across devices
- File versioning
- Sharing files/folders with others
- Real-time notifications
- Conflict resolution (conflicted copies)
- Bandwidth optimization

### Scale Requirements
```
Assumptions:
- 500 million monthly active users
- 1 trillion files stored
- 100 million daily active users
- Peak uploads: 100K files/second

Calculations:
Average file size: 1MB
Total storage: 1 trillion × 1MB = 1 Exabyte

Upload throughput = 100K files/sec × 1MB = 100GB/sec
Download throughput = similar

Critical Requirements:
✓ Minimize bandwidth (delta sync)
✓ Fast sync (< 1 second)
✓ 99.99% availability
✓ Strong consistency for user's own files
```

---

## Core Features

### 1. Delta Sync (Bandwidth Optimization)

```javascript
// Smart sync: Only upload changed blocks
async function syncFile(userId, filePath, fileContent) {
    // 1. Get file metadata
    const localFile = {
        path: filePath,
        size: fileContent.length,
        hash: sha256(fileContent),
        lastModified: Date.now()
    };

    // 2. Check if file already exists on server
    const remoteFile = await getRemoteFileMetadata(userId, filePath);

    if (!remoteFile) {
        // New file: upload entire content
        return await uploadNewFile(userId, localFile, fileContent);
    }

    // 3. Compare hashes - if same, no need to sync
    if (localFile.hash === remoteFile.hash) {
        return { synced: true, status: 'UP_TO_DATE' };
    }

    // 4. File changed: Use chunking and delta sync
    const chunks = createChunks(fileContent, CHUNK_SIZE);
    const remoteChunks = await getRemoteChunks(userId, filePath);

    const changedChunks = chunks
        .map((chunk, index) => ({
            index: index,
            chunk: chunk,
            hash: sha256(chunk),
            remoteHash: remoteChunks[index]?.hash
        }))
        .filter(c => c.hash !== c.remoteHash);

    // 5. Upload only changed chunks
    for (let changedChunk of changedChunks) {
        await uploadChunk(userId, filePath, changedChunk);
    }

    // 6. Update metadata
    await updateFileMetadata(userId, filePath, localFile);

    return { synced: true, chunksUploaded: changedChunks.length };
}
```

### 2. Conflict Resolution

```javascript
async function syncConflict(userId, filePath, localVersion, remoteVersion) {
    // 1. Detect if file was modified on both sides
    const localModTime = localVersion.lastModified;
    const remoteModTime = remoteVersion.lastModified;

    // 2. If one is clearly newer, use it
    if (localModTime > remoteModTime + CONFLICT_THRESHOLD) {
        return await overwriteRemote(userId, filePath, localVersion);
    }

    if (remoteModTime > localModTime + CONFLICT_THRESHOLD) {
        return await overwriteLocal(userId, filePath, remoteVersion);
    }

    // 3. Actual conflict: Create conflicted copy
    const conflictedPath = `${filePath} (conflicted copy ${Date.now()})`;

    await createConflictedCopy(userId, conflictedPath, localVersion);
    await updateLocalFile(userId, filePath, remoteVersion);

    // 4. Notify user
    await notifyUser(userId, {
        type: 'CONFLICT',
        message: `File "${filePath}" has conflicting changes`,
        original: filePath,
        conflicted: conflictedPath
    });

    return { status: 'CONFLICT_RESOLVED', conflictedCopy: conflictedPath };
}
```

### 3. File Sharing

```javascript
async function shareFile(ownerId, filePath, sharedWithUserIds, permissions) {
    const shareId = generateId();

    // 1. Create share record
    const share = {
        share_id: shareId,
        owner_id: ownerId,
        file_path: filePath,
        shared_with: sharedWithUserIds,
        permissions: permissions,  // READ, WRITE, ADMIN
        created_at: Date.now(),
        expires_at: null
    };

    await postgres.insert('file_shares', share);

    // 2. Notify shared users
    for (let userId of sharedWithUserIds) {
        await notifyUser(userId, {
            type: 'FILE_SHARED',
            from: ownerId,
            file: filePath,
            permissions: permissions,
            link: `https://dropbox.example.com/s/${shareId}`
        });
    }

    // 3. Cache share metadata for quick access
    await redis.setex(
        `share:${shareId}`,
        86400,
        JSON.stringify(share)
    );

    return { shareId: shareId, link: `https://dropbox.example.com/s/${shareId}` };
}
```

---

---

# Question 13: Design Notification System

## Problem Statement
Design a notification system that:
- Sends notifications across multiple channels (email, SMS, push, in-app)
- High throughput (millions of notifications/minute)
- Reliable delivery with retries
- User preferences and opt-out
- Real-time delivery
- Idempotent (no duplicates)

### Scale Requirements
```
Assumptions:
- 500 million daily active users
- 10 billion notifications per day
- Peak: 500K notifications per second
- Multiple channels per notification

Calculations:
10B notifications/day ÷ 86,400 seconds = ~116K RPS
With multiple channels, actual RPS = 116K × 3 = ~350K RPS
Peak = 350K × 3 = ~1M RPS

Critical Requirements:
✓ 99.99% delivery rate
✓ Sub-second latency
✓ Handle failures gracefully
✓ Cost optimization
```

---

## Architecture

```javascript
// Notification Service
async function sendNotification(userId, notification) {
    const notificationId = generateId();
    const timestamp = Date.now();

    try {
        // 1. Validate user preferences
        const userPrefs = await getUserNotificationPreferences(userId);
        if (!userPrefs.notifications_enabled) {
            return { status: 'SKIPPED', reason: 'USER_OPTED_OUT' };
        }

        // 2. Store notification record
        const record = {
            notification_id: notificationId,
            user_id: userId,
            title: notification.title,
            body: notification.body,
            type: notification.type,
            status: 'PENDING',
            created_at: timestamp,
            delivery_attempts: 0,
            max_retries: 3
        };

        await postgres.insert('notifications', record);

        // 3. Determine channels based on preferences
        const channels = getPreferredChannels(userPrefs);

        // 4. Send via each channel
        const results = {};
        for (let channel of channels) {
            results[channel] = await sendViaChannel(
                notificationId,
                userId,
                notification,
                channel
            );
        }

        // 5. Update status
        const allSucceeded = Object.values(results).every(r => r.success);
        await postgres.query(
            'UPDATE notifications SET status = ? WHERE notification_id = ?',
            [allSucceeded ? 'DELIVERED' : 'PARTIAL', notificationId]
        );

        // 6. Publish event
        await kafka.publish('notification-events', {
            event: 'notification_sent',
            notification_id: notificationId,
            user_id: userId,
            channels: results
        });

        return { status: 'SENT', results: results };
    } catch (error) {
        console.error('Notification error:', error);

        // Queue for retry
        await redis.lpush(`retry_queue:${userId}`, JSON.stringify(record));

        throw error;
    }
}

// Send via individual channels
async function sendViaChannel(notificationId, userId, notification, channel) {
    try {
        switch (channel) {
            case 'PUSH':
                return await sendPushNotification(notificationId, userId, notification);
            case 'EMAIL':
                return await sendEmailNotification(notificationId, userId, notification);
            case 'SMS':
                return await sendSMSNotification(notificationId, userId, notification);
            case 'IN_APP':
                return await sendInAppNotification(notificationId, userId, notification);
        }
    } catch (error) {
        console.error(`Failed to send via ${channel}:`, error);
        return { success: false, channel: channel, error: error.message };
    }
}

// Idempotent delivery with deduplication
async function sendEmailNotification(notificationId, userId, notification) {
    const dedupeKey = `email:${userId}:${notificationId}`;

    // Check if already sent
    const alreadySent = await redis.exists(dedupeKey);
    if (alreadySent) {
        return { success: true, cached: true };
    }

    try {
        // Send email
        const result = await sendGridAPI.send({
            to: await getUserEmail(userId),
            subject: notification.title,
            html: renderEmailTemplate(notification),
            custom_args: {
                notification_id: notificationId,
                user_id: userId
            }
        });

        // Mark as sent (idempotent)
        await redis.setex(dedupeKey, 86400, '1');

        return { success: true, messageId: result.id };
    } catch (error) {
        // Retry logic
        await queueForRetry(notificationId, 'EMAIL');
        throw error;
    }
}

// Retry mechanism
async function retryFailedNotifications() {
    const retryQueue = await redis.keys('retry_queue:*');

    for (let queueKey of retryQueue) {
        const notifications = await redis.lrange(queueKey, 0, -1);

        for (let notifStr of notifications) {
            const notification = JSON.parse(notifStr);

            if (notification.delivery_attempts >= notification.max_retries) {
                // Give up after max retries
                await redis.lrem(queueKey, 1, notifStr);
                await updateNotificationStatus(notification.notification_id, 'FAILED');
                continue;
            }

            // Exponential backoff: retry with increasing delays
            const backoffMs = Math.pow(2, notification.delivery_attempts) * 1000;
            const nextRetry = notification.last_attempt + backoffMs;

            if (Date.now() >= nextRetry) {
                try {
                    await sendNotification(notification.user_id, notification);
                    await redis.lrem(queueKey, 1, notifStr);
                } catch (error) {
                    notification.delivery_attempts++;
                    notification.last_attempt = Date.now();
                    // Keep in queue for retry
                }
            }
        }
    }
}

// Run retry job every 5 minutes
setInterval(retryFailedNotifications, 5 * 60 * 1000);
```

---

---

# Question 14: Design Distributed Cache (Redis-like)

## Problem Statement
Design a distributed caching system like Redis where:
- In-memory key-value storage
- Multiple data types (strings, lists, sets, hashes, sorted sets)
- TTL (time-to-live) for keys
- Persistence options
- Replication and failover
- Eviction policies
- Pub/Sub messaging

### Scale Requirements
```
Assumptions:
- 1 trillion cached items
- 1 million read operations/second
- 100K write operations/second
- Average value size: 1KB

Calculations:
Read latency target: < 1ms
Memory: 1 trillion items × 1KB = 1 Exabyte
But in practice: Distributed cache with sharding

Critical Requirements:
✓ Sub-millisecond latency
✓ High throughput
✓ Data persistence
✓ Horizontal scalability
```

---

## Core Implementation

```javascript
// Simple in-memory cache (single node)
class RedisLike {
    constructor() {
        this.store = new Map();  // key -> value
        this.ttl = new Map();    // key -> expiry timestamp
        this.subscriptions = new Map();  // channel -> [callbacks]
    }

    // Basic GET
    get(key) {
        if (!this.store.has(key)) {
            return null;
        }

        // Check if expired
        if (this.ttl.has(key) && this.ttl.get(key) < Date.now()) {
            this.store.delete(key);
            this.ttl.delete(key);
            return null;
        }

        return this.store.get(key);
    }

    // Basic SET
    set(key, value, expiryMs = null) {
        this.store.set(key, value);

        if (expiryMs) {
            this.ttl.set(key, Date.now() + expiryMs);
        }

        return 'OK';
    }

    // List operations
    lpush(key, ...values) {
        let list = this.get(key) || [];
        if (!Array.isArray(list)) {
            throw new Error('WRONGTYPE Operation against a key holding the wrong kind of value');
        }

        list.unshift(...values);
        this.set(key, list);
        return list.length;
    }

    lpop(key) {
        let list = this.get(key);
        if (!list || !Array.isArray(list)) return null;

        const value = list.shift();
        this.set(key, list);
        return value;
    }

    // Sorted set operations (for leaderboards, priority queues)
    zadd(key, score, member) {
        let sortedSet = this.get(key) || [];
        sortedSet.push({ score, member });
        sortedSet.sort((a, b) => a.score - b.score);

        this.set(key, sortedSet);
        return sortedSet.length;
    }

    zrange(key, start, end) {
        let sortedSet = this.get(key) || [];
        return sortedSet.slice(start, end + 1);
    }

    // TTL management
    expire(key, seconds) {
        if (!this.store.has(key)) return 0;
        this.ttl.set(key, Date.now() + (seconds * 1000));
        return 1;
    }

    // Pub/Sub
    subscribe(channel, callback) {
        if (!this.subscriptions.has(channel)) {
            this.subscriptions.set(channel, []);
        }
        this.subscriptions.get(channel).push(callback);
    }

    publish(channel, message) {
        const subscribers = this.subscriptions.get(channel) || [];
        for (let callback of subscribers) {
            callback(message);
        }
        return subscribers.length;
    }

    // Cleanup expired keys (background job)
    cleanupExpiredKeys() {
        const now = Date.now();
        for (let [key, expiry] of this.ttl.entries()) {
            if (expiry < now) {
                this.store.delete(key);
                this.ttl.delete(key);
            }
        }
    }
}

// Distributed Redis (with sharding)
class DistributedRedis {
    constructor(nodes = 3) {
        this.nodes = Array(nodes).fill(null).map(() => new RedisLike());
        this.nodeCount = nodes;
    }

    // Consistent hashing to route to correct node
    getNodeForKey(key) {
        const hash = this.hashKey(key);
        const nodeIndex = hash % this.nodeCount;
        return this.nodes[nodeIndex];
    }

    hashKey(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = ((hash << 5) - hash) + key.charCodeAt(i);
            hash = hash & hash;
        }
        return Math.abs(hash);
    }

    get(key) {
        const node = this.getNodeForKey(key);
        return node.get(key);
    }

    set(key, value, expiryMs) {
        const node = this.getNodeForKey(key);
        return node.set(key, value, expiryMs);
    }

    // Replication for fault tolerance
    async replicateToSecondary(key, value, expiryMs) {
        // Store on primary node
        this.set(key, value, expiryMs);

        // Replicate to backup nodes
        const primaryNode = this.getNodeForKey(key);
        for (let i = 0; i < this.nodeCount; i++) {
            if (this.nodes[i] !== primaryNode) {
                // Send replication to backup node
                this.nodes[i].set(key, value, expiryMs);
            }
        }
    }
}
```

---

---

# Question 15: Design API Gateway

## Problem Statement
Design an API Gateway that:
- Routes requests to microservices
- Rate limiting and throttling
- Authentication and authorization
- Request/response transformation
- Caching
- Load balancing
- Circuit breaking
- API versioning

### Scale Requirements
```
Assumptions:
- 1 million requests per second
- 500+ microservices
- 99.99% availability

Calculations:
QPS = 1M requests/second
Latency: < 50ms (gateway overhead)
Throughput: 1M req/sec × 1KB avg = 1GB/sec

Critical Requirements:
✓ Ultra-low latency (< 50ms)
✓ High throughput (1M+ RPS)
✓ 99.99% availability
✓ Horizontal scalability
```

---

## Implementation

```javascript
// API Gateway with all features
class APIGateway {
    constructor() {
        this.routes = new Map();
        this.rateLimiters = new Map();
        this.circuitBreakers = new Map();
        this.cache = new RedisCache();
    }

    // Register route
    registerRoute(path, serviceUrl, options = {}) {
        this.routes.set(path, {
            url: serviceUrl,
            methods: options.methods || ['GET', 'POST', 'PUT', 'DELETE'],
            version: options.version || 'v1',
            rateLimit: options.rateLimit || 100,
            cacheTTL: options.cacheTTL || 0,
            requireAuth: options.requireAuth !== false
        });
    }

    // Main request handler
    async handleRequest(req, res) {
        const startTime = Date.now();

        try {
            // 1. Authentication
            if (!this.validateAuth(req)) {
                return res.status(401).json({ error: 'Unauthorized' });
            }

            // 2. Rate limiting
            const rateLimitResult = await this.checkRateLimit(req);
            if (!rateLimitResult.allowed) {
                res.status(429);
                res.set('Retry-After', rateLimitResult.retryAfter);
                return res.json({ error: 'Rate limit exceeded' });
            }

            // 3. Find route
            const route = this.findRoute(req.path);
            if (!route) {
                return res.status(404).json({ error: 'Route not found' });
            }

            // 4. Check cache
            const cacheKey = `${req.method}:${req.path}:${JSON.stringify(req.query)}`;
            if (req.method === 'GET' && route.cacheTTL > 0) {
                const cachedResponse = await this.cache.get(cacheKey);
                if (cachedResponse) {
                    res.set('X-Cache', 'HIT');
                    return res.json(cachedResponse);
                }
            }

            // 5. Check circuit breaker
            const breaker = this.getCircuitBreaker(route.url);
            if (breaker.isOpen()) {
                return res.status(503).json({ error: 'Service temporarily unavailable' });
            }

            // 6. Forward request to microservice
            const response = await this.forwardRequest(route, req);

            // 7. Cache response if applicable
            if (req.method === 'GET' && route.cacheTTL > 0 && response.status === 200) {
                await this.cache.set(cacheKey, response.data, route.cacheTTL);
            }

            // 8. Send response
            res.set('X-Gateway-Time', `${Date.now() - startTime}ms`);
            return res.status(response.status).json(response.data);

        } catch (error) {
            console.error('Gateway error:', error);
            return res.status(500).json({ error: 'Internal server error' });
        }
    }

    // Forward request to microservice
    async forwardRequest(route, req) {
        const breaker = this.getCircuitBreaker(route.url);

        try {
            const response = await fetch(route.url, {
                method: req.method,
                headers: {
                    ...req.headers,
                    'X-Original-URL': req.url,
                    'X-Forwarded-For': req.ip
                },
                body: req.body,
                timeout: 30000
            });

            // Success: reset circuit breaker
            breaker.recordSuccess();

            return {
                status: response.status,
                data: await response.json()
            };
        } catch (error) {
            // Failure: increment circuit breaker
            breaker.recordFailure();
            throw error;
        }
    }

    // Circuit breaker
    getCircuitBreaker(serviceUrl) {
        if (!this.circuitBreakers.has(serviceUrl)) {
            this.circuitBreakers.set(serviceUrl, new CircuitBreaker());
        }
        return this.circuitBreakers.get(serviceUrl);
    }

    // Rate limiter
    async checkRateLimit(req) {
        const userId = req.user?.id || req.ip;
        const key = `rate_limit:${userId}`;

        if (!this.rateLimiters.has(key)) {
            this.rateLimiters.set(key, new TokenBucket(100, 100 / 60));
        }

        const limiter = this.rateLimiters.get(key);
        return limiter.tryConsume();
    }

    // Find matching route
    findRoute(path) {
        for (let [pattern, route] of this.routes) {
            if (this.matchPattern(pattern, path)) {
                return route;
            }
        }
        return null;
    }

    matchPattern(pattern, path) {
        // Simple pattern matching (can be extended with regex)
        const patternParts = pattern.split('/');
        const pathParts = path.split('/');

        if (patternParts.length !== pathParts.length) {
            return false;
        }

        for (let i = 0; i < patternParts.length; i++) {
            if (patternParts[i] === '*' || patternParts[i] === pathParts[i]) {
                continue;
            }
            return false;
        }

        return true;
    }

    // Authentication
    validateAuth(req) {
        const token = req.headers.authorization?.split(' ')[1];
        if (!token) return false;

        try {
            const decoded = jwt.verify(token, process.env.JWT_SECRET);
            req.user = decoded;
            return true;
        } catch (error) {
            return false;
        }
    }
}
```

---

## Summary of 15 Questions

| # | System | Difficulty | Key Concepts |
|---|--------|-----------|---|
| 1 | Twitter/X | ⭐⭐⭐ | Feed generation, Fanout, Caching |
| 2 | Instagram | ⭐⭐⭐ | Image storage, CDN, Counter cache |
| 3 | YouTube | ⭐⭐⭐⭐ | Video encoding, Adaptive bitrate, Recommendations |
| 4 | TinyURL | ⭐⭐ | Hashing, Caching, Analytics |
| 5 | Uber | ⭐⭐⭐⭐ | Geolocation, Real-time matching, Pricing |
| 6 | Netflix | ⭐⭐⭐⭐ | Streaming, Recommendations, Resume |
| 7 | WhatsApp | ⭐⭐⭐⭐ | Real-time messaging, E2E encryption, Delivery |
| 8 | Slack | ⭐⭐⭐⭐ | Full-text search, Threads, Persistence |
| 9 | Airbnb | ⭐⭐⭐ | Geospatial, Availability, Booking |
| 10 | Rate Limiter | ⭐⭐ | Token bucket, Distributed algorithms |
| 11 | Google Docs | ⭐⭐⭐⭐⭐ | CRDT, Conflict resolution, Real-time sync |
| 12 | Dropbox | ⭐⭐⭐ | Delta sync, File versioning, Conflict resolution |
| 13 | Notification System | ⭐⭐⭐ | Multi-channel, Retries, Deduplication |
| 14 | Distributed Cache | ⭐⭐⭐ | Sharding, Replication, TTL |
| 15 | API Gateway | ⭐⭐⭐ | Routing, Rate limiting, Circuit breaker |

---

**Congratulations!** You now have a comprehensive guide to 15 system design questions covering:
- ✅ Social media platforms
- ✅ Video/file storage and streaming
- ✅ E-commerce and booking systems
- ✅ Real-time messaging and collaboration
- ✅ Infrastructure and distributed systems

This should give you full understanding and confidence to tackle any system design interview! 🚀


