# RentEase Backend

RentEase is a RESTful vehicle rental backend built with Node.js, Express.js, and MongoDB.

## Features

- Customer and Admin authentication using JWT
- Password hashing with bcrypt
- Role-based vehicle administration
- Vehicle search, filtering, and pagination
- Booking overlap prevention
- Booking cancellation
- Input validation and sanitization
- Helmet security headers
- Authentication rate limiting
- Centralized error handling

## Prerequisites

- Node.js 18 or later
- MongoDB running locally or a MongoDB Atlas connection string
- Postman or Thunder Client

## Installation

```bash
npm install
```

## Environment Variables

Create a `.env` file in the project root:

```env
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/rentease
JWT_SECRET=replace-with-a-long-random-secret
ADMIN_EMAIL=admin@rentease.com
ADMIN_PASSWORD=Admin123!
```

The application also has development fallbacks for the MongoDB URI and JWT secret.

## Run the Server

```bash
npm start
```

For watch mode:

```bash
npm run dev
```

Server: `http://localhost:5000`

Health check:

```text
GET /api/health
```

## Default Admin

The server creates an admin automatically if the configured admin account does not exist.

- Email: `admin@rentease.com`
- Password: `Admin123!`

Change `ADMIN_EMAIL` and `ADMIN_PASSWORD` before production use.

## Authentication

Send protected requests with:

```text
Authorization: Bearer YOUR_JWT_TOKEN
```

### Register Customer

```text
POST /api/auth/register
```

```json
{
  "name": "Test Customer",
  "email": "customer@test.com",
  "password": "Password123"
}
```

### Login

```text
POST /api/auth/login
```

```json
{
  "email": "customer@test.com",
  "password": "Password123"
}
```

## Vehicle APIs

| Method | Endpoint | Access | Description |
|---|---|---|---|
| GET | `/api/vehicles` | Public | Retrieve paginated vehicles |
| GET | `/api/vehicles/:id` | Public | Retrieve a vehicle |
| POST | `/api/vehicles` | Admin | Add a vehicle |
| PUT | `/api/vehicles/:id` | Admin | Update a vehicle |
| DELETE | `/api/vehicles/:id` | Admin | Delete a vehicle |

### Vehicle Query Parameters

```text
GET /api/vehicles?page=1&limit=10&name=Toyota&vehicleType=SUV&location=Mumbai
```

- `page`: Page number
- `limit`: Vehicles per page, maximum 100
- `name`: Vehicle name search
- `vehicleType`: Vehicle type filter
- `location`: Location filter

### Add Vehicle Example

```text
POST /api/vehicles
Authorization: Bearer ADMIN_TOKEN
```

```json
{
  "vehicleName": "Toyota Fortuner",
  "vehicleType": "SUV",
  "registrationNumber": "MH01AB1234",
  "seatingCapacity": 7,
  "pricePerDay": 3500,
  "location": "Mumbai",
  "availabilityStatus": "Available"
}
```

## Booking APIs

| Method | Endpoint | Access | Description |
|---|---|---|---|
| POST | `/api/bookings` | Customer | Create a booking |
| GET | `/api/bookings` | Authenticated | Retrieve accessible bookings |
| GET | `/api/bookings/:id` | Owner or Admin | Retrieve booking details |
| PATCH | `/api/bookings/:id/cancel` | Owner or Admin | Cancel a booking |

### Create Booking Example

```text
POST /api/bookings
Authorization: Bearer CUSTOMER_TOKEN
```

```json
{
  "vehicleId": "VEHICLE_ID",
  "pickupDate": "2026-12-01",
  "returnDate": "2026-12-05"
}
```

## Booking Rules

- Only customers can create bookings.
- Pickup dates cannot be in the past.
- Return date must be later than pickup date.
- Unavailable vehicles cannot be booked.
- Confirmed bookings cannot overlap the requested date range.
- Rental duration is calculated in calendar days.
- Total amount is `numberOfDays × pricePerDay`.
- Customers can view and cancel only their own bookings.
- Cancelling a booking releases the relevant dates.

## API Testing Evidence

Test and capture screenshots for:

1. Customer registration.
2. Customer login.
3. Admin login.
4. Admin successfully adding a vehicle.
5. Customer receiving `403` while attempting to add a vehicle.
6. Request without a JWT receiving `401`.
7. Customer creating a successful booking.
8. Customer retrieving their bookings.
9. Overlapping booking attempt receiving `409`.
10. Invalid or past-date booking attempt receiving `400`.
11. Successful booking cancellation.
12. Rebooking the released date range successfully.

Example overlap:

```json
{
  "vehicleId": "VEHICLE_ID",
  "pickupDate": "2026-12-03",
  "returnDate": "2026-12-07"
}
```

## Project Structure

```text
config/
  db.js
controller/
  authController.js
  bookingController.js
  vehicleController.js
middleware/
  auth.js
  errorHandler.js
models/
  booking.js
  models.js
  user.js
  vehicles.js
routes/
  authRoutes.js
  bookingRoutes.js
  vehicleRoutes.js
server.js
package.json
README.md
```

## Status Codes

- `200` Success
- `201` Resource created
- `400` Validation or invalid date
- `401` Missing or invalid JWT
- `403` Insufficient role or ownership
- `404` Resource or route not found
- `409` Duplicate or booking conflict
- `500` Internal server error
