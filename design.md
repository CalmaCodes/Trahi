# Design Document: TRAHI  
## Multi-Agent AI Emergency Response System

---

## 1. Overview

TRAHI is an agentic, distributed emergency coordination platform designed to autonomously handle emergency intake, severity assessment, resource matching, routing, hospital coordination, and multi-agency response.

The system follows a closed-loop operational model:

**Sense → Decide → Act → Monitor → Re-optimize**

Each functional capability is implemented as an independent AI Agent coordinated by a central Orchestrator.

The architecture supports:

- Autonomous decision making
- Real-time re-optimization
- Human-in-the-loop fallback
- Continuous learning
- Horizontal scalability
- Fault tolerance

The prototype (MVP) demonstrates a single vertical slice while preserving full production architecture.

---

## 2. High-Level Architecture

TRAHI consists of six logical layers:

1. Intake Layer  
2. Orchestration Layer  
3. Agent Layer  
4. Data Layer  
5. Integration Layer  
6. Presentation Layer  

---

### System Layers

### Intake Layer
- Voice Emergency Intake
- Text Emergency Intake

### Orchestration Layer
- Central Orchestrator
- Incident State Manager
- Event Bus

### Agent Layer
- Listener Agent
- Non-Emergency Filter Agent
- Matcher Agent
- Router Agent
- Hospital Agent
- Multi-Agency Agent
- Learning Agent
- Human-in-the-Loop Controller

### Data Layer
- Incident Store
- Resource Store
- Hospital Capacity Store
- Audit Log Store
- Analytics Store

### Integration Layer
- Maps APIs
- Speech-to-Text APIs
- External Emergency Systems
- Notification Services

### Presentation Layer
- Citizen Web Interface
- Authority Dashboard

---

## 3. Architecture Diagram (Conceptual)

┌─────────────────┐
│  Citizen        │
│  (Voice/Text)   │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Listener Agent │
└────────┬────────┘         
┌─────────────────┐
│Non-Emergency    │
│Filter           │
└────────┬────────┘
         ↓
┌─────────────────┐
│ Incident Store  │
└────────┬────────┘         ↓
    ┌─────────────────────────────────────┐
    │       Central Orchestrator          │
    └───────┬──────┬──────┬──────┬────────┘
            ↓       ↓       ↓      ↓
    ┌───────┐ ┌─────────┐ ┌──────┐ ┌──────────┐
    │Matcher│ │ Router  │ │Hospital│  Multi-  │
    │Agent  │ │ Agent   │ │Agent │ │  Agency  │
    └───┬───┘ └────┬────┘ └───┬──┘ │  Agent   │
        ↓          ↓          ↓    └────┬─────┘
   ┌────────┐ ┌─────────┐ ┌─────────┐   │
   │Resource│ │ Maps    │ │Hospital │   │
   │Store   │ │ API     │ │Store    │   │
   └────────┘ └─────────┘ └─────────┘   │
                                        ↓
                                ┌─────────────────────┐
                                │ Human-in-loop       │
                                │ Controller          │
                                └──────────┬──────────┘                                            ↓
                                 ┌─────────────────────┐
                                 │ Authority Dashboard │
                                 └──────────┬──────────┘
                                            ↓
                                 ┌─────────────────────┐
                                 │    Responders       │
                                 └─────────────────────┘



---

## 4. Agent Communication Model

Agents communicate asynchronously through a shared Incident State and Event Bus.

Process:

1. Listener Agent creates Incident_Report
2. Orchestrator publishes IncidentCreated event
3. Agents subscribe and act independently
4. Results are written back to Incident Store
5. Orchestrator reconciles and advances state

This design ensures:

- Loose coupling
- Parallel execution
- Independent scaling
- Fault isolation

---

## 5. Technology Stack (Prototype)

### Frontend
- React
- Web Speech API

### Backend
- Python FastAPI

### AI Models
- Whisper (speech-to-text)
- LLM for classification
- ML models for severity and routing

### Mapping
- Google Maps / Mapbox

### Database
- SQLite / JSON (prototype)
- PostgreSQL (production)

