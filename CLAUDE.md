# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Build the project
./gradlew build

# Run the application (starts on port 8082)
./gradlew bootRun

# Run all tests
./gradlew test

# Run a single test class
./gradlew test --tests "com.mtb.springboot.service.DepartmentServiceTest"

# Run a single test method
./gradlew test --tests "com.mtb.springboot.controller.DepartmentControllerTest.saveDepartment"

# Build without running tests
./gradlew build -x test
```

## Architecture

Standard Spring Boot layered architecture:

```
Controller → Service (interface + impl) → Repository → H2 (test) / MySQL (runtime)
```

**Package:** `com.mtb.springboot`

- `controller/` — REST controllers (`@RestController`). Uses `@Valid` for request body validation.
- `service/` — `DepartmentService` interface + `DepartmentServiceImpl`. New services should follow this pattern.
- `repository/` — Spring Data JPA repositories extending `JpaRepository`.
- `entity/` — JPA entities using Lombok (`@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`).
- `error/` — `DepartmentNotFoundException` + `RestResponseEntityExceptionHandler` (`@ControllerAdvice`) returning `ErrorMessage` with HTTP status.
- `config/` — `FeatureEndpoint`: custom Spring Boot Actuator endpoint at `/actuator/features`.

## Configuration & Profiles

`application.yml` defines three Spring profiles — `dev`, `qa`, `prod` — all pointing to MySQL. Default active profile is `qa`.

Tests use `src/test/resources/application.properties` which configures an in-memory H2 database (`create-drop`), overriding the MySQL config automatically.

Actuator exposes all endpoints except `env` and `beans` (configured in the `qa` profile block).

## Test Strategies

Three distinct test slices are used:

| Layer | Annotation | DB |
|---|---|---|
| Controller | `@WebMvcTest` + `@MockBean DepartmentService` | None |
| Service | `@SpringBootTest` + `@MockBean DepartmentRepository` | H2 |
| Repository | `@DataJpaTest` + `TestEntityManager` | H2 |
