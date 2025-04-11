# Offline Delivery Verification via On-Device LLM
*A Proposal for DoorDash*

## Executive Summary

This document outlines a solution to implement a lightweight, on-device Large Language Model (LLM) that enables Dashers to complete the delivery verification process even when experiencing connectivity issues. The system will process delivery confirmation steps locally on the Dasher's device, ensuring seamless operations in areas with poor network coverage.

## Problem Statement

Currently, Dashers require an active internet connection to:
1. Verify they've reached the correct delivery location
2. Process delivery completion steps
3. Upload confirmation photos
4. Mark orders as delivered in the system

When connectivity fails, this creates delays, customer frustration, and accounting issues for Dashers who cannot officially complete their deliveries.

## Technical Solution

### On-Device LLM Architecture

We propose implementing a specialized 4-bit quantized transformer model (300-500MB) trained specifically on delivery verification tasks. This model would:

- Load automatically when the Dasher begins the delivery journey
- Operate completely locally when internet connectivity is lost
- Sync results when connectivity is restored

### Example Scenarios

#### Scenario 1: Apartment Complex Delivery
```
INPUT TO LOCAL LLM:
- Target GPS: 37.7858, -122.4064
- Current GPS: 37.7859, -122.4063
- Delivery Photo: [image data]
- Delivery Instructions: "Leave at door of Apt 302, building has blue door"
- Time: 7:32 PM (within delivery window of 7:15-7:45 PM)

LOCAL LLM ANALYSIS:
"The current location is within 15 meters of the target destination. 
The delivery photo shows a package placed in front of a blue door with 
visible apartment number '302'. Time of delivery is within expected window. 
Delivery instructions have been followed correctly."

RESULT: Verification approved with 97% confidence
```

#### Scenario 2: Business Delivery with Partial Mismatch
```
INPUT TO LOCAL LLM:
- Target GPS: 37.7881, -122.4036
- Current GPS: 37.7899, -122.4052
- Delivery Photo: [image data]
- Delivery Instructions: "Deliver to reception desk at Acme Corp"
- Time: 12:45 PM (within delivery window)

LOCAL LLM ANALYSIS:
"Current location is approximately 200 meters from target destination.
The delivery photo shows a reception desk but signage in image indicates
'TechStart Inc' not 'Acme Corp'. This suggests possible building or
office mismatch despite being at a reception area."

RESULT: Verification flagged for review with 68% confidence
```

### Core Verification Functions

The on-device LLM would evaluate:

1. **Location Accuracy**
   - GPS proximity to target
   - Matches known building/area patterns

2. **Visual Confirmation**
   - Food/package placement matches instructions
   - Identifiable landmarks in photos match expected location
   - Text recognition from signage or building numbers

3. **Instruction Compliance**
   - Special delivery instructions have been followed
   - Required steps completed

4. **Temporal Validation**
   - Delivery completed within expected time window

## Implementation Roadmap

### Phase 1: Model Development
- Create specialized dataset from successful deliveries
- Train base model on delivery verification tasks
- Distill and quantize model for mobile deployment

### Phase 2: Infrastructure Integration
- Develop offline-online sync mechanisms
- Implement confidence scoring system
- Create fallback protocols for edge cases

### Phase 3: Field Testing
- Deploy to limited Dasher test group
- Measure accuracy against server-side verification
- Gather performance metrics on various device types

### Phase 4: Full Deployment
- Gradual rollout with monitoring
- Continuous model improvement via federated learning
- Expand capabilities based on real-world performance

## Technical Requirements

### Device Specifications
- Minimum RAM: 3GB
- Storage: 500MB available for model
- Processor: Snapdragon 700 series or equivalent
- Operating Systems: iOS 14+, Android 10+

### Optimization Techniques
- 4-bit weight quantization
- Sparse activation
- Knowledge distillation
- Task-specific pruning

## Privacy and Security Considerations

- All sensitive customer data processed locally
- Photos never leave device when operating in offline mode
- Cryptographic verification of delivery completion
- Model cannot be used for purposes outside delivery verification

## Business Impact

### Expected Benefits
- 99.8% delivery completion rate regardless of connectivity
- Estimated 12% reduction in Dasher support calls
- Improved customer satisfaction scores for deliveries in low-connectivity areas
- Reduced payment delays for Dashers

