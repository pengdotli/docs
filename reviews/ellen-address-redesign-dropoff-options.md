# Code Review: ellen/address-redesign-dropoff-options

## Overview
This branch contains multiple changes across various service protos, with significant updates to service configurations and API endpoints.

## Changes by Service

### User Management Service
- Removed HTTP and authorization configurations from several RPCs
- Added new RPC: `ListUserGroups`
- Added new RPC: `CreateUser`
- Simplified RPC definitions by removing redundant configurations
- Version changes:
  - Removed version 7.3.28
  - Reverted to version 7.3.27
- Removed permissions:
  - `PERMISSION_DEV_PORTAL_INVENTORY_VIEW`
  - `PERMISSION_DEV_PORTAL_INVENTORY_EDIT`

### Waitlist Service
- Removed `UpdateEntityTypeNotifyMeMode` endpoint
- Removed notification-related enums and fields:
  - Removed `EntityTypeNotifyMe` enum
  - Removed notification preference fields from `RunnerPreferences`
- Version changes:
  - Changed from 0.1.10 to 0.1.9

### Weather Service
- Modified timeout configuration:
  - Reduced `GetLatestAlerts` timeout from 5000ms to 2000ms
- Version changes:
  - Changed from 0.0.11 to 0.0.10

### Zero2One Service
- Cart Payment changes:
  - Removed optional fields: `payment_method_id` and `idempotency_key`
  - Removed `CancelCartPayment` messages and endpoints
- POS Service changes:
  - Updated endpoint lifecycle from DEV to PROD for several endpoints
  - Updated tier from T0 to T1 for multiple endpoints
  - Removed gift card loading functionality
- Removed KDS (Kitchen Display System) related files:
  - Deleted `kds_device.proto`
  - Removed KDS device references from `station.proto`
- Version changes:
  - Changed from 0.0.341 to 0.0.335

### Zesty Service
- Removed several features and endpoints:
  - Removed store reaction functionality
  - Removed query suggestions endpoint
  - Removed recommendation run details endpoint
- Simplified models:
  - Removed rating types and store reactions
  - Removed citation functionality
- Version changes:
  - Changed from 0.0.38 to 0.0.30

## Impact Analysis

### Breaking Changes
1. Removal of several endpoints and features:
   - User Management permissions
   - Waitlist notification preferences
   - KDS device functionality
   - Store reactions in Zesty
   - Gift card loading in Zero2One

2. Configuration Changes:
   - Multiple endpoints moved from DEV to PROD
   - Tier changes from T0 to T1
   - Timeout reductions

### Non-Breaking Changes
1. Code cleanup:
   - Removal of redundant HTTP configurations
   - Simplification of RPC definitions
   - Removal of deprecated fields

## Recommendations

### For Review
1. Verify that all endpoint lifecycle changes (DEV to PROD) are intentional
2. Confirm that timeout reduction in Weather Service is tested
3. Ensure dependent services are aware of removed functionalities
4. Check if any client applications rely on removed fields/endpoints

### For Testing
1. Test affected endpoints with new configurations
2. Verify backward compatibility where applicable
3. Test timeout changes under load
4. Validate removal of notification preferences doesn't affect core functionality

### For Deployment
1. Consider phased rollout for major changes
2. Plan for coordinated deployment with dependent services
3. Prepare rollback plan for critical changes
4. Update documentation for removed features

## Questions for Follow-up
1. Are there any migration plans for removed features?
2. Have all stakeholders been notified of the breaking changes?
3. Is there a timeline for deprecating the removed functionalities?
4. Are there any performance implications from the timeout changes?