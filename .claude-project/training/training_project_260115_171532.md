# SkillBridge - Client Requirements

## Basic Info

**1. App Name:** SkillBridge

**2. App Type:** Web + App (iOS, Android)

**3. Deadline:** Training Project - No deadline

**4. All User Types:**
- Student
- Instructor
- Admin

**5. User Roles & Permissions:**
- **Student**: Browse courses, enroll in courses, watch video lessons, complete quizzes, track progress, earn certificates, leave course reviews, manage profile and learning preferences
- **Instructor**: Create and manage courses, upload video lessons, create quizzes, view student progress and analytics, respond to student questions, manage earnings/payouts, update instructor profile
- **Admin**: Manage all users, approve/reject courses, manage platform settings, view platform analytics, handle disputes and refunds, manage categories and tags

**6. User Relationships:**
- Student to Course: N:N (one student can enroll in many courses, one course has many students)
- Instructor to Course: 1:N (one instructor creates many courses)
- Student to Instructor: N:N (indirect through courses)

---

## Design Reference

**7. Reference App:**
- **Udemy**: Course catalog layout, video player with progress tracking, review system
- **Coursera**: Clean learning dashboard, certificate design, progress visualization
- **Skillshare**: Class discovery, short-form lesson structure

**8. What makes your app special:**
- Micro-learning focused: Lessons are 5-15 minutes max for busy professionals
- Skill verification: Built-in assessments that validate learning with shareable badges
- Learning paths: Curated course sequences for career progression
- Live Q&A sessions: Weekly scheduled sessions with instructors

**9. Preferred Main Color:** Indigo (#4F46E5) - professional and trustworthy

**10. Preferred Font:** Inter - clean and highly readable

---

## Features

**11. Main Features (per user type):**

### Student
- Browse and search course catalog with filters (category, level, duration, rating)
- Course detail view with curriculum preview
- Enroll in free or paid courses
- Video lesson player with playback controls and bookmarks
- Quiz and assessment completion
- Progress tracking dashboard
- Certificate generation upon course completion
- Course reviews and ratings
- Wishlist/saved courses
- Learning history

### Instructor
- Course creation wizard (title, description, curriculum structure)
- Video upload and management
- Quiz builder with multiple question types
- Student progress monitoring
- Q&A management (answer student questions)
- Earnings dashboard and payout requests
- Course analytics (enrollments, completion rates, ratings)
- Promotional tools (coupons, announcements)

### Admin
- User management (students, instructors)
- Course approval workflow
- Category and tag management
- Platform analytics dashboard
- Transaction and payout management
- Content moderation tools

**12. Main Feature Module Flow:**

### Module: Course Enrollment
1. Student browses course catalog on Home/Explore
2. Student taps course card to view Course Detail page
3. Student reviews curriculum, instructor info, and reviews
4. Student taps "Enroll Now" button
5. If paid course: Student completes payment flow
6. System enrolls student and grants course access
7. Student redirected to Course Learning page
8. System sends enrollment confirmation notification

### Module: Lesson Completion
1. Student opens enrolled course from My Learning
2. Student selects lesson from curriculum list
3. System loads video player with lesson content
4. Student watches video (progress auto-saved)
5. Student completes lesson by watching 90%+ of video
6. System marks lesson complete and updates progress bar
7. If quiz attached: System prompts student to take quiz
8. Student proceeds to next lesson or returns to curriculum

### Module: Course Creation (Instructor)
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

### Module: Certificate Generation
1. Student completes all lessons in a course
2. Student completes final assessment (if required)
3. System validates completion criteria met
4. System generates personalized certificate with:
   - Student name
   - Course title
   - Completion date
   - Unique certificate ID
5. Student can view, download (PDF), and share certificate
6. Certificate link is publicly verifiable

**13. Authentication Method:**
- **Login:** Email/Password, Google Sign-in, Apple Sign-in
- **Signup required fields:** Email, Password, Full Name, Account Type (Student/Instructor)
- **Signup optional fields:** Profile Photo, Bio, LinkedIn URL (for instructors)

**14. Communication Features:**
- **Chat:** No (Q&A through course discussion threads instead)
- **Video call:** No
- **Push notifications:** Yes - New lesson available, course announcements, enrollment confirmations, certificate ready, instructor responses to questions

---

## Data

**15. Data to Collect:**
- User profiles (name, email, bio, preferences)
- Course data (title, description, curriculum, videos, quizzes)
- Enrollment records (student, course, date, status)
- Progress data (lessons completed, quiz scores, time spent)
- Reviews and ratings (rating, comment, date)
- Transactions (payments, payouts)
- Certificates (issued certificates with verification data)

**16. Data to Export:**
- Student: Personal learning history, certificates (PDF)
- Instructor: Student progress reports (CSV), earnings reports (CSV)
- Admin: User lists (CSV), transaction reports (CSV), course analytics (CSV)

---

## Technical

**17. 3rd Party Integrations:**
- **Google Authentication**: Social login for users
- **Apple Authentication**: Social login for iOS users
- **Stripe**: Payment processing for course purchases and instructor payouts

**18. Domain Terminology:**

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

---

## Other

**19. Additional Important Information:**
- Video hosting should support streaming with adaptive bitrate
- Courses can be free or paid (instructor sets price)
- Platform takes 20% commission on paid course sales
- Minimum payout threshold for instructors: $50
- Certificates should have unique verification URLs
- Course content should be DRM-protected (no easy downloading)
- Support for closed captions/subtitles on videos
- Mobile app should support offline video downloads for enrolled courses

---

*Generated by /generate-random-project*
*Difficulty Level: 3*
*Domain: Education*
