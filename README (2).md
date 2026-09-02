# Nightwing Securities

> A late-night emergency response platform that connects people who need urgent assistance with verified nearby responders.

## Project Overview

Nightwing Securities is a full-stack web application inspired by location-based platforms such as Uber, but designed specifically for late-night safety incidents.

A user can create an account, trigger an emergency signal, share their location, and request assistance from an available responder. Responders can create profiles, set their availability, receive nearby incidents, and update the incident status. Security operators can monitor active incidents through an administrative dashboard.

The long-term vision is to create a platform that could support private security providers and, subject to appropriate partnerships, integrate with official emergency-response organisations.

> **Important:** This project is an academic and personal software project. It is not a replacement for emergency services. In an immediate life-threatening emergency, contact the relevant emergency number for your country.

## Objectives

- Provide a simple and fast way for users to request late-night assistance.
- Connect users with nearby, verified responders.
- Provide live incident status and location updates.
- Create secure communication between users and responders.
- Give security operators an overview of active incidents.
- Demonstrate full-stack engineering, real-time systems, authentication, databases, and secure software design.

## Planned User Roles

### User

- Register and log in.
- Manage a personal profile.
- Add trusted emergency contacts.
- Trigger an SOS request.
- Select an incident type and severity.
- Share their location during an active incident.
- View the assigned responder and estimated arrival.
- Communicate with the responder.
- Cancel or complete an incident.
- Submit feedback after assistance.

### Responder

- Register as a responder.
- Create a responder profile.
- Submit verification information.
- Set availability status.
- View nearby incidents.
- Accept an available incident.
- Navigate to the user’s location.
- Update the incident status.
- Communicate securely with the user.
- Submit an incident report after completion.

### Security Operator

- Sign in to an administrative dashboard.
- View active and historical incidents.
- Monitor incident locations and statuses.
- Manage users and responders.
- Review responder verification information.
- Reassign or escalate incidents.
- Suspend accounts where necessary.
- View operational metrics and response times.

## Core Incident Lifecycle

```text
CREATED
   ↓
SEARCHING_FOR_RESPONDER
   ↓
ASSIGNED
   ↓
EN_ROUTE
   ↓
ON_SCENE
   ↓
COMPLETED
```

Alternative outcomes include:

```text
CREATED → CANCELLED
CREATED → ESCALATED
ASSIGNED → UNASSIGNED → SEARCHING_FOR_RESPONDER
```

Every status change should be recorded with the actor, timestamp, and optional explanation.

## MVP Scope

The first working version should focus on a small but complete workflow:

1. A user creates an account.
2. A responder creates an account and is marked as verified by an administrator.
3. The responder changes their availability to online.
4. The user creates an SOS incident containing their location and incident type.
5. The responder sees the nearby incident.
6. The responder accepts the incident.
7. Both users see the incident status.
8. The responder marks the incident as completed.
9. The user submits a rating and feedback.
10. An administrator can inspect the incident in the dashboard.

Features such as payments, native mobile applications, police integration, video streaming, machine learning, and SMS fallback should be treated as later phases rather than initial MVP requirements.

## Proposed Technology Stack

| Layer | Proposed Technology |
|---|---|
| Frontend | React, JavaScript or TypeScript, HTML, CSS |
| Backend | Python, Flask, Flask-RESTful or Flask blueprints |
| Database | PostgreSQL for production, SQLite for early development |
| ORM | SQLAlchemy and Flask-Migrate |
| Authentication | Secure sessions or JWT-based authentication |
| Real-time updates | WebSockets using Flask-SocketIO or a dedicated real-time service |
| Maps and location | OpenStreetMap/Leaflet or a commercial mapping API |
| Notifications | Email initially; SMS as a later integration |
| Testing | Pytest, React Testing Library, and API integration tests |
| Deployment | Docker, a cloud hosting provider, and managed PostgreSQL |
| Version control | Git and GitHub |

