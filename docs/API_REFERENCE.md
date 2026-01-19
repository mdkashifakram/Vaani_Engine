# VAANI Engine - API Reference

**Complete API documentation for VAANI Engine REST and WebSocket endpoints**

---

## Table of Contents

1. [API Overview](#api-overview)
2. [Authentication](#authentication)
3. [Voice Endpoints](#voice-endpoints)
4. [Appointment Endpoints](#appointment-endpoints)
5. [Patient Endpoints](#patient-endpoints)
6. [Doctor Endpoints](#doctor-endpoints)
7. [Call Log Endpoints](#call-log-endpoints)
8. [Admin Endpoints](#admin-endpoints)
9. [WebSocket API](#websocket-api)
10. [Error Handling](#error-handling)
11. [Rate Limiting](#rate-limiting)
12. [Webhook Events](#webhook-events)

---

## 1. API Overview

### 1.1 Base URL

```
Development: http://localhost:8000
Staging: https://staging-api.vaani-engine.com
Production: https://api.vaani-engine.com
```

### 1.2 API Versioning

All API endpoints are versioned under `/api/v1/`. Future versions will use `/api/v2/`, etc.

### 1.3 Request/Response Format

**Content-Type**: `application/json`

**Request Headers:**
```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>  (for protected endpoints)
```

**Standard Response Format:**
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation completed successfully",
  "timestamp": "2026-01-20T10:00:00Z"
}
```

**Error Response Format:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "field": "phone",
      "issue": "Invalid phone number format"
    }
  },
  "timestamp": "2026-01-20T10:00:00Z"
}
```

---

## 2. Authentication

### 2.1 JWT Token Authentication

**Obtain Access Token:**

```http
POST /api/v1/auth/login
```

**Request Body:**
```json
{
  "username": "admin@clinic.com",
  "password": "secure_password"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "bearer",
    "expires_in": 3600,
    "user": {
      "id": 1,
      "username": "admin@clinic.com",
      "role": "admin"
    }
  }
}
```

**Using the Token:**
```http
GET /api/v1/admin/dashboard
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 2.2 API Key Authentication

For webhook endpoints (Twilio callbacks):

```http
POST /api/v1/voice/incoming
X-Twilio-Signature: <signature>
```

Signature validation is performed automatically.

---

## 3. Voice Endpoints

### 3.1 Incoming Call Webhook

**Endpoint:** `POST /api/v1/voice/incoming`

**Description:** Twilio webhook called when a patient calls the clinic number.

**Request Body (from Twilio):**
```
CallSid=CA1234567890abcdef
AccountSid=AC1234567890abcdef
From=+919876543210
To=+911234567890
CallStatus=ringing
Direction=inbound
```

**Response (TwiML):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <Say voice="alice">Hello! Please wait while we connect you to our AI assistant.</Say>
    <Connect>
        <Stream url="wss://api.vaani-engine.com/api/v1/voice/stream/CA1234567890abcdef">
            <Parameter name="patient_phone" value="+919876543210"/>
        </Stream>
    </Connect>
</Response>
```

**Status Codes:**
- `200 OK`: TwiML response returned
- `500 Internal Server Error`: Server error

### 3.2 Call Status Callback

**Endpoint:** `POST /api/v1/voice/status`

**Description:** Receives call status updates from Twilio.

**Request Body:**
```
CallSid=CA1234567890abcdef
CallStatus=completed
CallDuration=125
From=+919876543210
To=+911234567890
```

**Response:**
```json
{
  "success": true,
  "message": "Call status updated"
}
```

**Call Status Values:**
- `queued`: Call is queued
- `ringing`: Phone is ringing
- `in-progress`: Call is active
- `completed`: Call ended normally
- `busy`: Line was busy
- `failed`: Call failed
- `no-answer`: No one answered

### 3.3 WebSocket Stream

**Endpoint:** `WS /api/v1/voice/stream/{call_sid}`

**Description:** Real-time bidirectional audio streaming for voice calls.

See [WebSocket API](#websocket-api) section for detailed protocol.

---

## 4. Appointment Endpoints

### 4.1 List Appointments

**Endpoint:** `GET /api/v1/appointments`

**Description:** Retrieve list of appointments with optional filters.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `patient_id` | integer | No | Filter by patient ID |
| `doctor_id` | integer | No | Filter by doctor ID |
| `status` | string | No | Filter by status (scheduled, completed, cancelled) |
| `start_date` | date | No | Filter appointments after this date (YYYY-MM-DD) |
| `end_date` | date | No | Filter appointments before this date (YYYY-MM-DD) |
| `page` | integer | No | Page number (default: 1) |
| `limit` | integer | No | Items per page (default: 20, max: 100) |

**Example Request:**
```http
GET /api/v1/appointments?doctor_id=5&start_date=2026-01-20&limit=10
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "appointments": [
      {
        "id": 123,
        "patient_id": 45,
        "patient_name": "Rajesh Kumar",
        "patient_phone": "+919876543210",
        "doctor_id": 5,
        "doctor_name": "Dr. Priya Sharma",
        "scheduled_time": "2026-01-25T14:30:00Z",
        "duration_minutes": 30,
        "status": "scheduled",
        "reason": "Annual checkup",
        "created_at": "2026-01-20T10:15:00Z",
        "updated_at": "2026-01-20T10:15:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 45,
      "total_pages": 5
    }
  }
}
```

### 4.2 Get Appointment Details

**Endpoint:** `GET /api/v1/appointments/{appointment_id}`

**Description:** Retrieve details of a specific appointment.

**Path Parameters:**
- `appointment_id` (integer): Unique appointment identifier

**Example Request:**
```http
GET /api/v1/appointments/123
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 123,
    "patient": {
      "id": 45,
      "name": "Rajesh Kumar",
      "phone": "+919876543210",
      "date_of_birth": "1985-05-15",
      "language_preference": "hi"
    },
    "doctor": {
      "id": 5,
      "name": "Dr. Priya Sharma",
      "specialization": "General Physician",
      "phone": "+911234567890"
    },
    "scheduled_time": "2026-01-25T14:30:00Z",
    "duration_minutes": 30,
    "status": "scheduled",
    "reason": "Annual checkup",
    "notes": "Patient requested early morning slot",
    "call_sid": "CA1234567890abcdef",
    "created_at": "2026-01-20T10:15:00Z",
    "updated_at": "2026-01-20T10:15:00Z"
  }
}
```

### 4.3 Create Appointment

**Endpoint:** `POST /api/v1/appointments`

**Description:** Create a new appointment.

**Request Body:**
```json
{
  "patient_id": 45,
  "doctor_id": 5,
  "scheduled_time": "2026-01-25T14:30:00Z",
  "duration_minutes": 30,
  "reason": "Annual checkup",
  "notes": "Patient prefers Hindi"
}
```

**Validation Rules:**
- `scheduled_time` must be in the future
- `duration_minutes` must be 15, 30, 45, or 60
- No overlapping appointments for the same doctor
- Doctor must be available at requested time

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 124,
    "patient_id": 45,
    "doctor_id": 5,
    "scheduled_time": "2026-01-25T14:30:00Z",
    "duration_minutes": 30,
    "status": "scheduled",
    "reason": "Annual checkup",
    "created_at": "2026-01-20T10:30:00Z"
  },
  "message": "Appointment created successfully. SMS confirmation sent."
}
```

**Error Responses:**
- `400 Bad Request`: Invalid input data
- `404 Not Found`: Patient or doctor not found
- `409 Conflict`: Time slot already booked

### 4.4 Update Appointment

**Endpoint:** `PUT /api/v1/appointments/{appointment_id}`

**Description:** Update an existing appointment.

**Request Body:**
```json
{
  "scheduled_time": "2026-01-25T15:00:00Z",
  "reason": "Follow-up checkup",
  "notes": "Rescheduled by patient request"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 123,
    "scheduled_time": "2026-01-25T15:00:00Z",
    "reason": "Follow-up checkup",
    "updated_at": "2026-01-20T11:00:00Z"
  },
  "message": "Appointment updated. SMS notification sent to patient."
}
```

### 4.5 Cancel Appointment

**Endpoint:** `DELETE /api/v1/appointments/{appointment_id}`

**Description:** Cancel an appointment.

**Query Parameters:**
- `reason` (string, optional): Cancellation reason

**Example Request:**
```http
DELETE /api/v1/appointments/123?reason=Patient%20unavailable
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "message": "Appointment cancelled successfully. SMS notification sent."
}
```

### 4.6 Get Available Slots

**Endpoint:** `GET /api/v1/appointments/available-slots`

**Description:** Get available appointment slots for a doctor.

**Query Parameters:**
- `doctor_id` (integer, required): Doctor ID
- `date` (date, required): Date to check (YYYY-MM-DD)
- `duration_minutes` (integer, optional): Appointment duration (default: 30)

**Example Request:**
```http
GET /api/v1/appointments/available-slots?doctor_id=5&date=2026-01-25&duration_minutes=30
```

**Response:**
```json
{
  "success": true,
  "data": {
    "doctor_id": 5,
    "doctor_name": "Dr. Priya Sharma",
    "date": "2026-01-25",
    "available_slots": [
      {
        "start_time": "2026-01-25T09:00:00Z",
        "end_time": "2026-01-25T09:30:00Z"
      },
      {
        "start_time": "2026-01-25T10:00:00Z",
        "end_time": "2026-01-25T10:30:00Z"
      },
      {
        "start_time": "2026-01-25T14:00:00Z",
        "end_time": "2026-01-25T14:30:00Z"
      }
    ],
    "total_slots": 3
  }
}
```

---

## 5. Patient Endpoints

### 5.1 List Patients

**Endpoint:** `GET /api/v1/patients`

**Description:** Retrieve list of patients.

**Query Parameters:**
- `search` (string): Search by name or phone
- `language_preference` (string): Filter by language (hi, en, hi-en)
- `page` (integer): Page number
- `limit` (integer): Items per page

**Example Request:**
```http
GET /api/v1/patients?search=rajesh&limit=10
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "patients": [
      {
        "id": 45,
        "phone": "+919876543210",
        "name": "Rajesh Kumar",
        "date_of_birth": "1985-05-15",
        "age": 40,
        "language_preference": "hi",
        "total_appointments": 12,
        "last_appointment": "2025-12-15T10:00:00Z",
        "created_at": "2024-01-10T09:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 1,
      "total_pages": 1
    }
  }
}
```

### 5.2 Get Patient Details

**Endpoint:** `GET /api/v1/patients/{patient_id}`

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 45,
    "phone": "+919876543210",
    "name": "Rajesh Kumar",
    "date_of_birth": "1985-05-15",
    "age": 40,
    "language_preference": "hi",
    "created_at": "2024-01-10T09:00:00Z",
    "appointment_history": [
      {
        "id": 120,
        "doctor_name": "Dr. Priya Sharma",
        "scheduled_time": "2025-12-15T10:00:00Z",
        "status": "completed",
        "reason": "Checkup"
      }
    ],
    "total_appointments": 12,
    "upcoming_appointments": 1
  }
}
```

### 5.3 Create Patient

**Endpoint:** `POST /api/v1/patients`

**Request Body:**
```json
{
  "phone": "+919876543210",
  "name": "Rajesh Kumar",
  "date_of_birth": "1985-05-15",
  "language_preference": "hi"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 46,
    "phone": "+919876543210",
    "name": "Rajesh Kumar",
    "created_at": "2026-01-20T12:00:00Z"
  }
}
```

### 5.4 Update Patient

**Endpoint:** `PUT /api/v1/patients/{patient_id}`

**Request Body:**
```json
{
  "name": "Rajesh Kumar Sharma",
  "language_preference": "hi-en"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 45,
    "name": "Rajesh Kumar Sharma",
    "language_preference": "hi-en",
    "updated_at": "2026-01-20T12:30:00Z"
  }
}
```

---

## 6. Doctor Endpoints

### 6.1 List Doctors

**Endpoint:** `GET /api/v1/doctors`

**Query Parameters:**
- `specialization` (string): Filter by specialization
- `available` (boolean): Show only available doctors

**Response:**
```json
{
  "success": true,
  "data": {
    "doctors": [
      {
        "id": 5,
        "name": "Dr. Priya Sharma",
        "specialization": "General Physician",
        "phone": "+911234567890",
        "email": "priya.sharma@clinic.com",
        "available": true,
        "working_hours": {
          "monday": ["09:00-13:00", "14:00-18:00"],
          "tuesday": ["09:00-13:00", "14:00-18:00"],
          "wednesday": ["09:00-13:00"],
          "thursday": ["09:00-13:00", "14:00-18:00"],
          "friday": ["09:00-13:00", "14:00-18:00"],
          "saturday": ["09:00-13:00"],
          "sunday": []
        }
      }
    ]
  }
}
```

### 6.2 Get Doctor Schedule

**Endpoint:** `GET /api/v1/doctors/{doctor_id}/schedule`

**Query Parameters:**
- `date` (date): Specific date (YYYY-MM-DD)
- `week` (boolean): Show full week schedule

**Response:**
```json
{
  "success": true,
  "data": {
    "doctor_id": 5,
    "doctor_name": "Dr. Priya Sharma",
    "date": "2026-01-25",
    "appointments": [
      {
        "id": 120,
        "patient_name": "Rajesh Kumar",
        "start_time": "09:00",
        "end_time": "09:30",
        "status": "scheduled"
      },
      {
        "id": 121,
        "patient_name": "Priya Patel",
        "start_time": "10:00",
        "end_time": "10:30",
        "status": "scheduled"
      }
    ],
    "total_appointments": 8,
    "available_slots": 12
  }
}
```

---

## 7. Call Log Endpoints

### 7.1 List Call Logs

**Endpoint:** `GET /api/v1/call-logs`

**Query Parameters:**
- `patient_id` (integer): Filter by patient
- `direction` (string): inbound or outbound
- `start_date` (date): Filter from date
- `end_date` (date): Filter to date

**Response:**
```json
{
  "success": true,
  "data": {
    "call_logs": [
      {
        "id": 500,
        "call_sid": "CA1234567890abcdef",
        "patient_id": 45,
        "patient_name": "Rajesh Kumar",
        "direction": "inbound",
        "from_number": "+919876543210",
        "to_number": "+911234567890",
        "duration_seconds": 125,
        "status": "completed",
        "transcript": "Patient called to book appointment...",
        "summary": "Appointment booked for Jan 25 at 2:30 PM",
        "sentiment": "positive",
        "created_at": "2026-01-20T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "total_pages": 8
    }
  }
}
```

### 7.2 Get Call Log Details

**Endpoint:** `GET /api/v1/call-logs/{call_log_id}`

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 500,
    "call_sid": "CA1234567890abcdef",
    "patient": {
      "id": 45,
      "name": "Rajesh Kumar",
      "phone": "+919876543210"
    },
    "direction": "inbound",
    "duration_seconds": 125,
    "status": "completed",
    "transcript": "Full conversation transcript...",
    "summary": "Patient called to book appointment for annual checkup. Scheduled for Jan 25 at 2:30 PM with Dr. Sharma.",
    "sentiment": "positive",
    "intents_detected": [
      "book_appointment",
      "ask_doctor_availability"
    ],
    "appointment_created": 123,
    "created_at": "2026-01-20T10:00:00Z"
  }
}
```

---

## 8. Admin Endpoints

### 8.1 Dashboard Metrics

**Endpoint:** `GET /api/v1/admin/dashboard`

**Authentication:** Required (Admin role)

**Response:**
```json
{
  "success": true,
  "data": {
    "today": {
      "total_calls": 45,
      "completed_calls": 42,
      "failed_calls": 3,
      "appointments_booked": 28,
      "average_call_duration": 98
    },
    "active_sessions": 5,
    "voice_engine_status": "healthy",
    "average_latency_ms": 720,
    "system_health": {
      "database": "connected",
      "redis": "connected",
      "twilio": "connected",
      "deepgram": "connected",
      "cartesia": "connected"
    }
  }
}
```

### 8.2 System Health

**Endpoint:** `GET /api/v1/admin/health`

**Authentication:** Not required (public endpoint)

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2026-01-20T10:00:00Z",
  "version": "2.0.0",
  "components": {
    "database": {
      "status": "healthy",
      "response_time_ms": 5
    },
    "redis": {
      "status": "healthy",
      "response_time_ms": 2
    },
    "voice_providers": {
      "deepgram": "healthy",
      "cartesia": "healthy"
    }
  }
}
```

### 8.3 Analytics Data

**Endpoint:** `GET /api/v1/admin/analytics`

**Query Parameters:**
- `start_date` (date): Start date for analytics
- `end_date` (date): End date for analytics
- `metric` (string): Specific metric (calls, appointments, latency)

**Response:**
```json
{
  "success": true,
  "data": {
    "period": {
      "start": "2026-01-01",
      "end": "2026-01-20"
    },
    "calls": {
      "total": 850,
      "completed": 820,
      "failed": 30,
      "average_duration_seconds": 105
    },
    "appointments": {
      "total_booked": 520,
      "completed": 480,
      "cancelled": 25,
      "no_show": 15
    },
    "performance": {
      "average_latency_ms": 685,
      "p95_latency_ms": 950,
      "p99_latency_ms": 1200
    },
    "languages": {
      "hindi": 450,
      "english": 250,
      "hinglish": 150
    }
  }
}
```

---

## 9. WebSocket API

### 9.1 Connection Protocol

**Endpoint:** `wss://api.vaani-engine.com/api/v1/voice/stream/{call_sid}`

**Connection Lifecycle:**

1. **Client Initiates Connection**
```
→ WebSocket Handshake
← 101 Switching Protocols
```

2. **Start Event (from Twilio)**
```json
{
  "event": "start",
  "streamSid": "MZ1234567890abcdef",
  "callSid": "CA1234567890abcdef",
  "tracks": ["inbound"],
  "mediaFormat": {
    "encoding": "audio/x-mulaw",
    "sampleRate": 8000,
    "channels": 1
  }
}
```

3. **Media Events (bidirectional)**

**Inbound (Patient → Server):**
```json
{
  "event": "media",
  "streamSid": "MZ1234567890abcdef",
  "media": {
    "track": "inbound",
    "chunk": "2",
    "timestamp": "5",
    "payload": "base64-encoded-mulaw-audio"
  }
}
```

**Outbound (Server → Patient):**
```json
{
  "event": "media",
  "streamSid": "MZ1234567890abcdef",
  "media": {
    "track": "outbound",
    "payload": "base64-encoded-mulaw-audio"
  }
}
```

4. **Mark Events (tracking playback)**
```json
{
  "event": "mark",
  "streamSid": "MZ1234567890abcdef",
  "mark": {
    "name": "response_complete"
  }
}
```

5. **Stop Event**
```json
{
  "event": "stop",
  "streamSid": "MZ1234567890abcdef"
}
```

### 9.2 Custom Events

**Session State Update:**
```json
{
  "event": "session_state",
  "state": "listening",
  "timestamp": "2026-01-20T10:00:15Z"
}
```

**States:**
- `idle`: Connected, waiting for audio
- `listening`: Receiving patient speech
- `processing`: Transcribing audio
- `thinking`: Generating LLM response
- `speaking`: Playing AI response

**Transcript Event:**
```json
{
  "event": "transcript",
  "direction": "inbound",
  "text": "I want to book an appointment",
  "language": "en",
  "confidence": 0.95,
  "timestamp": "2026-01-20T10:00:20Z"
}
```

---

## 10. Error Handling

### 10.1 Error Codes

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| 400 | `VALIDATION_ERROR` | Invalid request data |
| 401 | `UNAUTHORIZED` | Missing or invalid authentication |
| 403 | `FORBIDDEN` | Insufficient permissions |
| 404 | `NOT_FOUND` | Resource not found |
| 409 | `CONFLICT` | Resource conflict (e.g., duplicate) |
| 422 | `UNPROCESSABLE_ENTITY` | Semantic error in request |
| 429 | `RATE_LIMIT_EXCEEDED` | Too many requests |
| 500 | `INTERNAL_SERVER_ERROR` | Server error |
| 503 | `SERVICE_UNAVAILABLE` | Service temporarily unavailable |

### 10.2 Error Response Examples

**Validation Error:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "phone": "Phone number must be in E.164 format (+919876543210)",
      "scheduled_time": "Scheduled time must be in the future"
    }
  },
  "timestamp": "2026-01-20T10:00:00Z"
}
```

**Resource Not Found:**
```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Appointment not found",
    "details": {
      "resource": "appointment",
      "id": 999
    }
  },
  "timestamp": "2026-01-20T10:00:00Z"
}
```

**Conflict Error:**
```json
{
  "success": false,
  "error": {
    "code": "CONFLICT",
    "message": "Appointment time slot already booked",
    "details": {
      "doctor_id": 5,
      "requested_time": "2026-01-25T14:30:00Z",
      "conflicting_appointment_id": 123
    }
  },
  "timestamp": "2026-01-20T10:00:00Z"
}
```

---

## 11. Rate Limiting

### 11.1 Rate Limit Rules

| Endpoint Category | Rate Limit | Window |
|------------------|------------|--------|
| Voice Webhooks | No limit | N/A |
| Authentication | 5 requests | 1 minute |
| Appointment API | 60 requests | 1 minute |
| Admin API | 120 requests | 1 minute |
| Public API | 30 requests | 1 minute |

### 11.2 Rate Limit Headers

**Response Headers:**
```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1642680000
```

**Rate Limit Exceeded Response:**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "details": {
      "limit": 60,
      "window_seconds": 60,
      "retry_after": 35
    }
  }
}
```

