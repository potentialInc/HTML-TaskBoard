---
description: Generate random complex project specifications for team training
argument-hint: "[--difficulty <1-5>]"
---

# Random Project Generator for Training

Generate detailed, realistic full-stack web/mobile app project specifications for team training purposes. The generated project is output in client questionnaire format and automatically triggers `/generate-prd` to create a comprehensive PRD.

---

## Usage

```bash
/generate-random-project
/generate-random-project --difficulty 3
/generate-random-project --difficulty 5
```

### Arguments

| Argument | Description | Default |
|----------|-------------|---------|
| `--difficulty <1-5>` | Project complexity level (1=simple, 5=expert) | Random (1-5) |

---

## Difficulty Scale

| Level | Complexity | User Types | Main Features | 3rd Party Integrations |
|:-----:|:-----------|:-----------|:--------------|:-----------------------|
| 1 | Simple | 1-2 types | 3-5 features | 0-1 integrations |
| 2 | Basic | 2 types | 5-7 features | 1-2 integrations |
| 3 | Medium | 2-3 types | 7-10 features | 2-3 integrations |
| 4 | Complex | 3-4 types | 10-15 features | 3-5 integrations |
| 5 | Expert | 4+ types | 15+ features | 5+ integrations |

---

## Execution Steps

### Step 1: Parse Arguments

Check `$ARGUMENTS` for the `--difficulty` flag.

**Parsing logic:**
1. If `--difficulty <N>` is provided:
   - Validate N is between 1-5
   - If invalid, output error and stop
2. If no argument provided:
   - Randomly select difficulty level (1-5)

**Error handling:**
```
Error: Invalid difficulty level. Please use a number between 1 and 5.
Usage: /generate-random-project --difficulty 3
```

---

### Step 2: Select Random Domain

Randomly select one domain from the following list. Each domain comes with associated characteristics.

| Domain | Example App Types | Common User Types |
|--------|-------------------|-------------------|
| Healthcare | Telemedicine, Patient Management, Fitness Tracking | Patient, Doctor, Admin, Caregiver |
| Education | LMS, Tutoring Platform, Course Marketplace | Student, Teacher, Parent, Admin |
| E-commerce | Marketplace, B2B Platform, Subscription Box | Buyer, Seller, Admin, Delivery Partner |
| Social | Community Platform, Dating App, Professional Network | User, Creator, Moderator, Admin |
| Fitness | Workout App, Personal Training, Nutrition Tracker | User, Trainer, Nutritionist, Admin |
| Finance | Budgeting App, Investment Platform, P2P Lending | User, Advisor, Investor, Admin |
| Food Delivery | Restaurant Ordering, Meal Prep, Grocery Delivery | Customer, Restaurant, Driver, Admin |
| Real Estate | Property Listing, Rental Management, Agent Platform | Buyer/Renter, Agent, Landlord, Admin |
| Travel | Booking Platform, Travel Planning, Tour Guide | Traveler, Host, Guide, Admin |
| Entertainment | Streaming, Event Booking, Gaming | User, Creator, Organizer, Admin |
| Logistics | Fleet Management, Warehouse, Delivery Tracking | Driver, Manager, Client, Admin |
| HR/Recruitment | Job Board, ATS, Employee Management | Candidate, Recruiter, Employee, Admin |

---

### Step 3: Generate Project Details Based on Difficulty

Generate realistic project details scaled to the selected difficulty level.

#### 3.1 Basic Information

**App Name Generation:**
- Combine domain-relevant prefix with creative suffix
- Examples: "MediConnect", "LearnHub Pro", "FitTrack Elite", "QuickBite"

**App Type Selection (based on difficulty):**
- Level 1-2: Single platform (App OR Web)
- Level 3: App + Web (user-facing)
- Level 4-5: App + Web + Admin Dashboard

**User Types Generation:**
- Select appropriate number of user types from domain list based on difficulty
- Define clear roles and permissions for each
- Create realistic relationships (1:1, 1:N, etc.)

#### 3.2 Design Reference

Generate realistic design references:
- Reference 1-3 real apps in similar domain
- Specify unique selling points/differentiators
- Select appropriate color scheme (professional for B2B, vibrant for consumer)
- Suggest appropriate fonts

#### 3.3 Features Generation

**Feature Categories to Consider:**

