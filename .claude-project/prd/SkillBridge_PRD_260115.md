# SkillBridge PRD (2026-01-15)

---

# Part 1: Basic Information

## Title
SkillBridge

## Terminology

| Term | Definition |
|------|------------|
| Course | A structured learning program with multiple lessons and assessments |
| Lesson | A single video-based learning unit within a course section |
| Section | A group of related lessons within a course curriculum |
| Curriculum | The complete structure of sections and lessons in a course |
| Enrollment | A student's registration to access a specific course |
| Quiz | An assessment with questions to test learning comprehension |
| Certificate | A digital credential issued upon successful course completion |
| Learning Path | A curated sequence of courses for achieving a specific skill goal |
| Instructor | A content creator who publishes courses on the platform |
| Progress | The percentage of course content a student has completed |

## Project Information

### Description
SkillBridge is a micro-learning focused online education platform that connects students with instructors through short-form video courses. The platform emphasizes quick, digestible lessons (5-15 minutes) designed for busy professionals, with built-in skill verification through assessments and shareable certificates.

### Goals
1. Provide accessible, bite-sized learning content for busy professionals
2. Enable instructors to monetize their expertise through course creation
3. Validate learning outcomes through assessments and verifiable certificates
4. Create curated learning paths for structured career progression

### User Types
- **Student**: Learners who browse, enroll in, and complete courses. Can track progress, earn certificates, and leave reviews.
- **Instructor**: Content creators who build and publish courses. Can upload videos, create quizzes, monitor student progress, and earn revenue.
- **Admin**: Platform administrators who manage users, approve courses, handle transactions, and oversee platform operations.

### User Relationships
- Student to Course: N:N (one student can enroll in many courses, one course has many students)
- Instructor to Course: 1:N (one instructor creates many courses)
- Student to Instructor: N:N (indirect relationship through course enrollments)

### Project Type
- iOS App - React Native
- Android App - React Native
- Web Application - React

## System Modules (Step-by-step Flows)

### Module 1 - Course Enrollment
1. Student browses course catalog on Home/Explore
2. Student taps course card to view Course Detail page
3. Student reviews curriculum, instructor info, and reviews
4. Student taps "Enroll Now" button
5. If paid course: Student completes payment flow via Stripe
6. System enrolls student and grants course access
7. Student redirected to Course Learning page
8. System sends enrollment confirmation push notification

### Module 2 - Lesson Completion
1. Student opens enrolled course from My Learning
2. Student selects lesson from curriculum list
3. System loads video player with lesson content
4. Student watches video (progress auto-saved)
5. Student completes lesson by watching 90%+ of video
6. System marks lesson complete and updates progress bar
7. If quiz attached: System prompts student to take quiz
8. Student proceeds to next lesson or returns to curriculum

### Module 3 - Course Creation (Instructor)
1. Instructor navigates to Instructor Dashboard
2. Instructor taps "Create New Course" button
3. Instructor enters course details (title, description, category, level)
4. Instructor creates curriculum structure (sections and lessons)
5. Instructor uploads video for each lesson
6. Instructor adds quizzes to relevant sections
7. Instructor sets pricing (free or paid with price)
8. Instructor submits course for review
9. Admin reviews and approves/rejects course
10. Upon approval: Course goes live in catalog

### Module 4 - Certificate Generation
1. Student completes all lessons in a course
2. Student completes final assessment (if required)
3. System validates completion criteria met
4. System generates personalized certificate with student name, course title, completion date, unique certificate ID
5. Student can view, download (PDF), and share certificate
6. Certificate link is publicly verifiable

### Module 5 - Instructor Payout
1. Student purchases paid course
2. System processes payment via Stripe
3. Platform deducts 20% commission
4. Remaining 80% added to instructor's earnings balance
5. Instructor requests payout when balance >= $50
6. Admin processes payout request
7. Funds transferred to instructor's connected account

