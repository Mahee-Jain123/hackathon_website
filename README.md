# Hackathon Website Backend

Backend API for a hackathon/team-registration platform built with **Node.js, Express.js, MongoDB, JWT authentication, bcrypt, and Zod**.

The backend provides separate access for **team leaders/users** and **administrators**:

* **User/Team Leader Panel** → Register a hackathon team.
* **Admin Panel** → View registered teams and their information.
* **Authentication** → User registration and login using JWT.
* **Authorization** → Protected admin-only endpoints.
* **Validation** → Request validation using Zod.
* **Database** → MongoDB with Mongoose.

---

## Tech Stack

| Technology | Purpose               |
| ---------- | --------------------- |
| Node.js    | JavaScript runtime    |
| Express.js | Backend/API framework |
| MongoDB    | Database              |
| Mongoose   | MongoDB ODM           |
| JWT        | Authentication        |
| bcrypt     | Password hashing      |
| Zod        | Request validation    |
| dotenv     | Environment variables |

---

## Project Structure

```text
hackathon_website/
│
├── config/
│   └── db.js
│
├── controllers/
│   ├── authController.js
│   └── teamController.js
│
├── middlewares/
│   └── authMiddleware.js
│
├── model/
│   ├── user.js
│   └── team.js
│
├── routes/
│   ├── authRoutes.js
│   ├── teamRoutes.js
│   └── adminRoutes.js
│
├── validators/
│   └── teamValidator.js
│
├── .env
├── .gitignore
├── index.js
├── package.json
└── package-lock.json
```

---

## Features

### 1. User Registration

Users can create an account with:

* Username
* Email
* Password

Passwords are hashed using **bcrypt** before being stored in MongoDB.

New users receive the default role:

```text
team leader
```

The role is not accepted directly from the registration request, preventing users from simply registering themselves as administrators.

---

### 2. User Login

Registered users can log in using their email and password.

After successful authentication, the server generates a **JWT access token**.

The token is then used to access protected routes.

Example authentication flow:

```text
Register
   ↓
Password hashed with bcrypt
   ↓
User stored in MongoDB
   ↓
Login
   ↓
Password verification
   ↓
JWT generated
   ↓
JWT used for protected routes
```

---

### 3. Team Registration

Authenticated users can submit their hackathon team information.

Example request:

```json
{
  "teamName": "Code Warriors",
  "teamLeaderEmail": "leader@example.com",
  "members": [
    {
      "name": "Alice",
      "email": "alice@example.com"
    },
    {
      "name": "Bob",
      "email": "bob@example.com"
    }
  ]
}
```

Team registration data is validated before being stored in MongoDB.

---

### 4. Admin-Only Team Access

The admin has access to registered team information.

The team listing endpoint is protected by both:

```text
JWT Authentication
        +
Admin Authorization
```

The request flow is:

```text
Request
   ↓
JWT verification
   ↓
Check user role
   ↓
Is role = admin?
   ↓
Yes → Access team database
No  → Access denied
```

This prevents normal users/team leaders from accessing all registered teams.

---

## API Endpoints

### Authentication

#### Register

```http
POST /api/auth/register
```

Example body:

```json
{
  "username": "mahee",
  "email": "mahee@example.com",
  "password": "password123"
}
```

---

#### Login

```http
POST /api/auth/login
```

Example body:

```json
{
  "email": "mahee@example.com",
  "password": "password123"
}
```

A successful login returns a JWT token.

Use the token for protected endpoints:

```http
Authorization: Bearer <your_token>
```

---

### Team

#### Register Team

```http
POST /api/teams/register
```

Example:

```json
{
  "teamName": "Code Warriors",
  "teamLeaderEmail": "leader@example.com",
  "members": [
    {
      "name": "Alice",
      "email": "alice@example.com"
    },
    {
      "name": "Bob",
      "email": "bob@example.com"
    }
  ]
}
```

---

#### Get All Teams

```http
GET /api/teams
```

This endpoint is restricted to administrators.

Required:

```http
Authorization: Bearer <admin_jwt>
```

A normal team leader/user should receive an authorization error.

---

## Authentication & Authorization

The project uses **JWT-based authentication**.

### Authentication

Authentication answers:

> "Who is this user?"

The JWT contains information about the authenticated user.

### Authorization

Authorization answers:

> "What is this user allowed to do?"

The application currently uses roles such as:

```text
admin
team leader
```

For example:

```text
Team Leader
   └── Register team

Admin
   └── View registered teams
```

---

## Database Models

### User

The User model contains information such as:

```text
username
email
password
role
createdAt
updatedAt
```

The password is stored as a bcrypt hash rather than plain text.

---

### Team

The Team model contains information such as:

```text
teamName
teamLeader
teamLeaderEmail
members
createdAt
updatedAt
```

Each member contains:

```text
name
email
```

---

## Validation

Team registration requests are validated using **Zod** before reaching the database.

Validation includes checks such as:

* Required team name
* Valid team leader email
* Valid member information
* Valid member email addresses
* Required members

Invalid requests are rejected instead of being directly inserted into MongoDB.

---

## Environment Variables

Create a `.env` file in the project root.

Example:

```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
```

Do **not** commit your `.env` file to GitHub.

The `.gitignore` should contain:

```text
node_modules/
.env
```

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/Mahee-Jain123/hackathon_website.git
```

### 2. Enter the project

```bash
cd hackathon_website
```

### 3. Install dependencies

```bash
npm install
```

### 4. Configure environment variables

Create:

```text
.env
```

and add your MongoDB connection string, JWT secret, and port.

### 5. Start the server

```bash
node index.js
```

The server runs on:

```text
http://localhost:5000
```

---

## Testing

The API can be tested using tools such as:

* Thunder Client
* Postman
* cURL

### Basic test flow

```text
1. Start server
       ↓
2. Register user
       ↓
3. Login
       ↓
4. Copy JWT
       ↓
5. Send authenticated request
       ↓
6. Register team
       ↓
7. Create/use admin account
       ↓
8. Login as admin
       ↓
9. Access GET /api/teams
```

---

## Security

The project currently implements several basic security measures:

* Password hashing using bcrypt
* JWT authentication
* Role-based authorization
* Environment variables for secrets
* Input validation with Zod
* Admin-only access to team information
* Unique user email/username constraints
* Unique team name constraint

---

## Current Architecture

```text
                ┌──────────────────┐
                │     Frontend     │
                └────────┬─────────┘
                         │
                         ▼
                ┌──────────────────┐
                │   Express API    │
                └────────┬─────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       Authentication          Team Routes
              │                     │
              ▼                     ▼
          JWT/Bcrypt            Validation
                                    │
                                    ▼
                              Controllers
                                    │
                                    ▼
                                Mongoose
                                    │
                                    ▼
                               MongoDB Atlas
```

---

## Project Status

### Implemented

* [x] Express server
* [x] MongoDB connection
* [x] User model
* [x] Team model
* [x] User registration
* [x] User login
* [x] Password hashing
* [x] JWT authentication
* [x] Role-based authorization
* [x] Team registration
* [x] Zod team validation
* [x] Admin-only team access
* [x] Environment configuration

---

## Author

**Mahee Jain**

GitHub: [Mahee-Jain123](https://github.com/Mahee-Jain123)

Repository: [hackathon_website](https://github.com/Mahee-Jain123/hackathon_website)

