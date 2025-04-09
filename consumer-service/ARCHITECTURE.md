# Consumer Profile Service Architecture

This document provides an overview of the key components in the `com.doordash.consumerservice` package and how they work together.

## Core Components

### Models

#### `DBConsumer`
- Core data model representing a consumer record in the database
- Contains essential consumer information like:
  - Consumer ID
  - User ID
  - Experience type (DoorDash, Caviar, etc.)
  - Tenant ID
  - Default address and country settings
  - VIP status and tier
  - Payment information (Stripe)
  - Contact information
  - Timestamps

#### `DecoratedConsumer`
- Enhanced consumer model that combines:
  - Base consumer information (`DBConsumer`)
  - Default address details (`DecoratedAddressLink`)
  - Scheduled delivery time
  - Terms of Service status
  - Blocked item types

#### `ConsumerTOS`
- Represents consumer's Terms of Service status
- Tracks:
  - Whether user accepted latest terms
  - Latest version of terms of service

### Services

#### `ConsumerService`
- Core service orchestrating business logic for consumer operations
- Key responsibilities:
  - Consumer CRUD operations
  - Address management
  - Terms of service handling
  - Scheduled delivery management
  - Consumer profile type management (guest, authenticated, etc.)
  - Integration with external systems (identity, geo, persona)

#### `ConsumerScheduledDeliveryTimeRepository`
- Manages scheduled delivery times for consumers
- Uses DashCache for caching delivery schedules
- Handles TTL and cache invalidation

### Repositories

#### `DatabaseRepository`
- Primary data access layer for consumer operations
- Handles:
  - Consumer CRUD operations
  - Address link management
  - Terms of service tracking
  - Consumer recycling
  - External consumer mappings

#### `ConsumerDatabaseRepository`
- Base repository implementation
- Manages cache interactions using DashCache
- Handles database migrations

### Cache Management

#### `DashCacheKeyType`
Enumerates different types of cache keys:
- `CONSUMER_ID`
- `CONSUMER_USER_ID`
- `GEO_ADDRESS_ID`
- `IMMUTABLE_USER_ID_AND_EXPERIENCE`
- And more

#### `DashCacheUseCase`
Defines different caching scenarios:
- `CONSUMER_CACHE`
- `IMMUTABLE_CONSUMER_CACHE`
- `CONSUMER_SCHEDULED_DELIVERY_TIME`
- `LITE_GUEST_CONSUMER`

### Event Handling

#### `ConsumerServiceTopic`
Kafka topics for consumer events:
- `CONSUMER_EVENTS`
- `CONSUMER_POST_SIGNUP_EVENTS`
- `CONSUMER_POST_SIGNUP_EVENTS_V2`

## Flow of Operations

1. **Consumer Creation**:
   - External request comes through gRPC controller
   - `ConsumerService` validates and processes request
   - `DatabaseRepository` creates consumer record
   - Cache is updated via `DashCache`
   - Events published to Kafka topics

2. **Profile Retrieval**:
   - Request handled by gRPC controller
   - `ConsumerService` checks cache first
   - If cache miss, `DatabaseRepository` fetches data
   - Data decorated with additional information (address, TOS, etc.)
   - Response returned to client

3. **Address Management**:
   - Address operations coordinated by `ConsumerService`
   - `DatabaseRepository` handles persistence
   - Cache updated accordingly
   - Events published for relevant changes

4. **Scheduled Delivery**:
   - Managed by `ConsumerScheduledDeliveryTimeRepository`
   - Uses caching for performance
   - Integrated with main consumer operations

## Error Handling

The service defines various custom exceptions:
- `ConsumerNotFoundException`
- `ConsumerAddressLinkNotFoundException`
- `ConsumerTOSNotFoundException`
- `ValidationException`
- And more

These exceptions are mapped to appropriate gRPC status codes for client communication.

## Integration Points

The service integrates with several other DoorDash systems:
- Identity service for user management
- Geo service for address validation
- Persona service for verification
- Risk service for consumer validation
- Payment systems (Stripe)
- Notification services (GCM/FCM)

## Configuration

Runtime configuration is managed through:
- `RuntimeConfig` for service-level settings
- `CopsDynamicValueConfig` for dynamic configuration
- Environment-specific settings

## Best Practices

1. **Caching Strategy**:
   - Use DashCache for performance
   - Implement appropriate TTLs
   - Handle cache invalidation properly

2. **Error Handling**:
   - Use appropriate custom exceptions
   - Proper gRPC status code mapping
   - Comprehensive error logging

3. **Event Publishing**:
   - Use appropriate Kafka topics
   - Ensure event ordering
   - Handle failed publications

4. **Data Consistency**:
   - Use transactions where necessary
   - Maintain cache consistency
   - Handle race conditions

## Future Considerations

1. Cache optimization opportunities
2. Event handling improvements
3. Additional consumer profile types
4. Enhanced integration points