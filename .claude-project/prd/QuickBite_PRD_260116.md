# QuickBite PRD (2026-01-16)

---

# Part 1: Basic Information

## Title
QuickBite

## Terminology

| Term | Definition |
|------|------------|
| Order | A transaction consisting of one or more menu items from a single restaurant |
| Cart | Temporary storage of items before checkout |
| Prep Time | Estimated time for restaurant to prepare the order |
| Delivery Zone | Geographic area within which a restaurant offers delivery |
| Surge Pricing | Dynamic pricing during high-demand periods |
| Split Payment | Feature allowing multiple users to pay portions of a group order |
| Kitchen Status | Real-time preparation stage of an order (received, preparing, ready) |
| Delivery ETA | Estimated time of arrival for the delivery driver |
| Eco Delivery | Low-carbon delivery option using bicycle or walking couriers |
| Hot Bag | Insulated container used to keep food warm during delivery |

## Project Information

### Description
QuickBite is a hyper-local food delivery platform focusing on neighborhood restaurants within a 3km radius. The app connects customers with local restaurants and delivery drivers, featuring real-time kitchen prep status updates, eco-friendly delivery options, and group ordering capabilities with split payment functionality.

### Goals
1. Provide seamless food ordering experience for customers from local restaurants
2. Enable restaurant owners to manage orders, menus, and track sales efficiently
3. Create a reliable delivery platform for drivers with clear earnings tracking
4. Differentiate through eco-friendly delivery options and real-time kitchen status updates

### User Types
- **Customer**: Browse restaurants, place orders, track deliveries, rate orders, manage payment methods, view order history
- **Restaurant Owner**: Manage menu items, receive and process orders, view sales analytics, set availability status, respond to reviews
- **Delivery Driver**: Accept delivery requests, update delivery status, view earnings, navigate to destinations, mark orders as delivered
- **Admin**: Full platform management, user management, analytics, support handling

### User Relationships
- Customer to Restaurant: 1:N (one customer can order from many restaurants)
- Customer to Driver: 1:N per order (one customer's order assigned to one driver)
- Restaurant to Driver: N:N (restaurants receive deliveries from multiple drivers, drivers deliver for multiple restaurants)
- All users operate independently but interact through order transactions

### Project Type
- Mobile App (iOS/Android) - React Native
- Web Application - Next.js
- Admin Dashboard - Next.js

## System Modules (Step-by-step Flows)

### Module 1 - Place Order (Customer)
1. Customer selects restaurant from list or search
2. Customer browses menu and adds items to cart
3. Customer reviews cart and applies promo code if available
4. Customer selects delivery address and time (ASAP or scheduled)
5. Customer selects payment method and confirms order
6. System sends order to restaurant for confirmation
7. Restaurant accepts order and estimated prep time is shown
8. Customer receives real-time status updates

### Module 2 - Order Management (Restaurant Owner)
1. Restaurant receives push notification for new order
2. Owner views order details (items, special requests, customer notes)
3. Owner accepts order and sets estimated prep time
4. System notifies customer of acceptance
5. Owner marks order as "preparing"
6. Owner marks order as "ready for pickup"
7. System assigns available driver for delivery

### Module 3 - Delivery Flow (Driver)
1. Driver receives notification of available delivery nearby
2. Driver views order details (restaurant, customer location, estimated pay)
3. Driver accepts delivery assignment
4. Driver navigates to restaurant
5. Driver marks "arrived at restaurant"
6. Driver picks up order and marks "picked up"
7. Driver navigates to customer location
8. Driver marks "delivered" and takes confirmation photo
9. System completes transaction and updates earnings

### Module 4 - Group Ordering (Customer)
1. Customer creates a group order and generates invite link
2. Customer shares invite link with friends/colleagues
3. Invited users join and add their items to shared cart
4. Order organizer reviews combined order
5. System calculates split payment amounts
6. Each participant pays their portion
7. Order is submitted to restaurant as single order
8. All participants receive tracking updates

## 3rd Party API List
- Google OAuth: Social authentication for customers
- Apple Sign-in: Social authentication for iOS users
- Kakao Login: Social authentication for Korean users
- Toss Payments: Payment processing
- Kakao Pay: Alternative payment processing
- Google Maps: Location services and navigation
- Kakao Maps: Alternative location services for Korean users
- Firebase: Push notifications

---

# Part 2: User Application PRD

## User Types: Customer, Restaurant Owner, Delivery Driver
All user types access the platform through dedicated mobile applications with role-specific interfaces.

## 1. Common

### Splash Page
- Design: App logo with loading animation
- Duration: 2-3 seconds while checking authentication status

### Login Page
- **Input**:
  - Email or Phone number
  - Password
- **Social Login Options**:
  - Google
  - Apple
  - Kakao
- **Next Action**:
  - Validate credentials
  - On success: Navigate to user-type specific home
  - On failure: Display error message with retry option
  - "Forgot Password" link available

### Forgot Password Page
- **Main**
  - Input: Email or Phone number
  - Submit button sends reset link/code
- **Reset Password Page**
  - Input: New password, Confirm password
  - Password strength indicator
  - Submit button saves new password

### Sign Up Page
- **Input**:
  - Name (required)
  - Email or Phone number (required)
  - Password (required)
  - Profile photo (optional)
  - Default delivery address (optional - Customer only)
- **User Type Selection**:
  - Customer
  - Restaurant Owner (requires business verification)
  - Delivery Driver (requires identity verification)
- **Rules**:
  - Password minimum 8 characters
  - Email format validation
  - Phone number format validation
  - Terms of service agreement required

---

## 2. Customer

### 2.1 Navigation Menu
1. Home (Restaurant Discovery)
2. Orders (Order History & Tracking)
3. Favorites
4. Profile

### 2.2 Page Architecture & Feature Specification

#### 1. Home Tab

**Main Page (Restaurant Discovery)**
1) Location Selector
   - Current location display
   - Tap to change delivery address
   - Saved addresses list

