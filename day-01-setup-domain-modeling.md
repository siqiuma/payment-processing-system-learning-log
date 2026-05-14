Day 1 -- Project Foundation & Core Domain Modeling
=================================================

What I worked on
----------------

Today I initialized the **Payment Processing System** project and focused on establishing a solid foundation before implementing any business logic.

### 1\. Project Setup

Created a new Spring Boot backend project using:

-   Java 21
-   Spring Boot
-   Spring Data JPA
-   PostgreSQL
-   Spring Kafka

Established the initial package structure to separate:

-   domain models
-   repositories
-   future service logic
-   gateway integration
-   asynchronous processing

The goal is to keep business correctness concerns (payments) clearly separated from infrastructure concerns (Kafka, outbox, consumers).

* * * * *

### 2\. Infrastructure Setup (Docker)

Configured local infrastructure using Docker Compose:

-   PostgreSQL
-   Kafka
-   Zookeeper

Used custom ports to avoid conflicts with the previously built AI anomaly detection project:

| Service | Port |
| --- | --- |
| PostgreSQL | 5433 |
| Kafka | 9093 |
| Zookeeper | 2182 |

Learned and clarified Docker port mapping format:

```
HOST_PORT:CONTAINER_PORT
```

Example:

```
5433:5432
```

Meaning:

-   `5433` → accessed from local machine
-   `5432` → PostgreSQL default port inside container

* * * * *

### 3\. Kafka Configuration

Configured Kafka bootstrap settings in `application.yml`.

Learned the purpose of:

```
KAFKA_ADVERTISED_LISTENERS
```

Specifically:

```
PLAINTEXT://localhost:9093
```

This allows Kafka to advertise a broker address that is reachable from the Spring Boot application running outside Docker.

Key insight:

Kafka clients first connect to retrieve metadata, then reconnect using the broker address returned by Kafka. If the advertised address is not reachable, the client connection fails.

* * * * *

### 4\. Core Domain Modeling

Designed and implemented the core entities that will support the payment lifecycle.

#### Payment Entity

Represents payment state and correctness.

Key fields include:

-   `paymentId`
-   `orderId`
-   `amount`
-   `currency`
-   `paymentMethod`
-   `status`
-   `gatewayReference`
-   `failureReason`
-   optimistic locking version

Implemented:

```
@Version
```

for optimistic locking to prevent concurrent update conflicts.

* * * * *

#### IdempotencyKey Entity

Designed to prevent duplicate payment processing caused by retries.

Stores:

-   idempotency key
-   associated payment
-   request hash

This will later support:

> safe retries without duplicate charges

* * * * *

#### OutboxEvent Entity

Created the foundation for the **Outbox Pattern**.

Stores:

-   aggregate type
-   event type
-   payload
-   publication status

This will later enable reliable asynchronous event publishing for payment capture.

* * * * *

### 5\. Enum Modeling

Created domain enums to make state transitions explicit:

#### PaymentStatus

```
INITIATEDAUTHORIZINGAUTHORIZEDCAPTUREDDECLINED
```

#### OutboxStatus

```
PENDINGPUBLISHEDFAILED
```

#### PaymentMethod

```
CARDBANK_TRANSFERWALLET
```

* * * * *

### 6\. Repository Layer

Implemented repositories for:

-   `Payment`
-   `IdempotencyKey`
-   `OutboxEvent`

Used:

```
Optional<T>
```

for query results that may or may not exist.

Example:

```
Optional<IdempotencyKey>
```

instead of returning `null`, making missing values explicit and reducing risk of `NullPointerException`.

* * * * *

### 7\. Database Design Learning

Learned why real systems commonly separate:

### Internal database key

```
Long id
```

from:

### Business identifier

```
paymentId
```

Reasons include:

-   faster indexing and joins
-   security (avoid enumeration attacks)
-   cleaner API exposure
-   better logging and debugging

This same pattern was applied to:

```
IdempotencyKey
```

* * * * *

### 8\. Repository Hygiene

Standardized `.gitignore` for:

-   Maven build output
-   IntelliJ files
-   logs
-   OS-generated files
-   future local secrets

Ensured only meaningful project artifacts are tracked by Git.

* * * * *

What I learned
--------------

### 1\. Kafka networking is non-trivial

Understanding:

```
advertised.listeners
```

was important for correctly connecting applications running outside Docker.

* * * * *

### 2\. Surrogate key vs business key

A system can intentionally maintain:

```
Long id
```

for database efficiency while exposing:

```
paymentId
```

externally for business operations.

* * * * *

### 3\. Optional improves null safety

`Optional<T>` forces developers to intentionally handle missing values instead of relying on `null`.

* * * * *

### 4\. Project boundaries matter

Unlike the DTP project (infrastructure reliability), this project is intentionally centered around:

> **business correctness under failure**

including:

-   idempotency
-   state transitions
-   payment guarantees

* * * * *

Decisions made
--------------

### Keep tech stack consistent

Chose:

```
Java + Spring Boot
```

instead of switching languages for variety.

Reason:

For backend-focused internship roles, depth in a production-grade stack provides stronger signal than artificial tech-stack diversity.

* * * * *

### Use Mock Payment Gateway

Decided **not** to integrate Stripe or a real payment provider.

Reason:

The interview value comes from learning:

-   correctness
-   retries
-   idempotency
-   external dependency handling

---not API plumbing.

* * * * *

### Keep only `application.yml`

Decided to avoid introducing:

```
application-local.yml
```

for now to avoid premature complexity.

* * * * *

Problems encountered
--------------------

No major blockers today.

Main focus was understanding foundational infrastructure concepts rather than debugging.
