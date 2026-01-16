# FleetSync Pro - Product Requirements Document (PRD)

**Version:** 1.0
**Date:** 2026-01-15
**Status:** Draft

---

# Part 1: Project Overview

## 1. Product Summary

| Item | Description |
|:-----|:------------|
| **Product Name** | FleetSync Pro |
| **Product Type** | Web + Mobile App + Admin Dashboard |
| **Target Users** | Logistics companies, delivery services, fleet operators |
| **Core Value** | AI-powered fleet management and delivery optimization platform |

## 2. User Types & Roles

| User Type | Primary Role | Key Permissions |
|:----------|:-------------|:----------------|
| **Driver** | Execute deliveries | View assignments, update status, submit POD, report issues |
| **Fleet Manager** | Manage vehicles & drivers | Monitor fleet, assign vehicles, view analytics, manage maintenance |
| **Dispatcher** | Coordinate deliveries | Create/assign tasks, optimize routes, track drivers real-time |
| **Admin** | System administration | Full access, user management, billing, configuration |

## 3. User Relationships

```
Admin (1)
    └── Fleet Manager (N)
            └── Driver (N)
                    ↕
            Dispatcher (N:N)
```

- **Driver ↔ Fleet Manager:** N:1 - Many drivers report to one fleet manager
- **Driver ↔ Dispatcher:** N:N - Dispatchers assign to multiple drivers, drivers receive from multiple dispatchers
- **Fleet Manager ↔ Admin:** N:1 - Multiple fleet managers per admin

## 4. Design Guidelines

| Aspect | Specification |
|:-------|:-------------|
| **Primary Color** | #1E3A5F (Navy Blue) |
| **Font Family** | Inter |
| **Design Reference** | Uber Freight (driver UI), Samsara (dashboard), Onfleet (route planning) |

## 5. Key Differentiators

1. AI-powered route optimization considering traffic, weather, and time windows
2. Unified platform for last-mile and long-haul fleet management
3. Real-time vehicle health monitoring with predictive maintenance
4. Seamless dispatcher shift handoff

---

# Part 2: Mobile App PRD

## 1. Driver App

### 1.1 Navigation Menu (Bottom Tab)

1. **Home** - Today's schedule and active delivery
2. **Deliveries** - Full delivery list and history
3. **Messages** - Chat with dispatcher/fleet manager
4. **Profile** - Settings, earnings, vehicle info

### 1.2 Page Architecture & Feature Specification

#### 1. Home Tab

**Main Page (Today's Dashboard)**

1) **Active Delivery Card**
   - Current delivery address and customer name
   - ETA display
   - Navigation button → Opens Google Maps
   - Status buttons: Arrived / Complete / Issue

2) **Today's Summary**
   - Total deliveries: [X] assigned, [Y] completed
   - Hours worked today
   - Break time remaining

3) **Quick Actions**
   - Start Route button
   - Report Issue button
   - Take Break button

**Delivery Detail Page**
- Full address with map preview
- Customer contact info (tap to call)
- Delivery notes/instructions
- Package details (size, quantity)
- Time window display
- Status update buttons
- Proof of Delivery section

#### 2. Deliveries Tab

**Main Page (Delivery List)**

1) **Filter Tabs**
   - Assigned | In Progress | Completed | Failed

2) **Delivery List Items**
   - Address (truncated)
   - Time window
   - Status indicator (color-coded)
   - Priority flag (if applicable)
   - Tap → Delivery Detail

3) **Search**
   - Search by address or customer name

**History Page**
- Past deliveries with POD thumbnails
- Filter by date range
- Earnings per delivery

#### 3. Messages Tab

**Main Page (Conversation List)**

1) **Conversation Items**
   - Contact name and role
   - Last message preview
   - Unread indicator
   - Timestamp

**Chat Page**
- Message bubbles (sent/received)
- Text input with send button
- Photo attachment option
- Real-time delivery status shared

#### 4. Profile Tab

**Main Page**

1) **Profile Section**
   - Profile photo
   - Name and driver ID
   - Contact info
   - Edit Profile button

2) **Earnings Section**
   - Today's earnings
   - This week's earnings
   - View Details → Earnings History