2) Search Bar
   - Search restaurants by name
   - Search menu items
   - Recent searches history

3) Filter Section
   - Cuisine type filter (Korean, Chinese, Japanese, Western, etc.)
   - Rating filter (4+ stars, 3+ stars)
   - Delivery time filter (under 30 min, under 45 min)
   - Price range filter ($, $$, $$$)
   - Eco Delivery filter toggle

4) Restaurant List
   - Restaurant card component:
     - Restaurant image
     - Restaurant name
     - Rating (stars + review count)
     - Cuisine type tags
     - Estimated delivery time
     - Delivery fee
     - Eco Delivery badge (if available)
   - Infinite scroll pagination

**Restaurant Detail Page**
1) Header Section
   - Restaurant cover image
   - Restaurant name
   - Rating and review count
   - Cuisine tags
   - Operating hours
   - Delivery zone info
   - Favorite button (heart icon)

2) Menu Section
   - Category tabs (horizontal scroll)
   - Menu item cards:
     - Item image
     - Item name
     - Description
     - Price
     - Popular/Best Seller badge
     - Add to cart button

3) Reviews Section
   - Average rating display
   - Review list with pagination
   - Photo reviews filter
   - Sort by: Recent, Highest rated, Lowest rated

**Menu Item Detail Page**
- Large item image
- Item name and description
- Base price
- Customization options (size, toppings, etc.)
- Special instructions text input
- Quantity selector
- Add to Cart button with total price

**Cart Page**
1) Cart Items List
   - Item name and customizations
   - Quantity adjuster (+/-)
   - Item price
   - Remove item button

2) Promo Code Section
   - Input field for promo code
   - Apply button
   - Discount display

3) Order Summary
   - Subtotal
   - Delivery fee
   - Discount (if applied)
   - Total

4) Delivery Options
   - Standard Delivery
   - Eco Delivery (bicycle/walking - if available)
   - Scheduled delivery time picker

5) Group Order Section
   - "Start Group Order" button
   - Share invite link
   - View participants and their items

6) Checkout Button

**Checkout Page**
1) Delivery Address
   - Selected address display
   - Change address button
   - Add new address option

2) Payment Method
   - Saved payment methods list
   - Add new payment method
   - Toss Payments integration
   - Kakao Pay integration

3) Order Notes
   - Special instructions for restaurant
   - Delivery instructions for driver

4) Order Summary (collapsed)

