# AutoMedic

A RESTful backend API for a medical monitoring platform built as a graduation project. It manages users with role-based access (patient, doctor, admin), and handles medical sensor data including ECG readings and pulse oximeter measurements.

---

## Tech Stack

| Layer     | Technology              |
| --------- | ----------------------- |
| Runtime   | Node.js (ES Modules)    |
| Framework | Express.js              |
| Database  | MongoDB + Mongoose      |
| Auth      | JWT + bcrypt            |
| Email     | Nodemailer              |
| Dev Tools | Nodemon, Morgan, dotenv |

---

## Project Structure

```
AutoMedic/
├── app.js                      # Express app setup, middleware, routes
├── server.js                   # DB connection & server start
├── controllers/
│   ├── auth-controller.js      # signup, login
│   ├── user-controller.js      # user & doctor/patient management
│   ├── ecg-controller.js       # ECG data retrieval & ingestion
│   ├── oximeter-controller.js  # Oximeter data retrieval & ingestion
│   └── error-controller.js     # Global error handler
├── middlewares/
│   ├── auth-middleware.js      # JWT protect + role restriction
│   └── access-midddleware.js   # Patient/doctor level access control
├── models/
│   ├── user-model.js           # User schema (patient & doctor variants)
│   ├── ecg-model.js            # ECG readings schema
│   └── oximeter-model.js       # Oximeter readings schema
├── routes/
│   ├── users-routes.js         # Auth & user management routes
│   ├── ecg-routes.js           # ECG data routes
│   └── oximeter-routes.js      # Oximeter data routes
└── utils/
    ├── api-features.js         # Filter, sort, fields, pagination
    ├── app-error.js            # Custom operational error class
    ├── catch-async.js          # Async error wrapper
    ├── controllers-handler.js  # Generic getAll / getOne / deleteOne
    ├── create-token.js         # JWT signing
    └── send-response.js        # Standardized JSON response helper
```

---

## Getting Started

### Prerequisites

- Node.js v18+
- MongoDB instance (local or Atlas)

### Installation

```bash
git clone https://github.com/your-username/automedic.git
cd automedic
npm install
```

### Environment Variables

Create a `.env` file in the project root:

```env
PORT=5000
NODE_ENV=development

DATABASE_URL=mongodb+srv://<user>:<password>@cluster.mongodb.net/automedic

JWT_SECRET=your_jwt_secret_key
JWT_EXPIRES_IN=90d
JWT_COOKIE_EXPIRES_IN=90
```

### Running the Server

```bash
# Development
npm run dev

# Production
npm run prod
```

---

## API Reference

Base URL: `/api/v1`

All protected routes require the `Authorization: Bearer <token>` header.

---

### Authentication — `/api/v1/users`

| Method | Endpoint       | Access | Description             |
| ------ | -------------- | ------ | ----------------------- |
| `POST` | `/auth/signup` | Public | Register a new user     |
| `POST` | `/auth/login`  | Public | Login and receive a JWT |

#### Signup Body (Patient)

```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "password": "password123",
  "passwordConfirm": "password123",
  "phoneNumber": "01012345678",
  "dateOfBirth": "1995-06-15",
  "role": "patient",
  "patientInfo": {
    "medicalHistory": "Diabetes",
    "emergencyContact": {
      "name": "Jane Doe",
      "relationship": "Sister",
      "phoneNumber": "01098765432"
    }
  }
}
```

#### Signup Body (Doctor)

```json
{
  "firstName": "Sarah",
  "lastName": "Smith",
  "email": "sarah@hospital.com",
  "password": "password123",
  "passwordConfirm": "password123",
  "role": "doctor",
  "doctorInfo": {
    "specialty": "Cardiology",
    "hospital": "City Medical Center"
  }
}
```

---

### User Management — `/api/v1/users`

| Method   | Endpoint       | Access | Description                             |
| -------- | -------------- | ------ | --------------------------------------- |
| `GET`    | `/`            | Admin  | Get all users                           |
| `GET`    | `/:id`         | Admin  | Get a single user by ID                 |
| `DELETE` | `/:id`         | Admin  | Delete a user by ID                     |
| `GET`    | `/doctor`      | Doctor | Get all patients assigned to the doctor |
| `GET`    | `/doctor/:pid` | Doctor | Get a specific patient by ID            |
| `POST`   | `/doctor/:pid` | Doctor | Add a patient to the doctor's list      |