The final technology choices can change during implementation. The priority is to keep the first version understandable, testable, and secure.

## High-Level Architecture

```text
┌─────────────────┐      HTTPS / WebSocket      ┌────────────────────┐
│  User Frontend  │ ─────────────────────────▶ │                    │
└─────────────────┘                            │                    │
                                              │   Flask Backend    │
┌─────────────────┐      HTTPS / WebSocket      │                    │
│ Responder UI    │ ─────────────────────────▶ │                    │
└─────────────────┘                            │                    │
                                              └─────────┬──────────┘
┌─────────────────┐      HTTPS                    │
│ Admin Dashboard │ ─────────────────────────────┘
└─────────────────┘                                      │
                                                        ▼
                                              ┌────────────────────┐
                                              │ PostgreSQL Database │
                                              └────────────────────┘
                                                        │
                                                        ▼
                                              ┌────────────────────┐
                                              │ External Services  │
                                              │ Maps, email, SMS   │
                                              └────────────────────┘
```

## Main Functional Requirements

### Authentication

- The system must allow users and responders to register.
- The system must support login and logout.
- Passwords must never be stored as plain text.
- Passwords must be securely hashed before storage.
- Protected routes must only be accessible to authenticated users.
- Role-based access control must restrict user, responder, and administrator functionality.
- The system should support email or phone verification in a later phase.

### User Profiles

- Users must be able to update their name, profile image, phone number, and emergency contacts.
- Responders must be able to provide their service area, availability, vehicle details where relevant, and verification information.
- Administrators must be able to approve or reject responder verification.
- Sensitive verification documents must not be publicly accessible.

### SOS Requests

- The user must be able to trigger an SOS request from the main application screen.
- The request must contain a timestamp and the user’s latest available location.
- The user should be able to select an incident type.
- The system should support a silent alert mode where appropriate.
- The system must show the user whether the request is searching, assigned, active, completed, cancelled, or escalated.

### Responder Matching

- Only verified and available responders should receive new incidents.
- The system should prioritise nearby responders.
- A responder must be able to accept an incident.
- Once accepted, an incident must not be accepted by another responder unless it is reassigned.
- Administrators must be able to reassign incidents.

### Live Updates

- The user and responder should receive live incident status updates.
- The responder should be able to share their current location while travelling to the incident.
- The user should be able to see the responder’s approximate location and status.
- The system should record relevant location updates only while an incident is active.

### Communication

- The user and responder should have access to an incident-specific message channel.
- Messages must be associated with the correct incident.
- Administrators should be able to access authorised incident records for operational review.
- Communication records should be protected and retained according to the project’s privacy policy.

### Notifications

- The system should notify responders when a suitable incident is available.
- The system should notify the user when a responder accepts the incident.
- The system should notify trusted contacts when the user enables that option.
- Email notifications can be implemented before SMS notifications.

## Proposed Database Model

### `users`

| Field | Type | Description |
|---|---|---|
| `id` | UUID / integer | Primary key |
| `name` | string | Display name |
| `email` | string | Unique login email |
| `phone` | string | Optional phone number |
| `password_hash` | string | Secure password hash |
| `role` | enum | `USER`, `RESPONDER`, or `ADMIN` |
| `is_active` | boolean | Account status |
| `created_at` | datetime | Account creation time |
| `updated_at` | datetime | Last update time |

### `responder_profiles`

| Field | Type | Description |
|---|---|---|
| `id` | UUID / integer | Primary key |
| `user_id` | foreign key | Related user account |
| `verification_status` | enum | `PENDING`, `APPROVED`, or `REJECTED` |
| `availability_status` | enum | `OFFLINE`, `ONLINE`, or `BUSY` |
| `service_area` | string | Area in which the responder operates |
| `vehicle_details` | JSON / text | Optional vehicle information |
| `created_at` | datetime | Profile creation time |

### `emergency_contacts`