5) Place Order Button

**Order Tracking Page**
1) Order Status Timeline
   - Order Placed ✓
   - Restaurant Confirmed
   - Preparing (with Kitchen Status)
   - Ready for Pickup
   - Driver Assigned
   - Driver Picked Up
   - On the Way
   - Delivered

2) Live Map
   - Restaurant location marker
   - Customer location marker
   - Driver location (real-time when in transit)
   - Route display

3) Driver Info Card (when assigned)
   - Driver name and photo
   - Vehicle/delivery method info
   - Chat button
   - Call button

4) Order Details (expandable)
   - Items list
   - Order total

#### 2. Orders Tab

**Main Page (Order History)**
1) Active Orders Section
   - Current orders with status
   - Tap to view tracking

2) Past Orders List
   - Order date
   - Restaurant name
   - Items summary
   - Total amount
   - Order status (Completed/Cancelled)
   - Reorder button
   - View Receipt button

**Order Detail Page**
- Full order information
- Receipt details
- Rate Order button (if not yet rated)
- Help/Support button

**Rate Order Page**
- Overall rating (1-5 stars)
- Food quality rating
- Delivery experience rating
- Photo upload option
- Written review text area
- Submit button

#### 3. Favorites Tab

**Main Page**
1) Favorite Restaurants List
   - Restaurant cards (same as Home)
   - Remove from favorites option

2) Empty State
   - Illustration
   - "No favorites yet" message
   - Browse restaurants button

#### 4. Profile Tab

**Main Page**
1) Profile Header
   - Profile photo
   - User name
   - Edit profile button

2) Settings List
   - Saved Addresses
   - Payment Methods
   - Notifications Settings
   - Language (Korean/English)
   - App Version
   - Help & Support
   - Terms of Service
   - Privacy Policy
   - Logout

**Saved Addresses Page**
- Address list with labels (Home, Work, etc.)
- Add new address
- Edit address
- Delete address
- Set default address

**Payment Methods Page**
- Saved cards/payment methods list
- Add new payment method
- Remove payment method
- Set default payment method

**Notifications Settings Page**
- Order updates toggle
- Promotions toggle
- New restaurants toggle
- Push notification toggle
- Email notification toggle

---

## 3. Restaurant Owner

### 3.1 Navigation Menu
1. Dashboard
2. Orders
3. Menu
4. Reviews
5. Settings

### 3.2 Page Architecture & Feature Specification

#### 1. Dashboard Tab

**Main Page**
1) Status Toggle
   - Open / Closed / Busy status switch
   - Operating hours display

2) Today's Summary Cards
   - Total orders today
   - Total revenue today
   - Average order value
   - Average prep time

3) Active Orders Widget
   - Count of pending orders
   - Count of preparing orders
   - Quick access to Orders tab

4) Weekly Performance Chart
   - Orders trend
   - Revenue trend

#### 2. Orders Tab

**Main Page**
1) Order Status Tabs
   - New (pending acceptance)
   - Preparing
   - Ready for Pickup
   - Completed
   - Cancelled

2) Order Cards
   - Order ID
   - Order time
   - Customer name
   - Items list (collapsed)
   - Total amount
   - Accept/Reject buttons (for new orders)
   - Status update buttons

**Order Detail Page**
1) Order Info
   - Order ID and time
   - Customer name
   - Items with customizations
   - Special instructions
   - Total amount

2) Action Buttons
   - Accept Order (set prep time)
   - Mark as Preparing
   - Mark as Ready
   - Cancel Order (with reason)

3) Driver Info (when assigned)
   - Driver name
   - Contact option
   - Pickup status

#### 3. Menu Tab

**Main Page**
1) Category List
   - Category name
   - Item count
   - Drag to reorder
   - Add category button

2) Menu Items by Category
   - Item card with image
   - Item name and price
   - Available/Unavailable toggle
   - Edit button
   - Delete button

**Add/Edit Item Page**
1) Item Photo
   - Upload/change photo
   - Photo guidelines

2) Item Details
   - Item name
   - Description
   - Category selection
   - Base price

3) Options/Customizations
   - Add option groups (size, toppings, etc.)
   - Option prices
   - Required/Optional toggle
   - Single/Multiple selection

