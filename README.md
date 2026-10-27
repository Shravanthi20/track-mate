# TrackMate 🚆
### A Scalable Railway Ticket Booking and Reservation System

TrackMate is a production-oriented railway ticket booking platform designed to provide reliable train discovery, real-time seat availability, secure online payments, and seamless reservation management.

Built using a microservices architecture, TrackMate explores distributed systems concepts such as event-driven communication, concurrency control, fault tolerance, and scalable container orchestration.

## Key Features

- **Train Search:** Search trains by source, destination, journey date, and travel class.
- **Seat Availability:** Check journey-specific seat availability and prevent double booking through transactional inventory management.
- **Reservation Management:** Create bookings, manage reservation states, generate PNRs, and handle cancellations.
- **Secure Payments:** Integrate Razorpay Standard Checkout in test mode with server-side signature verification and webhook processing.
- **Event-Driven Architecture:** Use Apache Kafka for asynchronous communication between services.
- **Caching and Coordination:** Use Redis for search caching, rate limiting, and temporary reservation coordination.
- **Authentication and Authorization:** Secure APIs and enforce role-based access control.
- **Notifications:** Send booking confirmations and payment updates asynchronously.
- **Containerization:** Package services using Docker for consistent development and deployment.
- **Orchestration:** Use Kubernetes for service discovery, health checks, deployment management, and horizontal scaling.
- **Observability:** Integrate structured logging, metrics, and distributed tracing.

## Architecture

TrackMate follows a microservices architecture in which each service owns a distinct business responsibility.

```text
                     React Web Application
                              |
                         API Gateway
                              |
        +---------------------+---------------------+
        |                     |                     |
   Auth Service         Train Service        Search Service
        |                     |                     |
   User Service         Inventory Service           |
                              |                     |
                       Booking Service <------------+
                              |
                       Payment Service
                              |
                     Razorpay Checkout
                              |
                       Payment Webhooks
                              |
                         Apache Kafka
                              |
                    Notification Service

```

### Microservices

| Service | Responsibility |
|---|---|
| API Gateway | Request routing, authentication enforcement, and rate limiting |
| Auth Service | Registration, login, tokens, and access control |
| User Service | Passenger profiles and saved traveller details |
| Train Service | Train information, routes, stations, and schedules |
| Search Service | Train discovery and cached search results |
| Inventory Service | Seat allocation, temporary holds, and availability |
| Booking Service | Reservation lifecycle, passenger details, and PNR generation |
| Payment Service | Razorpay orders, payment verification, refunds, and payment records |
| Notification Service | Booking confirmations and payment notifications |

## Technology Stack

| Component | Technology |
|---|---|
| Frontend | React, TypeScript, Vite, Tailwind CSS |
| Backend | Node.js, TypeScript, Express |
| Database | PostgreSQL |
| ORM | Prisma |
| Cache | Redis |
| Message Broker | Apache Kafka |
| Payment Gateway | Razorpay Standard Checkout |
| Validation | Zod |
| Containerization | Docker, Docker Compose |
| Orchestration | Kubernetes |
| Testing | Vitest, Supertest, Playwright |
| Observability | Pino, OpenTelemetry, Prometheus, Grafana |
| CI/CD | GitHub Actions |

## Booking Workflow

1. The passenger searches for trains and checks seat availability.
2. The Booking Service validates the request and creates a pending reservation.
3. The Inventory Service atomically holds the requested seats.
4. The Payment Service creates a Razorpay order using the calculated fare.
5. The passenger completes payment through Razorpay Checkout.
6. The backend verifies payment authenticity and processes authoritative payment updates.
7. The booking workflow confirms the reservation only when payment and inventory conditions are satisfied.
8. A unique PNR is generated for the confirmed booking.
9. Kafka events trigger asynchronous notifications and downstream processing.

### Booking Reliability

TrackMate is designed around the following consistency and reliability principles:

- Transactional seat allocation to prevent double booking.
- Idempotency keys for retry-safe booking requests.
- Server-side payment verification and webhook signature validation.
- Duplicate-event protection for Kafka consumers and payment webhooks.
- Transactional outbox patterns for reliable event publishing.
- Expiring seat holds and recovery of abandoned reservations.
- Reconciliation workflows for uncertain payment or booking outcomes.

PostgreSQL acts as the authoritative source for seat allocation and booking records. Redis caching does not replace transactional inventory checks.

## Project Structure

```text
track-mate/
├── apps/
│   └── web/
├── services/
│   ├── api-gateway/
│   ├── auth-service/
│   ├── user-service/
│   ├── train-service/
│   ├── inventory-service/
│   ├── booking-service/
│   ├── payment-service/
│   ├── train-service/
│   ├── search-service/
│   └── notification-service/
├── packages/
│   ├── contracts/
│   ├── logger/
│   ├── observability/
│   └── test-utils/
├── infrastructure/
│   ├── docker/
│   ├── compose/
│   └── kubernetes/
├── docs/
│   ├── architecture.md
│   ├── booking-lifecycle.md
│   ├── api-specification.md
│   └── adr/
├── .github/
│   └── workflows/
├── package.json
├── package-lock.json
├── tsconfig.base.json
├── .env.example
└── README.md
```

