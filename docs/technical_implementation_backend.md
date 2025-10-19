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

### Cloud Infrastructure
- **AWS (Amazon Web Services)** - Exclusive cloud provider for all services
- **AWS S3** - Object storage for place images and media
- **AWS RDS** - Managed PostgreSQL database (production)
- **AWS Elastic Beanstalk** / **ECS** - Application hosting
- **AWS CloudFront** - CDN for fast image delivery

### Monitoring & Logging
- **Spring Boot Actuator** - Health checks and metrics
- **SLF4J** + **Logback** - Logging
- **Micrometer** - Application metrics

---

## 📝 Content Management Strategy

### MVP Approach (Manual Curation)

For the MVP phase, place data will be manually curated and inserted:

**Manual Data Entry:**
- Places will be manually added to PostgreSQL database
- Images and media uploaded directly to AWS S3 buckets
- No automated curation interface
- Admin access via database client (pgAdmin, DBeaver)

**Process:**
```bash
# Example manual insertion
INSERT INTO places (id, name, location, categories, rating, photos)
VALUES (
  'place-001',
  'The Daily Grind',
  ST_SetSRID(ST_MakePoint(-122.4194, 37.7749), 4326),
  ARRAY['cafe', 'coffee'],
  4.5,
  ARRAY['https://xplorr-images.s3.amazonaws.com/place-001/photo1.jpg']
);
```

**AWS S3 Structure:**
```
xplorr-images/
├── place-001/
│   ├── photo1.jpg
│   ├── photo2.jpg
│   └── thumbnail.jpg
├── place-002/
│   └── ...
```

### Post-MVP: Automated Curation Workflow

**Planned Features (Future):**
- Admin web dashboard for place management
- Bulk import from Google Places API
- Image upload and moderation interface
- Automated photo optimization and CDN distribution
- Place verification and approval workflow
- Community contributions and reviews

**Future Tech Stack:**
- Admin panel (React or Spring Boot + Thymeleaf)
- AWS Lambda for image processing
- AWS Step Functions for curation workflow
- AWS Rekognition for image content moderation

> **Note:** The current API is designed to be forward-compatible with automated curation. All endpoints will work seamlessly once the curation system is implemented.

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

### AWS Deployment (Primary)

> **Note:** xplorr exclusively uses AWS for all cloud infrastructure.

#### AWS Elastic Beanstalk

**Initial Setup:**
```bash
# Install EB CLI
pip install awsebcli

# Initialize Elastic Beanstalk
eb init xplorr-api --platform java-17 --region us-east-1

# Create environment
eb create xplorr-api-prod \
  --database.engine postgres \
  --database.size 5 \
  --instance-type t3.small

# Configure environment variables
eb setenv SPRING_PROFILES_ACTIVE=prod \
  AWS_S3_BUCKET=xplorr-images \
  AWS_REGION=us-east-1

# Deploy
eb deploy
```

#### AWS RDS PostgreSQL

**Setup:**
```bash
# RDS will be created automatically by EB or manually via console
# Enable PostGIS extension after creation:
psql -h your-rds-endpoint.amazonaws.com -U postgres -d xplorr
CREATE EXTENSION postgis;
```

**Configuration:**
```yaml
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://${RDS_HOSTNAME}:${RDS_PORT}/${RDS_DB_NAME}
    username: ${RDS_USERNAME}
    password: ${RDS_PASSWORD}
```

#### AWS S3 Configuration

**Create S3 Bucket:**
```bash
# Via AWS CLI
aws s3 mb s3://xplorr-images --region us-east-1

# Configure bucket for public read access (images only)
aws s3api put-bucket-cors --bucket xplorr-images --cors-configuration file://cors.json
```

**CORS Configuration (cors.json):**
```json
{
  "CORSRules": [{
    "AllowedOrigins": ["*"],
    "AllowedMethods": ["GET"],
    "AllowedHeaders": ["*"]
  }]
}
```

**Spring Boot S3 Integration:**
```java
// Add to pom.xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-s3</artifactId>
    <version>1.12.529</version>
</dependency>

// Service class
@Service
public class S3Service {
    private final AmazonS3 s3Client;
    
    @Value("${aws.s3.bucket}")
    private String bucketName;
    
    public String uploadFile(MultipartFile file, String placeId) {
        String key = placeId + "/" + file.getOriginalFilename();
        s3Client.putObject(bucketName, key, file.getInputStream(), null);
        return s3Client.getUrl(bucketName, key).toString();
    }
}
```