3) **Vehicle Section**
   - Assigned vehicle info
   - License plate
   - Last inspection date
   - Report Issue button

4) **Settings**
   - Notification preferences
   - Language selection
   - Help & Support
   - Log Out

### 1.3 Key Flows

#### Proof of Delivery Flow

```
1. Driver taps "Arrived" on Active Delivery
   └── System logs GPS + timestamp

2. Driver taps "Complete Delivery"
   └── Camera opens automatically

3. Driver takes photo of package
   └── Photo preview with retake option

4. Signature capture screen (if required)
   └── Customer signs on screen

5. Optional notes input
   └── Text field for delivery notes

6. Submit confirmation
   └── Success animation
   └── Next delivery auto-loads
```

#### Vehicle Issue Report Flow

```
1. Driver taps "Report Issue"
   └── Issue category selection (Engine, Tire, Brakes, Other)

2. Photo capture
   └── Take photo of issue

3. Severity selection
   └── Low / Medium / High / Critical

4. Description input
   └── Text field for details

5. Submit
   └── Fleet Manager notified immediately
   └── If Critical: Dispatcher prompted to reassign
```

---

## 2. Fleet Manager App (Web Responsive)

### 2.1 Navigation Menu (Side Menu)

1. **Dashboard** - Overview and alerts
2. **Fleet Map** - Real-time vehicle locations
3. **Vehicles** - Vehicle management
4. **Drivers** - Driver management
5. **Maintenance** - Schedule and history
6. **Reports** - Analytics and exports

### 2.2 Page Architecture & Feature Specification

#### 1. Dashboard

**Main Page**

1) **Fleet Status Cards**
   - Active vehicles count
   - Idle vehicles count
   - In maintenance count
   - Total deliveries today

2) **Alerts Panel**
   - Vehicle issues reported
   - Maintenance due alerts
   - Driver requests pending
   - Geofence violations

3) **Performance Chart**
   - Deliveries completed (7-day trend)
   - On-time delivery rate

#### 2. Fleet Map

**Main Page**

1) **Interactive Map**
   - Vehicle markers with status colors
   - Click marker → Vehicle popup card
   - Cluster view for many vehicles

2) **Filter Panel**
   - By status (Active, Idle, Issue)
   - By vehicle type
   - By assigned driver

3) **Vehicle List Sidebar**
   - Scrollable list synced with map
   - Quick status indicators

#### 3. Vehicles Page

**Main Page (Vehicle List)**

1) **Top Actions**
   - Add Vehicle button
   - Filter dropdown
   - Search by plate/ID

2) **Vehicle Table**
   - Vehicle ID
   - License plate
   - Type (Van/Truck/Motorcycle)
   - Assigned driver
   - Status
   - Last location
   - Actions (Edit, View, Assign)

**Vehicle Detail Page**
- Full vehicle information
- Maintenance history
- Fuel consumption chart
- Assigned driver history
- Current location on map

#### 4. Drivers Page

**Main Page (Driver List)**

1) **Driver Table**
   - Name and photo
   - Contact info
   - Assigned vehicle
   - Status (Active/On Break/Offline)
   - Today's deliveries
   - Performance score
   - Actions

**Driver Detail Page**
- Profile information
- Performance metrics
- Delivery history
- Hours worked (weekly view)
- Approve/Deny requests

#### 5. Maintenance Page

**Main Page**

1) **Calendar View**
   - Scheduled maintenance events
   - Overdue items highlighted

2) **Pending Issues List**
   - Driver-reported issues
   - Severity indicator
   - Create Ticket button

**Maintenance Ticket Page**
- Issue details
- Assigned mechanic/vendor
- Status updates
- Cost tracking
- Resolution notes

#### 6. Reports Page

**Main Page**

1) **Report Types**
   - Fleet Utilization
   - Fuel Consumption
   - Driver Performance
   - Maintenance Costs

2) **Export Options**
   - Date range picker
   - Format selection (PDF, Excel, CSV)
   - Download button

---

# Part 3: Dispatcher Web Dashboard PRD

## Navigation Menu (Top + Side)

**Top Bar:**
- Organization logo
- Search (deliveries, drivers)
- Notifications bell
- Profile dropdown

