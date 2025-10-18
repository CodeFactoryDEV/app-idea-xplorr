# Backend Development Guidelines - xplorr API

This document provides backend-specific development guidelines using Spring Boot, Maven, and PostgreSQL for building the xplorr REST API.

> **Related Documentation:**
> - **[← Back to Main Technical Overview](./technical_implementation.md)**
> - **[iOS Development Guide](./technical_implementation_ios.md)** - For client implementation details

---

## 🛠️ Tech Stack

### Framework & Language
- **Spring Boot 3.x** - Java web framework
- **Maven** - Dependency management and build tool
- **Java 17+** - LTS version

### API Layer
- **Spring Web** - RESTful API endpoints
- **Spring Security** - Authentication and authorization
- **JWT** - Token-based authentication
- **Spring Validation** - Request validation

### Database
- **PostgreSQL** - Primary database
- **PostGIS** extension - Geospatial queries for nearby places
- **Spring Data JPA** - ORM and database access
- **HikariCP** - Connection pooling

### Caching
- **Spring Cache** - Method-level caching
- **Caffeine** - In-memory cache (for MVP, can migrate to Redis later)

### Third-party APIs
- **RestTemplate** / **WebClient** - HTTP client for external APIs
- Google Places API integration
- Foursquare API (fallback)

### Monitoring & Logging
- **Spring Boot Actuator** - Health checks and metrics
- **SLF4J** + **Logback** - Logging
- **Micrometer** - Application metrics

---

## 📐 Project Structure

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

---

## 📊 Data Models

### Java Entities

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

## 🚀 API Endpoints

### Places API

```java
@RestController
@RequestMapping("/api/places")
public class PlaceController {
    
    @GetMapping("/nearby")
    public ResponseEntity<List<PlaceDTO>> getNearbyPlaces(
        @RequestParam Double latitude,
        @RequestParam Double longitude,
        @RequestParam(defaultValue = "5.0") Double radius
    ) {
        // Return places within radius
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<PlaceDTO> getPlaceDetails(@PathVariable String id) {
        // Return detailed place information
    }
    
    @PostMapping("/random")
    public ResponseEntity<PlaceDTO> getRandomPlace(
        @RequestBody RandomizeRequest request
    ) {
        // Return randomly selected place
    }
}
```

### Itinerary API

```java
@RestController
@RequestMapping("/api/itineraries")
public class ItineraryController {
    
    @PostMapping
    public ResponseEntity<ItineraryDTO> createItinerary(
        @RequestBody CreateItineraryRequest request
    ) {
        // Create new itinerary
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ItineraryDTO> getItinerary(@PathVariable UUID id) {
        // Get itinerary by ID
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<ItineraryDTO> updateItinerary(
        @PathVariable UUID id,
        @RequestBody UpdateItineraryRequest request
    ) {
        // Update existing itinerary
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteItinerary(@PathVariable UUID id) {
        // Delete itinerary
    }
}
```

### User API

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping("/register")
    public ResponseEntity<UserDTO> register(@RequestBody RegisterRequest request) {
        // Register new user
    }
    
    @GetMapping("/me")
    public ResponseEntity<UserDTO> getCurrentUser(@AuthenticationPrincipal UserDetails user) {
        // Get current user profile
    }
    
    @PutMapping("/preferences")
    public ResponseEntity<UserDTO> updatePreferences(
        @RequestBody UserPreferences preferences
    ) {
        // Update user preferences
    }
    
    @PostMapping("/visit-history/{placeId}")
    public ResponseEntity<Void> addToVisitHistory(@PathVariable String placeId) {
        // Add place to visit history
    }
    
    @PostMapping("/no-go/{placeId}")
    public ResponseEntity<Void> addToNoGoList(@PathVariable String placeId) {
        // Add place to no-go list
    }
}
```

---

## 🔧 Configuration

### Maven Dependencies (pom.xml)

```xml
<dependencies>
    <!-- Spring Boot Starters -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    
    <!-- Database -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-spatial</artifactId>
    </dependency>
    
    <!-- Caching -->
    <dependency>
        <groupId>com.github.ben-manes.caffeine</groupId>
        <artifactId>caffeine</artifactId>
    </dependency>
    
    <!-- JWT -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    
    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Application Configuration (application-dev.yml)

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
    show-sql: true

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

logging:
  level:
    com.xplorr: DEBUG
    org.springframework.web: INFO
