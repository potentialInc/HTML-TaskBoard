# FitPulse PRD (2026-01-16)

---

# Part 1: Basic Information

## Title
FitPulse

## Terminology

| Term | Definition |
|------|------------|
| Workout | A single exercise session logged by the user |
| Intensity | Self-reported effort level (1-10 scale) |
| Progress | Aggregated view of workout frequency and duration over time |
| Active User | A user who has logged at least one workout in the past 7 days |
| Workout Streak | Consecutive days with at least one logged workout |

## Project Information

### Description
FitPulse is a minimalist fitness tracking web application focused on quick workout logging and progress visualization. The app prioritizes privacy and simplicity, offering personal fitness tracking without social features for privacy-focused users.

### Goals
1. Enable users to quickly log workouts with minimal friction
2. Provide clear progress visualization through weekly/monthly charts
3. Maintain user privacy with no social features or data sharing

### User Types
- **User (Fitness Enthusiast)**: Can create account, log workouts, track progress, view personal statistics, manage profile
- **Admin**: Can manage users, view platform analytics, moderate content, manage system settings

### User Relationships
- Users are independent of each other (no social connections)
- Admin has oversight of all Users (1:N relationship)

### Project Type
- Web Application - React (Frontend)
- Admin Dashboard - React
- Backend - NestJS

## System Modules (Step-by-step Flows)

### Module 1 - Log Workout
1. User clicks "Log Workout" button
2. System displays workout form (type dropdown, duration input, intensity slider)
3. User fills in workout details and submits
4. System saves workout and updates statistics
5. User sees confirmation and updated weekly progress

### Module 2 - View Progress
1. User navigates to "Progress" tab
2. System retrieves user's workout data
3. System displays weekly chart showing workout frequency and duration
4. User can toggle between weekly/monthly views

### Module 3 - Admin Dashboard
1. Admin logs into admin panel
2. System displays overview: total users, active users this week, total workouts logged
3. Admin can drill down into user list or platform statistics

## 3rd Party API List
- None (Level 1 project - no external integrations)

---

# Part 2: User Application PRD

## User Types: User (Fitness Enthusiast)
Privacy-focused fitness tracking without social features.

## 1. Common

### Splash Page
- Design: Simple loading screen with FitPulse logo
- Color: Energetic Orange (#FF6B35) background

### Login Page
- **Input**:
  - Email (required)
  - Password (required)
- **Next Action**:
  - Validate email format
  - Validate password not empty
  - On success: Navigate to Home (Dashboard)
  - On failure: Show error message "Invalid email or password"

### Forgot Password Page
- **Main**
  - Input: Email address
  - Action: Send password reset link to email
- **Reset Password Page**
  - Input: New password, Confirm password
  - Validation: Passwords must match, minimum 8 characters
  - On success: Navigate to Login page with success message

### Sign Up Page
- **Input**:
  - Email (required)
  - Password (required)
  - Name (required)
  - Age (optional)
  - Fitness Goal (optional)
- **Rules**:
  - Email must be unique
  - Password minimum 8 characters
  - Name minimum 2 characters

## 2. User (Fitness Enthusiast)

### 2.1 Navigation Menu
1. Home (Dashboard)
2. Progress
3. History
4. Profile

### 2.2 Page Architecture & Feature Specification

#### 1. Home (Dashboard) Tab

**Main Page**
1) Quick Stats Card
   - Total workouts this week
   - Current workout streak
   - Weekly goal progress (if set)

2) Log Workout Button (Primary CTA)
   - Prominent button for quick access
   - Opens Log Workout modal/page

3) Recent Activity
   - Last 3-5 workout entries
   - Quick view of recent activity

**Log Workout Page/Modal**
- Workout Type: Dropdown (Running, Cycling, Strength, Yoga, Swimming, Other)
- Duration: Number input (minutes)
- Intensity: Slider (1-10 scale)
- Date: Date picker (defaults to today)
- Notes: Optional text field
- Save Button / Cancel Button

