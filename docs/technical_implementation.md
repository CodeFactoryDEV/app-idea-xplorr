# Technical Implementation - xplorr

This document provides a high-level overview of the technical implementation for xplorr, a location-based discovery app.

---

## 📚 Documentation Structure

For detailed implementation guides, please refer to the platform-specific documentation:

- **[iOS Development Guidelines](./technical_implementation_ios.md)** 📱
  - Swift & SwiftUI implementation
  - Liquid Glass design system
  - MapKit integration
  - Complete code examples and workflow

- **[Backend Development Guidelines](./technical_implementation_backend.md)** ⚙️
  - Spring Boot & Java implementation
  - REST API endpoints
  - PostgreSQL & PostGIS setup
  - Deployment strategies

---

## 🎯 Core Algorithm

### Randomization Flow

```
1. User taps X logo → Trigger randomization
2. X logo rotates, map animates routes
3. Fetch places within radius (configurable)
4. Filter out:
   - Already visited places
   - No-go tagged locations
   - Currently closed venues (optional)
5. Weighted random selection based on:
   - Distance (closer = slightly higher weight)
   - Category diversity (avoid clustering)
   - Ratings (minimum threshold)
6. Animate route pulse → Present result
7. Store selection in history
```

### Weighting System

**Distance Weight:**
- Places within 0.5 mi: 1.5x weight
- Places 0.5-1 mi: 1.0x weight
- Places 1-2 mi: 0.75x weight
- Places 2+ mi: 0.5x weight

**Category Diversity:**
- If last 2 visits were same category: 0.5x weight for that category
- Different categories get 1.0x weight
- Ensures varied experiences

**Rating Threshold:**
- Minimum 3.0 star rating (configurable)
- Higher rated places get slight boost: `rating / 5.0` multiplier

---

## 🛠️ Tech Stack Overview

### Frontend (iOS Native)
- **Swift** with **SwiftUI**
- **MapKit** for maps
- **CoreLocation** for GPS
- **Combine** for reactive programming
- **URLSession** for networking

**See: [iOS Implementation Guide](./technical_implementation_ios.md)**

---

### Backend (Spring Boot API)
- **Java 17+** with **Spring Boot 3.x**
- **Maven** for build management
- **PostgreSQL** with **PostGIS**
- **Spring Data JPA**
- **JWT** authentication

**See: [Backend Implementation Guide](./technical_implementation_backend.md)**

---

## 📐 Architecture Overview

### System Components

```
┌─────────────┐
│  iOS App    │
│  (Swift)    │
└──────┬──────┘
       │ HTTPS/REST
       │
┌──────▼──────────┐
│  Spring Boot    │
│     API         │
└──────┬──────────┘
       │
┌──────▼──────────┐      ┌────────────────┐
│  PostgreSQL     │      │ External APIs  │
│  + PostGIS      │      │ (Google Maps)  │
└─────────────────┘      └────────────────┘
```

### Communication Flow

```
1. User Action (iOS)
   ↓
2. API Request (REST/JSON)
   ↓
3. Backend Processing (Spring Boot)
   ↓
4. Database Query (PostgreSQL/PostGIS)
   ↓
5. External API Call (Google Places)
   ↓
6. Response to iOS
   ↓
7. UI Update (SwiftUI)
```

---

## 📱 Data Models

### Shared Data Structure

Both iOS (Swift) and Backend (Java) use equivalent models:

**User:**
- id: UUID
- createdAt: Date
- preferences: Object (search radius, categories, min rating)
- visitHistory: Array of place IDs
- noGoList: Array of place IDs
- fixedLocation: Optional location

**Place:**
- id: String
- name: String
- location: Coordinates (lat/lng)
- categories: Array of strings
- rating: Number
- priceLevel: Number
- photos: Array of URLs
- hours: String
- estimatedVisitTime: Number (minutes)

**Itinerary:**
- id: UUID
- places: Array of ItineraryPlace
- createdAt: Date
- totalDuration: Number (minutes)

---

## 🔄 Development Workflow

### Phase 1: Foundation (Week 1-2)
- Set up iOS project structure
- Set up Spring Boot project
- Configure PostgreSQL with PostGIS
- Create base data models

### Phase 2: Core Features (Week 3-4)
- iOS: MapKit integration
- Backend: Place API endpoints
- iOS: Location services
- Backend: Randomization logic