```

---

## 🚀 Development Setup

### Prerequisites

```bash
- Java 17 or higher (LTS)
- Maven 3.8+
- PostgreSQL 14+
- PostGIS extension
- IntelliJ IDEA or VS Code (optional)
```

### Setup Steps

```bash
# Clone repository
git clone https://github.com/yourusername/xplorr.git
cd xplorr/xplorr-api

# Set up PostgreSQL
createdb xplorr
psql -d xplorr -c "CREATE EXTENSION postgis;"

# Configure application properties
cp src/main/resources/application-example.yml src/main/resources/application-dev.yml
# Edit application-dev.yml with your credentials

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

---

## 🧪 Testing

### Unit Tests (JUnit)

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
    
    @Test
    void testExcludesVisitedPlaces() {
        List<Place> places = createMockPlaces();
        List<String> visitHistory = Arrays.asList("place1", "place2");
        
        Place selected = randomizerService.selectRandom(places, visitHistory);
        assertFalse(visitHistory.contains(selected.getId()));
    }
}
```

### Integration Tests

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
    
    @Test
    void testGetPlaceDetails() throws Exception {
        mockMvc.perform(get("/api/places/test-place-id"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value("test-place-id"));
    }
}
```

### Run Tests

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

---

## 📦 Deployment

### Docker Deployment

**Dockerfile:**
```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/xplorr-api-0.0.1.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Build and Run:**
```bash
# Build JAR
mvn clean package -DskipTests

# Build Docker image
docker build -t xplorr-api .

# Run container
docker run -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/xplorr \
  xplorr-api
```

### Docker Compose

```yaml
version: '3.8'
services:
  db:
    image: postgis/postgis:14-3.2
    environment:
      POSTGRES_DB: xplorr
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/xplorr
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: password
    depends_on:
      - db

volumes:
  postgres_data:
```

### Cloud Platforms

**Heroku:**
```bash
heroku create xplorr-api
heroku addons:create heroku-postgresql:hobby-dev
heroku config:set SPRING_PROFILES_ACTIVE=prod
git push heroku main
```

**Railway:**
- Connect GitHub repository
- Add PostgreSQL service
- Configure environment variables
- Deploy automatically on push

**AWS Elastic Beanstalk:**
```bash
eb init xplorr-api
eb create xplorr-api-env
eb deploy
```

---

## 🔒 Security Best Practices

### JWT Implementation

```java
@Component
public class JwtTokenProvider {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private long expiration;
    
    public String generateToken(Authentication authentication) {
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + expiration);
        
        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(Keys.hmacShaKeyFor(secret.getBytes()))
            .compact();
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(Keys.hmacShaKeyFor(secret.getBytes()))
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
}
```

### Input Validation

```java
@PostMapping("/places/random")
public ResponseEntity<PlaceDTO> getRandomPlace(
    @Valid @RequestBody RandomizeRequest request
) {
    // Request is automatically validated
}

public class RandomizeRequest {
    @NotNull
    @DecimalMin("-90.0")
    @DecimalMax("90.0")
    private Double latitude;
    
    @NotNull
    @DecimalMin("-180.0")
    @DecimalMax("180.0")
    private Double longitude;
    
    @Min(0)
    @Max(50)
    private Double radius;
}
```

---

## 📊 Monitoring & Logging

### Actuator Endpoints

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
```

Access at:
- Health: `http://localhost:8080/actuator/health`
- Metrics: `http://localhost:8080/actuator/metrics`

### Logging Configuration

```yaml
logging:
  level:
    root: INFO
    com.xplorr: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
  file:
    name: logs/xplorr-api.log
```

---

## 🎯 Performance Optimization

### Caching Strategy

```java
@Service
public class PlaceService {
    
    @Cacheable(value = "places", key = "#id")
    public Place getPlaceById(String id) {
        // Cached for 1 hour
        return placeRepository.findById(id).orElseThrow();
    }
    
    @CacheEvict(value = "places", key = "#id")
    public void invalidatePlace(String id) {
        // Manually invalidate cache
    }
}
```

### Database Indexing

```sql
-- Create spatial index for location queries
CREATE INDEX idx_places_location ON places USING GIST(location);

-- Create index on frequently queried fields
CREATE INDEX idx_places_rating ON places(rating);
CREATE INDEX idx_places_category ON places USING GIN(categories);
```

---

## 📚 Additional Resources

- **[Main Technical Implementation](./technical_implementation.md)** - Overview and architecture
- **[iOS Development Guidelines](./technical_implementation_ios.md)** - iOS/Swift/SwiftUI guide
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [PostGIS Documentation](https://postgis.net/documentation/)

---

**Last Updated:** October 18, 2025