---

## 6. Agent Responsibilities

---

### Listener Agent

Responsible for:

- Voice transcription
- Language detection
- Panic detection
- Emergency type extraction
- Location extraction
- Confidence scoring

Interface:

processInput(
  input_type: enum {VOICE, TEXT},
  payload: bytes | string,
  metadata: {
    caller_id?: string,
    gps?: Location,
    language_hint?: string
  }
) -> IncidentReport


Outputs:

IncidentReport:
  incident_id: UUID
  raw_transcript: string
  detected_language: string
  location: Location
  emergency_type: string
  severity_indicators: array<string>
  panic_score: float
  confidence_score: float


---

### Non-Emergency Filter Agent

Classifies calls into:

- Emergency
- Non-Emergency

Routes non-urgent calls to chatbot or helpline.


Interface: 

classifyUrgency(
  incident: IncidentReport
) -> 


Output: 
UrgencyResult:
  category: enum {EMERGENCY, NON_EMERGENCY}
  confidence: float



---



### Matcher Agent

Selects optimal responders using:

- Distance / ETA
- Availability
- Equipment match
- Hospital capacity

Interface:

matchResources(
  incident: IncidentReport,
  available_rsources: Resource[]
) -> ResourceAssignment
e
Output: 

-ResourceAssignment:
-primary_resource: Resource
-alternatives: Resource[]
-reasoning: string[]

### Router Agent

Computes and monitors routes:

- Calculates multiple alternatives
- Tracks ETA
- Re-routes dynamically
- Supports Green Corridor

Interface:

calculateRoute(
  resource_location: Location,
  incident_location: Location
) -> Route

recalculateRoute(
  route_id: UUID,
  obstruction: Obstruction
) -> Route

Output:

Route:
  id: UUID
  waypoints: Location[]
  eta_minutes: float
  confidence: float


### Hospital Agent

Monitors and reserves beds:

- ICU
- Emergency
- Ventilators

Automatically blocks and releases beds.

Interface:

selectHospital(
  incident: IncidentReport,
  hospitals: Hospital[]
) -> HospitalAssignment

Output:

HospitalAssignment:
  hospital_id: UUID
  bed_type: string
  confirmation_status: boolean


---

### Multi-Agency Agent

Coordinates:

- Police
- Fire
- Ambulance

Creates unified response plans.


Interface: 

coordinateResponse(
  incident: IncidentReport,
  assigned_resources: Resource[]
) -> CoordinationPlan




---

### Learning Agent

Analyzes completed incidents:

- Identifies delays
- Detects bottlenecks
- Updates models nightly

Interface: 

analyzeIncident(
  incident_id: UUID
) -> LearningInsights


---

### Human-in-the-Loop Controller

Triggers manual intervention when:

- Confidence < threshold
- Child detected
- Mental health crisis
- User requests human
- Technical failures

Human receives full incident context.


Interface: 

requestHumanIntervention(
  incident_id: UUID,
  reason: string
) -> void



---

## 7. MVP Prototype Scope

The prototype implements:

- Voice + Text Intake
- Listener Agent
- Matcher Agent (mock resources)
- Router Agent (maps API)
- Hospital Agent (mock beds)
- Authority Dashboard
- Human override

Simulated:

- Emergency vehicles
- Hospital beds
- Traffic conditions

---

## 8. MVP Flow

User submits emergency
|
Listener Agent
|
Incident created
|
Matcher selects ambulance
|
Router computes route
|
Hospital selected
|
Dashboard updates

---

## 9. Incident Lifecycle

CREATED → CLASSIFIED → ROUTED → DISPATCHED → IN_PROGRESS → RESOLVED

---

## 10. Core Data Models

### Incident :

id: UUID
created_at: timestamp
updated_at: timestamp
status: enum {CREATED, CLASSIFIED, ROUTED, DISPATCHED, IN_PROGRESS, RESOLVED}
source: enum {VOICE, TEXT, IOT}
language: string
raw_transcript: text
location: Location
emergency_type: string
severity_level: enum {LOW, MEDIUM, HIGH, CRITICAL}
confidence_score: float
assigned_resources: array<UUID>
assigned_hospital: UUID | null
route_id: UUID | null
audit_log: json


