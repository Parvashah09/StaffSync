# StaffSync

StaffSync is an enterprise employee management system designed to streamline human resource operations, attendance tracking, leave processing, and payroll generation. The platform features role-based access control for administrators and employees, coupled with asynchronous background task processing for automated notifications and compliance workflows.

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Environment Configuration](#environment-configuration)
  - [Installation and Setup](#installation-and-setup)
  - [Database Seeding](#database-seeding)
  - [Running the Application](#running-the-application)
- [API Reference](#api-reference)
- [Background Workflows (Inngest)](#background-workflows-inngest)
- [Deployment](#deployment)
- [License](#license)

---

## Overview

StaffSync provides organizations with a centralized platform to manage employee records, track daily check-ins and working hours, process leave requests with multi-tier approval workflows, and generate printable employee payslips. It also integrates event-driven workflows for automated checkout, absence detection, and admin reminders.

---

## Key Features

### Role-Based Access Control
- Dedicated login portals for Administrators and Employees.
- JWT-based authentication with role verification middleware.
- Protected client-side routing based on authentication state.

### Employee Management
- Directory listing with department-based filtering and status tracking.
- Profile management for personal information, employment records, and banking details.
- Soft-deletion safeguards to preserve historical attendance and payroll data.

### Attendance Tracking
- One-click daily check-in and check-out with automatic working hours computation.
- Half-day, full-day, and late attendance classification.
- Scheduled morning attendance checks that identify absent staff and issue reminder notifications.
- Automatic checkout enforcement for open attendance sessions exceeding designated shift lengths.

### Leave Management
- Employee leave application submission specifying leave type, dates, and rationale.
- Administrative review workflow with approval and rejection actions.
- Automated escalations sent to administrators for unreviewed requests exceeding 24 hours.

### Payroll and Payslips
- Salary computation with basic salary, allowances, and statutory deductions.
- Generation and retrieval of monthly payslip records.
- Standardized print-ready payslip view suitable for archiving or physical distribution.

### Automated Notifications
- Email dispatch through Nodemailer for attendance reminders, checkout alerts, and leave status updates.

---

## System Architecture

The application follows a client-server architecture decoupled into two primary components:

1. **Client (Single Page Application)**: Developed using React 19, Vite, and Tailwind CSS. Communicates with the backend through REST endpoints using Axios.
2. **Server (REST API & Worker)**: Developed using Express 5 and Node.js. Connects to MongoDB via Mongoose for persistence and integrates Inngest to manage event-driven queues and recurring cron routines.

---

## Technology Stack

### Frontend
- **Framework**: React 19
- **Build Tool**: Vite
- **Styling**: Tailwind CSS v4
- **Routing**: React Router v7
- **HTTP Client**: Axios
- **Date Handling**: date-fns
- **Notifications**: react-hot-toast
- **Icons**: Lucide React

### Backend
- **Runtime**: Node.js (ES Modules)
- **Framework**: Express 5
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: JSON Web Tokens (JWT) and bcrypt password hashing
- **Background Jobs**: Inngest
- **File Handling**: Multer
- **Email Service**: Nodemailer

---

## Project Structure

```text
StaffSync/
├── client/                     # Frontend client application
│   ├── public/                 # Static assets and favicons
│   ├── src/
│   │   ├── api/                # Axios instance configuration
│   │   ├── assets/             # Global styles and static references
│   │   ├── components/         # Reusable UI elements (modals, cards, forms)
│   │   │   ├── attendance/     # Attendance-specific components
│   │   │   ├── leave/          # Leave management components
│   │   │   └── payslip/        # Payslip generation and list components
│   │   ├── context/            # Authentication and session state context
│   │   ├── pages/              # Primary route views
│   │   ├── App.jsx             # Route definitions and layout assembly
│   │   └── main.jsx            # React root mount
│   ├── package.json
│   ├── vite.config.js
│   └── vercel.json
│
├── server/                     # Backend API server
│   ├── config/                 # Database and mail transport configurations
│   ├── constants/              # Department and role enumerations
│   ├── controllers/            # Route request handlers
│   ├── inngest/                # Background function and cron declarations
│   ├── middleware/             # Authentication and authorization guards
│   ├── models/                 # Mongoose schemas (User, Employee, Attendance, etc.)
│   ├── routes/                 # Express route definitions
│   ├── seed.js                 # Initial admin account seed utility
│   ├── server.js               # Application entry point
│   ├── package.json
│   └── vercel.json
│
├── .gitignore
└── README.md
```

---

## Getting Started

### Prerequisites

Ensure the following runtimes and services are available on your development machine:
- Node.js (version 18 or later recommended)
- npm or yarn package manager
- A local or hosted MongoDB instance (MongoDB Atlas)
- An SMTP account or mail service (for transactional emails)

---

### Environment Configuration

#### 1. Server Configuration (`server/.env`)

Create a `.env` file in the `server` directory with the following variables:

```env
PORT=4000
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/staffsync?retryWrites=true&w=majority
JWT_SECRET=your_jwt_secret_key_here
ADMIN_EMAIL=admin@staffsync.com
SMTP_USER=your_smtp_username
SMTP_PASS=your_smtp_password
SENDER_EMAIL=noreply@staffsync.com
```

#### 2. Client Configuration (`client/.env`)

Create a `.env` file in the `client` directory:

```env
VITE_BASE_URL=http://localhost:4000
```

---

### Installation and Setup

Clone the repository and install dependencies for both the frontend and backend services:

```bash
# Clone the repository
git clone https://github.com/Parvashah09/StaffSync.git
cd StaffSync

# Install server dependencies
cd server
npm install

# Install client dependencies
cd ../client
npm install
```

---

### Database Seeding

Initialize the default administrative user by executing the seed script located in the `server` directory:

```bash
cd server
npm run seed
```

This provisions an administrative record using the email specified in `ADMIN_EMAIL` and a default temporary password (`admin123`). Change the password immediately upon initial authentication via the Settings interface.

---

### Running the Application

#### 1. Start the Backend Service
From the `server` directory:
```bash
# Development mode with auto-reload
npm run server

# Production mode
npm start
```
The server will bind to `http://localhost:4000`.

#### 2. Start the Frontend Service
From the `client` directory:
```bash
npm run dev
```
The client interface will become accessible at `http://localhost:5173`.

#### 3. Inngest Development Server (Optional for Local Background Jobs)
To execute and inspect background functions locally, launch the Inngest Dev Server:
```bash
npx inngest-cli@latest dev -u http://localhost:4000/api/inngest
```

---

## API Reference

The server exposes RESTful endpoints organized under `/api`:

| Base Path | Description | Access |
|---|---|---|
| `/api/auth` | Login authentication, token verification, password updates | Public / Authenticated |
| `/api/employees` | Employee records CRUD, status modifications, department queries | Admin |
| `/api/profile` | Authenticated user profile retrieval and modifications | Authenticated |
| `/api/attendance` | Check-in, check-out, monthly logs, and team attendance stats | Authenticated / Admin |
| `/api/leave` | Leave applications, pending approvals, history, and status updates | Authenticated / Admin |
| `/api/payslips` | Payslip creation, individual retrieval, and organizational summaries | Authenticated / Admin |
| `/api/dashboard` | Aggregated statistical metrics and administrative summaries | Admin |
| `/api/inngest` | Inngest event webhook and function execution gateway | Public / Inngest Signature |

---

## Background Workflows (Inngest)

StaffSync employs Inngest to manage event-driven background routines:

- **Automated Check-out (`auto-check-out`)**: Triggered upon an employee check-in event. Waits for 9 hours; if no check-out was recorded, sends a reminder notification. After 10 hours of inactivity, closes the shift session automatically with late/half-day indicators.
- **Leave Escalation (`leave-application-reminder`)**: Dispatched when a leave request is submitted. If the application remains in a pending state after 24 hours, an escalation notification is routed to the administrator.
- **Attendance Deadline Audit (`attendance-reminder-cron`)**: A scheduled cron job running daily at 11:30 AM IST (`TZ=Asia/Kolkata 30 11 * * *`). It identifies active employees who are neither checked in nor on an approved leave, dispatching attendance alert emails.

---

## Deployment

### Vercel Deployment

Both `client/` and `server/` contain configuration files (`vercel.json`) supporting deployment on the Vercel platform:

1. **Backend**:
   - Deploy the `server` directory as a Node.js project.
   - Configure all environment variables (`MONGODB_URI`, `JWT_SECRET`, `ADMIN_EMAIL`, etc.) in the Vercel Project Settings.
2. **Frontend**:
   - Deploy the `client` directory.
   - Configure `VITE_BASE_URL` to point to the production API URL.

---