| Category | Level 1-2 Features | Level 3-4 Features | Level 5 Features |
|----------|-------------------|-------------------|------------------|
| Authentication | Basic email/password | + Social login, Phone verification | + SSO, Biometric, 2FA |
| User Profile | Basic profile | + Avatar, Preferences | + Verification, Badges, Portfolio |
| Core Domain | 2-3 main features | 5-7 main features | 10+ main features |
| Communication | None or basic | In-app messaging | + Video calls, Group chat, Notifications |
| Payments | None | Basic payment | Subscriptions, Refunds, Multi-currency |
| Search/Discovery | Basic list | Filters, Search | Advanced filters, Recommendations, AI |
| Content | View only | Create/Edit | Rich media, Version history, Collaboration |
| Analytics | None | Basic stats | Detailed dashboards, Reports, Export |
| Admin | None | Basic CRUD | Full management, Bulk actions, Audit logs |

**Module Flow Generation:**
- Create step-by-step flows for each main feature
- Format: Actor -> Action -> System Response -> Result
- Include edge cases for higher difficulty levels

#### 3.4 Communication Features

Based on difficulty:
- Level 1: Push notifications only (optional)
- Level 2: Push notifications
- Level 3: + In-app messaging
- Level 4: + Real-time chat
- Level 5: + Video calls, Group messaging, Channels

#### 3.5 Data Requirements

**Data Collection (scaled by difficulty):**
- Level 1-2: Basic forms (registration, profile)
- Level 3: + Activity logs, User preferences
- Level 4: + Surveys, Analytics events, Usage metrics
- Level 5: + Advanced tracking, A/B testing data, ML training data

**Data Export (scaled by difficulty):**
- Level 1-2: None or basic list export
- Level 3: User data export (CSV)
- Level 4: + Reports, Activity logs
- Level 5: + Bulk exports, Scheduled reports, API access

#### 3.6 3rd Party Integrations

Select appropriate integrations based on difficulty and domain:

| Category | Options |
|----------|---------|
| Authentication | Google, Apple, Kakao, Facebook, LinkedIn |
| Payment | Stripe, PayPal, Toss, KakaoPay |
| Maps | Google Maps, Naver Maps, Kakao Maps |
| Communication | Twilio (SMS), SendGrid (Email), Firebase (Push) |
| Storage | AWS S3, Google Cloud Storage |
| Analytics | Mixpanel, Amplitude, Google Analytics |
| Video | Zoom, Agora, Twilio Video |
| AI/ML | OpenAI, Google Vision, AWS Rekognition |

#### 3.7 Domain Terminology

Generate 5-15 domain-specific terms based on difficulty:
- Include common industry terms
- Include app-specific coined terms
- Provide clear definitions

---

### Step 4: Output in Client Questionnaire Format

Generate the output file matching the `/generate-prd` input format:

```markdown
# [App Name] - Client Requirements

## Basic Info

**1. App Name:** [Generated App Name]

**2. App Type:** [Web / App / Web + App]

**3. Deadline:** Training Project - No deadline

**4. All User Types:**
- [User Type 1]
- [User Type 2]
- [etc.]

**5. User Roles & Permissions:**
- **[User Type 1]**: [Permissions and capabilities]
- **[User Type 2]**: [Permissions and capabilities]

**6. User Relationships:**
[Describe relationships - 1:1, 1:N, independent]

---

## Design Reference

**7. Reference App:**
- [Reference App 1]: [What to reference]
- [Reference App 2]: [What to reference]

**8. What makes your app special:**
[Unique selling points and differentiators]

**9. Preferred Main Color:** [Color choice with hex code]

**10. Preferred Font:** [Font suggestion]

---

## Features

**11. Main Features (per user type):**

### [User Type 1]
- [Feature 1]
- [Feature 2]
- [etc.]

### [User Type 2]
- [Feature 1]
- [Feature 2]
- [etc.]

**12. Main Feature Module Flow:**

### Module: [Feature Name]
1. [Actor] does [Action]
2. System [Response]
3. [Result/Next Step]

[Repeat for each main feature module]

**13. Authentication Method:**
- **Login:** [Methods - ID/PW, Social login options]
- **Signup required fields:** [List of required fields]
- **Signup optional fields:** [List of optional fields]

**14. Communication Features:**
- **Chat:** [Yes/No + details]
- **Video call:** [Yes/No + details]
- **Push notifications:** [Yes/No + details]

---

## Data

**15. Data to Collect:**
- [Data type 1]: [Description]
- [Data type 2]: [Description]

**16. Data to Export:**
- [Export type 1]: [Description]
- [Export type 2]: [Description]

---

## Technical

**17. 3rd Party Integrations:**
- [Service 1]: [Purpose]
- [Service 2]: [Purpose]

**18. Domain Terminology:**

| Term | Definition |
|------|------------|
| [Term 1] | [Definition] |
| [Term 2] | [Definition] |

---

## Other

**19. Additional Important Information:**
[Any additional context, constraints, or requirements]

---

*Generated by /generate-random-project*
*Difficulty Level: [N]*
*Domain: [Selected Domain]*
```

