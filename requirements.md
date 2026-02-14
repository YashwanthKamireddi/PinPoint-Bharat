# PinPoint-Bharat: Requirements Document

## Executive Summary

PinPoint-Bharat is a **Generative Geocoding Agent** that solves India's "Last Mile" delivery crisis by converting unstructured vernacular voice instructions (e.g., "Blue temple ke peeche") into precise GPS coordinates through multimodal AI analysis of real-time satellite imagery.

---

## 1. Problem Statement

### The $5 Billion Last-Mile Crisis

**Market Context:**
- India's logistics sector loses **$5 billion annually** due to failed last-mile deliveries
- 68% of rural addresses are **landmark-based** ("Near the banyan tree", "Behind the blue temple")
- Standard geocoding APIs (Google Maps, Mapbox) fail in rural India where formal addresses don't exist
- Delivery agents spend 40% of time calling customers for directions

**Critical Barriers:**
- **No formal addressing system** in 600,000+ villages
- **Vernacular descriptions** don't translate to coordinates (Hindi: "मंदिर के पीछे", Tamil: "கோவிலுக்கு பின்னால்")
- **Dynamic landmarks** (seasonal markets, temporary structures) not in maps
- **Cognitive load** on delivery agents navigating unfamiliar terrain

**Business Impact:**
- E-commerce companies: 15-20% failed deliveries in Tier-2/3 cities
- Logistics cost: ₹80-120 per failed delivery attempt
- Customer churn: 35% after 2 failed deliveries
- Emergency services: 12-minute average delay in rural areas

---

## 2. Solution Overview

### Core Value Proposition

**"Speak your location in any language → Get GPS coordinates in 3 seconds"**

### Technology Approach

A **multimodal agentic system** that:
1. Accepts vernacular voice descriptions via WhatsApp/mobile app
2. Extracts spatial entities using Amazon Bedrock (Anchor, Target, Vector)
3. Fetches satellite imagery from Amazon Location Service
4. Identifies landmarks using Amazon SageMaker computer vision
5. Calculates precise GPS coordinates using geospatial mathematics

---

## 3. Functional Requirements

### FR-1: Vernacular Voice Input
**Priority:** P0  
**Description:** Accept voice notes in 22 Indian languages

**Input Specifications:**
- **Supported Languages:** Hindi, Tamil, Telugu, Bengali, Marathi, Gujarati, Kannada, Malayalam, Punjabi, Odia, Assamese, Urdu, Kashmiri, Konkani, Nepali, Sindhi, Dogri, Manipuri, Bodo, Santali, Maithili, Sanskrit
- **Audio Format:** MP3, WAV, OGG (WhatsApp-compatible)
- **Duration:** 5-60 seconds
- **Quality:** 8 kHz minimum sampling rate

**Processing:**
- **Amazon Transcribe** with language auto-detection
- Noise reduction for outdoor environments
- Dialect handling (e.g., Bhojpuri variant of Hindi)

**Acceptance Criteria:**
- 95% transcription accuracy for clear audio
- <2 second transcription latency
- Handle background noise (traffic, wind)

---

### FR-2: Spatial Entity Extraction
**Priority:** P0  
**Description:** Parse natural language into structured spatial relationships

**Entity Types:**
- **Anchor Landmark:** Primary reference point (temple, school, water tank)
- **Target Location:** Destination (house, shop, farm)
- **Directional Vector:** Spatial relationship (behind, 50m north, opposite)
- **Contextual Attributes:** Color, size, unique features (blue temple, big banyan tree)

**Example Parsing:**
```
Input: "नीले मंदिर के पीछे 50 मीटर पर लाल घर"
(Blue temple ke peeche 50 meter par lal ghar)

Output:
{
  "anchor": {
    "type": "temple",
    "attributes": ["blue"],
    "confidence": 0.92
  },
  "vector": {
    "direction": "behind",
    "distance": 50,
    "unit": "meters"
  },
  "target": {
    "type": "house",
    "attributes": ["red"],
    "confidence": 0.88
  }
}
```