4) Availability
   - Available toggle
   - Scheduled availability (optional)

5) Save/Update Button

**Add/Edit Category Page**
- Category name
- Category description (optional)
- Display order
- Save button

#### 4. Reviews Tab

**Main Page**
1) Rating Summary
   - Overall rating
   - Rating distribution chart
   - Total review count

2) Review List
   - Customer rating
   - Review text
   - Review photos (if any)
   - Review date
   - Reply button
   - Replied indicator

**Reply to Review Page**
- Original review display
- Reply text input
- Submit reply button
- Edit existing reply

#### 5. Settings Tab

**Main Page**
1) Restaurant Profile
   - Restaurant name
   - Address
   - Phone number
   - Operating hours
   - Delivery zone settings

2) Notification Settings
   - New order alerts
   - Sound settings
   - Order reminder intervals

3) Promotion Management
   - Active promotions list
   - Create new promotion
   - Edit/Delete promotions

4) Analytics
   - Daily/Weekly/Monthly reports
   - Popular items report
   - Peak hours analysis
   - Export data (CSV)

5) Support
   - Help center
   - Contact support
   - FAQs

**Promotion Management Page**
- Promotion type (% off, fixed amount, free delivery)
- Minimum order amount
- Valid dates
- Usage limit
- Promo code (auto-generated or custom)
- Active/Inactive toggle

---

## 4. Delivery Driver

### 4.1 Navigation Menu
1. Available Deliveries
2. Active Delivery
3. Earnings
4. Profile

### 4.2 Page Architecture & Feature Specification

#### 1. Available Deliveries Tab

**Main Page**
1) Online/Offline Toggle
   - Go online to receive delivery requests
   - Go offline to stop receiving requests

2) Available Delivery Cards
   - Restaurant name and distance
   - Customer distance from restaurant
   - Estimated earnings
   - Delivery type badge (Standard/Eco)
   - Accept button
   - Time limit countdown for acceptance

3) Empty State (when no deliveries)
   - "No deliveries nearby" message
   - Map showing current location

#### 2. Active Delivery Tab

**Main Page (Active Delivery)**
1) Status Progress Bar
   - Heading to Restaurant
   - At Restaurant
   - Picked Up
   - Heading to Customer
   - Delivered

2) Map View
   - Current location
   - Destination marker (restaurant or customer)
   - Route guidance
   - Navigation button (opens Google/Kakao Maps)

3) Order Details Card
   - Restaurant name and address
   - Customer name and address
   - Items list (for verification)
   - Special delivery instructions

4) Action Buttons
   - "Arrived at Restaurant"
   - "Picked Up Order"
   - "Arrived at Customer"
   - "Complete Delivery" (with photo capture)

5) Communication
   - Chat with Customer button
   - Chat with Restaurant button
   - Call buttons

**Delivery Completion Page**
- Photo capture for proof of delivery
- Delivery notes (optional)
- Complete button
- Issue report button

#### 3. Earnings Tab

**Main Page**
1) Earnings Summary
   - Today's earnings
   - This week's earnings
   - This month's earnings

2) Earnings Chart
   - Daily earnings trend
   - Weekly comparison

3) Delivery History List
   - Delivery date/time
   - Restaurant → Customer
   - Earnings amount
   - Delivery type (Standard/Eco)
   - Bonus/Tips (if applicable)

**Earnings Detail Page**
- Base delivery fee
- Distance bonus
- Peak hour bonus
- Customer tip
- Total earnings

#### 4. Profile Tab

**Main Page**
1) Driver Profile
   - Profile photo
   - Name
   - Rating
   - Total deliveries completed
   - Edit profile button

2) Vehicle/Delivery Method
   - Current method (Bicycle, Motorcycle, Walking)
   - Update method

3) Documents
   - ID verification status
   - Background check status
   - Vehicle documents (if applicable)

4) Settings
   - Notification preferences
   - Navigation app preference
   - Language
   - Support/Help
   - Logout

---

# Part 3: Admin Dashboard PRD

## Page Architecture & Feature Specification

### Dashboard Page

