# ETA Fulfillment Estimates Service Architecture

## Overview
The ETA Fulfillment Estimates Service is a Kotlin-based gRPC service that provides delivery time estimates for DoorDash's delivery platform. It's built using the Asgard framework and follows a clean architecture pattern with distinct layers.

## Service Architecture

### High-Level Components

```
eta-fulfillment-estimates-service/
├── applicationlayer/
│   ├── controllers/      # gRPC endpoint handlers
│   ├── services/        # Business logic orchestration
│   ├── orchestrators/   # Complex workflow coordination
│   ├── protocolconversion/ # Data model conversion
│   └── iguazu/         # Event processing
├── domainlayer/
│   └── dxeta/         # Core domain models and logic
└── util/              # Shared utilities
```

### Key Components

1. **Controllers Layer**
   - `DxEtaController`: Main gRPC endpoint handler
   - Handles request validation and error mapping
   - Converts gRPC-specific errors to appropriate status codes

2. **Services Layer**
   - `DxEtaService`: Core business logic orchestration
   - Manages request lifecycle
   - Handles timeout management
   - Coordinates between different components
   - Manages observability and metrics

3. **Protocol Conversion**
   - Converts between external (gRPC) and internal data models
   - Handles request/response validation
   - Maintains clean separation between external and internal representations

4. **Domain Layer**
   - Contains core business logic and domain models
   - Implements delivery time estimation algorithms
   - Pure business logic, independent of external protocols

### Technology Stack

1. **Framework & Runtime**
   - Asgard Server Framework
   - Kotlin Coroutines for async operations
   - gRPC for API communication
   - Guice for dependency injection

2. **Monitoring & Observability**
   - Custom metrics using FEAPlatformCounterKit
   - Iguazu for event processing
   - Comprehensive logging with AsgardLogger

3. **Configuration**
   - Runtime configuration management
   - Environment-specific settings
   - Feature flags support

### Request Flow

1. gRPC request received by `DxEtaController`
2. Request converted to internal model by `RequestConverter`
3. `DxEtaService` orchestrates the processing:
   - Validates request
   - Applies business logic
   - Handles timeouts
   - Manages observability
4. Response converted back to gRPC format
5. Metrics and events published

### Error Handling

- Structured error hierarchy with `DxEtaException` as base
- Specific exception types for different failure scenarios:
  - `DxEtaRequestValidationException`
  - `DxEtaRequestExtractionException`
  - `DxEtaOrchestrationException`
  - `DxEtaRequestTimeoutException`
- Automatic mapping to appropriate gRPC status codes

### Observability

1. **Metrics**
   - Request latency measurements
   - Success/failure counters
   - Custom business metrics

2. **Logging**
   - Structured logging with correlation IDs
   - Error context preservation
   - Performance monitoring

3. **Event Processing**
   - Iguazu events for system integration
   - Async event publishing
   - Request/response tracking

### Configuration Management

- Runtime configuration with caching
- Environment-specific settings
- Timeout configurations
- Feature flag support

## Best Practices

1. **Clean Architecture**
   - Clear separation of concerns
   - Domain-driven design principles
   - Dependency injection
   - Interface-based design

2. **Error Handling**
   - Structured exception hierarchy
   - Proper error propagation
   - Detailed error context
   - Appropriate status code mapping

3. **Performance**
   - Async operations with coroutines
   - Timeout management
   - Resource cleanup
   - Efficient data conversion

4. **Maintainability**
   - Clear code organization
   - Comprehensive documentation
   - Consistent naming conventions
   - Modular design

## Deployment

The service is containerized and can be deployed using:
- Docker
- Kubernetes
- Standard deployment scripts (run-job.sh, healthcheck.sh)

## Development Setup

Required tools and dependencies:
- Kotlin
- Gradle
- Docker
- Development environment configurations in devbox/

## Future Considerations

1. **Scalability**
   - Horizontal scaling capabilities
   - Load balancing considerations
   - Cache implementation options

2. **Monitoring**
   - Enhanced metrics collection
   - Advanced tracing integration
   - Performance optimization opportunities

3. **Integration**
   - New client integration patterns
   - Additional protocol support
   - Enhanced event processing 