| Field | Type | Description |
|---|---|---|
| `id` | UUID / integer | Primary key |
| `user_id` | foreign key | Contact owner |
| `name` | string | Contact name |
| `phone` | string | Contact phone number |
| `email` | string | Optional contact email |
| `is_primary` | boolean | Primary contact flag |

### `incidents`

| Field | Type | Description |
|---|---|---|
| `id` | UUID / integer | Primary key |
| `user_id` | foreign key | User requesting assistance |
| `responder_id` | foreign key | Assigned responder, nullable |
| `incident_type` | string | Type of emergency or safety concern |
| `severity` | enum | `LOW`, `MEDIUM`, `HIGH`, or `CRITICAL` |
| `status` | enum | Current incident state |
| `latitude` | decimal | Initial latitude |
| `longitude` | decimal | Initial longitude |
| `description` | text | Optional user description |
| `created_at` | datetime | Creation timestamp |
| `assigned_at` | datetime | Assignment timestamp |
| `completed_at` | datetime | Completion timestamp |

### `incident_events`

| Field | Type | Description |
|---|---|---|
| `id` | UUID / integer | Primary key |
| `incident_id` | foreign key | Related incident |
| `actor_id` | foreign key | User who caused the event |
| `event_type` | string | Status change, message, reassignment, or escalation |
| `metadata` | JSON | Additional event information |
| `created_at` | datetime | Event timestamp |

### `location_updates`

| Field | Type | Description |
|---|---|---|
| `id` | UUID / integer | Primary key |
| `incident_id` | foreign key | Related incident |
| `user_id` | foreign key | Location owner |
| `latitude` | decimal | Latitude |
| `longitude` | decimal | Longitude |
| `accuracy` | decimal | Location accuracy in metres |
| `created_at` | datetime | Collection timestamp |

### `ratings`

| Field | Type | Description |
|---|---|---|
| `id` | UUID / integer | Primary key |
| `incident_id` | foreign key | Completed incident |
| `user_id` | foreign key | Person giving rating |
| `responder_id` | foreign key | Rated responder |
| `score` | integer | Rating from 1 to 5 |
| `comment` | text | Optional feedback |
| `created_at` | datetime | Rating timestamp |

## API Outline

### Authentication

```text
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/auth/me
```

### User and Contacts

```text
GET    /api/users/me
PATCH  /api/users/me
GET    /api/users/me/contacts
POST   /api/users/me/contacts
PATCH  /api/users/me/contacts/{contact_id}
DELETE /api/users/me/contacts/{contact_id}
```

### Incidents

```text
POST   /api/incidents
GET    /api/incidents/{incident_id}
GET    /api/incidents/active
POST   /api/incidents/{incident_id}/cancel
POST   /api/incidents/{incident_id}/complete
POST   /api/incidents/{incident_id}/rate
```

### Responders

```text
GET    /api/responders/me
PATCH  /api/responders/me
POST   /api/responders/me/availability
GET    /api/responders/incidents/nearby
POST   /api/responders/incidents/{incident_id}/accept
POST   /api/responders/incidents/{incident_id}/status
POST   /api/responders/incidents/{incident_id}/location
```

### Administration

```text
GET    /api/admin/incidents
GET    /api/admin/incidents/{incident_id}
POST   /api/admin/incidents/{incident_id}/reassign
POST   /api/admin/incidents/{incident_id}/escalate
GET    /api/admin/users
POST   /api/admin/users/{user_id}/suspend
POST   /api/admin/responders/{user_id}/verify
```

### WebSocket Events

```text
incident.created
incident.assigned
incident.status_updated
incident.location_updated
incident.message_sent
incident.escalated
incident.completed
```

## Example API Request

### Create an incident

```http
POST /api/incidents
Content-Type: application/json
Authorization: Bearer <access-token>
```

```json
{
  "incident_type": "FEELING_UNSAFE",
  "severity": "HIGH",
  "latitude": 52.4068,
  "longitude": -1.5197,
  "description": "I am walking home and believe I am being followed",
  "notify_contacts": true
}
```