## 3rd Party API List
- Google Authentication: Social login for users
- Apple Authentication: Social login for iOS users
- Stripe: Payment processing for course purchases and instructor payouts

---

# Part 2: User Application PRD

## User Types: Student, Instructor
Both user types share common authentication flows but have distinct app experiences.

## 1. Common

### Splash Page
- Design: App logo with loading indicator
- Auto-redirect to Home (if logged in) or Login (if not)

### Login Page
- **Input**:
  - Email
  - Password
- **Social Login Options**:
  - Google Sign-in button
  - Apple Sign-in button
- **Next Action**:
  - Validate email format and password not empty
  - On success: Redirect to Home (Student) or Instructor Dashboard (Instructor)
  - On failure: Display error message
  - "Forgot Password?" link → Forgot Password Page

### Forgot Password Page
- **Main**
  - Input: Email address
  - Action: Send password reset link
- **Reset Password Page**
  - Input: New password, Confirm password
  - Validation: Passwords match, minimum 8 characters
  - On success: Redirect to Login with success message

### Sign Up Page
- **Input**:
  - Email (required)
  - Password (required)
  - Full Name (required)
  - Account Type: Student / Instructor (required)
  - Profile Photo (optional)
  - Bio (optional)
  - LinkedIn URL (optional, shown only for Instructor)
- **Social Signup Options**:
  - Google Sign-up button
  - Apple Sign-up button
- **Rules**:
  - Email must be unique
  - Password minimum 8 characters
  - Account type determines app experience after signup

## 2. Student

### 2.1 Navigation Menu
1. Home
2. Search
3. My Learning
4. Profile

### 2.2 Page Architecture & Feature Specification

#### 1. Home Tab

**Main Page**
1) Featured Courses Section
   - Horizontal scrollable carousel
   - Course cards with thumbnail, title, instructor, rating
   - Tap → Course Detail Page

2) Categories Section
   - Grid of category icons/tiles
   - Tap category → Filtered course list

3) Recommended For You Section
   - Personalized course suggestions
   - Based on enrolled courses and browsing history

4) New Courses Section
   - Recently published courses
   - Horizontal scrollable list

5) Top Rated Courses Section
   - Highest rated courses
   - Horizontal scrollable list

#### 2. Search Tab

**Main Page**
1) Search Bar
   - Text input with search icon
   - Real-time search suggestions

2) Filter Options
   - Category dropdown
   - Level (Beginner, Intermediate, Advanced)
   - Duration (< 1hr, 1-3hr, 3-5hr, > 5hr)
   - Rating (4+, 4.5+)
   - Price (Free, Paid)

3) Search Results List
   - Course cards with thumbnail, title, instructor, rating, price
   - Sort by: Relevance, Rating, Newest, Price

**Course Detail Page**
- Course thumbnail/preview video
- Title, instructor name and photo
- Rating and review count
- Price / "Enrolled" badge
- Course description
- What you'll learn (bullet points)
- Curriculum (expandable sections and lessons)
- Instructor bio
- Reviews section
- "Enroll Now" / "Continue Learning" button
- Add to Wishlist button

#### 3. My Learning Tab

**Main Page**
1) In Progress Section
   - Courses with progress bar
   - Last accessed date
   - "Continue" button

2) Completed Section
   - Finished courses
   - Certificate download button
   - "View Certificate" link

3) Wishlist Section
   - Saved courses for later
   - Remove from wishlist option

**Course Learning Page**
- Video player (top)
  - Playback controls (play/pause, seek, speed, fullscreen)
  - Bookmark button
  - Progress indicator
- Curriculum list (below video)
  - Sections with lessons
  - Checkmarks for completed lessons
  - Current lesson highlighted
- Course Q&A tab
  - Ask question button
  - List of questions and answers
- Notes tab
  - Personal notes for the course

**Quiz Page**
- Question text
- Answer options (multiple choice / true-false)
- Submit button
- Results summary after completion
- Retry option if failed

