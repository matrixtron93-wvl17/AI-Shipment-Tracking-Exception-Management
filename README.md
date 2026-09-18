# AI Shipment Tracking & Exception Management Automation

An AI-powered shipment tracking and exception management workflow built with n8n, Google Gemini, Google Sheets, JavaScript, and webhook-based event processing.

The workflow is designed for logistics and cargo operations that need to monitor shipment events, identify delivery exceptions, prioritize operational issues, prepare customer notifications, and maintain an operational shipment tracker.

---

## Overview

This project started from an open-source n8n workflow focused on order tracking notifications.

The workflow was adapted and customized into an AI-powered shipment tracking and exception management system.

The original workflow was designed around order-status notifications and Google Sheets logging.

This implementation changes the business logic to focus on shipment operations by introducing:

- AI-based shipment classification
- Shipment exception detection
- Priority classification
- Confidence scoring
- Human-review requirements
- Shipment-specific notification logic
- Operational shipment tracking
- Structured AI output
- Logistics-focused test scenarios

The final workflow uses Google Sheets as the operational tracker and currently simulates WhatsApp notification delivery for portfolio testing.

---

## Business Problem

Logistics and cargo operations receive shipment events that may require monitoring and operational action.

Common situations include:

- Shipments moving normally
- Delayed shipments
- Failed deliveries
- Damaged packages
- Customs holds
- Missing shipment information
- Shipment exceptions requiring human review

Manually reviewing every shipment event can make it difficult to identify important exceptions quickly.

This automation demonstrates how AI can assist with shipment-event analysis and operational classification while maintaining a structured record of the results.

---

## Solution

The workflow receives shipment events, normalizes the incoming information, sends the shipment data to Google Gemini for classification, validates the AI result, prepares an appropriate notification, records the result in Google Sheets, and returns a structured API response.

### Main Workflow

Shipment Event Webhook
→ Normalize Shipment Data
→ AI Shipment Classifier
→ Validate Shipment Event
→ API Rate Limit Protection
→ Build Shipment Notification
→ Send Shipment Notification
→ Capture Notification Result
→ Update Shipment Operations Tracker
→ Shipment API Response

An error-handling path is also connected to the notification and tracking stages.

---

# Workflow Architecture

## 1. Trigger & Intake

This section receives and prepares incoming shipment information.

```text
Shipment Event Webhook
        ↓
Normalize Shipment Data
        ↓
AI Shipment Classifier
```

An optional polling trigger can also provide shipment events.

---

## 2. AI Classification & Validation

This section analyzes the normalized shipment data and validates the AI classification.

```text
AI Shipment Classifier
        ↓
Validate Shipment Event
        ↓
API Rate Limit Protection
        ↓
Build Shipment Notification
```

Google Gemini and the Structured Output Parser are connected to the AI classification node.

---

## 3. Notification & Operations Tracking

This section prepares the notification, records the notification result, updates the operational tracker, and returns the API response.

```text
Build Shipment Notification
        ↓
Send Shipment Notification
        ↓
Capture Notification Result
        ↓
Update Shipment Operations Tracker
        ↓
Shipment API Response
```

The Shipment Error Handler provides an error path for failures.

---

# Node-by-Node Documentation

## Node 1

### Original Node Name
`Webhook - Order Event`

### Renamed Node
`Shipment Event Webhook`

### Node Type
Webhook

### Purpose

Receives incoming shipment events from an external shipment system.

### Function

The webhook accepts HTTP POST requests containing shipment information such as:

- Shipment ID
- Tracking number
- Customer information
- Origin
- Destination
- Current location
- Shipment status
- Expected delivery
- Last updated time
- Remarks

### Why This Node Was Used

A webhook is appropriate for event-driven logistics automation because a shipment system can send an event whenever a shipment status changes.

It allows the workflow to process shipment events without requiring manual input.

### Customization

The original webhook was designed for order tracking.

