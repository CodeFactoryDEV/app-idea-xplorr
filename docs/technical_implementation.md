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

## 🛠️ Tech Stack (MVP)

### Frontend - iOS Native

**Framework:**
- **Swift** - Native iOS development
- **SwiftUI** - Modern declarative UI framework
- **UIKit** (where needed) - For complex animations and map interactions

**Map Integration:**
- **MapKit** - Apple's native mapping framework
- **CoreLocation** - GPS and location services
- **MKAnnotation** - Custom markers and pins

**Animation:**
- **SwiftUI Animations** - Built-in animation system
- **Core Animation** - Advanced transitions and effects
- **UIView.animate** - Logo rotation and route pulse

**State Management:**
- **@StateObject** / **@ObservableObject** - SwiftUI state management
- **Combine** - Reactive programming for async operations
- **UserDefaults** / **Keychain** - Local data persistence

**Networking:**
- **URLSession** - Native HTTP client
- **Codable** - JSON encoding/decoding
- **async/await** - Modern Swift concurrency

**UI Components:**
- Custom SwiftUI views
- SF Symbols for icons
- Native iOS design patterns

---

### Backend - Spring Boot API

**Framework:**
- **Spring Boot 3.x** - Java web framework
- **Maven** - Dependency management and build tool
- **Java 17+** - LTS version

**API Layer:**
- **Spring Web** - RESTful API endpoints
- **Spring Security** - Authentication and authorization
- **JWT** - Token-based authentication
- **Spring Validation** - Request validation

**Database:**
- **PostgreSQL** - Primary database
- **PostGIS** extension - Geospatial queries for nearby places
- **Spring Data JPA** - ORM and database access
- **HikariCP** - Connection pooling

**Caching:**
- **Spring Cache** - Method-level caching
- **Caffeine** - In-memory cache (for MVP, can migrate to Redis later)

**Third-party APIs:**
- **RestTemplate** / **WebClient** - HTTP client for external APIs
- Google Places API integration
- Foursquare API (fallback)

**Monitoring & Logging:**
- **Spring Boot Actuator** - Health checks and metrics
- **SLF4J** + **Logback** - Logging
- **Micrometer** - Application metrics

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

### iOS App Structure

```
xplorr-ios/
├── xplorr/
│   ├── App/
│   │   ├── xplorrApp.swift           # App entry point
│   │   └── AppDelegate.swift         # App lifecycle
│   ├── Views/                        # SwiftUI views
│   │   ├── MapView.swift             # Main map screen
│   │   ├── RouteAnimationView.swift  # Randomizer animation
│   │   ├── PlaceDetailView.swift     # Place details screen
│   │   ├── ItineraryView.swift       # Trip itinerary
│   │   └── SettingsView.swift        # Settings screen
│   ├── Components/                   # Reusable UI components
│   │   ├── Map/
│   │   │   ├── MapViewController.swift
│   │   │   └── PlaceAnnotation.swift
│   │   ├── XLogoView.swift           # Animated X logo
│   │   └── UI/
│   │       ├── PillButton.swift
│   │       └── FloatingCard.swift
│   ├── ViewModels/                   # State and business logic
│   │   ├── MapViewModel.swift
│   │   ├── RandomizerViewModel.swift
│   │   └── ItineraryViewModel.swift
│   ├── Services/                     # API and data services
│   │   ├── PlacesService.swift
│   │   ├── APIClient.swift
│   │   └── LocationManager.swift
│   ├── Models/                       # Data models
│   │   ├── Place.swift
│   │   ├── Itinerary.swift
│   │   └── User.swift
│   ├── Storage/                      # Local persistence
│   │   ├── UserDefaultsManager.swift
│   │   └── KeychainManager.swift
│   ├── Utils/                        # Helper extensions
│   │   └── Extensions/
│   └── Resources/                    # Assets, colors, fonts
│       ├── Assets.xcassets
│       └── Colors.swift
├── xplorrTests/                      # Unit tests
└── xplorrUITests/                    # UI tests
```

### Backend Structure

```
xplorr-api/
├── src/
│   ├── main/
│   │   ├── java/com/xplorr/
│   │   │   ├── XplorrApplication.java     # Spring Boot entry point
│   │   │   ├── config/                    # Configuration
│   │   │   │   ├── SecurityConfig.java
│   │   │   │   ├── CacheConfig.java
│   │   │   │   └── WebConfig.java
│   │   │   ├── controller/                # REST endpoints
│   │   │   │   ├── PlaceController.java
│   │   │   │   ├── ItineraryController.java
│   │   │   │   └── UserController.java
│   │   │   ├── service/                   # Business logic
│   │   │   │   ├── PlaceService.java
│   │   │   │   ├── RandomizerService.java
│   │   │   │   └── ThirdPartyAPIService.java
│   │   │   ├── repository/                # Data access
│   │   │   │   ├── PlaceRepository.java
│   │   │   │   ├── ItineraryRepository.java
│   │   │   │   └── UserRepository.java
│   │   │   ├── model/                     # Entities
│   │   │   │   ├── Place.java
│   │   │   │   ├── Itinerary.java
│   │   │   │   └── User.java
│   │   │   ├── dto/                       # Data Transfer Objects
│   │   │   │   ├── PlaceDTO.java
│   │   │   │   └── ItineraryDTO.java
│   │   │   ├── security/                  # Auth & security
│   │   │   │   ├── JwtTokenProvider.java
│   │   │   │   └── JwtAuthFilter.java
│   │   │   └── exception/                 # Error handling
│   │   │       ├── GlobalExceptionHandler.java
│   │   │       └── ResourceNotFoundException.java
│   │   └── resources/
│   │       ├── application.yml            # App configuration
│   │       ├── application-dev.yml
│   │       └── application-prod.yml
│   └── test/
│       └── java/com/xplorr/
│           ├── controller/
│           ├── service/
│           └── repository/
├── pom.xml                                # Maven dependencies
└── Dockerfile                             # Container deployment
```