**Amazon Bedrock Integration:**
- Use **Claude 3.5 Sonnet** for complex spatial reasoning
- Prompt engineering for Indian context (local landmarks, measurement units)
- Few-shot learning with 500+ annotated examples

**Acceptance Criteria:**
- Extract anchor, vector, target with >85% accuracy
- Handle ambiguous descriptions ("near the big tree")
- Support compound directions ("50m north, then 20m east")

---

### FR-3: Visual Grounding (Satellite Object Detection)
**Priority:** P0  
**Description:** Identify landmarks in satellite imagery

**Data Sources:**
- **Amazon Location Service** (Esri satellite tiles, 30cm resolution)
- **Sentinel-2** (10m resolution, free, updated every 5 days)
- **Google Earth Engine** (historical imagery for validation)

**Object Detection:**
- **SageMaker Serverless Inference** with ResNet-50 backbone
- **Custom training dataset:** 10,000 labeled images of Indian landmarks
  - Temples (Hindu, Muslim, Christian, Sikh) - distinctive roof structures
  - Schools (government, private, anganwadi) - large rectangular buildings
  - Water bodies (tanks, ponds, rivers) - blue/dark patches
  - Agricultural features (tube wells, grain silos) - circular structures
  - Infrastructure (cell towers, transformers) - vertical structures
  - Large trees (banyan, peepal) - distinctive canopy shapes

**Resolution Constraints:**
- **30cm resolution (Esri):** Can detect building roofs, painted surfaces, large structures
- **10m resolution (Sentinel-2):** Can detect water bodies, large buildings, tree clusters
- **Cannot detect:** Doors, windows, small signs, individual people, vehicle colors
- **Can detect:** Temple domes, school compounds, water tank roofs, large tree canopies

**Detection Pipeline:**
1. Fetch 1km² satellite tile centered on user's approximate GPS
2. Run object detection model (SageMaker endpoint)
3. Filter detections by confidence >0.75
4. Match detected objects to anchor landmark description

**Acceptance Criteria:**
- Detect 90% of major landmarks (temples, schools, water tanks)
- <5% false positive rate
- Handle occlusions (trees covering buildings)
- Process 1km² tile in <3 seconds

---

### FR-4: Geospatial Triangulation
**Priority:** P0  
**Description:** Calculate target GPS coordinates from spatial relationships

**Mathematical Operations:**
- **Bearing calculation:** Convert "behind" to azimuth (0-360°)
- **Distance projection:** Apply Haversine formula for curved Earth
- **Coordinate transformation:** WGS84 (GPS) ↔ UTM (meters)

**Directional Mapping:**
```python
DIRECTION_BEARINGS = {
    "north": 0,
    "northeast": 45,
    "east": 90,
    "southeast": 135,
    "south": 180,
    "southwest": 225,
    "west": 270,
    "northwest": 315,
    "behind": 180,  # Relative to anchor orientation
    "front": 0,
    "left": 270,
    "right": 90
}
```

**Algorithm:**
```
1. Identify anchor landmark GPS (from satellite detection)
2. Determine anchor orientation (building facade direction)
3. Convert relative direction to absolute bearing
4. Project distance along bearing to get target GPS
5. Apply confidence radius based on description ambiguity
```

**Acceptance Criteria:**
- <10 meter error for precise descriptions ("50m north")
- <50 meter error for vague descriptions ("near the temple")
- Return confidence radius (e.g., "±15m")

---

### FR-5: Multi-Turn Clarification
**Priority:** P1  
**Description:** Interactive refinement for ambiguous locations

**Trigger Conditions:**
- Multiple matching landmarks detected (3 blue temples in area)
- Insufficient spatial information ("near the temple" without direction)
- Low confidence detection (<0.7)

**Clarification Flow:**
1. Agent asks follow-up question via WhatsApp
   - "मैंने 3 नीले मंदिर देखे। क्या आपका मतलब बड़े वाले से है?"
   - (I see 3 blue temples. Do you mean the big one?)