It was renamed and adapted to:

`Shipment Event Webhook`

with the path:

`shipment-tracking-inbound`

This changes the workflow's business context from generic orders to logistics shipment events.

---

# Node 2

### Original Node Name
`Poll Order System`

### Renamed Node
`Poll Shipment System`

### Node Type
Schedule Trigger

### Purpose

Provides an optional scheduled mechanism for retrieving shipment events.

### Function

The node can trigger the workflow on a schedule instead of relying only on real-time webhook events.

### Why This Node Was Used

Some logistics systems may not provide real-time webhooks.

A polling mechanism can therefore act as an alternative event source or fallback mechanism.

### Current Scope

The node is treated as an optional shipment-system integration point.

It can be connected to an external shipment API or tracking system in a production implementation.

---

# Node 3

### Original Node Name
`Prepare Order Context`

### Renamed Node
`Normalize Shipment Data`

### Node Type
Set / Data Transformation

### Purpose

Standardizes incoming shipment information into a consistent structure before AI processing.

### Function

The node prepares fields including:

- `shipmentId`
- `trackingNumber`
- `customerName`
- `customerPhone`
- `origin`
- `destination`
- `currentLocation`
- `shipmentStatus`
- `expectedDelivery`
- `lastUpdated`
- `remarks`
- `ingestedAt`

### Why This Node Was Used

Incoming data may have different structures depending on the source system.

Normalizing the information before AI processing makes the downstream workflow more predictable and easier to maintain.

### Customization

The original order-related fields were replaced with shipment-specific fields.

For example:

`orderId`

became:

`shipmentId`

and:

`orderStatus`

became:

`shipmentStatus`

---

# Node 4

### Original Node Name
`JS - Detect Order Event`

### Renamed Node
`AI Shipment Classifier`

### Node Type
Basic LLM Chain

### Purpose

Analyzes shipment information and classifies the shipment status, exception type, priority, confidence, and human-review requirement.

### Function

The node sends normalized shipment information to Google Gemini.

The AI produces structured classification output containing:

- `shipment_status`
- `exception_type`
- `priority`
- `confidence`
- `requires_human_review`
- `reason`

### Why This Node Was Used

The original workflow used JavaScript to detect predefined order events.

For this project, the business requirement was expanded from simple event detection to AI-assisted shipment exception classification.

Replacing the original event-detection logic with an LLM-based classifier allows the workflow to analyze shipment information and remarks to identify potential operational exceptions.

### AI Rules

The classifier uses controlled categories.

#### Shipment Status

- `IN_TRANSIT`
- `OUT_FOR_DELIVERY`
- `DELIVERED`
- `PENDING`
- `DELAYED`
- `EXCEPTION`
- `FAILED_DELIVERY`
- `UNKNOWN`

#### Exception Type

- `NONE`
- `NO_MOVEMENT`
- `LATE_DELIVERY`
- `ADDRESS_ISSUE`
- `FAILED_DELIVERY`
- `DAMAGED`
- `CUSTOMS_HOLD`
- `MISSING_INFORMATION`
- `OTHER`

#### Priority

- `LOW`
- `MEDIUM`
- `HIGH`
- `CRITICAL`

### Human Review Logic

The AI can flag a shipment for human review when:

- A serious exception is detected
- Required information is missing
- AI confidence is below the configured threshold
- The shipment information is unclear
- The shipment requires operational intervention

---

# Node 5

### Supporting Node
`Google Gemini Chat Model`

### Purpose

Provides the Google Gemini model used by the AI Shipment Classifier.

### Function

The Gemini model processes the normalized shipment information and generates the structured classification requested by the workflow.

### Why This Node Was Used

The project requires an AI component capable of interpreting shipment information and identifying potential exceptions.

Google Gemini was selected as the LLM used for the prototype.

---

# Node 6

### Supporting Node
`Structured Output Parser`