## Getting Started

### Prerequisites

Install the following tools before setting up the project:

- Node.js and npm
- Docker and Docker Compose
- Git
- A code editor
- A Razorpay account with test-mode API credentials for payment integration

Kubernetes tooling such as Minikube or kind can be installed later for local orchestration.

### Installation

Clone the repository:

```bash
git clone https://github.com/Shravanthi20/track-mate
cd track-mate
```

Install dependencies:

```bash
npm install
```

Configure the environment:

```bash
cp .env.example .env
```

Update the environment variables with your local database configuration and other required service settings.

Start the local infrastructure:

```bash
docker compose up -d
```

Apply database migrations:

```bash
npx prisma migrate deploy
```

Generate the Prisma Client if required by the service setup:

```bash
npx prisma generate
```

Start the development environment using the scripts defined in the root `package.json`.

> These commands describe the intended setup. The final scripts, migration paths, and service startup commands should match the implemented repository.

## Environment Configuration

Use `.env.example` to document the configuration required by the application.

```dotenv
NODE_ENV=development

DATABASE_URL=
REDIS_URL=
KAFKA_BROKERS=

JWT_SECRET=
JWT_ACCESS_TOKEN_TTL=

RAZORPAY_KEY_ID=
RAZORPAY_KEY_SECRET=
RAZORPAY_WEBHOOK_SECRET=

WEB_BASE_URL=
API_BASE_URL=
```

Keep real credentials in local environment files or a secret manager. Never commit production credentials, private keys, payment secrets, or real customer information.

Razorpay must remain in test mode during development. Payment verification and webhook endpoints must be implemented on the backend.

## API Overview

The initial API surface is planned as follows:

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/auth/register` | Register an account |
| POST | `/api/v1/auth/login` | Authenticate a user |
| GET | `/api/v1/trains/search` | Search available trains |
| GET | `/api/v1/trains/:trainId` | Retrieve train details |
| GET | `/api/v1/inventory/availability` | Check seat availability |
| POST | `/api/v1/bookings` | Create a reservation request |
| GET | `/api/v1/bookings/:bookingId` | Retrieve booking status |
| POST | `/api/v1/bookings/:bookingId/payment-order` | Create a payment order |
| POST | `/api/v1/payments/verify` | Verify payment details |
| POST | `/api/v1/payments/webhooks/razorpay` | Process payment webhooks |
| POST | `/api/v1/bookings/:bookingId/cancel` | Request booking cancellation |
| GET | `/api/v1/bookings/:bookingId/ticket` | Retrieve a confirmed ticket |

These endpoints describe the planned API contract and should be updated as implementation progresses.

## Testing

TrackMate is intended to support multiple levels of automated testing:

- **Unit tests:** Validate service logic, fare calculations, and state transitions.
- **Integration tests:** Verify database transactions, repositories, and external-service adapters.
- **Contract tests:** Validate API schemas and Kafka event contracts.
- **Concurrency tests:** Simulate simultaneous attempts to reserve the same seat.
- **Payment tests:** Exercise invalid signatures, duplicate webhooks, payment failures, and late payment outcomes.
- **End-to-end tests:** Verify the passenger journey from train search to ticket confirmation.
- **Load tests:** Evaluate API throughput, latency, and behavior under contention.

Run the appropriate test scripts defined in the repository's `package.json`.

## Deployment

### Docker

Each service is designed to have an independent Docker image. Docker Compose provides a convenient local environment for the application and its supporting infrastructure.

### Kubernetes

Kubernetes deployment configurations will support:

- Independent service deployments and internal service discovery.
- Readiness and liveness probes.
- Resource requests and limits.
- Horizontal scaling where appropriate.
- Rolling updates and controlled rollbacks.
- Secret management and network policies.
- Persistent storage and backup strategies for stateful infrastructure.

Production deployments require tested recovery procedures, appropriate monitoring, security controls, and durable database and message-broker configurations.

## Engineering Principles

TrackMate emphasizes:

- Clear service boundaries and data ownership.
- Transactional consistency for critical booking operations.
- Idempotent APIs and event consumers.
- Asynchronous communication through Kafka.
- Cache-aware system design with PostgreSQL as the source of truth.
- Secure payment processing.
- Observability and fault recovery.
- Automated testing and repeatable deployment workflows.

## Project Status

**Development status:** In progress.

The project is intended to evolve incrementally from a functional booking system into a distributed, containerized application with payment integration, event-driven workflows, and production-oriented reliability controls.

## Disclaimer

TrackMate is an independent educational and engineering project inspired by railway reservation platforms. It is not affiliated with or endorsed by Indian Railways or IRCTC. Railway schedules, fares, and reservations require appropriate data sources and integrations; the project does not imply access to official booking infrastructure.

## License

Choose and add an appropriate open-source license before distributing the project.