**Card Components**
- Total Orders (Today/Week/Month)
- Total Revenue (Today/Week/Month)
- Active Users (Customers/Restaurants/Drivers)
- Average Delivery Time
- Order Success Rate
- New User Registrations

**Chart Components**
- Orders Trend Chart (Line)
- Revenue Trend Chart (Line)
- Orders by Cuisine Type (Pie)
- Peak Hours Heatmap
- Geographic Distribution Map

**Recent Activity Feed**
- New restaurant registrations
- Support tickets
- System alerts

---

## User Management

### Customer Management Page

**Main Page**
1. Top Area:
   - Filters: Status (Active, Suspended, Deleted), Registration date range
   - Search: Name, Email, Phone number
   - Export button (CSV)

2. List Item Component:
   - Customer ID
   - Name
   - Email
   - Phone
   - Registration date
   - Total orders
   - Total spent
   - Status
   - Action column (View, Suspend, Delete)

3. Sort Order: Registration date (newest), Total orders, Total spent

**Detail Drawer**
- Header Info: Profile photo, Name, Contact info
- Sub-tabs:
  - Overview (registration date, last order, lifetime value)
  - Order History (list of all orders)
  - Addresses (saved addresses)
  - Payment Methods (masked card info)
  - Activity Log

---

### Restaurant Management Page

**Main Page**
1. Top Area:
   - Filters: Status (Pending, Active, Suspended), Cuisine type, Rating range
   - Search: Restaurant name, Owner name, Address
   - Create button → Restaurant Onboarding Flow
   - Export button (CSV)

2. List Item Component:
   - Restaurant ID
   - Restaurant name
   - Owner name
   - Cuisine type
   - Rating
   - Total orders
   - Status
   - Action column (View, Approve, Suspend, Delete)

3. Bulk Action: Approve selected (for pending restaurants)

**Creation Modal (Onboarding)**
- Restaurant name
- Owner information
- Business registration upload
- Address
- Cuisine type
- Initial menu setup option

**Detail Drawer**
- Header Info: Restaurant image, Name, Rating, Status
- Sub-tabs:
  - Overview (registration, total orders, revenue)
  - Menu (view/edit menu items)
  - Orders (order history)
  - Reviews (customer reviews)
  - Documents (business verification)
  - Settings (operating hours, delivery zone)

---

### Driver Management Page

**Main Page**
1. Top Area:
   - Filters: Status (Pending Verification, Active, Suspended), Delivery method, Rating range
   - Search: Name, Phone, Email
   - Export button (CSV)

2. List Item Component:
   - Driver ID
   - Name
   - Phone
   - Delivery method
   - Rating
   - Total deliveries
   - Status
   - Verification status
   - Action column (View, Approve, Suspend, Delete)

3. Bulk Action: Approve selected (for pending drivers)

**Detail Drawer**
- Header Info: Photo, Name, Rating, Status
- Sub-tabs:
  - Overview (registration, total deliveries, earnings)
  - Delivery History (completed deliveries)
  - Documents (ID, background check, vehicle docs)
  - Earnings (payment history)
  - Activity Log

---

## Order Management

### Orders Management Page

**Main Page**
1. Top Area:
   - Filters: Status (Pending, Preparing, In Transit, Delivered, Cancelled), Date range, Restaurant, Customer
   - Search: Order ID, Customer name, Restaurant name
   - Export button (CSV)

2. List Item Component:
   - Order ID
   - Order date/time
   - Customer name
   - Restaurant name
   - Driver name
   - Items count
   - Total amount
   - Status
   - Action column (View, Cancel, Refund)

**Detail View**
- Order Information
- Customer details
- Restaurant details
- Driver details
- Items list with prices
- Payment information
- Status timeline
- Action buttons (Issue refund, Contact participants)

---

## Promotion Management

### Promotions Page

**Main Page**
1. Top Area:
   - Filters: Status (Active, Scheduled, Expired), Type (% off, Fixed, Free delivery)
   - Search: Promo code, Promotion name
   - Create button → Creation Modal

2. List Item Component:
   - Promotion ID
   - Promo code
   - Type
   - Discount value
   - Usage count
   - Valid dates
   - Status
   - Action column (View, Edit, Deactivate, Delete)