### Example response

```json
{
  "id": "incident_123",
  "status": "SEARCHING_FOR_RESPONDER",
  "incident_type": "FEELING_UNSAFE",
  "severity": "HIGH",
  "created_at": "2026-09-02T21:30:00Z"
}
```

## Security Requirements

- Use HTTPS in every deployed environment.
- Hash passwords using a modern password-hashing library such as Argon2 or bcrypt.
- Validate and sanitise all user input on the server.
- Use parameterised database queries through the ORM.
- Apply role-based authorisation to every protected endpoint.
- Do not expose exact location data to users who are not part of the incident or authorised operators.
- Use short-lived access tokens or secure server-side sessions.
- Protect against CSRF, XSS, SQL injection, brute-force login attempts, and insecure direct object references.
- Rate-limit registration, login, SOS creation, messaging, and location-update endpoints.
- Store audit events for account changes, incident access, assignments, escalations, and administrative actions.
- Encrypt sensitive data in transit and protect sensitive data at rest where appropriate.
- Avoid storing unnecessary location history after an incident has been closed.
- Define a retention and deletion policy before collecting real user data.
- Never present the platform as a guaranteed emergency service without the operational capability, legal arrangements, staffing, and testing to support that claim.

## Privacy and Safety Principles

- Collect only the information needed to provide the service.
- Explain clearly why location, contact, and incident information is collected.
- Request explicit consent before notifying trusted contacts or sharing optional media.
- Make location sharing visible to the user.
- Provide a way to stop active location sharing when the incident ends.
- Use approximate locations in responder discovery where exact precision is unnecessary.
- Provide accessible, low-light interfaces with clear status messages.
- Design for poor connectivity and clearly communicate when location or network data is stale.
- Include a prominent instruction to contact official emergency services for immediate life-threatening danger.

## Suggested Repository Structure

```text
nightwing-securities/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── auth/
│   │   ├── users/
│   │   ├── incidents/
│   │   ├── responders/
│   │   ├── admin/
│   │   ├── models/
│   │   ├── services/
│   │   └── extensions.py
│   ├── migrations/
│   ├── tests/
│   ├── config.py
│   ├── requirements.txt
│   └── run.py
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── hooks/
│   │   ├── context/
│   │   └── App.jsx
│   ├── public/
│   ├── package.json
│   └── vite.config.js
├── docs/
│   ├── requirements.md
│   ├── architecture.md
│   ├── api.md
│   └── threat-model.md
├── .env.example
├── .gitignore
├── docker-compose.yml
├── LICENSE
└── README.md
```

## Development Roadmap

### Phase 1: Planning and Design

- Confirm the target user journey and incident types.
- Create user stories and acceptance criteria.
- Produce wireframes for the user, responder, and admin experiences.
- Define the initial database schema and API contract.
- Create a threat model and privacy plan.

### Phase 2: Backend Foundation

- Set up the Flask application factory.
- Configure SQLAlchemy and database migrations.
- Implement registration, login, logout, and role-based access control.
- Add automated unit tests for authentication and permissions.

### Phase 3: User and Responder Workflows

- Build user and responder profiles.
- Implement responder verification and availability.
- Create and persist incidents.
- Implement nearby responder discovery and incident acceptance.

### Phase 4: Real-Time Experience

- Add WebSocket support.
- Implement live incident status updates.
- Add location updates during active incidents.
- Build incident-specific messaging.

### Phase 5: Admin Dashboard

- Display active incidents on a map and list view.
- Add responder and user management.
- Add reassignment and escalation controls.
- Add audit logs and basic response-time analytics.

### Phase 6: Testing and Deployment

- Add unit, integration, end-to-end, and security tests.
- Test race conditions when multiple responders accept the same incident.
- Test poor network connectivity and stale location data.
- Containerise the application.
- Deploy a staging environment.
- Conduct user acceptance testing with synthetic or consented test data only.