---

### ECG Data — `/api/v1/ecg`

| Method | Endpoint      | Access           | Description                                                |
| ------ | ------------- | ---------------- | ---------------------------------------------------------- |
| `GET`  | `/`           | Patient / Doctor | Get ECG data (patient: own; doctor: all assigned patients) |
| `GET`  | `/:patientId` | Doctor           | Get ECG data for a specific patient                        |
| `POST` | `/add-ecg`    | Public           | Ingest ECG readings (for IoT devices)                      |

#### Add ECG Body

```json
{
  "patientId": "664abc123...",
  "readings": [
    [0.1, 0.2, 0.3],
    [0.4, 0.5, 0.6]
  ]
}
```

---

### Oximeter Data — `/api/v1/oximeter`

| Method | Endpoint | Access           | Description                                 |
| ------ | -------- | ---------------- | ------------------------------------------- |
| `GET`  | `/`      | Patient / Doctor | Get oximeter data                           |
| `POST` | `/`      | Protected        | Add oximeter reading                        |
| `GET`  | `/:id`   | Doctor           | Get oximeter data for a specific patient    |
| `POST` | `/:id`   | Protected        | Add oximeter reading for a specific patient |

#### Add Oximeter Body

```json
{
  "patientId": "664abc123...",
  "heartRate": 78,
  "oxygenPercentage": 98
}
```

---

## Data Models

### User

| Field                    | Type    | Notes                                                  |
| ------------------------ | ------- | ------------------------------------------------------ |
| `firstName` / `lastName` | String  | Required                                               |
| `email`                  | String  | Unique, validated                                      |
| `password`               | String  | Hashed with bcrypt, min 8 chars                        |
| `role`                   | String  | `patient` \| `doctor` \| `admin`                       |
| `phoneNumber`            | String  | Optional                                               |
| `dateOfBirth`            | Date    |                                                        |
| `address`                | Object  | street, city, state, zipCode, country                  |
| `doctorInfo`             | Object  | specialty (required for doctors), hospital, patients[] |
| `patientInfo`            | Object  | medicalHistory, primaryDoctor, emergencyContact        |
| `active`                 | Boolean | Default: `true`                                        |

### ECG Reading

| Field       | Type         | Notes                      |
| ----------- | ------------ | -------------------------- |
| `patientId` | ObjectId     | Ref: User                  |
| `readings`  | `[[Number]]` | 2D array of signal samples |
| `timestamp` | Date         | Auto-set                   |

### Oximeter Reading

| Field              | Type     | Notes     |
| ------------------ | -------- | --------- |
| `patientId`        | ObjectId | Ref: User |
| `heartRate`        | Number   | Required  |
| `oxygenPercentage` | Number   | Required  |
| `timestamp`        | Date     | Auto-set  |

---

## Roles & Permissions

| Action                       | Patient | Doctor | Admin |
| ---------------------------- | ------- | ------ | ----- |
| View own ECG / oximeter data | ✅      | —      | —     |
| View assigned patients' data | —       | ✅     | —     |
| Manage patient list          | —       | ✅     | —     |
| View / delete all users      | —       | —      | ✅    |

---

## Query Parameters (Admin endpoints)

The `GET /` (all users) endpoint supports advanced querying via `APIQuery`:

| Parameter        | Example                   | Description                       |
| ---------------- | ------------------------- | --------------------------------- |
| `filter`         | `?role=doctor`            | Filter by any field               |
| `sort`           | `?sort=firstName`         | Sort results                      |
| `fields`         | `?fields=firstName,email` | Select specific fields            |
| `page` + `limit` | `?page=2&limit=10`        | Pagination                        |
| Advanced filter  | `?heartRate[gte]=60`      | Supports `gte`, `gt`, `lte`, `lt` |

---

## Error Handling

All errors are handled globally via the error controller.

- **Development**: full error details + stack trace
- **Production**: sanitized messages for operational errors; generic `500` for unexpected errors

Handled error types: `CastError`, duplicate fields (`11000`), `ValidationError`, `JsonWebTokenError`, `TokenExpiredError`
