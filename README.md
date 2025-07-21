# InfluData API Documentation

## Authentication

All API endpoints require authentication via an access token. Pass your access token as a query parameter:
```
?accessToken=YOUR_ACCESS_TOKEN
```

## Token System

- Each API user has a token balance (`tokensLeft`)
- Tokens are deducted based on the data retrieved
- Users with `allowNegativeBalance` flag can continue using the API even with 0 tokens

## Endpoints

### GET /api/v1/getContent

Retrieves content pieces from the Elasticsearch index with powerful filtering capabilities.

#### Parameters

| Parameter | Type | Description | Default | Max |
|-----------|------|-------------|---------|-----|
| `accessToken` | string | **Required.** Your API access token | - | - |
| `limit` | integer | Number of content pieces to return | 1 | 1000 |
| `offset` | integer | Number of results to skip (for pagination) | 0 | - |
| `keywords` | string | Comma-separated keywords to search in captions, hashtags, mentions | - | - |
| `followerMin` | integer/string | Minimum follower count for the creator | - | - |
| `followerMax` | integer/string | Maximum follower count for the creator | - | - |
| `viewsMin` | integer/string | Minimum views/reach for content | - | - |
| `viewsMax` | integer/string | Maximum views/reach for content | - | - |
| `uploadedFrom` | date | Start date for content upload (ISO format) | - | - |
| `uploadedTo` | date | End date for content upload (ISO format) | - | - |
| `country` | string | Creator's country | - | - |
| `city` | string | Creator's city | - | - |
| `gender` | string | Creator's gender (m/f) | - | - |
| `language` | string | Content language | - | - |
| `user_country` | string | Audience country filter | - | - |
| `sorting` | string | Sort order: `viral`, `uploaded`, or default by reach | - | - |
| `contentType` | string | Type of content to filter | - | - |
| `platform` | string | Social media platform | - | - |

#### Token Calculation
- **1 token per content piece returned**
- Example: If you request `limit=100` and 100 content pieces are found, 100 tokens will be deducted

#### Response Format
```json
{
  "content": [
    {
      "_id": "content_id",
      "_index": "contentdata_v0.2",
      "score": 95.5,
      "contentId": "unique_content_id",
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
      "username": "creator_username",
      "userId": "user_123",
      "followers": 45000,
      "imageUrl": "https://signed-url-to-image.com/...",
      "videoUrl": "https://signed-url-to-video.com/...",
      "profilePicUrl": "https://signed-url-to-profile.com/...",
      "captions": "Post caption text...",
      "hashtags": ["fashion", "style"],
      "mentions": ["@brand1", "@user2"],
      "keywords": "Formatted keywords with highlights",
      // ... all other Elasticsearch fields
    }
  ],
  "count": 1000,  // Total matches in Elasticsearch
  "limit": 100,   // Requested limit
  "offset": 0,    // Current offset
  "tokensDeducted": 100,
  "tokensRemaining": 4900
}
```

#### Error Responses
- `401 Unauthorized` - Invalid or missing access token
- `403 Forbidden` - Out of API tokens (and allowNegativeBalance is false)
- `404 Not Found` - No content found matching criteria
- `500 Internal Server Error` - Server error

### POST /api/v1/returnDataToInfludata (Private Endpoint)

**⚠️ PRIVATE ENDPOINT - NOT FOR PUBLIC USE**

This endpoint is reserved for internal data synchronization and should not be used by external API clients.

#### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `accessToken` | string | **Required.** Your API access token (query or body) |
| `*` | any | Any JSON data in request body |

#### Response Format
```json
{
  "success": true,
  "message": "Data received successfully",
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```

### GET /api/v1/checkDataStatus

Check the enrichment status of a creator profile and optionally trigger updates.

#### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `accessToken` | string | **Required.** Your API access token |
| `platform` | string | **Required.** Platform: `instagram`, `tiktok`, `youtube`, `twitch` |
| `userId` | string | Platform user ID (one of userId, username, or _id required) |
| `username` | string | Platform username (one of userId, username, or _id required) |
| `_id` | string | MongoDB ID (one of userId, username, or _id required) |
| `addToWeeklyEnrichment` | string | Set to "true" to add profile to weekly enrichment queue |

### POST /api/v1/bulkAddToEnrich

Add multiple profiles to the enrichment queue.

#### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `accessToken` | string | **Required.** Your API access token |
| `platform` | string | **Required.** Platform: `instagram`, `tiktok`, `youtube`, `twitch` |
| `usernames` | array | Array of usernames to enrich |
| `userIds` | array | Array of user IDs to enrich |
| `addToWeeklyEnrichment` | boolean | Add to weekly enrichment queue |

Maximum 100 items per request.

## Rate Limits

API rate limits depend on your subscription plan. Contact support for details about your specific limits.

## Support

For API support, contact: api-support@infludata.com