#### 2. Progress Tab

**Main Page**
1) Progress Chart
   - Weekly view (default): Bar chart showing workouts per day
   - Monthly view: Bar chart showing workouts per week
   - Toggle button for weekly/monthly view

2) Stats Summary
   - Total workouts (selected period)
   - Total duration (selected period)
   - Average intensity (selected period)
   - Most frequent workout type

#### 3. History Tab

**Main Page**
1) Workout List
   - Chronological list of all workouts
   - Each item shows: Date, Type, Duration, Intensity
   - Click to view details

2) Filter Options
   - Filter by workout type
   - Filter by date range

**Workout Detail Page**
- Full workout information
- Edit button
- Delete button (with confirmation)

#### 4. Profile Tab

**Main Page**
1) Profile Information
   - Name
   - Email (read-only)
   - Age
   - Fitness Goal

2) Edit Profile Button
   - Opens edit form

3) Account Actions
   - Change Password
   - Logout

**Edit Profile Page**
- Name input
- Age input
- Fitness Goal input (dropdown or text)
- Save Button / Cancel Button

---

# Part 3: Admin Dashboard PRD

## Page Architecture & Feature Specification

### Dashboard Page
Overview of platform metrics and user activity.

**Card Components**
- Total Users: Count of registered users
- Active Users: Users with workouts in last 7 days
- Total Workouts: All workouts logged on platform
- Workouts This Week: Workouts logged in current week

**Additional Components**
- User Growth Chart: Line chart showing user registrations over time
- Workout Activity Chart: Bar chart showing daily workout logs

## User Management

### User Management Page

**Main Page**
1. Top Area:
   - Filters: Status (Active/Inactive), Date Range (Registration Date)
   - Search: By name or email
   - Export button (future consideration)

2. List Item Component:
   - Name
   - Email
   - Registration Date
   - Last Active Date
   - Total Workouts
   - Status (Active/Inactive)
   - Action column (View Details)

3. Sort Order: Registration Date (newest first), Name (A-Z), Last Active

**Detail Drawer**
- Header Info: Name, Email, Registration Date
- Statistics: Total Workouts, Workout Streak, Most Common Workout Type
- Recent Activity: Last 10 workouts
- Actions: Deactivate Account, Delete Account (with confirmation)

## System Settings

### Settings Page

**Main Page**
- Platform Name
- Support Email
- Maintenance Mode Toggle

---

# Additional Questions (Client Confirmation Required)

## Required Clarifications
| # | Question | Context |
|:-:|:---------|:--------|
| 1 | What workout types should be available in the dropdown? | Currently assumed: Running, Cycling, Strength, Yoga, Swimming, Other. Client may want specific types. |
| 2 | Should users be able to add custom workout types? | Not specified in requirements |
| 3 | What is the maximum duration allowed for a single workout? | Need validation rules |

## Recommended Clarifications
| # | Question | Context |
|:-:|:---------|:--------|
| 1 | Should there be a weekly workout goal feature? | Could enhance progress tracking |
| 2 | Should workout history have pagination or infinite scroll? | Performance consideration for heavy users |
| 3 | Should the admin be able to send announcements to users? | Admin communication feature |
| 4 | What happens when a user account is deactivated vs deleted? | Data retention policy |

---

# Feature Change Log

## Version 1.0 (2026-01-16)

| Change Type | Before | After | Source |
|:-----------|:-------|:------|:-------|
| **Initial Creation** | - | FitPulse PRD v1.0 | FitPulse_260116_154208.md |

### Change Details
#### Initial PRD Creation
- **Source Document**: FitPulse_260116_154208.md (Training Project)
- **Change Description**: Initial PRD created from generated training project specification

---

# Validation Results

## Validation Checklist