### Purpose

Ensures that the AI classification follows a defined JSON structure.

### Function

The parser expects fields such as:

```json
{
  "shipment_status": "IN_TRANSIT",
  "exception_type": "NONE",
  "priority": "LOW",
  "confidence": 0.97,
  "requires_human_review": false,
  "reason": "Shipment is moving normally toward the destination."
}
```

### Why This Node Was Used

A structured parser makes the AI output easier for downstream n8n nodes to process.

Instead of relying on free-form AI text, the workflow receives predictable fields that can be mapped into Google Sheets and notification logic.

---

# Node 7

### Original Node Name
`Filter Valid Order Events`

### Renamed Node
`Validate Shipment Event`

### Node Type
IF

### Purpose

Checks whether the AI classification contains the required information before continuing.

### Function

The current validation checks that the AI output contains:

- Shipment status
- Exception type

Only valid classifications continue through the workflow.

### Why This Node Was Used

AI output should be validated before it is used by downstream automation.

This prevents the workflow from continuing when required classification information is missing.

### Customization

The original validation logic was changed from order-event validation to shipment-event validation.

---

# Node 8

### Original Node Name
`Wait - Rate Limit`

### Renamed Node
`API Rate Limit Protection`

### Node Type
Wait

### Purpose

Introduces a short delay before continuing the workflow.

### Function

The current configuration waits:

`2 seconds`

before proceeding.

### Why This Node Was Used

External APIs and AI services may have request-rate limitations.

A controlled delay provides basic protection against sending requests too quickly, especially during repeated testing or higher-volume processing.

### Customization

The original rate-limit step was retained but renamed to reflect its role in the shipment automation.

---

# Node 9

### Original Node Name
`JS - Build Message Payload`

### Renamed Node
`Build Shipment Notification`

### Node Type
Code

### Purpose

Creates the shipment notification message using shipment data and AI classification results.

### Function

The node combines:

- Shipment information
- Shipment status
- Exception type
- Priority
- Confidence
- Human-review requirement
- AI reason

The workflow creates different messages for normal shipments and shipment exceptions.

### Normal Shipment

A normal shipment receives a status-update style message.

### Shipment Exception

An exception generates an alert containing information such as:

- Shipment ID
- Tracking number
- Status
- Exception
- Priority
- Current location
- Expected delivery
- Reason
- Human-review requirement

### Why This Node Was Used

A Code node allows the notification message to be dynamically generated based on the AI classification.

This is more flexible than using one fixed message for every shipment.

---

# Node 10

### Original Node Name
`Send WhatsApp Message`

### Renamed Node
`Send Shipment Notification`

### Node Type
Code

### Purpose

Represents the shipment notification delivery stage.

### Function

For the current portfolio prototype, the node simulates WhatsApp delivery.

It generates information such as:

- Notification channel
- Notification status
- Notification result
- Simulated message ID
- Recipient phone
- Message content
- Simulation timestamp

### Why This Node Was Used

The original workflow used an HTTP Request node to connect to the WhatsApp Business API.

For this portfolio implementation, real WhatsApp delivery is intentionally simulated so the workflow can be demonstrated without sending real customer messages.

### Current Status

```text
notificationChannel = WHATSAPP_SIMULATION
notificationStatus = SIMULATED
notificationResult = TEST_ONLY
```

No real WhatsApp message is sent.

### Production Upgrade

The node can later be replaced or reconfigured to connect to the WhatsApp Business API.

---

# Node 11

### Original Node Name
`JS - Capture Delivery Receipt`

### Renamed Node
`Capture Notification Result`

### Node Type
Code

### Purpose

Captures the result of the notification stage.

### Function

The node records:

- Notification channel
- Notification status
- Notification result
- Notification message ID
- Shipment reference
- Tracking reference
- Processing status
- Processed timestamp

### Why This Node Was Used

The notification result needs to be available to the operational tracking stage.

