# UniVerse API  
A modern, scalable, and production-ready **.NET 9 Web API** powering the UniVerse mobile and web applications.  
This API is deployed on **Render**, uses **PostgreSQL**, supports **JWT authentication**, **Google login**, **file uploads**, **course/assessment management**, **offline caching**, and more.

---

## Table of Contents  
- [Overview](#overview)  
- [Features](#features)  
- [Tech Stack](#tech-stack)  
- [Project Structure](#project-structure)
- [API Endpoints](#api-endpoints) 
- [Environment Variables](#environment-variables)  
- [Running the API Locally](#running-the-api-locally)  
- [Database & Migrations](#database--migrations)  
- [Docker Support](#docker-support)  
- [Deployment (Render)](#deployment-render)  

---

## Overview  
The **UniVerse API** is the backend service responsible for all core functionality of the UniVerse ecosystem.  
It manages:

✔ Authentication (Email/Password + Google OAuth)  
✔ Users & profiles  
✔ Announcements  
✔ Assessments & file attachments  
✔ Courses & Modules  
✔ Chat/Messaging  
✔ Notifications  
✔ Scheduling & Calendar  
✔ File uploads (PDFs, Docs, Images)  
✔ Offline storage support for mobile apps  

The API follows **REST best practices**, includes **Swagger documentation**, and uses **Entity Framework Core** with PostgreSQL.

---

## Features  
- 🔐 **Secure JWT Authentication**  
- 🔑 **Google OAuth Login Support**  
- 📄 **File Uploads (PDF, DOCX, Images, etc.)**  
- 📢 **Announcements Management**  
- 📝 **Assessment Upload & Metadata Tracking**  
- 📚 **Courses & Modules API**  
- 💬 **Messaging**  
- 🗂 **Serve Uploaded Files via Static Hosting (wwwroot/uploads)**  
- 🌐 **Swagger UI** for simple testing  
- 🛠 **Configurable via Environment Variables**  
- 📦 **Docker-ready**  
- ☁️ **Fully deployed and running on Render**

---

## Tech Stack  
| Component | Technology |
|----------|------------|
| Backend Framework | **.NET 9 (C#)** |
| Database | **PostgreSQL** |
| ORM | **Entity Framework Core (Npgsql)** |
| Authentication | **JWT**, **Google OAuth** |
| Hosting | **Render Web Service** |
| File Storage | **Local (wwwroot/uploads)** |
| API Docs | **Swagger / Swashbuckle** |
| Containerization | **Dockerfile included** |

---

## 📁 Project Structure 

UniVerse/
├── Controllers/ # All API route controllers
├── Data/ # DbContext & configuration
├── Models/ # Entities & DTOs
├── Migrations/ # EF Core migrations
├── Services/ # External + internal service classes
├── wwwroot/uploads/ # Folder where files are stored
├── Program.cs # Application startup
├── appsettings.json
└── Dockerfile # Builds the app container

---

## API Endpoints

### 📢 Announcements Controller

- **Get All Announcements**
  - Endpoint: GET /api/announcements
    - Description: Retrieves all announcements sorted by date (newest first)
    - Response: Array of announcement objects

- **Get Announcements by Module**
  - Endpoint: GET /api/announcements/module/{moduleId}
    - Description: Retrieves announcements for a specific module
    - Parameters: moduleId (string) - The module identifier
    - Response: Array of announcement objects for the specified module

- **Create New Announcement**
  - Endpoint: POST /api/announcements
    - Description: Creates a new announcement
    - Body: Announcement object (JSON)
    - Required Fields: ModuleId must be provided
    - Response: Created announcement object

---

### 📝 Assessments Controller

- All assessment endpoints require authentication

- **Get Assessments by Course**
  - Endpoint: GET /assessments/course/{courseId}
    - Description: Retrieves all assessments for a specific course
    - Authentication: Required
    - Parameters: courseId (string) - The course identifier
    - Response: Array of assessment objects with course and submission data

- **Get Assessments by Module**
  - Endpoint: GET /assessments/module/{moduleId}
    - Description: Retrieves all assessments for a specific module
    - Authentication: Required
    - Parameters: moduleId (string) - The module identifier
    - Response: Array of assessment objects with course and submission data

- **Get Assessment by ID**
  - Endpoint: GET /assessments/{assessmentId}
    - Description: Retrieves a specific assessment by its ID
    - Authentication: Required
    - Parameters: assessmentId (integer) - The assessment identifier
    - Response: Assessment object with detailed information

- **Create New Assessment**
  - Endpoint: POST /assessments/create
    - Description: Creates a new assessment (Lecturer role required)
    - Authentication: Required (Lecturer role)
    - Method: Form-data

- **Body Parameters:**
  -Title (string) - Assessment title
  - Description (string) - Assessment description
  - DueDate (DateTime) - Due date in UTC
  - MaxMarks (decimal) - Maximum marks
  - CourseID (string) - Course identifier
  - ModuleID (string, optional) - Module identifier
  - file (file, optional) - Attachment file (max ~50MB)
  - Response: Created assessment details with file URL

### 🔐 Authentication Notes

- Assessment endpoints require valid JWT token
- Create assessment endpoint requires "Lecturer" role
- File uploads are stored in /uploads/ directory with unique filenames

### 🗂️ Response Format

All endpoints return JSON responses with appropriate HTTP status codes:
- 200 OK - Successful request
- 400 Bad Request - Invalid input/data
- 401 Unauthorized - Authentication required
- 403 Forbidden - Insufficient permissions
- 404 Not Found - Resource not found
- 500 Internal Server Error - Server-side error

---

### 📅 Calendar Controller

All calendar endpoints require authentication

- **Get All Calendar Events**
  - Endpoint: GET /api/calendar
    - Description: Retrieves all calendar events sorted by start time
    - Authentication: Required
    - Response: Array of calendar event objects with user information

- **Get Calendar Event by ID**
  - Endpoint: GET /api/calendar/{id}
    - Description: Retrieves a specific calendar event by its ID
    - Authentication: Required
    - Parameters: id (string) - The event identifier
    - Response: Calendar event object with user details

- **Create Calendar Event**
  - Endpoint: POST /api/calendar/events
    - Description: Creates a new calendar event (Lecturer role required)
    - Authentication: Required (Lecturer role)
    - Body: CalendarEvent object (JSON)
    - Required Fields: UserID must reference an existing user
    - Response: Success message with generated event ID

- **Update Calendar Event**
  - Endpoint: PUT /api/calendar/{id}
    - Description: Updates an existing calendar event (Lecturer role required)
    - Authentication: Required (Lecturer role)
    - Parameters: id (string) - The event identifier to update
    - Body: Updated CalendarEvent object (JSON)
    - Response: Success message

- **Delete Calendar Event**
  - Endpoint: DELETE /api/calendar/{id}
    - Description: Deletes a calendar event (Lecturer role required)
    - Authentication: Required (Lecturer role)
    - Parameters: id (string) - The event identifier to delete
    - Response: Success message

---

### 🎓 Courses Controller

- **Get All Courses**
  - Endpoint: GET /courses
    - Description: Retrieves all courses with lecturer, modules, and assessments
    - Authentication: Required
    - Response: Array of course objects with complete details

- **Get Course by ID**
  - Endpoint: GET /courses/{id}
    - Description: Retrieves a specific course by its ID
    - Authentication: Required
    - Parameters: id (string) - The course identifier
    - Response: Course object with lecturer, modules, and assessments

- **Create New Course**
  - Endpoint: POST /courses/create
    - Description: Creates a new course (Lecturer role required)
    - Authentication: Required (Lecturer role)
    - Body: Course object (JSON)
    - Response: Success message

- **Enroll in Course**
  - Endpoint: POST /courses/{id}/enroll
    - Description: Enrolls the authenticated student in a course
    - Authentication: Required (Student role)
    - Parameters: id (string) - The course identifier to enroll in
    - Response: Success message or error if already enrolled

- **Get Enrolled Courses**
  - Endpoint: GET /courses/enrolled
    - Description: Retrieves all courses the authenticated student is enrolled in
    - Authentication: Required (Student role)
    - Response: Array of enrolled course objects with complete details

---

### 📁 Files Controller

- All file endpoints require authentication

- **Upload File**
  - Endpoint: POST /files/upload
  - Description: Uploads a new file record to the system
  - Authentication: Required
  - Body: ClassFile object (JSON)
  - Response: Success message

- **Get File by ID**
  - Endpoint: GET /files/{id}
    - Description: Retrieves a specific file by its ID
    - Authentication: Required
    - Parameters: id (integer) - The file identifier
    - Response: File object with uploader information

- **Get Files by Uploader**
  - Endpoint: GET /files/uploader/{userId}
    - Description: Retrieves all files uploaded by a specific user
    - Authentication: Required
    - Parameters: userId (integer) - The user identifier
    - Response: Array of file objects with uploader details

### 🔐 Authentication & Role Requirements

- All endpoints require valid JWT token authentication
- Lecturer-only endpoints: Calendar create/update/delete, Course creation
- Student-only endpoints: Course enrollment, Enrolled courses list
- User ID is automatically extracted from JWT token for enrollment

### 🗂️ Response Format

- All endpoints return JSON responses with appropriate HTTP status codes:
  - 200 OK - Successful request
  - 400 Bad Request - Invalid input/data
  - 401 Unauthorized - Authentication required
  - 403 Forbidden - Insufficient permissions
  - 404 Not Found - Resource not found
  - 500 Internal Server Error - Server-side error

### 📋 Data Models

- Calendar Events: Include title, type, start/end times, color coding, description
- Courses: Include modules, assessments, lecturer information, dates
- Files: Include uploader information, file metadata

---

### 🎮 Gamification Controller
All gamification endpoints require authentication

- **Get User Stats**
  - Endpoint: GET /api/gamification/user/{userId}
    - Description: Retrieves gamification statistics, badges, and rewards for a specific user
    - Authentication: Required
    - Parameters: userId (integer) - The user identifier
    - Response: UserStatsDto with points, streak, badges, and redeemed rewards

- **Get Leaderboard**
  - Endpoint: GET /api/gamification/leaderboard
    - Description: Retrieves top 50 users ranked by points
    - Authentication: Required
    - Response: Array of LeaderboardEntryDto objects

- **Get Available Rewards**
  - Endpoint: GET /api/gamification/rewards
    - Description: Retrieves all rewards available for redemption
    - Authentication: Required
    - Response: Array of RewardDto objects sorted by cost

- **Add Points to User**
  - Endpoint: POST /api/gamification/user/{userId}/addpoints?points={points}
    - Description: Adds points to a user's score and checks for badge awards
    - Authentication: Required
  - Parameters:
    - userId (integer) - The user identifier
    - points (integer) - Number of points to add (query parameter)
    - Response: Updated UserStatsDto

- **Play Mini-Game**
  - Endpoint: POST /api/gamification/user/{userId}/play?gameType={type}
    - Description: Plays a mini-game and awards random points
    - Authentication: Required
  - Parameters:
    - userId (integer) - The user identifier
    - gameType (string) - Type of game: "spin" (default), "math", or "guess"
    - Response: GamePlayResultDto with points awarded and message

- **Redeem Reward**
  - Endpoint: POST /api/gamification/user/{userId}/redeem/{rewardId}
    - Description: Redeems a reward using user's points
    - Authentication: Required
  - Parameters:
    - userId (integer) - The user identifier
    - rewardId (integer) - The reward identifier to redeem
    - Response: RedeemResponseDto with success status and updated stats

---

### 💬 Messages Controller
  - **Get User Messages**
    - Endpoint: GET /messages/{userId}
    - Description: Retrieves all messages for a specific user (both sent and received)
    - Authentication: Not required (consider adding for production)
    - Parameters: userId (integer) - The user identifier
    - Response: Array of MessageResponseDto objects sorted by most recent

- **Get Conversation**
  - Endpoint: GET /messages/conversation/{senderId}/{receiverId}
    - Description: Retrieves conversation history between two users
    - Authentication: Not required (consider adding for production)
  - Parameters:
    - senderId (integer) - First user identifier
    - receiverId (integer) - Second user identifier
    - Response: Array of MessageResponseDto objects in chronological order

- **Send Message**
  - Endpoint: POST /messages
    - Description: Sends a new message between users
    - Authentication: Not required (consider adding for production)
    - Body: MessageRequest object (JSON)
    - Required Fields: SenderID, ReceiverID, Content
    - Response: MessageResponseDto of the created message

- **Mark Message as Read**
  - Endpoint: PUT /messages/{id}/read
    - Description: Updates a message's read status to "Read"
    - Authentication: Not required (consider adding for production)
  - Parameters: id (integer) - The message identifier
  - Response: Success message

---

### 📚 Modules Controller
All module endpoints require authentication

- **Get All Modules**
  - Endpoint: GET /modules
    - Description: Retrieves all modules with course information
    - Authentication: Required
    - Response: Array of module objects with course details

- **Get Module by ID**
  - Endpoint: GET /modules/{id}
    - Description: Retrieves a specific module by its ID
    - Authentication: Required
    - Parameters: id (string) - The module identifier
    - Response: Module object with course information

- **Create Module**
  - Endpoint: POST /modules/create
    - Description: Creates a new module (Lecturer role required)
    - Authentication: Required (Lecturer role)
    - Body: Module object (JSON)
    - Response: Success message

- **Get Modules by Course**
  - Endpoint: GET /modules/byCourse/{courseId}
    - Description: Retrieves all modules for a specific course
    - Authentication: Required
    - Parameters: courseId (string) - The course identifier
    - Response: Array of ModuleDto objects

---

### 🔔 Notifications Controller

- **Get All Notifications**
  - Endpoint: GET /notifications
    - Description: Retrieves all notifications with user information
    - Authentication: Required
    - Response: Array of NotificationDTO objects

- **Create Notification**
  - Endpoint: POST /notifications/create
    - Description: Creates a new notification (Lecturer role required)
    - Authentication: Required (Lecturer role)
    - Body: Notification object (JSON)
    - Response: Success message

---

### 🏆 Gamification Features

- Badges: Automatically awarded for achievements (FIRST_WIN, SCORE_100, STREAK_7)
- Points System: Earn points through mini-games and activities
- Rewards: Redeem points for available rewards
- Leaderboard: Competitive ranking system

### 💌 Message Features

- Conversation Management: Full conversation history between users
- Read Status: Track message delivery and reading
- File Attachments: Support for file attachments in messages
- User Validation: Ensures sender and receiver exist

### 📖 Module Features

- Course Integration: Modules linked to specific courses
- Content Support: Various content types and links
- Completion Tracking: Track module completion status

### 🔔 Notification Features

- Priority Levels: Different notification priorities
- Status Tracking: Notification delivery status
- User Association: Notifications linked to specific users

### 🔐 Authentication Notes

- Gamification, Modules, Notifications: Require authentication
- Messages: Currently no authentication (recommend adding for production)
- Lecturer-only endpoints: Module creation, Notification creation

### 🗂️ Response Format
All endpoints return JSON responses with appropriate HTTP status codes:
  - 200 OK - Successful request
  - 400 Bad Request - Invalid input/data
  - 401 Unauthorized - Authentication required
  - 403 Forbidden - Insufficient permissions
  - 404 Not Found - Resource not found
  - 500 Internal Server Error - Server-side error

---

## 🔑 Environment Variables

| Key | Description |
|-----|-------------|
| `ConnectionStrings__DefaultConnection` | PostgreSQL connection string |
| `Jwt__Key` | Secret key for signing JWT tokens |
| `Jwt__Issuer` | JWT issuer |
| `Jwt__Audience` | JWT audience |
| `Authentication__Google__ClientId` | Google OAuth client ID |
| `Authentication__Google__ClientSecret` | Google OAuth secret |
| `ASPNETCORE_ENVIRONMENT` | Development/Production |
| `PORT` | Render automatically sets this |