---

## 12. Webhook Events

### 12.1 Webhook Configuration

VAANI can send webhooks to external systems for event notifications.

**Configure Webhooks:**
```http
POST /api/v1/admin/webhooks
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "url": "https://your-app.com/webhooks/vaani",
  "events": [
    "appointment.created",
    "appointment.updated",
    "appointment.cancelled",
    "call.completed"
  ],
  "secret": "your-webhook-secret"
}
```

### 12.2 Webhook Payload Examples

**Appointment Created:**
```json
{
  "event": "appointment.created",
  "timestamp": "2026-01-20T10:00:00Z",
  "data": {
    "id": 123,
    "patient_id": 45,
    "doctor_id": 5,
    "scheduled_time": "2026-01-25T14:30:00Z",
    "status": "scheduled"
  }
}
```

**Call Completed:**
```json
{
  "event": "call.completed",
  "timestamp": "2026-01-20T10:05:00Z",
  "data": {
    "call_sid": "CA1234567890abcdef",
    "patient_id": 45,
    "duration_seconds": 125,
    "summary": "Appointment booked successfully",
    "appointment_id": 123
  }
}
```

### 12.3 Webhook Signature Verification

**Signature Header:**
```http
X-VAANI-Signature: sha256=<signature>
```

**Verification (Python):**
```python
import hmac
import hashlib

def verify_webhook_signature(payload: bytes, signature: str, secret: str) -> bool:
    expected_signature = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(f"sha256={expected_signature}", signature)
```