Capturing the result separately also keeps the workflow modular.

### Customization

The original delivery-receipt logic was adapted for the simulated notification system.

---

# Node 12

### Original Node Name
`Update Google Sheet Tracker`

### Renamed Node
`Update Shipment Operations Tracker`

### Node Type
Google Sheets

### Purpose

Records processed shipment events and classification results in a centralized operational tracker.

### Function

The node appends shipment information to:

`AI Shipment Operations Tracker`

The tracker contains fields including:

- Shipment ID
- Tracking number
- Customer name
- Customer phone
- Origin
- Destination
- Current location
- Shipment status
- Exception type
- Priority
- Confidence
- Human-review requirement
- Reason
- Expected delivery
- Last updated
- Notification channel
- Notification status
- Notification result
- Notification message ID
- Processing status
- Processed timestamp

### Why This Node Was Used

Google Sheets provides a simple operational view that can be easily reviewed by logistics or operations teams.

It also makes the prototype easy to demonstrate to a client without requiring a separate dashboard or database.

### Customization

The original tracker was designed around order tracking.

It was redesigned around shipment operations and AI classification results.

---

# Node 13

### Original Node Name
`Respond to Webhook`

### Renamed Node
`Shipment API Response`

### Node Type
Respond to Webhook

### Purpose

Returns the processing result to the system that submitted the shipment event.

### Function

The response can include:

- Success status
- Shipment ID
- Tracking number
- Shipment status
- Exception type
- Priority
- AI confidence
- Human-review requirement
- Notification status
- Processing status
- Processed timestamp

### Why This Node Was Used

A webhook-based integration should provide a clear response to the calling system.

This makes the automation behave like an API endpoint rather than a workflow that silently processes data.

---

# Node 14

### Original Node Name
`Error Handler`

### Renamed Node
`Shipment Error Handler`

### Node Type
Error Handling / Code

### Purpose

Captures and formats workflow failures.

### Function

The handler records information such as:

- Shipment ID
- Tracking number
- Shipment status
- Exception type
- Priority
- Customer phone
- Notification status
- Error message
- Error code
- Failure timestamp

### Why This Node Was Used

External integrations and workflow nodes can fail.

An error-handling path provides visibility into failures and makes troubleshooting easier.

### Customization

The error information was changed from generic order-processing errors to shipment-specific operational information.

---

# Documentation Nodes

The workflow also contains sticky notes used to organize the canvas into logical sections.

## Customized Documentation Sections

### 1. Trigger & Intake

Accepts incoming shipment events through a webhook.

Supports real-time shipment events and scheduled polling as an optional fallback.

### 2. AI Classification & Validation

Analyzes shipment events, validates shipment data, and classifies shipment status, exceptions, and priority using AI.

### 3. Notification & Operations Tracking

Sends shipment notifications using the simulated WhatsApp layer, records shipment events in Google Sheets, and provides an operational view of shipment status and exceptions.

---

# AI Classification Logic

The classifier uses controlled categories to produce consistent results.

## Shipment Status

- `IN_TRANSIT`
- `OUT_FOR_DELIVERY`
- `DELIVERED`
- `PENDING`
- `DELAYED`
- `EXCEPTION`
- `FAILED_DELIVERY`
- `UNKNOWN`

## Exception Type

- `NONE`
- `NO_MOVEMENT`
- `LATE_DELIVERY`
- `ADDRESS_ISSUE`
- `FAILED_DELIVERY`
- `DAMAGED`
- `CUSTOMS_HOLD`
- `MISSING_INFORMATION`
- `OTHER`

## Priority

- `LOW`
- `MEDIUM`
- `HIGH`
- `CRITICAL`

## Human Review

The classifier can require human review when:

- A serious exception is detected
- Required information is missing
- AI confidence is below the configured threshold
- Shipment information is unclear
- Operational intervention is required

---