2. User responds with additional context
3. Agent re-runs detection with refined parameters

**Acceptance Criteria:**
- <2 clarification questions per request
- 90% resolution rate after clarification

---

### FR-6: WhatsApp Bot Integration
**Priority:** P0  
**Description:** Seamless user experience via WhatsApp Business API

**User Flow:**
1. User sends voice note to WhatsApp number
2. Bot acknowledges: "आपका स्थान खोज रहे हैं..." (Searching your location...)
3. Bot returns Google Maps link with GPS coordinates
4. Optional: Send static map image with marker

**Technical Integration:**
- **Twilio WhatsApp API** or **Meta WhatsApp Business API**
- **Amazon API Gateway** webhook for incoming messages
- **AWS Lambda** for message processing
- **Amazon S3** for voice note storage (ephemeral, 24h retention)

**Acceptance Criteria:**
- <5 second end-to-end response time
- Support 1,000 concurrent users
- Handle media attachments (voice, images)

---

## 4. Non-Functional Requirements

### NFR-1: Latency
**Requirement:** <10 seconds end-to-end processing

**Latency Budget:**
- Voice transcription (Transcribe): 1.2s
- Spatial entity extraction (Bedrock): 1.8s
- Satellite tile fetch (Location Service): 0.8s (cached: 0.2s)
- Object detection (SageMaker): 2.5s
- Geospatial calculation (Lambda): 0.3s
- Response delivery (WhatsApp): 0.4s
- Network overhead: 1.0s
- **Total:** 8.0s (realistic)

**Context:** Manual phone calls take 5+ minutes. We deliver 30x faster.

**Optimization Strategies:**
- Cache satellite tiles for frequently requested areas (90% hit rate)
- Pre-warm SageMaker endpoints during peak hours
- Use Bedrock streaming for faster perceived latency

**Acceptance Criteria:**
- P95 latency <10s
- P99 latency <15s

---

### NFR-2: Cost Efficiency
**Requirement:** <₹0.50 per geocoding request

**Cost Breakdown (Target):**
- Amazon Transcribe: ₹0.06 (30s audio)
- Amazon Bedrock (Claude 3.5 Sonnet): ₹0.12 (500 tokens)
- Amazon Location Service: ₹0.08 (1 tile fetch)
- SageMaker Serverless Inference: ₹0.15 (1 inference)
- AWS Lambda: ₹0.02 (compute)
- Amazon S3: ₹0.01 (storage)
- WhatsApp API: ₹0.05 (message delivery)
- **Total:** ₹0.49

**Cost Optimization:**
- Use **SageMaker Serverless** (no idle GPU costs)
- Cache satellite tiles in **ElastiCache** (₹0.02 vs ₹0.08 per fetch)
- Batch Bedrock requests for high-volume users
- Use **Amazon Nova Lite** for simple queries (₹0.06 vs ₹0.12)

**Acceptance Criteria:**
- Average cost <₹0.50 per request
- 90% of requests use cached tiles

---

### NFR-3: Accuracy
**Requirement:** 85% of coordinates within 50m of ground truth

**Validation Methodology:**
- Ground truth dataset: 1,000 manually verified locations
- Test across diverse terrains (urban, rural, hilly, coastal)
- Measure Euclidean distance between predicted and actual GPS

**Error Sources:**
- Satellite imagery resolution (30cm-10m)
- Landmark ambiguity (multiple similar buildings)
- Outdated imagery (new construction not visible)
- User description vagueness

**Acceptance Criteria:**
- 85% accuracy within 50m
- 95% accuracy within 100m
- <2% catastrophic failures (>500m error)

---

### NFR-4: Scalability
**Requirement:** Handle 10,000 requests/hour during peak

**Architecture Constraints:**
- Fully serverless (Lambda, SageMaker Serverless)
- Auto-scaling for all components
- No single point of failure

**Load Profile:**
- Peak: 10,000 requests/hour (2.78 requests/second)
- Average: 2,000 requests/hour
- Burst: 50 requests/second (flash sales, emergencies)