### Phase 7: Future Enhancements

- Progressive Web App or native mobile clients.
- Background location tracking with explicit consent.
- SMS fallback for limited connectivity.
- Payment support for commercial security services.
- Multi-organisation tenancy for security providers.
- Formal integrations with emergency services after legal and operational review.
- Advanced reporting and risk-area visualisation.

## Testing Strategy

### Unit Tests

- Password hashing and authentication helpers.
- Incident state-transition rules.
- Responder eligibility and matching logic.
- Permission checks.
- Validation functions.

### Integration Tests

- Registration and login flows.
- SOS creation and persistence.
- Responder acceptance and assignment locking.
- Incident cancellation and completion.
- Trusted-contact notification triggering.

### End-to-End Tests

- User creates an SOS and tracks a responder.
- Responder accepts and completes an incident.
- Administrator reassigns an incident.
- Unauthorized users cannot view another user’s incident.

### Security and Reliability Tests

- Rate-limit testing.
- Authentication and session-expiry testing.
- Input-validation testing.
- Access-control testing.
- Concurrent acceptance tests.
- Database backup and recovery tests.
- WebSocket reconnect tests.
- Poor-network and location-permission tests.

## MVP Acceptance Criteria

The MVP is complete when:

- A user can register, log in, and manage their basic profile.
- A responder can register, become approved by an administrator, and go online.
- A user can create an SOS incident containing an incident type and location.
- An eligible responder can view and accept the incident.
- The system prevents two responders from successfully claiming the same incident.
- The user can view assignment and status updates.
- The responder can update the incident through arrival and completion.
- The administrator can monitor, reassign, and review incidents.
- Incident actions are persisted and auditable.
- Automated tests cover the main user, responder, and administrator workflows.
- The application clearly communicates that it is not a replacement for official emergency services.

## Contribution Guidelines

1. Create a feature branch from `main`.
2. Use descriptive commit messages.
3. Add or update tests for every functional change.
4. Run the test suite before opening a pull request.
5. Update documentation when changing the API, database, or user workflow.
6. Do not commit secrets, API keys, private certificates, or real personal data.
7. Open a pull request describing the problem, implementation, testing, and known limitations.

## Local Development

### Prerequisites

- Python 3.11 or later.
- Node.js 20 or later.
- PostgreSQL, or Docker Desktop for the local database.
- Git.

### Setup

```bash
git clone https://github.com/<your-username>/nightwing-securities.git
cd nightwing-securities
```

Create the backend environment:

```bash
cd backend
python -m venv .venv
```

Activate it on macOS/Linux:

```bash
source .venv/bin/activate
```

Activate it on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install backend dependencies:

```bash
pip install -r requirements.txt
```

Create the frontend environment:

```bash
cd ../frontend
npm install
```

Create a local environment file based on `.env.example`, configure the database connection, and then run the backend and frontend in separate terminals.

```bash
# Backend
cd backend
flask run --debug
```

```bash
# Frontend
cd frontend
npm run dev
```

## Environment Variables

Never commit the real `.env` file. A typical configuration may include:

```text
FLASK_ENV=development
SECRET_KEY=replace-me
DATABASE_URL=postgresql://user:password@localhost:5432/nightwing
MAPS_API_KEY=replace-me
EMAIL_PROVIDER_API_KEY=replace-me
SMS_PROVIDER_API_KEY=replace-me
```

## Project Status

This project is currently in the specification and planning stage.

Planned initial milestone:

- Repository setup.
- Flask backend skeleton.
- React frontend skeleton.
- Database configuration.
- Authentication workflow.
- Initial user, responder, and admin roles.

## Licence

A licence has not yet been selected. Choose a licence before public distribution or accepting external contributions.

## Disclaimer

Nightwing Securities is a personal software-engineering project and should not be relied upon as the sole method of obtaining emergency assistance. The application must undergo legal, security, privacy, accessibility, reliability, and operational review before any real-world deployment.