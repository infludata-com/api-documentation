# InfluData API Documentation

**Updated: August 1st, 2025**

## 🆕 Recent Updates

**August 2025 - Enhanced Keyword Search Features:**
- **Targeted Field Searching**: New `keywordFields` parameter allows you to specify which profile fields to search (bio, hashtags, content, website)
- **Bi-gram Phrase Matching**: Keywords with spaces are now treated as exact phrases for more accurate matching
- **Website Content Search**: Search within creator websites and external links (Instagram, TikTok, YouTube)
- **Improved Relevance**: More precise search results with targeted field selection

## Table of Contents

1. [Authentication](#authentication)
2. [API Rate Limit](#api-rate-limit)
3. [Endpoints](#endpoints)
   - [Influencer Profiles](#influencer-profiles)
   - [Client Information](#client-information)
   - [Influencer Discovery](#influencer-discovery)
   - [Content Discovery](#content-discovery)
   - [Private Endpoints](#private-endpoints)
   - [Utility](#utility)
4. [Search Parameters by Platform](#search-parameters-by-platform)
5. [Error Handling](#error-handling)
6. [Platform Coverage](#platform-coverage)
7. [Best Practices](#best-practices)
8. [Examples](#examples)
9. [Support](#support)

## Authentication

All API requests require authentication via Bearer Token. Set your access token in the Authorization header:

```http
Authorization: Bearer YOUR_ACCESS_TOKEN
```

Access tokens and API users are created and managed exclusively by influData based on contractual terms. Each client receives:
- Unique access token
- Client ID (defaults to "default")
- Token balance for API consumption
- Rate limit configurations

## API Rate Limit

**Token-based consumption model:**
- **Basic profile data**: 1 token per request
- **Profile data with audience report**: 20 tokens per request
- **Discovery searches**: 1 token per request (20 results per search)
- **Utility endpoints**: No token consumption

**Token Management:**
- Monitor balance via `/getClientInfo` endpoint
- Tokens are deducted upon successful data retrieval
- Failed requests do not consume tokens
- Negative balance handling varies by client configuration

## Endpoints

### Influencer Profiles

#### Get Influencer Data

```
GET https://app.infludata.com/api/externalAPI/getUserData
```

Retrieve comprehensive influencer data including demographics, statistics, and optionally detailed audience analysis.

**Parameters:**
- `platform` (mandatory): `instagram`, `tiktok`, `youtube`, or `twitch`
- `username`, `userId`, or `_id` (one mandatory): Influencer identifier
  - `username`: Platform username (e.g., "adidas", "@MrBeast")
  - `userId`: Platform-specific user ID (see formats below)
  - `_id`: Internal database ID (works across all platforms)
- `includeAudienceReport` (optional): `true` to include full audience report (default: `false`)
- `showDatalogAndScore` (optional): `true` to include profile scores and data logs (default: `false`)

**Platform-specific User ID formats:**
- **Instagram**: `instagramId` (numeric) - Platform's internal user ID
- **TikTok**: `secTikTokId` (string) - TikTok's secure user identifier
- **YouTube**: `channelId` (string) - YouTube channel identifier
- **Twitch**: `twitchId` (string) - Twitch user identifier
- **Internal ID**: `_id` (string) - InfluData's internal database ID (platform-agnostic)

**Examples:**
```http
GET https://app.infludata.com/api/externalAPI/getUserData?platform=instagram&username=adidas&includeAudienceReport=true

GET https://app.infludata.com/api/externalAPI/getUserData?platform=tiktok&userId=MS4wLjABAAAA...&includeAudienceReport=false

GET https://app.infludata.com/api/externalAPI/getUserData?platform=youtube&username=@MrBeast&showDatalogAndScore=true

GET https://app.infludata.com/api/externalAPI/getUserData?platform=twitch&username=ninja

GET https://app.infludata.com/api/externalAPI/getUserData?_id=507f1f77bcf86cd799439011&includeAudienceReport=true
```

**Responses:**
- **200**: Success, returns influencer data
- **201**: User added to enrichment queue, no tokens charged (will be available within 24-48 hours)
- **202**: User found but audience analysis pending (1 token charged, check back later)
- **400**: Input error (missing platform, invalid parameters)
- **401**: Unauthorized (invalid access token)
- **403**: Insufficient tokens (balance too low)
- **404**: User not found on platform
- **500**: Server error

**Response Data Structure:**

*Basic Profile Data (1 token):*
- Profile information (username, display name, bio, profile picture)
- Follower/subscriber counts and following counts
- Post/video counts and engagement metrics
- Account verification status and account type
- Demographics (country, language, gender, age estimate)
- Timeline data showing collaboration mentions (Instagram/TikTok creators)
- Quality scores and growth metrics

*Full Audience Report (20 tokens):*
- All basic profile data plus:
- Detailed audience demographics (age, gender, location breakdown)
- Audience interests and affinities
- Audience quality metrics (real vs. fake followers)
- Geographic audience distribution
- Device and platform usage patterns
- Lookalike audience suggestions

**Data Exclusions for Security:**
- Post/video content arrays are not included
- Internal collection and campaign data removed
- Administrative fields and internal scoring details filtered
- Raw engagement data limited to aggregated metrics

---

#### Check Data Status

```
GET https://app.infludata.com/api/externalAPI/checkDataStatus
```

Get comprehensive metadata about an influencer without consuming tokens. This endpoint also automatically triggers data refresh and optionally adds the creator to weekly refresh schedules.

**Parameters:**
- `platform` (mandatory): `instagram`, `tiktok`, `youtube`, or `twitch`
- `username`, `userId`, or `_id` (one mandatory): Influencer identifier
- `addToWeeklyEnrichment` (optional): Set to `true` to add creator to weekly refresh schedule (default: `false`)

**Examples:**
```http
GET https://app.infludata.com/api/externalAPI/checkDataStatus?platform=instagram&userId=248312442

GET https://app.infludata.com/api/externalAPI/checkDataStatus?platform=tiktok&username=charlidamelio&addToWeeklyEnrichment=true

GET https://app.infludata.com/api/externalAPI/checkDataStatus?_id=507f1f77bcf86cd799439011
```

**Responses:**
- **200**: Comprehensive metadata object
  ```json
  {
    "isInDatabase": true,
    "creatorId": "507f1f77bcf86cd799439011",
    "username": "example_user",
    "displayName": "Example User",
    "platform": "instagram",
    "followers": 125000,
    "following": 500,
    "posts": 250,
    "isPrivate": false,
    "lastRefreshDate": "2024-01-20T10:30:00Z",
    "lastAudienceDataRefresh": "2024-01-19T08:00:00Z",
    "hasAudienceData": true,
    "firstTimelineDate": "2023-07-20T00:00:00Z",
    "recentTimelineDate": "2024-01-20T00:00:00Z",
    "timelineDatapoints": 180,
    "isAddedToRefresh": false,
    "isAddedToWeeklyRefresh": true
  }
  ```

**For creators not in database:**
  ```json
  {
    "isInDatabase": false,
    "isBeingProcessed": true,
    "username": "new_creator",
    "platform": "instagram",
    "isAddedToRefresh": true,
    "isAddedToWeeklyRefresh": false
  }
  ```

**Response Fields:**
- `isInDatabase`: Boolean indicating if creator exists in database
- `creatorId`: Internal database ID (if exists)
- `username`: Creator's username
- `displayName`: Creator's display name (if exists)
- `platform`: Platform name
- `followers`: Number of followers (if exists)
- `following`: Number following (if exists)
- `posts`: Number of posts (if exists)
- `isPrivate`: Boolean for private accounts
- `lastRefreshDate`: Date of last data refresh (from Elasticsearch)
- `lastAudienceDataRefresh`: Date of last audience analysis update (null if no audience data)
- `hasAudienceData`: Boolean indicating if audience analysis exists
- `firstTimelineDate`: Date of first timeline data point
- `recentTimelineDate`: Date of most recent timeline data point
- `timelineDatapoints`: Number of timeline data points available
- `isAddedToRefresh`: Boolean indicating if creator was just added to instant refresh queue
- `isAddedToWeeklyRefresh`: Boolean indicating if creator was just added to weekly refresh list

**Features:**
- **No Token Consumption**: This endpoint does not consume any API tokens
- **Automatic Instant Update**: Creator is always added to instant update queue
- **Optional Weekly Refresh**: Creator can be added to weekly refresh schedule
- **Comprehensive Metadata**: Detailed information about data availability and freshness

**Error Responses:**
- **400**: Invalid parameters or missing required fields
- **401**: Unauthorized (invalid access token)
- **500**: Server error

---

#### Bulk Add to Enrichment

```
POST https://app.infludata.com/api/externalAPI/bulkAddToEnrich
```

Add multiple creators to enrichment queues in a single request. Process up to 100 creators at once with automatic instant refresh and optional weekly refresh scheduling.

**Request Body:**
```json
{
  "accessToken": "YOUR_ACCESS_TOKEN",
  "platform": "instagram",
  "usernames": ["username1", "username2", "username3"],
  "userIds": ["userId1", "userId2"],
  "addToWeeklyEnrichment": false
}
```

**Parameters:**
- `accessToken` (required): Your API access token (can be in query or body)
- `platform` (required): Platform name (`instagram`, `tiktok`, `youtube`, `twitch`)
- `usernames` (optional): Array of usernames to process
- `userIds` (optional): Array of platform-specific user IDs to process
- `addToWeeklyEnrichment` (optional): Set to `true` to add creators to weekly refresh schedule (default: `false`)

**Examples:**
```http
POST https://app.infludata.com/api/externalAPI/bulkAddToEnrich
Content-Type: application/json

{
  "accessToken": "YOUR_TOKEN",
  "platform": "instagram",
  "usernames": ["cristiano", "leomessi", "neymarjr"],
  "userIds": ["123456789", "987654321"]
}
```

**With weekly refresh enabled:**
```http
POST https://app.infludata.com/api/externalAPI/bulkAddToEnrich
Content-Type: application/json

{
  "accessToken": "YOUR_TOKEN",
  "platform": "tiktok",
  "usernames": ["charlidamelio", "addisonre"],
  "addToWeeklyEnrichment": true
}
```

**Response Format:**
```json
{
  "processed": [
    {
      "identifier": "cristiano",
      "type": "username",
      "isNewUser": false,
      "creatorId": "507f1f77bcf86cd799439011",
      "addedToInstant": true,
      "addedToWeekly": true
    },
    {
      "identifier": "123456789",
      "type": "userId",
      "username": "leomessi",
      "creatorId": "507f1f77bcf86cd799439012",
      "addedToInstant": false,
      "addedToWeekly": false
    }
  ],
  "failed": [
    {
      "identifier": "invaliduser",
      "type": "username",
      "error": "Invalid username format"
    }
  ],
  "summary": {
    "total": 10,
    "existingInDatabase": 8,
    "newlyAdded": 1,
    "addedToInstant": 9,
    "addedToWeekly": 8,
    "failed": 1
  }
}
```

**Response Fields:**

*Processed Array:*
- `identifier`: The username or userId that was processed
- `type`: Either "username" or "userId"
- `isNewUser`: Boolean indicating if this was a new creator (only for usernames)
- `creatorId`: Internal database ID (if creator exists in database)
- `username`: Creator's username (for userId requests)
- `addedToInstant`: Boolean indicating if added to instant refresh queue
- `addedToWeekly`: Boolean indicating if added to weekly refresh schedule

*Failed Array:*
- `identifier`: The username or userId that failed
- `type`: Either "username" or "userId"
- `error`: Description of the failure reason

*Summary Object:*
- `total`: Total number of creators processed
- `existingInDatabase`: Number of creators already in database
- `newlyAdded`: Number of new creators queued for enrichment
- `addedToInstant`: Number of creators added to instant refresh queue
- `addedToWeekly`: Number of creators added to weekly refresh schedule
- `failed`: Number of creators that failed processing

**Features:**
- **No Token Consumption**: This endpoint does not consume any API tokens
- **Bulk Processing**: Process up to 100 creators in a single request
- **Mixed Input Types**: Accept both usernames and userIds in the same request
- **Automatic Instant Refresh**: All valid creators are added to instant update queue
- **Optional Weekly Refresh**: Controlled by `addToWeeklyEnrichment` parameter
- **Detailed Error Reporting**: Clear feedback on which creators succeeded or failed
- **Duplicate Prevention**: Won't re-add creators already in enrichment queues

**Error Responses:**
- **400**: Invalid parameters, empty arrays, or exceeding 100 item limit
- **401**: Unauthorized (invalid access token)
- **500**: Server error

**Limitations:**
- Maximum 100 items per request (combined usernames + userIds)
- Weekly refresh only applies to creators already in the database
- New creators must be enriched before they can be added to weekly refresh

---

### Client Information

#### Get Client Info

```
GET https://app.infludata.com/api/externalAPI/getClientInfo
```

Retrieve current token balance, usage statistics, and account information.

**Parameters:**
- `clientId` (optional): Client identifier (defaults to "default")

**Example:**
```http
GET https://app.infludata.com/api/externalAPI/getClientInfo
```

**Responses:**
- **200**: Client information object
  ```json
  {
    "clientId": "your_client_id",
    "reportsLeft": 1250,
    "reports": [
      {
        "package": "starter",
        "tokensRemaining": 500,
        "expirationDate": "2025-04-21"
      }
    ],
    "createdDate": "2025-01-15",
    "lastUsed": "2025-03-21"
  }
  ```
- **400**: Client not found or invalid input
- **401**: Unauthorized access token
- **500**: Server error

---

### Influencer Discovery

#### Discovery Search

```
GET https://app.infludata.com/api/externalAPI/discovery
```

Search and discover influencers across all supported platforms using comprehensive filtering options.

**Parameters:**
- `platform` (mandatory): `instagram`, `tiktok`, `youtube`, or `twitch`
- `skipCount` (optional): Pagination offset (increments of 20, default: 0)
- `categories` (optional): Creator categories (comma-separated, see available categories below)
- Additional platform-specific search parameters (see below)

**Examples:**
```http
GET https://app.infludata.com/api/externalAPI/discovery?platform=instagram&skipCount=0&country=Germany&followerMin=10000

GET https://app.infludata.com/api/externalAPI/discovery?platform=tiktok&categories=gaming,tech&followerMin=5000&country=United States

GET https://app.infludata.com/api/externalAPI/discovery?platform=youtube&categories=fitness&keywords=workout&sorting=descFollowers

# NEW: Targeted keyword searching
GET https://app.infludata.com/api/externalAPI/discovery?platform=instagram&keywords=dog,pet&keywordFields=bio&followerMin=10000

GET https://app.infludata.com/api/externalAPI/discovery?platform=tiktok&keywords=fitness&keywordFields=bio,content&country=Germany

GET https://app.infludata.com/api/externalAPI/discovery?platform=instagram&keywords=shopify&keywordFields=website&followerMin=5000

# NEW: Bi-gram phrase matching examples
GET https://app.infludata.com/api/externalAPI/discovery?platform=instagram&keywords=personal trainer,fitness coach&keywordFields=bio&followerMin=1000

GET https://app.infludata.com/api/externalAPI/discovery?platform=youtube&keywords=weight loss,healthy eating&keywordFields=content&country=United States

GET https://app.infludata.com/api/externalAPI/discovery?platform=tiktok&keywords=dog lover,pet care,animal rescue&keywordFields=bio,content
```

**Responses:**
- **200**: Search results object
  ```json
  {
    "users": [
      {
        "_id": "profile_id_123",
        "username": "example_user",
        "displayName": "Example User",
        "followers": 125000,
        "platform": "instagram",
        "country": "Germany",
        "engagementMean": 4.2,
        "score": 95.8
      }
    ],
    "count": 1842,
    "dataSuccess": true
  }
  ```
- **400**: Invalid platform or search parameters
- **401**: Unauthorized access token
- **500**: Server error

**Search Results:**
- **Results per search**: 20 influencers maximum
- **Token consumption**: 1 token per search request
- **Pagination**: Use `skipCount` parameter (0, 20, 40, etc.)
- **Sorting**: Results can be sorted by various metrics
- **Score**: Relevance score (0-100) when using keyword search

---

### Search Parameters by Platform

**Common Parameters (All Platforms):**
- `keywords` (optional): Search terms in creator content, bio, username (comma-separated). **NEW**: Now supports bi-gram phrase matching - keywords with spaces are treated as exact phrases for better accuracy
  - Single words: `keywords=fitness,yoga` searches for "fitness" OR "yoga"
  - Phrases: `keywords=personal trainer,weight loss` searches for exact phrases "personal trainer" OR "weight loss"
  - Mixed: `keywords=fitness,personal trainer,yoga` combines single words and phrases
- `keywordFields` (optional): **NEW** Specify which profile fields to search for keywords - allows targeted keyword searching (comma-separated). Default: searches all fields
  - `bio` - Search in profile descriptions/bios
  - `hashtags` - Search in hashtags used by creators  
  - `content` - Search in post captions and content
  - `website` - Search in websites and external links (Instagram, TikTok, YouTube only)
  - Example: `keywordFields=bio,content` searches only in bios and post content
- `categories` (optional): Creator categories (comma-separated, see Available Categories section)
- `country` (optional): Creator location country (see Supported Countries section)
- `city` (optional): Creator location city  
- `language` (optional): Creator language (2-letter code, see Supported Languages section)
- `gender` (optional): Creator gender (`m` for male, `w` for female)
- `followerMin` (optional): Minimum follower count
- `followerMax` (optional): Maximum follower count
- `engagementRate` (optional): Minimum engagement rate (0-20, where 1 = 1%)
- `growthRate` (optional): Minimum monthly growth rate percentage
- `sorting` (optional): Sort order - `relevance`, `username`, `profileScore`, `followers`, `engagementMean`, `growthRate`

#### Instagram Specific Parameters

**Audience Size:**
- `followerMin/followerMax` - Follower count range (1 to 500,000,000)

**Content Performance:**
- `viewsMin/viewsMax` - Average views per post range
- `reelPlaysMin/reelPlaysMax` - Average reel plays range

**Demographics:**
- `city` - City-level location filter (use with country parameter)
- `gender` - Creator gender: `male`, `female`, `unknown`
- `ageMin/ageMax` - Creator age range (13-80)

**Account Type:**
- `businessSearch` - Set to `true` for business account mode (focuses on brand accounts)

**Additional Sorting Options:**
- `viewsPost` - Average views per post
- `reelPlays` - Average reel plays

#### TikTok Specific Parameters

**Audience Size:**
- `followerMin/followerMax` - Follower count range (1 to 200,000,000)

**Content Performance:**
- `viewsMin/viewsMax` - Average views per video range
- `totalViews` - Total cumulative views

**Demographics:**
- `city` - City-level location filter
- `gender` - Creator gender: `male`, `female`, `unknown`
- `ageMin/ageMax` - Creator age range (13-80)

**Additional Sorting Options:**
- `viewsPost` - Average views per video
- `totalViews` - Total video views

#### YouTube Specific Parameters

**Audience Size:**
- `followerMin/followerMax` - Subscriber count range (1 to 200,000,000)

**Content Performance:**
- `viewsMin/viewsMax` - Average views per video range

**Additional Sorting Options:**
- `viewsPost` - Average views per video (mapped to `medianViewCountPosts`)

#### Twitch Specific Parameters

**Audience Size:**
- `followerMin/followerMax` - Follower count range (1 to 20,000,000)

**Streaming Performance:**
- `averageViewersMin/averageViewersMax` - Average concurrent viewer count range

**Additional Sorting Options:**
- `averageViewers` - Average concurrent viewers during streams

---

### Content Discovery

#### Get Content

```
GET https://app.infludata.com/api/externalAPI/getContent
```

Retrieve content pieces from the Elasticsearch index with powerful filtering capabilities. Access to all content across platforms with detailed metrics and engagement data.

**Parameters:**
- `accessToken` (required): Your API access token
- `limit` (optional): Number of content pieces to return (default: 1, max: 1000)
- `offset` (optional): Number of results to skip for pagination (default: 0)
- `keywords` (optional): Comma-separated keywords to search in captions, hashtags, mentions
- `followerMin` (optional): Minimum follower count for the content creator
- `followerMax` (optional): Maximum follower count for the content creator
- `viewsMin` (optional): Minimum views/reach for content
- `viewsMax` (optional): Maximum views/reach for content
- `uploadedFrom` (optional): Start date for content upload (ISO format)
- `uploadedTo` (optional): End date for content upload (ISO format)
- `country` (optional): Creator's country
- `city` (optional): Creator's city
- `gender` (optional): Creator's gender (`m` for male, `f` for female)
- `language` (optional): Content language (2-letter code)
- `user_country` (optional): Audience country filter
- `sorting` (optional): Sort order: `viral`, `uploaded`, or default by reach
- `contentTypes` (optional): Comma-separated content types to filter
  - Instagram: `post`, `reel`, `story` (e.g., `contentTypes=post,reel`)
  - TikTok: `video`
  - YouTube: `video`, `short`
  - Twitch: `video`
- `platform` (optional): Social media platform: `instagram`, `tiktok`, `youtube`, `twitch`

**Examples:**
```http
GET https://app.infludata.com/api/externalAPI/getContent?limit=10&keywords=fashion,style&followerMin=10000

GET https://app.infludata.com/api/externalAPI/getContent?limit=100&uploadedFrom=2024-01-01&country=Germany&sorting=viral

GET https://app.infludata.com/api/externalAPI/getContent?offset=50&limit=50&viewsMin=100000&platform=instagram&contentTypes=post,reel
```

**Token Consumption:**
- **1 token per content piece returned**
- Example: If you request `limit=100` and 100 content pieces are found, 100 tokens will be deducted
- Token deduction occurs only for successfully returned content

**Response Format:**
```json
{
  "content": [
    {
      "_id": "65abc123def456789",
      "_index": "contentdata_v0.2",
      "score": 95.5,
      "contentId": "IG_12345678901234567",
      "platform": "instagram",
      "contentType": "reel",
      "uploaded": "2024-01-15T10:30:00Z",
      "reach": 125000,
      "likes": 8500,
      "comments": 342,
      "shares": 156,
      "saves": 89,
      "engagementRate": 0.0745,
      "viralFactor": 1.23,
      "username": "fashioninfluencer",
      "userId": "123456789",
      "followers": 45000,
      "following": 890,
      "verified": true,
      "country": "Germany",
      "city": "Berlin",
      "language": "de",
      "gender": "female",
      "imageUrl": "https://signed-url-to-image.amazonaws.com/...",
      "videoUrl": "https://signed-url-to-video.amazonaws.com/...",
      "profilePicUrl": "https://signed-url-to-profile.amazonaws.com/...",
      "captions": "Check out this amazing outfit! #fashion #style #ootd",
      "hashtags": ["fashion", "style", "ootd", "berlin", "streetstyle"],
      "mentions": ["@brandname", "@photographer"],
      "keywords": "<b>fashion</b> <b>style</b> outfit berlin streetstyle",
      "isSponsored": false,
      "brandMentions": ["brandname"],
      "audienceGender": {
        "male": 23.5,
        "female": 76.5
      },
      "audienceAge": {
        "13-17": 5.2,
        "18-24": 42.3,
        "25-34": 31.8,
        "35-44": 15.2,
        "45+": 5.5
      },
      "audienceCountries": {
        "DE": 45.2,
        "AT": 12.3,
        "CH": 8.7,
        "US": 5.4,
        "Other": 28.4
      }
    }
  ],
  "count": 15842,
  "limit": 10,
  "offset": 0,
  "tokensDeducted": 10,
  "tokensRemaining": 4990
}
```

**Response Fields:**
- All fields from Elasticsearch are included in the response
- Image and video URLs are pre-signed for direct access (valid for 24 hours)
- Keywords field contains highlighted search terms when using keyword search
- Collection-related fields are excluded for API users
- All metrics and engagement data are included

**Responses:**
- **200**: Success, returns content array
- **401**: Unauthorized (invalid access token)
- **403**: Insufficient tokens (and allowNegativeBalance is false)
- **404**: No content found matching criteria
- **500**: Server error

---

### Private Endpoints

#### Return Data to InfluData

```
POST https://app.infludata.com/api/externalAPI/returnDataToInfludata
```

**⚠️ PRIVATE ENDPOINT - NOT FOR PUBLIC USE**

This endpoint is reserved for internal data synchronization between InfluData systems. External API clients should not use this endpoint. Any unauthorized use may result in API access termination.

**Purpose:** Internal data collection and synchronization

**Authorization:** Requires special permissions beyond standard API access

---

### Utility

#### Get Cities for Country

```
GET https://app.infludata.com/api/externalAPI/getCitiesForCountry
```

Retrieve a list of available cities for location-based filtering within a specific country.

**Parameters:**
- `country` (mandatory): Valid country name (case-sensitive)

**Examples:**
```http
GET https://app.infludata.com/api/externalAPI/getCitiesForCountry?country=Germany

GET https://app.infludata.com/api/externalAPI/getCitiesForCountry?country=United%20States

GET https://app.infludata.com/api/externalAPI/getCitiesForCountry?country=United%20Kingdom
```

**Responses:**
- **200**: Array of city names
  ```json
  [
    "Berlin",
    "Munich",
    "Hamburg",
    "Cologne",
    "Frankfurt am Main",
    "Stuttgart",
    "Düsseldorf",
    "Dortmund",
    "Essen",
    "Leipzig"
  ]
  ```
- **400**: Invalid country name or country not supported
- **401**: Unauthorized access token
- **500**: Server error

---

### **Available Categories**

Use the `categories` parameter to filter creators by their content categories. Multiple categories can be specified using comma separation.

**Supported Categories:**
- `fashion` - Fashion & Style
- `fitness` - Fitness & Wellness  
- `beauty` - Beauty & Cosmetics
- `sports` - Athletics & Sports
- `food` - Food & Drink
- `diet` - Healthy Nutrition & Diet
- `veganism` - Veganism & Vegetarianism
- `travel` - Travel & Adventure
- `books` - Books & Literature
- `interior` - Home & Interior Design
- `comedy` - Comedy
- `tech` - Technology & Gadgets
- `art` - Art & Creativity
- `lifestyle` - Lifestyle
- `education` - Education & Learning
- `family` - Parenting & Family
- `media` - Entertainment & Media
- `music` - Music
- `lgbtq` - LGBTQ+
- `gaming` - Gaming
- `business` - Business & Finance
- `automotive` - Automotive & Vehicles
- `sustainability` - Sustainability & Environment
- `animals` - Animals & Pets
- `charity` - Charity & Activism
- `politics` - Politics

**Category Examples:**
```http
GET https://app.infludata.com/api/externalAPI/discovery?platform=instagram&categories=fitness,fashion&followerMin=50000

GET https://app.infludata.com/api/externalAPI/discovery?platform=tiktok&categories=gaming&country=Germany&skipCount=20
```

## Supported Countries

**Europe:**
Albania, Andorra, Austria, Belarus, Belgium, Bosnia and Herzegovina, Bulgaria, Croatia, Cyprus, Czech Republic, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Iceland, Ireland, Italy, Latvia, Lithuania, Luxembourg, Malta, Moldova, Monaco, Montenegro, Netherlands, North Macedonia, Norway, Poland, Portugal, Romania, San Marino, Serbia, Slovakia, Slovenia, Spain, Sweden, Switzerland, Ukraine, United Kingdom, Vatican City

**North America:**
Canada, Mexico, United States

**South America:**
Argentina, Bolivia, Brazil, Chile, Colombia, Ecuador, French Guiana, Guyana, Paraguay, Peru, Suriname, Uruguay, Venezuela

**Asia:**
Afghanistan, Armenia, Azerbaijan, Bahrain, Bangladesh, Bhutan, Brunei, Cambodia, China, Cyprus, Georgia, India, Indonesia, Iran, Iraq, Israel, Japan, Jordan, Kazakhstan, Kuwait, Kyrgyzstan, Laos, Lebanon, Malaysia, Maldives, Mongolia, Myanmar, Nepal, North Korea, Oman, Pakistan, Palestine, Philippines, Qatar, Russia, Saudi Arabia, Singapore, South Korea, Sri Lanka, Syria, Taiwan, Tajikistan, Thailand, Timor-Leste, Turkey, Turkmenistan, United Arab Emirates, Uzbekistan, Vietnam, Yemen

**Africa:**
Algeria, Angola, Benin, Botswana, Burkina Faso, Burundi, Cameroon, Cape Verde, Central African Republic, Chad, Comoros, Democratic Republic of the Congo, Djibouti, Egypt, Equatorial Guinea, Eritrea, Eswatini, Ethiopia, Gabon, Gambia, Ghana, Guinea, Guinea-Bissau, Ivory Coast, Kenya, Lesotho, Liberia, Libya, Madagascar, Malawi, Mali, Mauritania, Mauritius, Morocco, Mozambique, Namibia, Niger, Nigeria, Republic of the Congo, Rwanda, São Tomé and Príncipe, Senegal, Seychelles, Sierra Leone, Somalia, South Africa, South Sudan, Sudan, Tanzania, Togo, Tunisia, Uganda, Zambia, Zimbabwe

**Oceania:**
Australia, Fiji, Kiribati, Marshall Islands, Micronesia, Nauru, New Zealand, Palau, Papua New Guinea, Samoa, Solomon Islands, Tonga, Tuvalu, Vanuatu

## Supported Languages

**Language Codes:**
- `en` - English
- `es` - Spanish
- `fr` - French
- `de` - German
- `it` - Italian
- `pt` - Portuguese
- `ru` - Russian
- `ja` - Japanese
- `ko` - Korean
- `zh` - Chinese (Simplified)
- `zh-TW` - Chinese (Traditional)
- `ar` - Arabic
- `hi` - Hindi
- `th` - Thai
- `vi` - Vietnamese
- `id` - Indonesian
- `ms` - Malay
- `tl` - Filipino
- `tr` - Turkish
- `pl` - Polish
- `nl` - Dutch
- `sv` - Swedish
- `da` - Danish
- `no` - Norwegian
- `fi` - Finnish
- `cs` - Czech
- `sk` - Slovak
- `hu` - Hungarian
- `ro` - Romanian
- `bg` - Bulgarian
- `hr` - Croatian
- `sr` - Serbian
- `sl` - Slovenian
- `et` - Estonian
- `lv` - Latvian
- `lt` - Lithuanian
- `el` - Greek
- `he` - Hebrew
- `fa` - Persian
- `ur` - Urdu
- `bn` - Bengali
- `ta` - Tamil
- `te` - Telugu
- `ml` - Malayalam
- `kn` - Kannada
- `gu` - Gujarati
- `pa` - Punjabi
- `mr` - Marathi
- `ne` - Nepali
- `si` - Sinhala
- `my` - Burmese
- `km` - Khmer
- `lo` - Lao
- `ka` - Georgian
- `am` - Amharic
- `sw` - Swahili
- `zu` - Zulu
- `af` - Afrikaans
- `is` - Icelandic
- `mt` - Maltese
- `eu` - Basque
- `ca` - Catalan
- `gl` - Galician
- `cy` - Welsh
- `ga` - Irish
- `gd` - Scottish Gaelic
- `br` - Breton
- `co` - Corsican
- `lb` - Luxembourgish
- `rm` - Romansh
- `fur` - Friulian
- `sc` - Sardinian
- `vec` - Venetian
- `lmo` - Lombard
- `pms` - Piedmontese
- `lij` - Ligurian
- `nap` - Neapolitan
- `scn` - Sicilian

## Error Handling

### HTTP Status Codes

**Success Codes:**
- **200 OK**: Request successful, data returned
- **201 Created**: User added to enrichment queue, will be available soon

**Client Error Codes:**
- **400 Bad Request**: Invalid parameters, missing required fields, or malformed request
- **401 Unauthorized**: Invalid or missing access token
- **403 Forbidden**: Insufficient tokens or access denied
- **404 Not Found**: Requested user/resource not found
- **429 Too Many Requests**: Rate limit exceeded (if applicable)

**Server Error Codes:**
- **500 Internal Server Error**: Server-side error occurred
- **502 Bad Gateway**: Upstream service unavailable
- **503 Service Unavailable**: Service temporarily unavailable

### Error Response Format

```json
{
  "error": {
    "code": "INSUFFICIENT_TOKENS",
    "message": "Not enough tokens to complete request",
    "details": {
      "required": 20,
      "available": 5
    }
  }
}
```

### Common Error Scenarios

**Authentication Errors:**
- Invalid access token format
- Expired or revoked access token
- Missing Authorization header

**Parameter Errors:**
- Missing mandatory platform parameter
- Invalid platform value
- Multiple identifiers provided simultaneously
- No identifier provided (username, userId, or _id required)
- Invalid date ranges or numeric values

**Resource Errors:**
- User not found on specified platform
- Country not supported for city filtering
- Invalid language code

**Rate Limiting:**
- Token balance insufficient for request type
- Daily/monthly usage limits exceeded
- Concurrent request limits exceeded

## Platform Coverage

| Feature | Instagram | TikTok | YouTube | Twitch |
|---------|-----------|---------|---------|--------|
| **User Data Retrieval** | ✅ | ✅ | ✅ | ✅ |
| **Discovery Search** | ✅ | ✅ | ✅ | ✅ |
| **Audience Reports** | ✅ | ✅ | ✅ | ❌ |
| **Timeline Data** | ✅ | ✅ | ✅ | ✅ |
| **Content Performance** | ✅ | ✅ | ✅ | ✅ |
| **Demographics** | ✅ | ✅ | ✅ | ✅ |
| **City-level Filtering** | ✅ | ✅ | ❌ | ❌ |
| **Gender Filtering** | ✅ | ✅ | ❌ | ❌ |
| **Age Filtering** | ✅ | ✅ | ❌ | ❌ |
| **Business Mode** | ✅ | ❌ | ❌ | ❌ |
| **Internal ID Support** | ✅ | ✅ | ✅ | ✅ |

### Platform-Specific Notes

**Instagram:**
- Most comprehensive feature set
- Includes Reels performance metrics
- Business account detection and filtering
- Detailed demographic filtering options

**TikTok:**
- Full feature parity with Instagram except business mode
- Video performance metrics included
- Strong engagement analytics

**YouTube:**
- Subscriber and view-based metrics
- Channel-level analytics
- Limited demographic filtering (no age/gender/city)

**Twitch:**
- Streaming-specific metrics (average viewers)
- No audience reports available
- Focus on live streaming performance

## Best Practices

### Token Management
1. **Monitor token consumption** regularly via `/getClientInfo`
2. **Use metadata-first approach** with `/checkDataStatus` before consuming tokens
3. **Check data readiness** to avoid spending tokens on incomplete audience reports
4. **Implement token threshold alerts** in your application
5. **Cache metadata results** to reduce unnecessary API calls

### Data Retrieval Workflow
1. **Start with metadata checks** using `/checkDataStatus` to understand data availability
2. **Use bulk operations** when processing multiple creators with `/bulkAddToEnrich`
3. **Implement retry logic** for creators pending enrichment
4. **Leverage automatic refresh** by enabling weekly updates for important creators
5. **Handle different data states** gracefully (new, pending, ready)

### Performance Optimization
1. **Batch creator processing** using the bulk enrichment endpoint
2. **Enable weekly refresh** for frequently accessed creators
3. **Use appropriate filtering** in discovery searches to reduce result set size
4. **Implement client-side caching** with metadata-driven cache invalidation
5. **Consider time zones** for optimal API response times

### Enrichment Strategy
1. **Use bulk enrichment** for onboarding new creator lists
2. **Enable weekly refresh** for creators in active campaigns
3. **Monitor enrichment status** through metadata responses
4. **Plan for enrichment delays** (24-48 hours for new creators)
5. **Combine instant and weekly refresh** based on use case priority

### Error Handling
1. **Implement exponential backoff** for temporary failures
2. **Handle enrichment states** appropriately (new vs. pending vs. ready)
3. **Process bulk operation results** carefully, handling partial successes
4. **Log detailed metadata** for debugging and monitoring
5. **Provide user-friendly status updates** based on enrichment progress

### Security Considerations
1. **Never expose API tokens** in client-side code
2. **Use HTTPS** for all API communications
3. **Implement proper token rotation** policies
4. **Monitor for unusual usage patterns**
5. **Validate bulk operation inputs** to prevent abuse

## Examples

### Complete Workflow Example

```javascript
// 1. Check client token balance
const clientInfo = await fetch('/api/externalAPI/getClientInfo', {
  headers: { 'Authorization': 'Bearer YOUR_TOKEN' }
});
const { reportsLeft } = await clientInfo.json();

if (reportsLeft < 20) {
  console.log('Insufficient tokens for full audience report');
}

// 2. Check creator metadata and trigger refresh (no tokens consumed)
const metadataCheck = await fetch(
  '/api/externalAPI/checkDataStatus?platform=instagram&username=cristiano&addToWeeklyEnrichment=true',
  { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
);
const metadata = await metadataCheck.json();

console.log('Creator metadata:', {
  inDatabase: metadata.isInDatabase,
  hasAudienceData: metadata.hasAudienceData,
  lastRefresh: metadata.lastRefreshDate,
  timelinePoints: metadata.timelineDatapoints
});

// 3. Retrieve full user data if ready and needed
if (metadata.isInDatabase && metadata.hasAudienceData) {
  const userData = await fetch(
    `/api/externalAPI/getUserData?platform=instagram&username=cristiano&includeAudienceReport=true`,
    { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
  );
  const user = await userData.json();
  console.log('Full user data retrieved');
} else if (metadata.isInDatabase) {
  console.log('Creator found, but audience data pending - check back later');
} else {
  console.log('Creator queued for enrichment - will be available within 24-48 hours');
}
```

### Bulk Operations Example

```javascript
// Bulk add multiple creators to enrichment queues
async function bulkEnrichCreators(platform, creatorList) {
  const response = await fetch('/api/externalAPI/bulkAddToEnrich', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer YOUR_TOKEN',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      platform: platform,
      usernames: creatorList.usernames,
      userIds: creatorList.userIds,
      addToWeeklyEnrichment: true
    })
  });

  const results = await response.json();
  
  console.log(`Processed ${results.summary.total} creators:`);
  console.log(`- ${results.summary.existingInDatabase} already in database`);
  console.log(`- ${results.summary.newlyAdded} newly added for enrichment`);
  console.log(`- ${results.summary.addedToInstant} added to instant refresh`);
  console.log(`- ${results.summary.addedToWeekly} added to weekly refresh`);
  console.log(`- ${results.summary.failed} failed`);

  // Handle failed items
  if (results.failed.length > 0) {
    console.log('Failed items:', results.failed);
  }

  return results;
}

// Usage
const creators = {
  usernames: ['cristiano', 'leomessi', 'neymarjr'],
  userIds: ['123456789', '987654321']
};

const bulkResults = await bulkEnrichCreators('instagram', creators);
```

### Metadata-First Approach Example

```javascript
// Check multiple creators' metadata before deciding to use tokens
async function analyzeCreatorReadiness(creatorList) {
  const readyCreators = [];
  const pendingCreators = [];
  
  for (const creator of creatorList) {
    const response = await fetch(
      `/api/externalAPI/checkDataStatus?platform=${creator.platform}&username=${creator.username}`,
      { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
    );
    
    const metadata = await response.json();
    
    if (metadata.isInDatabase && metadata.hasAudienceData) {
      readyCreators.push({
        ...creator,
        metadata,
        readyForFullData: true
      });
    } else if (metadata.isInDatabase) {
      pendingCreators.push({
        ...creator,
        metadata,
        needsAudienceAnalysis: true
      });
    } else {
      pendingCreators.push({
        ...creator,
        metadata,
        needsEnrichment: true
      });
    }
  }
  
  console.log(`${readyCreators.length} creators ready for full data retrieval`);
  console.log(`${pendingCreators.length} creators pending enrichment`);
  
  return { readyCreators, pendingCreators };
}

// Only retrieve full data for ready creators to optimize token usage
const analysis = await analyzeCreatorReadiness([
  { platform: 'instagram', username: 'cristiano' },
  { platform: 'tiktok', username: 'charlidamelio' },
  { platform: 'youtube', username: 'mrbeast' }
]);
```

### Discovery Search with Pagination

```javascript
async function searchInfluencers(query, platform) {
  let allResults = [];
  let skipCount = 0;
  let hasMore = true;

  while (hasMore && allResults.length < 100) {
    const response = await fetch(
      `/api/externalAPI/discovery?platform=${platform}&skipCount=${skipCount}&${query}`,
      { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
    );
    
    const data = await response.json();
    allResults.push(...data.users);
    
    // Check if there are more results
    hasMore = data.users.length === 20;
    skipCount += 20;
  }

  return allResults;
}

// Usage examples
const fashionInfluencers = await searchInfluencers(
  'keywords=fashion&country=Germany&followerMin=10000&engagementMin=3',
  'instagram'
);

// NEW: Using targeted field searching
const dogLoversBios = await searchInfluencers(
  'keywords=dog lover,pet owner&keywordFields=bio&followerMin=5000',
  'instagram'
);

// NEW: Using bi-gram phrase matching
const fitnessTrainers = await searchInfluencers(
  'keywords=personal trainer,fitness coach&keywordFields=bio,content&country=United States',
  'instagram'
);
```

### NEW: Advanced Keyword Search Examples

```javascript
// Example 1: Find fitness trainers using targeted bio search
async function findFitnessTrainers() {
  const response = await fetch(
    '/api/externalAPI/discovery?platform=instagram&keywords=personal trainer,fitness coach,certified trainer&keywordFields=bio&followerMin=1000&country=United States',
    { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
  );
  return await response.json();
}

// Example 2: Search for creators posting about specific topics
async function findContentCreators() {
  const response = await fetch(
    '/api/externalAPI/discovery?platform=tiktok&keywords=healthy recipes,meal prep&keywordFields=content&followerMin=5000',
    { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
  );
  return await response.json();
}

// Example 3: Find creators with e-commerce websites
async function findEcommerceCreators() {
  const response = await fetch(
    '/api/externalAPI/discovery?platform=instagram&keywords=shop,store,shopify,etsy&keywordFields=website&followerMin=10000',
    { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
  );
  return await response.json();
}

// Example 4: Combined search across bio and content
async function findPetInfluencers() {
  const response = await fetch(
    '/api/externalAPI/discovery?platform=instagram&keywords=dog lover,pet care,animal rescue&keywordFields=bio,content&followerMin=2000&country=Germany',
    { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
  );
  return await response.json();
}

// Example 5: Hashtag-specific search
async function findHashtagUsers() {
  const response = await fetch(
    '/api/externalAPI/discovery?platform=instagram&keywords=veganfood,plantbased&keywordFields=hashtags&followerMin=1000',
    { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
  );
  return await response.json();
}
```

### Multi-platform Search

```javascript
async function searchAllPlatforms(keywords, keywordFields = 'bio,content') {
  const platforms = ['instagram', 'tiktok', 'youtube', 'twitch'];
  const results = {};

  for (const platform of platforms) {
    try {
      // Skip website field for Twitch (not supported)
      const fields = platform === 'twitch' 
        ? keywordFields.replace(',website', '').replace('website,', '').replace('website', '')
        : keywordFields;
        
      const response = await fetch(
        `/api/externalAPI/discovery?platform=${platform}&keywords=${keywords}&keywordFields=${fields}&followerMin=1000`,
        { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
      );
      
      const data = await response.json();
      results[platform] = data.users;
    } catch (error) {
      console.error(`Error searching ${platform}:`, error);
      results[platform] = [];
    }
  }

  return results;
}

// NEW: Usage with targeted searching
const dogInfluencers = await searchAllPlatforms('dog lover,pet care', 'bio,content');
const fitnessCreators = await searchAllPlatforms('personal trainer,fitness coach', 'bio');
```

### Using Internal _id for Cross-platform Operations

```javascript
async function getInfluencerAcrossPlatforms(internalId) {
  try {
    // Get user data using internal _id (platform auto-detected)
    const response = await fetch(
      `/api/externalAPI/getUserData?_id=${internalId}&includeAudienceReport=false`,
      { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
    );
    
    const user = await response.json();
    console.log(`Found ${user.username} on ${user.platform}`);
    
    return user;
  } catch (error) {
    console.error('Error fetching user by internal ID:', error);
    return null;
  }
}
```

### Error Handling Implementation

```javascript
async function safeApiCall(url, options = {}) {
  const maxRetries = 3;
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, {
        ...options,
        headers: {
          'Authorization': 'Bearer YOUR_TOKEN',
          ...options.headers
        }
      });

      if (response.ok) {
        return await response.json();
      }

      // Handle specific error codes
      switch (response.status) {
        case 401:
          throw new Error('Invalid API token');
        case 403:
          throw new Error('Insufficient tokens');
        case 429:
          // Wait before retry
          await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
          continue;
        case 500:
          if (attempt < maxRetries) {
            await new Promise(resolve => setTimeout(resolve, 2000 * attempt));
            continue;
          }
          throw new Error('Server error');
        default:
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
    } catch (error) {
      lastError = error;
      if (attempt < maxRetries) {
        await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
      }
    }
  }

  throw lastError;
}
```

## Rate Limiting Details

### Token Consumption by Endpoint

| Endpoint | Basic Call | With Audience Report | Notes |
|----------|------------|---------------------|--------|
| `/getUserData` | 1 token | 20 tokens | Depends on `includeAudienceReport` |
| `/discovery` | 1 token | 1 token | Per search (20 results) |
| `/getContent` | Variable | N/A | 1 token per content piece returned |
| `/checkDataStatus` | 0 tokens | 0 tokens | Free utility endpoint with comprehensive metadata |
| `/bulkAddToEnrich` | 0 tokens | 0 tokens | Free bulk enrichment endpoint |
| `/getClientInfo` | 0 tokens | 0 tokens | Free utility endpoint |
| `/getCitiesForCountry` | 0 tokens | 0 tokens | Free utility endpoint |
| `/returnDataToInfludata` | N/A | N/A | Private endpoint - not for public use |

### Fair Usage Guidelines

- **Discovery searches**: Recommended maximum 100 searches per hour
- **Profile requests**: Recommended maximum 500 profiles per hour  
- **Concurrent requests**: Maximum 10 simultaneous connections
- **Data caching**: Recommended 24-hour cache for profile data

## Support

### Technical Support

For technical issues, integration questions, or API troubleshooting:
- **Primary Contact**: Your designated account manager
- **Technical Documentation**: Available in your client portal
- **Response Time**: 24-48 hours for standard inquiries

### Integration Support

For assistance with API integration, custom implementations, or scaling requirements:
- **Integration Team**: Available via your account manager
- **Custom Solutions**: Enterprise-level integrations available
- **Development Resources**: Code samples and SDKs upon request

### Account Management

For token purchases, usage monitoring, or account configuration:
- **Account Manager**: Direct contact provided upon onboarding
- **Billing Inquiries**: Handled through account management
- **Usage Analytics**: Detailed reports available upon request

### Emergency Contact

For critical production issues or service outages:
- **Emergency Hotline**: Provided to enterprise clients
- **Status Page**: Monitor service availability
- **Incident Response**: 4-hour response for critical issues

---

**Document Version**: 2.2  
**Last Updated**: August 1st, 2025  
**API Version**: v1  
**Effective Date**: March 21st, 2025

*This documentation is proprietary and confidential. Distribution is restricted to authorized API clients only.*
