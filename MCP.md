# Travel API MCP

MCP server for hotels and roundtrip travel packages, enriched with Google Places data.

**Live URL:** https://mcp-dev.agentify.travel/74ed4a32-8542-4d1b-bf44-c8332ed0cfc1/

> ‼️ The MCP server supports HTTP streaming transport only. Clients must use streaming HTTP for tool invocation and responses. 

## MCP Client SDKs

Official client SDKs (see Model Context Protocol site):

- TypeScript / Node.js  
  - Docs: https://modelcontextprotocol.org/docs/clients/typescript  
  - Repo: https://github.com/modelcontextprotocol/typescript-sdk  
  - Package: https://www.npmjs.com/package/@modelcontextprotocol/sdk

- Python  
  - Docs: https://modelcontextprotocol.org/docs/clients/python  
  - Repo: https://github.com/modelcontextprotocol/python-sdk  
  - Package: https://pypi.org/project/mcp

Use these SDKs with streaming HTTP transport for invoking this server’s tools. 

### MCP Inspector

You can use the MCP inspector locally to get familiar with the MNP server.

1. Open MCP Inspector: `npx @modelcontextprotocol/inspector`
2. Add Server: paste the live URL (above) and select Streaming HTTP transport.
3. Connect and open Tools panel to see available tool names.
4. Invoke a tool by supplying JSON arguments. Examples:

getHotels (top-rated Greece hotels):
```json
{
  "name": "getHotels",
  "arguments": {
    "country": "GR",
    "limit": 5,
    "sort": "rating",
    "order": "desc"
  }
}
```

getDestinations (list all destination codes):
```json
{ "name": "getDestinations", "arguments": {} }
```

getRoundtrips (sample with hotel expansion):
```json
{
  "name": "getRoundtrips",
  "arguments": {
    "destination": "GR",
    "expandHotels": true,
    "limit": 3
  }
}
```

resolvePhoto (after copying a base64 photo ref from a hotel response):
```json
{
  "name": "resolvePhoto",
  "arguments": {
    "ref": "cGxhY2VzLzEyMy9waG90b3MvYWJj...",
    "size": "medium"
  }
}
```

5. Inspect Response: success/data/meta or error structures match the documented format.
6. Iterate quickly: adjust arguments (paging, sorting) and re-run to learn data shapes.

## Overview

This MCP serves travel data including:
- **413 hotels** across 62 countries
- **345 roundtrip packages** across 77 destinations
- Google Places enrichment (ratings, reviews, photos, coordinates)
- Photo proxy with caching for Google Places images

## MCP Tools

### Hotels (`getDestinations`)

List destinations. Returns all unique destination codes from the roundtrip database
E.g. GR for Greece, IT for Italy, etc.

### Hotels (`getHotels`)

List hotels. Returns a paginated list of hotels with optional filtering and sorting

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `country` | string | Filter by country code(s), comma-separated | `country=GR,IT` |
| `city` | string | Filter by city name (partial match) | `city=athens` |
| `minRating` | number | Minimum Google rating | `minRating=4.0` |
| `maxRating` | number | Maximum Google rating | `maxRating=5.0` |
| `minPrice` | number | Minimum price (EUR) | `minPrice=100` |
| `maxPrice` | number | Maximum price (EUR) | `maxPrice=500` |
| `sort` | string | Sort field: `name`, `price`, `rating`, `reviews` | `sort=rating` |
| `order` | string | Sort order: `asc`, `desc` | `order=desc` |
| `page` | number | Page number (default: 1) | `page=2` |
| `limit` | number | Items per page (default: 20) | `limit=50` |

### Hotel by Id (`getHotelById`)

Get hotel by ID. Returns a single hotel by its unique identifier

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `id` | string | hotel id | `hotel-12345` |