**Certificate Page**
- Certificate preview
- Student name, course title, date
- Unique certificate ID
- Download PDF button
- Share buttons (LinkedIn, Twitter, copy link)
- Verification URL

#### 4. Profile Tab

**Main Page**
1) Profile Header
   - Profile photo
   - Full name
   - Email
   - Edit profile button

2) Learning Stats
   - Courses enrolled
   - Courses completed
   - Certificates earned
   - Total learning hours

3) Settings Section
   - Notification preferences
   - Language
   - Privacy settings

4) Account Section
   - Change password
   - Connected accounts (Google, Apple)
   - Delete account
   - Logout

**Edit Profile Page**
- Profile photo upload
- Full name input
- Bio text area
- Save button

## 3. Instructor

### 3.1 Navigation Menu
1. Dashboard
2. Courses
3. Earnings
4. Profile

### 3.2 Page Architecture & Feature Specification

#### 1. Dashboard Tab

**Main Page**
1) Stats Overview Cards
   - Total students
   - Total courses
   - Total earnings
   - Average rating

2) Recent Enrollments
   - List of recent student enrollments
   - Student name, course name, date

3) Pending Questions
   - Unanswered Q&A items
   - Quick reply option

4) Course Performance Chart
   - Enrollments over time
   - Line/bar chart

#### 2. Courses Tab

**Main Page**
1) Course List
   - All instructor's courses
   - Status badge (Draft, Pending Review, Published, Rejected)
   - Enrollments count, rating
   - Edit / View Analytics buttons

2) Create New Course Button
   - Floating action button
   - → Course Creation Wizard

**Course Creation Wizard**
- Step 1: Basic Info
  - Title, description, category, level
  - Course thumbnail upload
- Step 2: Curriculum
  - Add sections
  - Add lessons to sections
  - Reorder via drag-drop
- Step 3: Content
  - Upload video for each lesson
  - Add quiz to lessons (optional)
- Step 4: Pricing
  - Free or Paid toggle
  - Price input (if paid)
- Step 5: Review & Submit
  - Preview all details
  - Submit for review button

**Quiz Builder**
- Add question
- Question types: Multiple choice, True/False
- Add answer options
- Mark correct answer
- Save quiz

**Course Analytics Page**
- Enrollment trend chart
- Completion rate
- Average rating
- Review list
- Revenue (if paid course)

#### 3. Earnings Tab

**Main Page**
1) Balance Card
   - Current available balance
   - Pending balance
   - Request Payout button (if >= $50)

2) Earnings Chart
   - Monthly earnings trend

3) Transaction History
   - List of enrollments with amount
   - Payout history with status

**Payout Request Page**
- Available balance display
- Payout amount input
- Connected payout method (Stripe)
- Submit request button

#### 4. Profile Tab

**Main Page**
1) Instructor Profile Header
   - Profile photo
   - Full name
   - Bio
   - LinkedIn URL
   - Edit profile button

2) Instructor Stats
   - Total students
   - Total courses
   - Average rating
   - Total reviews

3) Settings Section
   - Notification preferences
   - Payout settings

4) Account Section
   - Change password
   - Connected accounts
   - Logout

---

# Part 3: Admin Dashboard PRD

## Page Architecture & Feature Specification

### Dashboard Page

**Card Components**
- Total Users (Students + Instructors)
- Total Courses (Published)
- Total Revenue (Platform earnings)
- Pending Approvals count

**Additional Components**
- User Growth Chart (line chart, last 30 days)
- Revenue Chart (bar chart, last 12 months)
- Recent Activity Feed (enrollments, signups, course submissions)

## User Management

### Student Management Page

**Main Page**
1. Top Area:
   - Filters: Status (Active, Suspended), Date joined
   - Search: Name, Email
   - Export button (CSV)