**Creation Modal**
- Promotion name
- Promo code (auto-generate or custom)
- Discount type and value
- Minimum order amount
- Maximum discount cap
- Valid date range
- Usage limit (per user, total)
- Applicable restaurants (all or specific)
- Target users (all, new users, specific segment)

---

## Support Management

### Support Tickets Page

**Main Page**
1. Top Area:
   - Filters: Status (Open, In Progress, Resolved, Closed), Priority, Category
   - Search: Ticket ID, User name
   - Export button (CSV)

2. List Item Component:
   - Ticket ID
   - User name and type
   - Subject
   - Category
   - Priority
   - Created date
   - Status
   - Assigned to
   - Action column (View, Assign, Close)

**Detail View**
- Ticket information
- User details
- Related order (if applicable)
- Conversation thread
- Internal notes
- Action buttons (Reply, Escalate, Resolve, Close)

---

## Analytics

### Analytics Dashboard

**Reports Available**
1. Sales Report
   - Total revenue by period
   - Revenue by restaurant
   - Revenue by cuisine type
   - Average order value trends

2. User Report
   - User growth trends
   - User retention rates
   - Active user metrics
   - User demographics

3. Operations Report
   - Average delivery time
   - Order success rate
   - Cancellation reasons
   - Driver performance metrics

4. Restaurant Report
   - Top performing restaurants
   - Restaurant ratings trends
   - Menu item popularity
   - Restaurant churn

**Export Options**
- Date range selector
- Format: CSV
- Schedule automated reports (daily, weekly, monthly)

---

## Export / Data Download

**Data Download**
- Customer Data: User profiles, order history
- Restaurant Data: Restaurant info, menu items, reviews
- Order Data: All orders with details
- Driver Data: Driver profiles, delivery history
- Financial Data: Revenue, payments, refunds
- Format: CSV
- Filter: All time / Custom date range

---

# Additional Questions (Client Confirmation Required)

## Required Clarifications

| # | Question | Context |
|:-:|:---------|:--------|
| 1 | What is the exact delivery zone radius configuration? Is 3km the only option or can restaurants set custom zones? | Need to determine delivery zone logic and restaurant settings |
| 2 | How does surge pricing work? What triggers it and what are the multipliers? | Listed in terminology but no details on implementation |
| 3 | What is the commission structure for restaurants and drivers? | Critical for business logic but not specified |
| 4 | What verification documents are required for restaurant onboarding? | Business verification mentioned but specifics unclear |
| 5 | What are the minimum/maximum scheduled delivery time windows? | Scheduled delivery mentioned but time constraints unclear |
| 6 | How long does a driver have to accept a delivery request before it's reassigned? | Timeout logic needed for driver assignment |

## Recommended Clarifications

| # | Question | Context |
|:-:|:---------|:--------|
| 1 | Should customers be able to tip drivers? If so, when and how? | Common feature but not explicitly mentioned |
| 2 | What happens if a restaurant rejects an order? Full refund? | Order rejection flow needs clarification |
| 3 | Can customers cancel orders? At what stages and with what penalties? | Cancellation policy not specified |
| 4 | How does the eco delivery bonus/pricing work for drivers? | Eco delivery mentioned but incentive structure unclear |
| 5 | What is the dispute resolution process for order issues? | Support process not detailed |
| 6 | Are there any loyalty/rewards program plans? | Could affect user engagement features |
| 7 | What metrics determine a driver's rating calculation? | Rating system details not specified |
| 8 | How are delivery fees calculated? Distance-based, flat rate, or hybrid? | Delivery fee structure not detailed |

---

# Feature Change Log

## Version 1.0 (2026-01-16)

| Change Type | Before | After | Source |
|:-----------|:-------|:------|:-------|
| **Initial Release** | - | Full PRD created | QuickBite_260116_153135.md |

### Change Details
#### Initial PRD Creation
- **Source Document**: QuickBite_260116_153135.md (Training project)
- **Change Description**: Complete PRD generated from client requirements document including all three user types (Customer, Restaurant Owner, Delivery Driver) plus Admin Dashboard specifications.

---

*Generated by /generate-prd*
*Source: /Users/dongsub/Documents/Potential/projects/claude-base/.claude-project/training/QuickBite_260116_153135.md*
