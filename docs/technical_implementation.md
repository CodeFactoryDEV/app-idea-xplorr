# Technical Implementation - xplorr

This document contains the technical details, architecture decisions, and implementation guidelines for xplorr.

---

## 🎯 The Algorithm

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

### Animation Timing

1. **Route Pulse Animation**: 1.5-3 seconds (randomized duration)
2. **X Logo Rotation**: 360° over animation duration
3. **Place Name Cycling**: Changes every 200ms during spin
4. **Destination Reveal**: 500ms fade-in after selection

---

## 🛠️ Tech Stack

### Frontend

**Framework:**
- **React Native** - Cross-platform mobile (iOS/Android)
- **Expo** (optional) - Faster development and deployment

**Map Integration:**
- **react-native-maps** - Unified map component
- **MapKit** (iOS) / **Google Maps** (Android)
- Custom marker clustering for performance

**Animation:**
- **Reanimated 2** - 60fps animations on native thread
- **Gesture Handler** - Native gesture recognition
- **Lottie** - Complex animations (logo spin, route pulse)

**State Management:**
- **Zustand** / **Redux Toolkit** - Global state
- **React Query** - Server state & caching
- **AsyncStorage** - Persistent local data

**UI Components:**
- Custom design system
- **React Native Paper** (optional base)
- SVG support via **react-native-svg**

---

### Backend

**API Server:**
- **Node.js** + **Express** - RESTful API
- **TypeScript** - Type safety
- Alternative: **Firebase Functions** for serverless

**Database:**
- **PostgreSQL** (primary) - User data, visit history
- **PostGIS** extension - Geospatial queries
- **Redis** - Session management, rate limiting, place data caching

**Authentication:**
- **Firebase Auth** / **Auth0** - OAuth, anonymous auth
- JWT tokens for API requests
- Device-based auth for privacy

**File Storage:**
- **AWS S3** / **Cloudflare R2** - Place images
- CDN for fast image delivery

---

### APIs & Services

**Place Data:**
- **Google Places API** (primary)
  - Nearby search
  - Place details
  - Place photos
- **Foursquare API** (fallback/enrichment)
- **OpenStreetMap** - Free alternative for basic data

**Geolocation:**
- **React Native Geolocation API**
- Foreground location only (privacy-focused)
- GPS + Network positioning

**Analytics:**
- **PostHog** / **Mixpanel** - User analytics (opt-in)
- Custom event tracking
- No PII collection

**Monitoring:**
- **Sentry** - Error tracking
- **LogRocket** - Session replay (development only)

---

## 📐 Architecture

### App Structure

```
xplorr/
├── src/
│   ├── screens/          # Main app screens
│   │   ├── MapScreen.tsx
│   │   ├── RouteAnimationScreen.tsx
│   │   ├── PlaceDetailsScreen.tsx
│   │   ├── ItineraryScreen.tsx
│   │   └── SettingsScreen.tsx
│   ├── components/       # Reusable components
│   │   ├── Map/
│   │   ├── Markers/
│   │   ├── XLogo/
│   │   └── UI/
│   ├── hooks/           # Custom React hooks
│   │   ├── useLocation.ts
│   │   ├── useRandomizer.ts
│   │   └── usePlaces.ts
│   ├── services/        # API & business logic
│   │   ├── places.ts
│   │   ├── storage.ts
│   │   └── randomizer.ts
│   ├── store/           # State management
│   ├── utils/           # Helper functions
│   └── types/           # TypeScript types
├── assets/              # Images, fonts, animations
├── docs/                # Documentation
└── tests/              # Unit & integration tests
```

### Data Models

**User Profile:**
```typescript
interface UserProfile {
  id: string;
  createdAt: Date;
  preferences: {
    searchRadius: number; // miles
    categories: string[];
    minRating: number;
  };
  visitHistory: string[]; // place IDs
  noGoList: string[];
  fixedLocation?: Location;
}
```

**Place:**
```typescript
interface Place {
  id: string;
  name: string;
  location: Location;
  category: string[];
  rating: number;
  priceLevel: number;
  photos: string[];
  hours: string;
  estimatedVisitTime: number; // minutes
  distance?: number; // calculated
}
```

**Itinerary:**
```typescript
interface Itinerary {
  id: string;
  places: {
    place: Place;
    startTime: Date;
    duration: number;
    walkingTime?: number; // to next place
  }[];
  createdAt: Date;
  totalDuration: number;
}
```

---

## 🔒 Privacy & Security

### Data Storage

**Local Only:**
- Visit history (encrypted)
- No-go list (encrypted)
- Location preferences
- App settings

**Server-Side:**
- Anonymous user ID (UUID)
- Aggregated usage statistics (if opted in)
- No location data stored server-side

### Encryption

- **AsyncStorage encryption** via `react-native-encrypted-storage`
- **API requests** over HTTPS only
- **No tracking** without explicit user consent

### Permissions

**Required:**
- Location (when in use)

**Optional:**
- Analytics (can be disabled)

**Never Requested:**
- Background location
- Contacts
- Camera (unless user wants to contribute photos)

---

## 🚀 Development Setup

### Prerequisites

```bash
- Node.js 18+
- npm or yarn
- Xcode (for iOS)
- Android Studio (for Android)
- CocoaPods (for iOS dependencies)
```

### Installation

```bash
# Clone repository
git clone https://github.com/yourusername/xplorr.git
cd xplorr

# Install dependencies
npm install

# iOS only
cd ios && pod install && cd ..

# Start Metro bundler
npm start

# Run on iOS
npm run ios

# Run on Android
npm run android
```

### Environment Variables

Create `.env` file:

```bash
GOOGLE_MAPS_API_KEY=your_key_here
FOURSQUARE_API_KEY=your_key_here
API_BASE_URL=http://localhost:3000
```

### Backend Setup

```bash
cd backend

# Install dependencies
npm install

# Set up database
npm run db:migrate

# Start development server
npm run dev
```

---

## 🧪 Testing

### Unit Tests

```bash
npm test
```

### E2E Tests

```bash
npm run test:e2e
```

### Coverage

```bash
npm run test:coverage
```

---

## 📦 Deployment

### Mobile App

**iOS:**
- TestFlight for beta testing
- App Store Connect for production

**Android:**
- Google Play Console
- Internal testing track → Beta → Production

### Backend

**Options:**
- **Railway** / **Render** - Easy deployment
- **AWS EC2** / **ECS** - Full control
- **Firebase** - Serverless option

### CI/CD

- **GitHub Actions** for automated builds
- **Fastlane** for mobile deployment automation
- **Sentry** for release monitoring

---

## 🔧 Configuration

### Customizable Parameters

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

---

## 📚 Additional Resources

- [API Documentation](./api_documentation.md) (coming soon)
- [Design System](./design_system.md) (coming soon)
- [Contributing Guidelines](../CONTRIBUTING.md) (coming soon)
- [Testing Strategy](./testing_strategy.md) (coming soon)

---

**Last Updated:** October 18, 2025