2. List Item Component:
   - Profile photo, Name, Email
   - Courses enrolled, Date joined
   - Status badge
   - Action column: View, Suspend/Activate

3. Sort Order: Newest first, Name A-Z

**Detail Drawer**
- Header Info: Photo, Name, Email, Join date
- Sub-tabs:
  - Enrollments (course list)
  - Certificates (issued certificates)
  - Activity (recent actions)

### Instructor Management Page

**Main Page**
1. Top Area:
   - Filters: Status (Active, Suspended, Pending Verification)
   - Search: Name, Email
   - Export button (CSV)

2. List Item Component:
   - Profile photo, Name, Email
   - Courses count, Total students, Rating
   - Status badge
   - Action column: View, Suspend/Activate, Verify

3. Sort Order: Newest first, Rating high to low

**Detail Drawer**
- Header Info: Photo, Name, Email, Bio, LinkedIn
- Sub-tabs:
  - Courses (list with status)
  - Earnings (balance, payout history)
  - Reviews (received reviews)

## Feature Management

### Course Management Page

**Main Page**
- Filters: Status (All, Pending Review, Published, Rejected), Category
- Search: Title, Instructor name
- List Component:
  - Thumbnail, Title, Instructor
  - Status, Enrollments, Rating
  - Actions: View, Approve, Reject, Unpublish

**Course Review Modal**
- Course details preview
- Curriculum preview
- Approve button with notes
- Reject button with reason (required)

### Category Management Page

**Main Page**
- List of categories with icon
- Course count per category
- Add Category button
- Edit / Delete actions

**Category Modal**
- Category name
- Icon upload
- Description
- Save / Cancel

### Payout Management Page

**Main Page**
- Filters: Status (Pending, Completed, Failed)
- List Component:
  - Instructor name, Amount, Request date
  - Status badge
  - Actions: Process, Reject

**Process Payout Modal**
- Instructor details
- Amount
- Stripe payout confirmation
- Mark as Completed button

## Export / Data Download

**Data Download**
- Users Export: Student list, Instructor list (CSV)
- Courses Export: All courses with stats (CSV)
- Transactions Export: All payments and payouts (CSV)
- Format: CSV
- Filter: All time / Custom date range

---

# Additional Questions (Client Confirmation Required)

## Required Clarifications
| # | Question | Context |
|:-:|:---------|:--------|
| 1 | What is the video hosting solution? | Need to determine if using self-hosted, AWS S3 + CloudFront, Vimeo, or other streaming service |
| 2 | What are the specific quiz question types needed? | Currently assuming multiple choice and true/false - are fill-in-blank or essay questions needed? |
| 3 | What is the certificate verification process? | How should third parties verify certificate authenticity? |
| 4 | What is the course approval criteria? | What specific guidelines should Admin use to approve/reject courses? |

## Recommended Clarifications
| # | Question | Context |
|:-:|:---------|:--------|
| 1 | Should instructors be able to offer course coupons/discounts? | Mentioned in features but needs detail on discount types and limits |
| 2 | What offline download restrictions apply? | Need DRM solution details for offline content protection |
| 3 | Is there a free trial or preview for paid courses? | Can students preview first lesson before purchasing? |
| 4 | What are the Learning Path creation rules? | Who creates Learning Paths - Admin only or Instructors too? |
| 5 | Live Q&A session implementation details? | Mentioned as differentiator - need scheduling, video platform, and capacity details |

---

# Feature Change Log

## Version 1.0 (2026-01-15)

| Change Type | Before | After | Source |
|:-----------|:-------|:------|:-------|
| **Initial Version** | - | Complete PRD created | training_project_260115_171532.md |

### Change Details
#### Initial PRD Creation
- **Source Document**: training_project_260115_171532.md
- **Change Description**: Initial PRD generated from random project training file

---

# UX Flow Improvement Suggestions (Optional)

*Skipped per workflow - user can request UX analysis separately*

---