#### AWS CloudFront (CDN)

**Setup:**
- Create CloudFront distribution pointing to S3 bucket
- Configure caching policies for images (long TTL)
- Use CloudFront URLs in API responses for fast delivery

```yaml
# application-prod.yml
aws:
  s3:
    bucket: xplorr-images
    region: us-east-1
  cloudfront:
    domain: d1234567890.cloudfront.net
```

#### AWS ECS (Alternative to Elastic Beanstalk)

For more control, use ECS with Fargate:
```bash
# Create ECR repository
aws ecr create-repository --repository-name xplorr-api

# Build and push Docker image
docker build -t xplorr-api .
docker tag xplorr-api:latest <account-id>.dkr.ecr.us-east-1.amazonaws.com/xplorr-api:latest
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/xplorr-api:latest

# Create ECS cluster and service (via console or CLI)
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

## 🗄️ Manual Data Management (MVP)

### Database Setup

**Create Database Schema:**
```sql
-- Run after PostGIS extension is enabled
CREATE TABLE places (
    id VARCHAR(255) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    location GEOMETRY(Point, 4326) NOT NULL,
    categories TEXT[] NOT NULL,
    rating DECIMAL(2,1),
    price_level INTEGER,
    photos TEXT[],
    hours VARCHAR(500),
    estimated_visit_time INTEGER,
    cached_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create spatial index
CREATE INDEX idx_places_location ON places USING GIST(location);
```

**Manual Place Insertion:**
```sql
-- Example: Insert a coffee shop
INSERT INTO places (
    id, 
    name, 
    location, 
    categories, 
    rating, 
    price_level,
    photos, 
    hours, 
    estimated_visit_time
) VALUES (
    'daily-grind-001',
    'The Daily Grind',
    ST_SetSRID(ST_MakePoint(-122.4194, 37.7749), 4326),
    ARRAY['cafe', 'coffee', 'breakfast'],
    4.5,
    2,
    ARRAY[
        'https://d1234567890.cloudfront.net/daily-grind-001/exterior.jpg',
        'https://d1234567890.cloudfront.net/daily-grind-001/interior.jpg'
    ],
    'Mon-Fri: 7:00 AM - 8:00 PM, Sat-Sun: 8:00 AM - 9:00 PM',
    45
);

-- Verify insertion
SELECT 
    id,
    name,
    ST_AsText(location) as coordinates,
    categories,
    rating
FROM places
WHERE id = 'daily-grind-001';
```

### AWS S3 Image Upload

**Upload via AWS CLI:**
```bash
# Upload place images
aws s3 cp ./images/daily-grind-001/ \
    s3://xplorr-images/daily-grind-001/ \
    --recursive \
    --acl public-read

# Verify upload
aws s3 ls s3://xplorr-images/daily-grind-001/
```

**Upload via AWS Console:**
1. Navigate to S3 bucket: `xplorr-images`
2. Create folder with place ID: `daily-grind-001/`
3. Upload images (jpg, png, webp)
4. Set permissions to public-read for images

**Image Naming Convention:**
```
{place-id}/
  ├── thumbnail.jpg      (300x300, for markers)
  ├── hero.jpg          (1200x800, for detail view)
  ├── photo-01.jpg      (800x600, gallery)
  ├── photo-02.jpg
  └── photo-03.jpg
```

### Data Import Script (Optional)

```java
// Utility class for bulk import
@Component
public class PlaceDataImporter {
    
    @Autowired
    private PlaceRepository placeRepository;
    
    @Autowired
    private S3Service s3Service;
    
    public void importFromCSV(String filePath) throws IOException {
        // Read CSV file
        // Parse place data
        // Insert into database
        // Log results
    }
    
    public void importFromGooglePlaces(String placeId) {
        // Fetch from Google Places API
        // Transform to our model
        // Save to database
        // Download and upload images to S3
    }
}
```

---

## 📚 Additional Resources

### Internal Documentation
- **[Main Technical Implementation](./technical_implementation.md)** - Overview and architecture
- **[iOS Development Guidelines](./technical_implementation_ios.md)** - iOS/Swift/SwiftUI guide

### External Resources
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [PostGIS Documentation](https://postgis.net/documentation/)
- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [AWS RDS PostgreSQL](https://docs.aws.amazon.com/rds/index.html)

---

**Last Updated:** October 18, 2025
**Cloud Provider:** AWS (Exclusive)

