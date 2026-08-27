# API Gateway

Edge server and API gateway for the EventSphere microservices platform, built with **Spring Cloud Gateway** and **Spring WebFlux**.

## Overview

The API Gateway serves as the single entry point for all client requests. It routes incoming HTTP traffic to the appropriate downstream microservices, handles cross-origin resource sharing (CORS), and provides resilience patterns like retries and circuit breaking.

## Tech Stack

- **Framework:** Spring Boot 4.1.0
- **API Gateway:** Spring Cloud Gateway (WebFlux)
- **Load Balancing:** Spring Cloud LoadBalancer
- **Service Discovery:** Netflix Eureka Client
- **Configuration:** Spring Cloud Config Client
- **Monitoring:** Spring Boot Actuator
- **Java:** 25

## Port

| Property | Value |
|----------|-------|
| Server Port | 8080 |

## Routing Rules

| Route ID | Path Pattern | Target Service | Filter |
|----------|-------------|----------------|--------|
| `user-service` | `/api/users/**` | `lb://user-service` | StripPrefix=1 |
| `event-booking-service` | `/api/events/**`, `/api/ticket-types/**`, `/api/bookings/**` | `lb://event-booking-service` | StripPrefix=1 |
| `review-notification-service` | `/api/reviews/**` | `lb://review-notification-service` | StripPrefix=1 |
| `review-notification-service-multipart` | `POST /api/reviews` | `lb://review-notification-service` | StripPrefix=1 |

## Resilience

Global retry filter is applied to all routes:

- **Retries:** 3
- **Retry Statuses:** `BAD_GATEWAY`, `SERVICE_UNAVAILABLE`, `GATEWAY_TIMEOUT`
- **Methods:** `GET`, `POST`
- **Backoff Strategy:** Exponential backoff
  - Initial backoff: 100ms
  - Max backoff: 500ms
  - Multiplier: 2

## CORS Configuration

Allowed origins:
- `http://localhost:3000` (local development)
- `http://136.68.42.194` (frontend IP)
- `https://eventsphere-frontend-149096254626.asia-south1.run.app` (Cloud Run production)

Allowed methods:
- `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `OPTIONS`

Exposed headers:
- `Content-Type`
- `Authorization`

## Architecture

```
Client (React Frontend)
    ↓ HTTPS / HTTP
API Gateway (Port 8080)
    ├── /api/users/**         → User Service (lb://user-service)
    ├── /api/events/**        → Event Booking Service (lb://event-booking-service)
    ├── /api/reviews/**       → Review & Notification Service (lb://review-notification-service)
    └── /api/bookings/**      → Event Booking Service (lb://event-booking-service)
```

## Running Locally

### Prerequisites

- Java 25
- Maven 3.9+
- Eureka Server running on port 8761
- Config Server running on port 8888

### Steps

```bash
# Build the service
mvn clean package

# Run the service
java -jar target/api-gateway-0.0.1-SNAPSHOT.jar
```

## Actuator Endpoints

- `GET /actuator/health` - Health check
- `GET /actuator/info` - Application info
- `GET /actuator/gateway` - Gateway routes and filters
- `GET /actuator/metrics` - Application metrics
- `POST /actuator/refresh` - Refresh configuration

## Logging

- `org.springframework.cloud.gateway`: DEBUG
- `org.springframework.cloud.loadbalancer`: DEBUG
- `com.netflix.discovery`: INFO

## CORS Configuration

CORS is configured via `CorsConfig.java` to allow requests from the React frontend running on localhost and the deployed Cloud Run instance.