### Phase 3: Integration (Week 5-6)
- Connect iOS to backend API
- Implement authentication
- Add caching layer
- Test end-to-end flows

### Phase 4: Polish (Week 7-8)
- Animations and transitions
- Error handling
- Performance optimization
- Testing and bug fixes

---

## 🔒 Privacy & Security

### Data Protection
- **Location Data**: Stored locally on device, only sent to server for queries
- **Visit History**: Encrypted and stored per device
- **No-Go List**: Private, never shared with external services
- **JWT Authentication**: Stateless, secure token-based auth
- **HTTPS Only**: All API communications encrypted

### User Permissions
**Required:**
- Location (when in use)

**Optional:**
- Analytics (can be disabled)

**Never Requested:**
- Background location
- Contacts
- Camera (unless contributing photos)

---

## 🧪 Testing Strategy

### iOS Testing
- **XCTest**: Unit tests for ViewModels and Services
- **XCUITest**: UI and integration tests
- **Instruments**: Performance profiling

### Backend Testing
- **JUnit**: Unit tests for services
- **MockMvc**: Controller integration tests
- **TestContainers**: Database integration tests

### End-to-End Testing
- Test complete user flows
- Verify iOS ↔ Backend communication
- Test randomization algorithm
- Performance benchmarks

---

## 📦 Deployment

### iOS App
- **TestFlight**: Beta distribution
- **App Store**: Production release
- Xcode Cloud for CI/CD

### Backend API
- **Docker**: Containerized deployment
- **Railway/Heroku**: Cloud hosting
- **PostgreSQL**: Managed database service
- GitHub Actions for CI/CD

---

## 🔧 Configuration

### Configurable Parameters

**Randomization:**
- `SEARCH_RADIUS_DEFAULT`: 5 miles
- `MIN_RATING`: 3.0
- `SPIN_DURATION_MIN`: 1500ms
- `SPIN_DURATION_MAX`: 3000ms

**UI:**
- `ANIMATION_SPEED`: 1.0 (multiplier)
- `MAP_ZOOM_LEVEL`: 14
- `MARKER_CLUSTER_RADIUS`: 50 pixels

**Caching:**
- `PLACE_CACHE_TTL`: 1 hour
- `USER_LOCATION_UPDATE_INTERVAL`: 30 seconds

**API:**
- `API_TIMEOUT`: 30 seconds
- `MAX_RETRIES`: 3
- `RATE_LIMIT`: 100 requests/minute

---

## 📊 API Endpoints Summary

### Places
- `GET /api/places/nearby` - Get nearby places
- `GET /api/places/{id}` - Get place details
- `POST /api/places/random` - Get random place

### Itineraries
- `POST /api/itineraries` - Create itinerary
- `GET /api/itineraries/{id}` - Get itinerary
- `PUT /api/itineraries/{id}` - Update itinerary
- `DELETE /api/itineraries/{id}` - Delete itinerary

### Users
- `POST /api/users/register` - Register user
- `GET /api/users/me` - Get current user
- `PUT /api/users/preferences` - Update preferences
- `POST /api/users/visit-history/{id}` - Add to history
- `POST /api/users/no-go/{id}` - Add to no-go list

---

## 📚 Additional Resources

### Implementation Guides
- **[iOS Development Guidelines](./technical_implementation_ios.md)** - Complete iOS guide
- **[Backend Development Guidelines](./technical_implementation_backend.md)** - Complete backend guide

### Design & Documentation
- [Main README](../README.md) - Project overview
- [Name Candidates](./name_candidates.md) - Branding decisions

### External References
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [Apple Liquid Glass Design](https://developer.apple.com/documentation/TechnologyOverviews/liquid-glass)
- [SwiftUI Documentation](https://developer.apple.com/documentation/swiftui/)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [PostGIS Documentation](https://postgis.net/documentation/)

---

## 🚀 Getting Started

1. **Read the Overview** (you are here)
2. **iOS Developers**: See [iOS Implementation Guide](./technical_implementation_ios.md)
3. **Backend Developers**: See [Backend Implementation Guide](./technical_implementation_backend.md)
4. **Set up local environment** following platform-specific guides
5. **Follow the development workflow** outlined in your respective guide

---

**Last Updated:** October 18, 2025

**Status:** MVP Development Phase