### Metrics to Track
- Offline verification accuracy
- False positive/negative rates
- Processing time on various devices
- Battery consumption
- Storage impact

---

## Example Code Implementation (Kotlin)

```kotlin
import org.tensorflow.lite.Interpreter
import android.graphics.Bitmap
import android.location.Location
import java.nio.ByteBuffer
import java.nio.ByteOrder
import kotlin.math.abs

class OfflineDeliveryVerifier(context: Context) {

    private val interpreter: Interpreter
    private val locationProvider: FusedLocationProviderClient
    private val modelFile = "delivery_verification_model.tflite"
    
    init {
        val assetManager = context.assets
        val model = loadModelFile(assetManager, modelFile)
        interpreter = Interpreter(model)
        locationProvider = LocationServices.getFusedLocationProviderClient(context)
    }
    
    /**
     * Verify delivery without server connection
     */
    fun verifyDelivery(
        targetLocation: LatLng,
        deliveryPhoto: Bitmap,
        deliveryInstructions: String,
        expectedDeliveryWindow: TimeWindow
    ): VerificationResult {
        
        // Get current location
        val currentLocation = getCurrentLocation()
        
        // Process delivery photo
        val processedImage = preprocessImage(deliveryPhoto)
        
        // Prepare input data for model
        val inputData = ByteBuffer.allocateDirect(INPUT_SIZE)
            .order(ByteOrder.nativeOrder())
        
        // Add location data
        inputData.putFloat(targetLocation.latitude.toFloat())
        inputData.putFloat(targetLocation.longitude.toFloat())
        inputData.putFloat(currentLocation.latitude.toFloat())
        inputData.putFloat(currentLocation.longitude.toFloat())
        
        // Add processed image data
        processedImage.forEach { inputData.putFloat(it) }
        
        // Add text embedding of delivery instructions
        val instructionsEmbedding = generateTextEmbedding(deliveryInstructions)
        instructionsEmbedding.forEach { inputData.putFloat(it) }
        
        // Add time data
        val currentTime = System.currentTimeMillis()
        inputData.putFloat(currentTime.toFloat())
        inputData.putFloat(expectedDeliveryWindow.startTime.toFloat())
        inputData.putFloat(expectedDeliveryWindow.endTime.toFloat())
        
        // Run inference
        val outputBuffer = ByteBuffer.allocateDirect(OUTPUT_SIZE)
            .order(ByteOrder.nativeOrder())
        
        interpreter.run(inputData, outputBuffer)
        
        // Process results
        val confidence = outputBuffer.getFloat(0)
        val isVerified = outputBuffer.getFloat(1) > 0.5
        val hasIssues = outputBuffer.getFloat(2) > 0.5
        
        // Store verification for later sync
        val verificationRecord = DeliveryVerificationRecord(
            timestamp = currentTime,
            location = currentLocation,
            isVerified = isVerified,
            confidence = confidence,
            photoPath = savePhotoLocally(deliveryPhoto)
        )
        
        // Store in local database for sync when online
        VerificationDatabase.insert(verificationRecord)
        
        return VerificationResult(
            isVerified = isVerified,
            confidence = confidence,
            hasIssues = hasIssues,
            offlineMode = true
        )
    }
    
    /**
     * Sync stored verification data when connection returns
     */
    fun syncStoredVerifications() {
        if (!isNetworkAvailable()) return
        
        val pendingVerifications = VerificationDatabase.getPendingVerifications()
        
        pendingVerifications.forEach { record ->
            DeliveryApi.submitVerification(
                deliveryId = record.deliveryId,
                timestamp = record.timestamp,
                location = record.location,
                isVerified = record.isVerified,
                confidence = record.confidence,
                photoPath = record.photoPath
            )
        }
    }
}
```

## Conclusion

Implementing an on-device LLM for offline delivery verification represents a significant advancement in DoorDash's ability to handle deliveries reliably regardless of connectivity conditions. The proposed solution balances accuracy and device resource requirements while maintaining security and privacy standards. By enabling true offline operation, we can improve both the Dasher experience and customer satisfaction metrics in challenging delivery environments.
