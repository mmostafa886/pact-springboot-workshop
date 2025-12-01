# Repository Information

## Overview
This is a clean Pact Contract Testing Workshop repository containing a Spring Boot microservices example demonstrating consumer-driven contract testing.

## Repository Location
`/Users/mohamedsolaiman/Desktop/Storage/Professional/API/ContractTesting/Pact/pact-springboot-workshop-clean/`

## Repository Structure

```
pact-springboot-workshop-clean/
├── consumer/                      # Consumer application (ProductCatalogue)
│   ├── src/main/java/            # Application source code
│   ├── src/test/java/            # Tests including Pact consumer tests
│   ├── pom.xml                   # Maven dependencies
│   └── mvnw                      # Maven wrapper
│
├── provider/                      # Provider application (ProductService)
│   ├── src/main/java/            # Application source code
│   ├── src/test/java/            # Pact provider verification tests
│   ├── pom.xml                   # Maven dependencies
│   └── mvnw                      # Maven wrapper
│
├── diagrams/                      # Visual documentation
├── docker-compose.yaml            # Pact Broker setup
├── README.md                      # Workshop guide (step-by-step)
├── PROJECT_EXPLANATION.md         # Detailed technical explanation
├── LEARNING.md                    # Learning objectives
├── LICENSE                        # MIT License
├── .gitignore                     # Git ignore rules
└── .java-version                  # Java version specification

```

## What's Included

### Consumer (ProductCatalogue)
- Spring Boot web application
- REST client for ProductService
- Pact consumer tests
- WireMock unit tests
- Thymeleaf templates for UI

### Provider (ProductService)
- Spring Boot REST API
- JPA with H2 database
- Bearer token authentication
- Pact provider verification tests
- Provider state handlers

### Infrastructure
- Docker Compose for Pact Broker
- PostgreSQL database for broker
- Complete workshop documentation

## Getting Started

### Prerequisites
- JDK 17+
- Maven 3+
- Docker (for Pact Broker)

### Running the Applications

**Consumer:**
```bash
cd consumer
./mvnw spring-boot:run
# Access at http://localhost:8080
```

**Provider:**
```bash
cd provider
./mvnw spring-boot:run
# API available at http://localhost:9000
```

**Pact Broker:**
```bash
docker-compose up
# Access at http://localhost:9292
# Credentials: pact_workshop / pact_workshop
```

### Running Tests

**Consumer Pact Tests:**
```bash
cd consumer
./mvnw test
# Generates pact files in target/pacts/
```

**Provider Verification:**
```bash
cd provider
./mvnw test
# Verifies pacts against running provider
```

## Documentation Files

- **README.md**: Complete step-by-step workshop (11 steps)
- **PROJECT_EXPLANATION.md**: Deep dive into Spring Boot, annotations, and Pact concepts
- **LEARNING.md**: Workshop facilitator guide

## Key Features Demonstrated

1. **Consumer-Driven Contracts**: Consumer defines API expectations
2. **Pact Consumer Tests**: Mock server testing with flexible matching
3. **Provider Verification**: Real API validation against contracts
4. **Provider States**: Database setup for different test scenarios
5. **Authentication**: Bearer token implementation and testing
6. **Request Filtering**: Dynamic header injection in tests
7. **Pact Broker Integration**: Contract publishing and versioning
8. **Can-I-Deploy**: Safe deployment checks

## Technologies Used

- **Spring Boot 2.x**: Framework for both applications
- **Spring MVC**: REST controllers
- **Spring Security**: Authentication filter
- **Spring Data JPA**: Database access
- **Pact JVM 4.x**: Contract testing framework
- **JUnit 5**: Testing framework
- **WireMock**: HTTP mocking for unit tests
- **H2**: In-memory database
- **Thymeleaf**: Template engine
- **Maven**: Build tool
- **Docker Compose**: Container orchestration

## Git Information

- **Branch**: main
- **Initial Commit**: Includes all source code and documentation
- **Excluded**: Build artifacts (target/), IDE files (.idea/), generated pacts

## Next Steps

1. Follow the README.md workshop steps sequentially
2. Read PROJECT_EXPLANATION.md for detailed concepts
3. Experiment with adding new endpoints
4. Try breaking contracts to see failures
5. Publish contracts to the Pact Broker

## Notes

- The `.gitignore` excludes build artifacts to keep repository clean
- Both consumer and provider have Maven wrappers for consistency
- Java version is specified in `.java-version` (JDK 17+)
- All tests are configured for the step11 branch features

---

Created: December 1, 2025
🤖 Generated with Claude Code