### Roundtrips (`getRoundtrips`)

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `destination` | string | Filter by destination code(s), comma-separated | `destination=GR,TR` |
| `hasHotels` | boolean | Only roundtrips with hotels | `hasHotels=true` |
| `expandHotels` | boolean | Include full hotel objects | `expandHotels=true` |
| `sort` | string | Sort field: `name`, `price`, `duration` | `sort=price` |
| `order` | string | Sort order: `asc`, `desc` | `order=asc` |
| `page` | number | Page number (default: 1) | `page=1` |
| `limit` | number | Items per page (default: 20) | `limit=10` |

### Roundtrip by Id (`getRoundtripById`)

Get roundtrip by ID. Returns a single roundtrip package by its unique identifier

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `id` | string | roundtrip ID | roundtrip-1224352435 | 

### Roundtrips with Hotel (`getHotelRoundtrips`)

Get roundtrips for hotel. Returns all roundtrip packages that include this hotel

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `id` | string | hotel id | `hotel-12345` |


### Get Hotels for Roundtrip (`getRoundtripHotels`)

Get hotels in roundtrip. Returns all hotels included in this roundtrip package

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `id` | string | rondtrip id | `roundtrip-12345` |


### Photos (`resolvePhoto`)

Resolve photo URL. Resolves a base64-encoded Google Places photo reference to an actual image URL.
Returns a 302 redirect to the Google CDN URL.

Photo URLs are cached for 20 hours to minimize API calls.


| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `ref` | base64 | base64-encoded photo reference from the other tools. | cGxhY2VzL0NoSU...
| `size` | string | Image size: `small` (400px), `medium` (800px), `large` (1200px) | `size=large` |

---

## Response Format

All responses follow this structure:

### Success Response

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "total": 413,
    "page": 1,
    "limit": 20,
    "pages": 21
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Hotel not found"
  }
}
```

---

## Data Models

### Hotel

```json
{
  "id": "hotel-123",
  "name": "Grand Hotel Athens",
  "type": "hotel",
  "country": "GR",
  "city": "Athens",
  "roomCount": 150,
  "description": "Luxury hotel in the heart of Athens...",
  "location": {
    "lat": 37.9755,
    "lng": 23.7348,
    "address": "123 Syntagma Square, Athens",
    "timezone": "Europe/Athens"
  },
  "pricing": {
    "basePrice": 245.00,
    "currency": "EUR"
  },
  "google": {
    "placeId": "ChIJ...",
    "rating": 4.5,
    "reviewCount": 1234,
    "priceLevel": 3,
    "types": ["lodging", "point_of_interest"],
    "photos": [
      "cGxhY2VzLzEyMy9waG90b3MvYWJj..."
    ],
    "url": "https://maps.google.com/?cid=..."
  },
  "timezoneUtcOffset": 120
}
```

### Roundtrip

```json
{
  "id": "RT-456",
  "name": "Greek Islands Explorer",
  "destination": "GR",
  "duration": "8 days / 7 nights",
  "description": "Discover the beauty of the Greek islands...",
  "pricing": {
    "basePrice": 1899.00,
    "currency": "EUR"
  },
  "hotels": [
    { "id": "hotel-123", "name": "Grand Hotel Athens" }
  ],
  "hotelCount": 3
}
```

With `expandHotels=true`, the `hotels` array contains full hotel objects.

---

## Photo Proxy

Photos from Google Places have temporary URLs that expire after 24-48 hours. The photo proxy solves this by:

1. Storing base64-encoded photo references in hotel/roundtrip data
2. Resolving references to actual URLs on-demand via `:ref`
3. Caching resolved URLs for 20 hours
4. Returning HTTP 302 redirects for direct `<img>` tag usage

### Size Options

| Size | Max Height | Use Case |
|------|------------|----------|
| `small` | 400px | Thumbnails, lists |
| `medium` | 800px | Cards, previews |
| `large` | 1200px | Full-screen, detail views |

---

## Data Sources

- **Hotels & Roundtrips:** Internal travel catalog
- **Google Places:** Enrichment via Places API (New)
  - Ratings and review counts
  - Photos (up to 10 per hotel)
  - Precise coordinates
  - Place types and price levels