### Data Models

**Swift Models (iOS):**
```swift
struct User: Codable {
    let id: UUID
    let createdAt: Date
    var preferences: UserPreferences
    var visitHistory: [String] // place IDs
    var noGoList: [String]
    var fixedLocation: Location?
}

struct UserPreferences: Codable {
    var searchRadius: Double // miles
    var categories: [String]
    var minRating: Double
}

struct Place: Codable, Identifiable {
    let id: String
    let name: String
    let location: Location
    let categories: [String]
    let rating: Double
    let priceLevel: Int
    let photos: [String]
    let hours: String
    let estimatedVisitTime: Int // minutes
    var distance: Double? // calculated
}

struct Location: Codable {
    let latitude: Double
    let longitude: Double
}

struct Itinerary: Codable, Identifiable {
    let id: UUID
    var places: [ItineraryPlace]
    let createdAt: Date
    var totalDuration: Int
}

struct ItineraryPlace: Codable, Identifiable {
    let id: UUID
    let place: Place
    let startTime: Date
    let duration: Int
    let walkingTime: Int? // to next place
}
```

**Java Entities (Spring Boot):**
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    
    private LocalDateTime createdAt;
    
    @Column(columnDefinition = "jsonb")
    private UserPreferences preferences;
    
    @ElementCollection
    private List<String> visitHistory;
    
    @ElementCollection
    private List<String> noGoList;
    
    @Column(columnDefinition = "geometry(Point,4326)")
    private Point fixedLocation;
}

@Entity
@Table(name = "places")
public class Place {
    @Id
    private String id; // from external API
    
    private String name;
    
    @Column(columnDefinition = "geometry(Point,4326)")
    private Point location;
    
    @ElementCollection
    private List<String> categories;
    
    private Double rating;
    private Integer priceLevel;
    
    @ElementCollection
    private List<String> photos;
    
    private String hours;
    private Integer estimatedVisitTime;
    
    @CreatedDate
    private LocalDateTime cachedAt;
}

@Entity
@Table(name = "itineraries")
public class Itinerary {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    
    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
    
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "itinerary")
    private List<ItineraryPlace> places;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    private Integer totalDuration;
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

**iOS Development:**
```bash
- macOS (for iOS development)
- Xcode 15+
- Swift 5.9+
- CocoaPods or Swift Package Manager
- iOS Simulator or physical device
```

**Backend Development:**
```bash
- Java 17 or higher (LTS)
- Maven 3.8+
- PostgreSQL 14+
- PostGIS extension
- IntelliJ IDEA or VS Code (optional)
```

### iOS App Setup

```bash
# Clone repository
git clone https://github.com/yourusername/xplorr.git
cd xplorr/xplorr-ios

# Install dependencies (if using CocoaPods)
pod install

# Open in Xcode
open xplorr.xcworkspace

# Or use Swift Package Manager (managed by Xcode)
# File -> Add Packages -> Add your dependencies

# Build and run
# Select simulator/device in Xcode
# Press Cmd+R to build and run
```

### Backend Setup

```bash
cd xplorr-api

# Set up PostgreSQL
createdb xplorr
psql -d xplorr -c "CREATE EXTENSION postgis;"

# Configure application properties
cp src/main/resources/application-example.yml src/main/resources/application-dev.yml
# Edit application-dev.yml with your database credentials

# Build the project
mvn clean install

# Run the application
mvn spring-boot:run

# Or run with specific profile
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Run tests
mvn test

# Package for deployment
mvn clean package
```

### Environment Configuration

**iOS - Info.plist or Config.xcconfig:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
    <key>API_BASE_URL</key>
    <string>http://localhost:8080</string>
    <key>GOOGLE_MAPS_API_KEY</key>
    <string>your_key_here</string>
</dict>
</plist>
```

**Backend - application-dev.yml:**
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/xplorr
    username: postgres
    password: your_password
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.spatial.dialect.postgis.PostgisDialect

server:
  port: 8080

jwt:
  secret: your_jwt_secret_key_change_in_production
  expiration: 86400000 # 24 hours

external-api:
  google-places:
    api-key: your_google_places_key
  foursquare:
    api-key: your_foursquare_key

cache:
  ttl:
    places: 3600 # 1 hour in seconds
```