---

### Resource:

id: UUID
type: enum {AMBULANCE, POLICE, FIRE}
location: Location
availability: enum {AVAILABLE, BUSY, RESPONDING, OFF_DUTY}
capabilities: array<string>
eta_minutes: float

---

### Hospital

id: UUID
name: string
location: Location
beds:
ICU: integer
Emergency: integer
General: integer
Ventilator: integer
last_updated: timestamp

---
### Route

Route:
id: UUID
origin: Location
destination: Location
waypoints: array<Location>
eta_minutes: float
recalculation_count: integer

---

### AuditEntry

AuditEntry:
timestamp: datetime
agent: string
action: string
confidence: float
reasoning: array<string>


---

## 11. Routing and Green Corridor Design

### Route Selection Pipeline

1. Router Agent queries Maps API for top 3 routes
2. ETA calculated for each
3. Fastest route selected
4. Alternatives stored for fallback

Routing score:

score = ETA + congestion_penalty + obstruction_risk



---

### Dynamic Rerouting

Recalculation triggers:

- Traffic delay > 2 minutes
- Road obstruction detected
- Manual override

---

### Green Corridor Workflow

1. Identify traffic signals on selected route
2. Send ETA to traffic control
3. Request green wave 30 seconds ahead
4. If API unavailable, notify traffic police manually

---

## 12. Hospital Selection and Bed Blocking

### Hospital Assignment Algorithm

1. Filter hospitals within radius
2. Remove hospitals with zero relevant beds
3. Rank by ETA + bed availability
4. Select optimal hospital
5. Automatically reserve bed
6. Notify ambulance crew

---

### Bed Release Logic

- If reroute occurs → release previous bed
- If incident cancelled → release bed
- If arrival confirmed → mark occupied

---

## 13. Learning Agent Pipeline

Executed after incident resolution.

### Analysis Steps

1. Reconstruct incident timeline
2. Measure:
   - Call to dispatch time
   - Travel duration
   - Hospital wait
3. Identify bottlenecks
4. Store analytics

---

### Model Updates

- Nightly retraining on new incidents
- Update:
  - Severity models
  - Routing predictions
  - Resource availability forecasts

---

## 14. Authority Dashboard

### Dashboard Views

- Active incidents map
- Resource locations
- Hospital capacity
- ETA indicators
- Confidence alerts

---

### Dispatcher Controls

- Take over incident
- Change hospital
- Reassign resource
- Add notes

---

## 15. Incident Persistence and Audit Trail

Each agent action produces an AuditEntry.

Audit fields:

- Timestamp
- Agent ID
- Action
- Reasoning
- Confidence score

All incidents retained minimum 7 years.

Encryption at rest: AES-256  
Encryption in transit: TLS 1.3  

---

## 16. Orchestration Logic

Central Orchestrator advances incident states:

CREATED
→ CLASSIFIED
→ ROUTED
→ DISPATCHED
→ IN_PROGRESS
→ RESOLVED


Transitions only allowed if required agent outputs exist.

---

## 17. Security Architecture

- Role-based access control
- Dispatcher / Responder / Admin roles
- Full access logging
- Suspicious activity alerts
- Citizen data anonymized in analytics

---

## 18. Scalability Strategy

- Stateless agents
- Horizontal scaling
- Event-driven processing
- Cached routing
- Degraded-mode operation when APIs unavailable

---

---

## 19. Correctness Properties

The following properties define system-wide guarantees derived directly from requirements.md.

---

### Property 1: Incident Creation Completeness

For any emergency submission, an Incident record must be created containing:

- incident_id
- emergency_type
- location
- severity_level

If any field is missing, clarification must be requested.

---

### Property 2: Language Consistency

For any incident submitted in language L, all system responses (guidance, notifications, dashboard labels) must remain in language L.

---

### Property 3: Critical Keyword Escalation