**Side Menu:**
1. **Dispatch Board** - Main delivery management
2. **Live Map** - Real-time tracking
3. **Deliveries** - All deliveries list
4. **Route Planner** - Optimization tool
5. **Customers** - Customer communication
6. **Handoff** - Shift handoff notes

## Page Architecture & Feature Specification

### 1. Dispatch Board

**Main Page (Kanban-style Board)**

1) **Column Headers**
   - Unassigned | Driver 1 | Driver 2 | Driver N...
   - Add Driver column button

2) **Delivery Cards**
   - Address snippet
   - Time window
   - Priority indicator
   - Package count
   - Drag handle

3) **Card Actions**
   - Click → Delivery detail modal
   - Drag → Assign to driver
   - Right-click → Priority menu

4) **Top Actions**
   - Import CSV button
   - Auto-assign button
   - Optimize All Routes button

5) **AI Suggestion Panel**
   - Suggested assignments
   - Accept / Dismiss buttons
   - Reasoning tooltip

### 2. Live Map

**Main Page**

1) **Map View**
   - Driver markers (real-time)
   - Active delivery routes
   - Traffic overlay toggle

2) **Driver Panel (Right)**
   - List of active drivers
   - Current delivery info
   - ETA to next stop
   - Message button

3) **Delivery Panel (Left)**
   - Unassigned deliveries count
   - Urgent deliveries highlight
   - Quick assign dropdown

### 3. Deliveries Page

**Main Page**

1) **Filter Bar**
   - Status filter
   - Date range
   - Driver filter
   - Priority filter

2) **Delivery Table**
   - ID
   - Customer
   - Address
   - Driver
   - Status
   - Time window
   - Actions

3) **Bulk Actions**
   - Select multiple
   - Bulk assign
   - Bulk cancel
   - Export selected

**Delivery Detail Modal**
- Full delivery info
- Status history timeline
- POD viewer (if completed)
- Customer communication log
- Edit / Cancel buttons

### 4. Route Planner

**Main Page**

1) **Driver Selection**
   - Dropdown to select driver
   - Or "New Route" option

2) **Delivery Selector**
   - Available deliveries list
   - Multi-select with checkboxes
   - Add to route button

3) **Route Preview**
   - Map with numbered stops
   - Ordered stop list (drag to reorder)
   - Total distance and time
   - Time window violations highlighted

4) **Optimization Panel**
   - Optimize button
   - Options: Fastest / Shortest / Balanced
   - Before/After comparison
   - Savings display

5) **Actions**
   - Save Route
   - Push to Driver
   - Clear Route

### 5. Customers Page

**Main Page**

1) **Communication Log**
   - Customer inquiries list
   - Status (Open / Resolved)
   - Assigned dispatcher

**Customer Detail**
- Customer info
- Delivery history
- Communication thread
- Reply input

### 6. Handoff Page

**Main Page**

1) **Current Shift Summary**
   - Deliveries completed
   - Pending issues
   - Driver notes

2) **Handoff Notes Editor**
   - Rich text editor
   - Tag drivers/deliveries
   - Priority items highlight

3) **Previous Handoffs**
   - History list
   - Read status

---

# Part 4: Admin Dashboard PRD

## Navigation Menu

1. **Overview** - System-wide dashboard
2. **Users** - User management
3. **Organizations** - Multi-tenant management
4. **Billing** - Subscription and invoices
5. **Integrations** - API keys and connections
6. **Settings** - System configuration
7. **Audit Logs** - Activity history

## Page Architecture & Feature Specification

### Dashboard (Overview)

**Card Components**
- Total users by role
- Active deliveries (real-time)
- System health status
- Revenue this month

**Charts**
- User growth trend
- Delivery volume by organization
- API usage metrics

### User Management

**Main Page**

1) **Top Area**
   - Create User button
   - Role filter dropdown
   - Organization filter
   - Search by name/email

2) **User Table**
   - Name
   - Email
   - Role
   - Organization
   - Status (Active/Suspended)
   - Last login
   - Actions

3) **Bulk Actions**
   - Bulk suspend
   - Bulk activate
   - Export users