---

## API Client Examples

### Python Example

```python
import httpx
import asyncio

class VAANIClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url
        self.headers = {"Authorization": f"Bearer {api_key}"}

    async def create_appointment(self, appointment_data: dict):
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{self.base_url}/api/v1/appointments",
                json=appointment_data,
                headers=self.headers
            )
            response.raise_for_status()
            return response.json()

# Usage
async def main():
    client = VAANIClient(
        base_url="https://api.vaani-engine.com",
        api_key="your_api_key"
    )

    appointment = await client.create_appointment({
        "patient_id": 45,
        "doctor_id": 5,
        "scheduled_time": "2026-01-25T14:30:00Z",
        "reason": "Checkup"
    })

    print(f"Appointment created: {appointment['data']['id']}")

asyncio.run(main())
```

### JavaScript Example

```javascript
const axios = require('axios');

class VAANIClient {
  constructor(baseUrl, apiKey) {
    this.baseUrl = baseUrl;
    this.headers = { Authorization: `Bearer ${apiKey}` };
  }

  async createAppointment(appointmentData) {
    const response = await axios.post(
      `${this.baseUrl}/api/v1/appointments`,
      appointmentData,
      { headers: this.headers }
    );
    return response.data;
  }
}

// Usage
const client = new VAANIClient(
  'https://api.vaani-engine.com',
  'your_api_key'
);

client.createAppointment({
  patient_id: 45,
  doctor_id: 5,
  scheduled_time: '2026-01-25T14:30:00Z',
  reason: 'Checkup'
}).then(data => {
  console.log('Appointment created:', data.data.id);
});
```

---

## Interactive API Documentation

Visit the interactive API documentation at:

- **Swagger UI**: `https://api.vaani-engine.com/docs`
- **ReDoc**: `https://api.vaani-engine.com/redoc`
- **OpenAPI Schema**: `https://api.vaani-engine.com/openapi.json`

---

**Document Version**: 1.0
**Last Updated**: 2026-01-20
**Maintained By**: VAANI API Team