If an incident transcript contains life-threatening keywords ("unconscious", "bleeding", "not breathing"), severity must be CRITICAL.

---

### Property 4: Confidence-Based Human Handoff

If Confidence_Score < 85% on critical fields, Human_Handoff must be triggered.

---

### Property 5: Resource Capability Matching

Any assigned Resource must support the emergency type and required equipment.

---

### Property 6: Route Completeness

All computed routes must include:

- origin
- destination
- waypoints
- ETA

---

### Property 7: Reroute on Obstruction

If obstruction detected, Router Agent must recompute route within 3 seconds.

---

### Property 8: Delay Notification

If ETA increases by more than 5 minutes, responders and hospital must be notified.

---

### Property 9: Hospital Bed Reservation

Before ambulance departure, Hospital Agent must reserve an appropriate bed.

---

### Property 10: Bed Release on Reroute

If hospital assignment changes, previous reservation must be released.

---

### Property 11: Multi-Incident Prioritization

When resources are limited, CRITICAL incidents must be serviced before HIGH, MEDIUM, or LOW.

---

### Property 12: Audit Logging Completeness

Every agent action must generate an AuditEntry.

---

### Property 13: Unique Incident Identifiers

All Incident IDs must be globally unique.

---

### Property 14: Duplicate Incident Merging

Reports within 100 meters and 5 minutes must be merged.

---

### Property 15: Automatic Callback

Disconnected emergency calls must be retried within 5 seconds.

---

### Property 16: Non-Emergency Filtering

Calls classified as non-emergency must be routed away without blocking emergency pipeline.

---

### Property 17: Learning Trigger

Every RESOLVED incident must trigger Learning Agent analysis.

---

### Property 18: Dashboard Consistency

Authority dashboard must reflect incident state changes within 2 seconds.

---

### Property 19: Encryption Guarantees

All stored data must be AES-256 encrypted; all transit must use TLS 1.3.

---

### Property 20: Fail-Safe Dispatch

Failure of non-critical agents must not prevent emergency dispatch.

---

## 20. Error Handling Strategy

### Error Categories

#### Input Errors
- Missing location
- Unclear incident type
- Unsupported language

Action: Request clarification or trigger Human_Handoff.

---

#### External API Failures
- Maps unavailable
- Hospital systems unreachable

Action: Use cached data and flag degraded accuracy.

---

#### Agent Failures
- ML model timeout
- Routing failure

Action: Fall back to rule-based logic.

---

#### Resource Exhaustion
- No available ambulances

Action: Queue incidents, alert authorities, activate multi-agency escalation.

---

## 21. Fault Tolerance

TRAHI assumes components will fail.

Design principles:

- Stateless agents
- Backup routing logic
- Cached hospital data
- Human override always available
- Incident persistence across restarts

---

## 22. Edge Case Handling

### Missing Location
System requests landmarks, uses cell triangulation, flags for human review.

---

### Duplicate Calls
Merged automatically to improve accuracy.

---

### False Emergencies
Processed normally but flagged for review.

---

### Network Loss
Incident queued locally and transmitted on reconnection.

---

## 23. Testing Strategy

### Unit Testing

- Each agent tested independently
- Mock APIs for routing and hospitals
- Orchestrator state transitions verified

---

### Integration Testing

- End-to-end incident flow
- Agent coordination
- Dashboard updates

---

### Property-Based Testing

Each correctness property must be validated using randomized inputs.

Framework:
- Python: Hypothesis

Minimum:
- 100 iterations per property

---

### Load Testing

- 1000 concurrent incidents
- Measure p95 latency
- Validate degraded mode behavior

---

### Security Testing

- Role access enforcement
- PII anonymization verification
- Penetration testing

---

## 24. Summary

TRAHI is architected as a resilient agentic system capable of autonomous emergency coordination.

The MVP demonstrates core Sense → Decide → Act behavior on a single incident flow, while the full design supports national-scale deployment across medical, fire, and police services.

The system prioritizes life-critical functionality, graceful degradation, and continuous learning, with humans integrated only when required.