---

### Step 5: Save Output File

**Create output directory:**
```bash
mkdir -p .claude-project/training
```

**Save file:**
- Location: `.claude-project/training/`
- Filename format: `[ProjectName]_[YYMMDD_HHMMSS].md`
- Sanitize project name: Replace spaces with underscores, remove special characters
- Example: `HealthBridge_260115_143022.md`

---

### Step 6: Report Generation Summary

Output a summary before triggering PRD generation. **Always display the full absolute path** to the generated file:

```markdown
## Random Project Generated

### Project Details
- **Project Name:** [Generated Name]
- **Domain:** [Selected Domain]
- **Difficulty:** Level [N] ([Complexity Name])
- **User Types:** [Count] types
- **Main Features:** [Count] features
- **Integrations:** [Count] 3rd party services

### Output File
**Full Path:** `/Users/dongsub/Documents/Potential/projects/claude-base/.claude-project/training/[filename].md`

### Next Step
Triggering /generate-prd with the generated file...
```

**Note:** The full absolute path must be displayed so users can easily reference or manually trigger `/generate-prd` with the correct path.

---

### Step 7: Trigger /generate-prd

After saving the file, automatically trigger the PRD generation using the **full absolute path** of the generated training file:

```
/generate-prd /Users/dongsub/Documents/Potential/projects/claude-base/.claude-project/training/[filename].md
```

**Example with actual filename:**
```
/generate-prd /Users/dongsub/Documents/Potential/projects/claude-base/.claude-project/training/HealthBridge_260115_143022.md
```

**IMPORTANT:** Always use the complete absolute path when triggering `/generate-prd`. The path format is:
```
[PROJECT_ROOT]/.claude-project/training/[ProjectName]_[YYMMDD_HHMMSS].md
```

This will:
1. Read the generated client questionnaire from the training file
2. Generate a comprehensive PRD document
3. Save the PRD to `.claude-project/prd/`

---

### Step 8: Generate PDF from PRD

After `/generate-prd` completes successfully, convert the PRD markdown to PDF:

```
/md-to-pdf .claude-project/prd/[ProjectName]_PRD_[YYMMDD].md
```

**Example:**
```
/md-to-pdf .claude-project/prd/HealthBridge_Pro_PRD_260115.md
```

This creates a PDF at `.claude-project/prd/[ProjectName]_PRD_[YYMMDD].pdf`

---

## Error Handling

**Invalid difficulty:**
```
Error: Invalid difficulty level "[value]"
Difficulty must be a number between 1 and 5.

Usage:
  /generate-random-project              # Random difficulty
  /generate-random-project --difficulty 3  # Specific difficulty
```

**File save error:**
```
Error: Unable to save training project file.
Please check write permissions for .claude-project/training/ directory.
```

---

## Example Output Summary

Running `/generate-random-project --difficulty 4`:

```
## Random Project Generated

### Project Details
- **Project Name:** HealthBridge Pro
- **Domain:** Healthcare
- **Difficulty:** Level 4 (Complex)
- **User Types:** 4 types (Patient, Doctor, Caregiver, Admin)
- **Main Features:** 12 features
- **Integrations:** 4 services (Google Auth, Stripe, Twilio, Google Maps)

### Output File
**Full Path:** /Users/dongsub/Documents/Potential/projects/claude-base/.claude-project/training/HealthBridge_Pro_260115_143022.md

### Next Step
Triggering /generate-prd with the generated file...

/generate-prd /Users/dongsub/Documents/Potential/projects/claude-base/.claude-project/training/HealthBridge_Pro_260115_143022.md

### PDF Generation
/md-to-pdf .claude-project/prd/HealthBridge_Pro_PRD_260115.md
```

---

## Notes

- Each run generates a unique, randomized project
- Projects are designed to be realistic and challenging
- Higher difficulty levels include more edge cases and complex user flows
- Generated projects can be used for:
  - Developer training exercises
  - PRD writing practice
  - Design sprint exercises
  - Estimation practice
  - Architecture planning exercises