---

## 🧪 Testing

### iOS Testing

**Unit Tests (XCTest):**
```swift
import XCTest
@testable import xplorr

class RandomizerServiceTests: XCTestCase {
    func testRandomPlaceSelection() {
        let service = RandomizerService()
        let places = [/* mock places */]
        let selected = service.selectRandom(from: places)
        XCTAssertNotNil(selected)
    }
}
```

**Run tests:**
```bash
# Command line
xcodebuild test -workspace xplorr.xcworkspace \
  -scheme xplorr -destination 'platform=iOS Simulator,name=iPhone 14'

# Or in Xcode: Cmd+U
```

**UI Tests:**
```swift
class xplorrUITests: XCTestCase {
    func testMapViewLoads() {
        let app = XCUIApplication()
        app.launch()
        XCTAssert(app.maps.element.exists)
    }
}
```

### Backend Testing

**Unit Tests (JUnit):**
```java
@SpringBootTest
class RandomizerServiceTest {
    
    @Autowired
    private RandomizerService randomizerService;
    
    @Test
    void testSelectRandomPlace() {
        List<Place> places = createMockPlaces();
        Place selected = randomizerService.selectRandom(places);
        assertNotNull(selected);
    }
}
```

**Integration Tests:**
```java
@SpringBootTest
@AutoConfigureMockMvc
class PlaceControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void testGetNearbyPlaces() throws Exception {
        mockMvc.perform(get("/api/places/nearby")
                .param("latitude", "37.7749")
                .param("longitude", "-122.4194")
                .param("radius", "5"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.length()").isNumber());
    }
}
```

**Run tests:**
```bash
# All tests
mvn test

# Specific test class
mvn test -Dtest=PlaceControllerIntegrationTest

# With coverage (using JaCoCo)
mvn clean test jacoco:report

# View coverage report
open target/site/jacoco/index.html
```

**Test Coverage:**
```bash
# iOS - use Xcode's built-in coverage
# Enable in scheme: Edit Scheme → Test → Options → Code Coverage

# Backend - JaCoCo plugin in pom.xml
mvn jacoco:report
```

---

## 📦 Deployment

### iOS App

**Development:**
- Run directly from Xcode to simulator or connected device
- Use Xcode debugging tools

**Beta Testing:**
- **TestFlight** - Apple's beta distribution platform
- Upload IPA via Xcode or Application Loader
- Invite internal and external testers

**Production:**
- **App Store Connect** - Apple's app distribution portal
- Archive app in Xcode (Product → Archive)
- Upload to App Store Connect
- Submit for App Review
- Release to App Store

**Build Process:**
```bash
# Archive for distribution
xcodebuild archive \
  -workspace xplorr.xcworkspace \
  -scheme xplorr \
  -archivePath ./build/xplorr.xcarchive

# Export IPA
xcodebuild -exportArchive \
  -archivePath ./build/xplorr.xcarchive \
  -exportPath ./build \
  -exportOptionsPlist ExportOptions.plist
```

### Backend API

**Local Development:**
```bash
mvn spring-boot:run
# API available at http://localhost:8080
```

**Production Options:**

**1. Docker Deployment:**
```dockerfile
# Dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/xplorr-api-0.0.1.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```bash
# Build and run
mvn clean package
docker build -t xplorr-api .
docker run -p 8080:8080 xplorr-api
```

**2. Cloud Platforms:**
- **Heroku** - Easy Java app deployment with PostgreSQL addon
- **Railway** - Modern platform with PostgreSQL support
- **AWS Elastic Beanstalk** - Managed Java application hosting
- **Google Cloud Run** - Container-based serverless
- **Azure App Service** - Managed Spring Boot hosting

**3. Traditional Server:**
```bash
# Build JAR
mvn clean package -DskipTests

# Run on server
java -jar target/xplorr-api-0.0.1.jar \
  --spring.profiles.active=prod

# Or use systemd service
sudo systemctl start xplorr-api
```

**Database Migration:**
```bash
# Use Flyway or Liquibase for schema versioning
# Or Spring Boot's built-in schema management

# Example Flyway migration
# src/main/resources/db/migration/V1__initial_schema.sql
```

### CI/CD

**GitHub Actions - iOS:**
```yaml
name: iOS Build
on: [push]
jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: |
          xcodebuild build -workspace xplorr.xcworkspace \
            -scheme xplorr -destination 'platform=iOS Simulator,name=iPhone 14'
```

**GitHub Actions - Backend:**
```yaml
name: Spring Boot Build
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Build with Maven
        run: mvn clean package
      - name: Run Tests
        run: mvn test
```

**Fastlane (Optional - iOS automation):**
```ruby
# fastlane/Fastfile
lane :beta do
  build_app(scheme: "xplorr")
  upload_to_testflight
end
```

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