# AI Classification Example

```json
{
  "shipment_status": "DELAYED",
  "exception_type": "LATE_DELIVERY",
  "priority": "MEDIUM",
  "confidence": 0.95,
  "requires_human_review": false,
  "reason": "Shipment is past the expected delivery date."
}
```

---

# Test Scenarios

The workflow was tested using six logistics scenarios.

| Test ID | Scenario | Shipment Status | Exception | Priority |
|----------|----------|-----------------|-----------|----------|
| SHP-TEST-001 | Normal shipment | IN_TRANSIT | NONE | LOW |
| SHP-TEST-002 | Delayed shipment | DELAYED | LATE_DELIVERY | MEDIUM |
| SHP-TEST-003 | Failed delivery | FAILED_DELIVERY | FAILED_DELIVERY | HIGH |
| SHP-TEST-004 | Missing customer information | IN_TRANSIT | MISSING_INFORMATION | MEDIUM |
| SHP-TEST-005 | Damaged package | EXCEPTION | DAMAGED | HIGH |
| SHP-TEST-006 | Customs hold | PENDING | CUSTOMS_HOLD | HIGH |

All six scenarios successfully passed through the production webhook and were recorded in the Google Sheets operational tracker.

---

# Client Operational View

Google Sheets is used as the operational tracking layer for the prototype.

The operations team can review:

- Current shipment status
- Shipment exceptions
- Exception priority
- AI confidence
- Human-review requirements
- Classification reason
- Notification result
- Processing status

This provides a simple operational view without requiring a separate dashboard application.

---

# Notification Architecture

The current notification stage uses a simulated WhatsApp implementation.

```text
Build Shipment Notification
          ↓
Send Shipment Notification
          ↓
WHATSAPP_SIMULATION
          ↓
Capture Notification Result
```

No real customer WhatsApp message is sent during portfolio testing.

For production deployment, the notification stage can be connected to the WhatsApp Business API.

---

# Technology Stack

- n8n
- Google Gemini
- Google Sheets
- JavaScript
- Webhooks
- Structured Output Parser
- WhatsApp notification simulation

## Optional Production Integration

- WhatsApp Business API
- Shipment tracking API
- Transportation Management System
- Logistics management platform
- Production database

---

# Portfolio Scope

This project demonstrates practical experience with:

- n8n workflow automation
- Webhook-based integrations
- AI/LLM integration
- Google Gemini
- Structured AI output
- JavaScript data transformation
- Shipment tracking automation
- Exception classification
- Priority classification
- Human-review workflows
- Notification workflow design
- Google Sheets integration
- API response handling
- Error handling
- Testing and troubleshooting
- Logistics operations automation

---

# Current Project Status

**Working Prototype**

The workflow has successfully processed six logistics test scenarios through the production webhook.

The AI classifier successfully identified normal shipment activity and multiple exception conditions.

Google Sheets successfully recorded the processed shipment events and classification results.

The WhatsApp notification layer is currently simulated for portfolio testing.

---

# Future Enhancements

Potential production enhancements include:

- Real WhatsApp Business API integration
- Live shipment tracking API
- Duplicate-event protection
- Production database integration
- Automated escalation routing
- Slack or Microsoft Teams alerts
- Operations dashboard
- Shipment SLA monitoring
- Automated customer communication
- Historical shipment analytics
- Advanced exception prioritization

---

# Project Summary

This project demonstrates how an AI-powered workflow can transform incoming shipment events into structured operational decisions.

The system combines:

**Event Intake**
→ **Data Normalization**
→ **AI Classification**
→ **Validation**
→ **Notification Preparation**
→ **Notification Delivery**
→ **Operational Tracking**
→ **API Response**
→ **Error Handling**

The result is a logistics-focused automation prototype designed to assist teams in monitoring shipment status, identifying exceptions, prioritizing operational issues, and maintaining a centralized shipment operations tracker.
