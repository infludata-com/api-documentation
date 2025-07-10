# InfluData API Documentation

**Updated: March 21st, 2025**

## Table of Contents

1. [Authentication](#authentication)
2. [API Rate Limit](#api-rate-limit)
3. [Endpoints](#endpoints)
   - [Influencer Profiles](#influencer-profiles)
   - [Client Information](#client-information)
   - [Influencer Discovery](#influencer-discovery)
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

Check if an influencer's data and audience report are ready for retrieval.

**Parameters:**
- `platform` (mandatory): `instagram`, `tiktok`, `youtube`, or `twitch`
- `username`, `userId`, or `_id` (one mandatory): Influencer identifier

**Examples:**
```http
GET https://app.infludata.com/api/externalAPI/checkDataStatus?platform=instagram&userId=248312442

GET https://app.infludata.com/api/externalAPI/checkDataStatus?platform=tiktok&username=charlidamelio

GET https://app.infludata.com/api/externalAPI/checkDataStatus?_id=507f1f77bcf86cd799439011
```

**Responses:**
- **200**: JSON object with status information
  ```json
  {
    "status": "READY" | "NOT READY" | "NOT FOUND" | "BAD REQUEST"
  }
  ```
  - `READY`: Full data including audience analysis available
  - `NOT READY`: Basic profile exists, audience analysis in progress
  - `NOT FOUND`: User not found in our database
  - `BAD REQUEST`: Invalid parameters provided
- **500**: Server error

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
- `keywords` (optional): Search terms in creator content, bio, username (comma-separated)
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
2. **Use `includeAudienceReport=false`** for basic profile data to save tokens
3. **Cache results** appropriately to minimize redundant API calls
4. **Implement token threshold alerts** in your application

### Data Retrieval Workflow
1. **Check data status** before requesting expensive audience reports
2. **Handle 201/202 responses** gracefully for new user enrichment
3. **Implement retry logic** for 202 responses (audience analysis pending)
4. **Use pagination** effectively for discovery searches

### Performance Optimization
1. **Batch similar requests** when possible
2. **Use appropriate filtering** to reduce result set size
3. **Implement client-side caching** for frequently accessed profiles
4. **Consider time zones** for optimal API response times

### Error Handling
1. **Implement exponential backoff** for temporary failures
2. **Log error details** for debugging and monitoring
3. **Provide user-friendly error messages** in your application
4. **Handle platform-specific limitations** gracefully

### Security Considerations
1. **Never expose API tokens** in client-side code
2. **Use HTTPS** for all API communications
3. **Implement proper token rotation** policies
4. **Monitor for unusual usage patterns**

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

// 2. Check if user data is ready (using internal _id)
const statusCheck = await fetch(
  '/api/externalAPI/checkDataStatus?_id=507f1f77bcf86cd799439011',
  { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
);
const { status } = await statusCheck.json();

// 3. Retrieve user data based on status
if (status === 'READY') {
  const userData = await fetch(
    '/api/externalAPI/getUserData?_id=507f1f77bcf86cd799439011&includeAudienceReport=true',
    { headers: { 'Authorization': 'Bearer YOUR_TOKEN' } }
  );
  const user = await userData.json();
  console.log('Full user data:', user);
} else if (status === 'NOT READY') {
  console.log('User found, audience analysis in progress');
} else {
  console.log('User not found');
}
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

// Usage
const fashionInfluencers = await searchInfluencers(
  'keywords=fashion&country=Germany&followerMin=10000&engagementMin=3',
  'instagram'
);
```

### Multi-platform Search

```javascript
async function searchAllPlatforms(keywords) {
  const platforms = ['instagram', 'tiktok', 'youtube', 'twitch'];
  const results = {};

  for (const platform of platforms) {
    try {
      const response = await fetch(
        `/api/externalAPI/discovery?platform=${platform}&keywords=${keywords}&followerMin=1000`,
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
| `/checkDataStatus` | 0 tokens | 0 tokens | Free utility endpoint |
| `/getClientInfo` | 0 tokens | 0 tokens | Free utility endpoint |
| `/getCitiesForCountry` | 0 tokens | 0 tokens | Free utility endpoint |

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
**Last Updated**: March 21st, 2025  
**API Version**: v1  
**Effective Date**: March 21st, 2025

*This documentation is proprietary and confidential. Distribution is restricted to authorized API clients only.*