### User Types Verification
- [x] All user types from input are included in Part 1 User Types
- [x] Each user type's permissions are accurately reflected
- [x] User relationships (independent users, Admin 1:N) are correct
- [x] Part 2 Section Mapping: User (Fitness Enthusiast) has separate section
- [x] Navigation Menu and Page Architecture defined for User

### Feature Verification
- [x] All main features from input are included (Log Workout, View History, Track Progress, Manage Profile)
- [x] Module flows match the input file
- [x] Navigation menu structure covers all features

### Authentication Verification
- [x] Login method included (Email/Password)
- [x] Signup required fields correct (Email, Password, Name)
- [x] Signup optional fields correct (Age, Fitness Goal)

### Tech Stack Verification
- [x] No 3rd party APIs (as specified - Level 1)
- [x] Platform: Web (React)
- [x] Backend: NestJS
- [x] Admin Dashboard: React

### Terminology Verification
- [x] All 5 domain terms included (Workout, Intensity, Progress, Active User, Workout Streak)
- [x] Terms used consistently throughout PRD

### Validation Complete
- Total items verified: 15
- Items modified: 0
- Items moved to questions: 7

---

# UX Flow Improvement Suggestions

UX flow issues identified during PRD creation:

## Suggestions
| # | Current Flow | Issue | Suggestion | Priority |
|:-:|:------------|:------|:-----------|:--------:|
| 1 | Log Workout requires navigating to Home first | Users may want to log from any screen | Add floating action button (FAB) for quick workout logging accessible from all screens | High |
| 2 | No empty state guidance | New users see empty dashboard/history | Add onboarding prompts and empty state CTAs encouraging first workout log | High |
| 3 | Progress tab only shows charts | No motivational feedback | Add streak badges or weekly completion messages | Medium |
| 4 | History filtering is basic | Heavy users may have difficulty finding specific workouts | Add search by notes/keywords and date range presets (This Week, Last Month) | Medium |
| 5 | No quick repeat workout | Users often do same workouts repeatedly | Add "Log Again" button on workout history items | Low |

### Suggestion Details

#### Suggestion 1: Floating Action Button for Workout Logging
- **Current**: User must navigate to Home tab to access Log Workout button
- **Issue**: Extra navigation step for the core action
- **Suggestion**: Add a floating action button (FAB) visible on all main screens (Home, Progress, History) for instant workout logging
- **Expected Benefit**: Reduced friction for primary user action, improved engagement
- **Complexity**: Simple

#### Suggestion 2: Empty State Onboarding
- **Current**: New users see empty dashboard with no guidance
- **Issue**: Unclear next steps, potential user drop-off
- **Suggestion**: Add illustrated empty states with clear CTA ("Log Your First Workout!") and brief feature highlights
- **Expected Benefit**: Better new user activation, clearer value proposition
- **Complexity**: Simple

#### Suggestion 3: Motivational Feedback
- **Current**: Progress tab shows data but no emotional engagement
- **Issue**: Users may lose motivation without positive reinforcement
- **Suggestion**: Add streak celebration animations, weekly summary messages ("Great week! 5 workouts logged"), and milestone badges
- **Expected Benefit**: Increased user retention and engagement
- **Complexity**: Medium

#### Suggestion 4: Enhanced History Search
- **Current**: Basic type and date range filters only
- **Issue**: Difficult to find specific past workouts for reference
- **Suggestion**: Add keyword search (searches notes field), quick date presets ("This Week", "Last Month", "Last 3 Months")
- **Expected Benefit**: Better usability for long-term users with extensive history
- **Complexity**: Simple

#### Suggestion 5: Quick Repeat Workout
- **Current**: To log a similar workout, user must manually enter all fields again
- **Issue**: Repetitive data entry for routine workouts
- **Suggestion**: Add "Log Again" button on history items that pre-fills the workout form with previous values
- **Expected Benefit**: Faster workout logging for routine exercises
- **Complexity**: Simple