**Acceptance Criteria:**
- Zero throttling at peak load
- API Gateway: 10,000 RPS limit
- SageMaker: 10 concurrent inferences

---

### NFR-5: Security & Privacy
**Requirement:** Protect user location data, comply with DPDP Act 2023

**Data Protection:**
- Voice notes deleted after 24 hours (ephemeral processing)
- GPS coordinates not stored (stateless API)
- Encryption at rest (S3 SSE-KMS)
- Encryption in transit (TLS 1.3)

**Access Control:**
- API authentication via API keys
- Rate limiting: 100 requests/hour per user
- No PII logging (only anonymized metrics)

**Compliance:**
- DPDP Act 2023 (Indian data privacy)
- No cross-border data transfer (all processing in ap-south-1)

**Acceptance Criteria:**
- Zero data breaches
- Automated voice note deletion
- Audit logs retained for 90 days

---

## 5. Success Metrics

### Primary KPIs
1. **Geocoding Accuracy:** >85% within 50m
2. **Response Time:** P95 <3 seconds
3. **Cost Efficiency:** <₹0.50 per request
4. **User Adoption:** 10,000 requests/month within 6 months

### Secondary KPIs
1. **Failed Delivery Reduction:** 30% decrease for pilot logistics partners
2. **User Satisfaction:** NPS >60
3. **Clarification Rate:** <15% of requests require follow-up
4. **Satellite Cache Hit Rate:** >90%

---

## 6. Out of Scope (Phase 1)

- Indoor navigation (building floor plans)
- Real-time traffic routing
- Multi-stop route optimization
- Integration with navigation apps (Waze, Google Maps)
- Offline mode (requires internet)

---

## 7. Assumptions & Dependencies

### Assumptions
1. Users have smartphones with WhatsApp
2. 4G network available (minimum 1 Mbps)
3. Landmarks are visible in satellite imagery
4. Users can describe locations in natural language

### Dependencies
1. Amazon Location Service availability in ap-south-1
2. SageMaker model training dataset (10,000 labeled images)
3. WhatsApp Business API approval
4. Sentinel-2 satellite imagery access

---

## 8. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Low satellite resolution in remote areas | High | Medium | Fallback to 10m Sentinel-2 imagery |
| Bedrock rate limits during peak | High | Low | Implement request queuing with SQS |
| Ambiguous landmark descriptions | Medium | High | Multi-turn clarification workflow |
| Outdated satellite imagery | Medium | Medium | Display imagery date to user |

---

## Appendix A: Example Use Cases

### Use Case 1: E-Commerce Delivery
**Scenario:** Flipkart delivery agent in rural Maharashtra

**Input (Marathi):** "शाळेच्या मागे 100 मीटर, लाल दरवाजा"  
(Shaalecha maage 100 meter, laal darwaja)

**Translation:** "100 meters behind the school, red door"

**Processing:**
1. Transcribe Marathi audio
2. Extract: Anchor=school, Vector=100m behind, Target=red door
3. Fetch satellite tile, detect school building
4. Calculate GPS 100m south of school
5. Return: 19.1234°N, 73.5678°E (±20m)

**Outcome:** Delivery agent navigates directly, saves 15 minutes

---

### Use Case 2: Emergency Medical Response
**Scenario:** Ambulance dispatch in rural Rajasthan

**Input (Hindi):** "बड़े पीपल के पेड़ के पास, गाँव के बाहर"  
(Bade peepal ke ped ke paas, gaon ke bahar)

**Translation:** "Near the big banyan tree, outside the village"

**Processing:**
1. Transcribe Hindi audio
2. Extract: Anchor=banyan tree, Vector=near, Context=outside village
3. Detect large tree in satellite imagery (distinctive canopy)
4. Return GPS with 50m confidence radius

**Outcome:** Ambulance reaches patient 8 minutes faster

---

**Document Version:** 1.0  
**Last Updated:** February 14, 2026  
**Owner:** PinPoint-Bharat Team