**User Detail/Edit Modal**
- Edit user info
- Change role
- Reset password
- Suspend/Activate
- Activity log

### Billing Page

**Main Page**

1) **Revenue Summary**
   - MRR display
   - Active subscriptions count
   - Overdue invoices alert

2) **Subscription Table**
   - Organization
   - Plan
   - Status
   - Next billing date
   - Actions

**Invoice Detail**
- Line items
- Payment history
- Resend invoice
- Mark as paid

### Integrations Page

**Main Page**

1) **Active Integrations**
   - Google Maps: Connected ✓
   - Stripe: Connected ✓
   - Twilio: Connected ✓
   - Firebase: Connected ✓

2) **API Keys**
   - Key list (masked)
   - Create new key
   - Revoke key
   - Usage stats

### Settings Page

**Sections**
- Organization defaults
- Notification templates
- Role permissions matrix
- White-label settings (logo, colors)
- Compliance settings

### Audit Logs

**Main Page**

1) **Filter Bar**
   - User filter
   - Action type
   - Date range

2) **Log Table**
   - Timestamp
   - User
   - Action
   - Resource
   - IP Address
   - Details link

---

# Additional Questions (Client Confirmation Required)

## Required Clarifications

| # | Question | Context |
|:-:|:---------|:--------|
| 1 | What is the maximum number of stops per route? | Needed for route optimization limits |
| 2 | Should POD photos be required for all deliveries or configurable? | Some deliveries may not need photo proof |
| 3 | What are the specific HOS (Hours of Service) regulations to enforce? | Varies by region/country |
| 4 | Should customers receive real-time tracking links? | Common feature but not specified |

## Recommended Clarifications

| # | Question | Context |
|:-:|:---------|:--------|
| 1 | What offline data sync strategy is preferred? | For driver app in low connectivity |
| 2 | Should there be driver-to-driver messaging? | Currently only driver ↔ dispatcher specified |
| 3 | What vehicle types need specific handling? | Weight limits, route restrictions for trucks |
| 4 | Is there a maximum photo size/quality setting? | Affects storage costs and upload time |

---

# Terminology

| Term | Definition |
|:-----|:-----------|
| **Last-mile delivery** | Final leg of delivery from distribution center to customer |
| **Proof of Delivery (POD)** | Documentation confirming successful delivery (photo, signature) |
| **Deadhead** | Driving without cargo, typically returning from delivery |
| **ETA** | Estimated Time of Arrival |
| **Geofence** | Virtual perimeter around a geographic location |
| **Hours of Service (HOS)** | Regulations limiting driver working hours |
| **Route optimization** | Algorithm to find most efficient delivery sequence |
| **Telematics** | Technology for monitoring vehicle location and diagnostics |
| **Dispatch** | Process of assigning drivers to deliveries |
| **Fleet utilization** | Percentage of fleet actively being used |
| **Time window** | Specific period when delivery must occur |
| **Stop** | Individual delivery location on a route |
| **Manifest** | List of all deliveries assigned to a driver |

---

# Technical Requirements

## Authentication
- Email/Password for all users
- Phone + OTP option for drivers
- SSO (SAML/OAuth) for enterprise clients
- 2FA required for Admin and Fleet Manager roles

## Integrations
| Service | Purpose |
|:--------|:--------|
| Google Maps Platform | Navigation, geocoding, traffic data |
| Stripe | Payment processing for billing |
| Twilio | SMS notifications, phone verification |
| Firebase | Push notifications, real-time database |
| AWS S3 | Photo storage (POD, vehicle issues) |
| Samsara API | Fleet telematics (optional) |
| QuickBooks API | Accounting sync (enterprise) |

## Non-Functional Requirements
- Offline mode support for driver app
- Multi-language: English, Spanish, Korean
- iOS and Android mobile apps
- Battery optimization for GPS tracking
- Real-time updates via WebSocket

---

# Feature Change Log

## Version 1.0 (2026-01-15)

| Change Type | Description | Source |
|:-----------|:------------|:-------|
| **Initial Release** | Complete PRD from client requirements | FleetSync_260115_162345.md |

---

*Generated by /generate-prd*
*Source: FleetSync_260115_162345.md*
