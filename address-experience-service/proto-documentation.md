# Address Experience Service Proto Documentation

## Overview
The Address Experience Service provides functionality for managing consumer address-related operations in the DoorDash ecosystem. This service handles address book management, address validation, and manual address entry workflows.

## Service Definition
```protobuf
service AddressExperienceService
```

### Service Configuration
- **Service Name**: address-experience-service
- **Service App**: web
- **Port**: 50051
- **Owner**: geo-intelligence team
  - Slack: #ask-geo-intel
  - Email: geo@doordash.com
- **Target Products**: CX, MX, DX

## Key RPCs

### 1. GetConsumerAddressBook
Loads and decorates consumer address book data, including:
- Sorting consumer saved addresses
- Retrieving nearby addresses via Google integration
- Response timeout: 500ms
- Max retry attempts: 2

### 2. GetAddressDetailsBySourcePlaceId
Retrieves address details using a source place ID.
- Response timeout: 500ms
- Max retry attempts: 2

### 3. FindManualAddressEntryCandidate
Handles manual address entry validation and suggestions.
- HTTP: GET /addresses/v1/manual-entry
- Response timeout: 2000ms
- Authorization: CX_USER, CX_GUEST, CX_LITE_GUEST, DX_USER
- Tier: T0

### 4. SaveManualAddressEntryCandidate
Saves manually entered address information.
- HTTP: POST /addresses/v1/manual-entry
- Response timeout: 1000ms
- Authorization: CX_USER, CX_GUEST, CX_LITE_GUEST, DX_USER
- Tier: T0

## Key Message Types

### FindManualAddressEntryCandidateRequest
Required fields:
- street_address
- locality
- admin_level_1
- postal_code
- country_code
- supported_map_vendors

### FindManualAddressEntryCandidateResponse
Contains:
- candidate_token
- lat/lng coordinates
- Formatted address components
- Map vendor information
- Detailed address components (locality, neighborhood, etc.)

### SaveManualAddressEntryCandidateResponse
Contains:
- geo_address_id
- Formatted address components
- Location information
- Dropoff preferences
- Consumer overrides
- Building details

### GetConsumerAddressBookResponse
Returns:
- Labeled address entities
- Nearby address suggestions
- Saved consumer addresses

## Circuit Breaker Configuration
Most RPCs include circuit breaker configuration:
- Failure rate threshold: 20%
- Counter sliding window: 10000ms

## Notes
- The service integrates with multiple map vendors (Google, Mapbox)
- Supports address validation and geocoding
- Handles both structured and unstructured address inputs
- Provides formatted address outputs in multiple formats
- Supports address customization and consumer preferences

## Dependencies
The service imports from several proto packages:
- common/personal_address_label
- common/service_client_config
- doordash.api/annotations
- geo-intelligence/address
- geo-intelligence/doordash_place
- google/api/annotations
- google/protobuf/wrappers