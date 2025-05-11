# Airbnb Clone Backend Requirements Specification

This document outlines the technical and functional requirements for three core backend features: User Authentication, Property Management, and Booking System.

---

## 1. User Authentication

### Overview
Handles user registration, login, and authentication for Guests, Hosts, and Admins.

### API Endpoints

- **POST /api/v1/auth/register**
  - Registers a new user (Guest or Host).
- **POST /api/v1/auth/login**
  - Authenticates a user and returns a JWT token.
- **GET /api/v1/auth/profile**
  - Returns the authenticated user's profile.

### Input/Output Specifications

#### Register
- **Input:**  
  - `email` (string, required, valid email format)
  - `password` (string, required, min 8 chars)
  - `role` (string, required, enum: guest, host, admin)
  - `first_name` (string, required)
  - `last_name` (string, required)
  
- **Output:**  
  - `201 Created` with user object (excluding password)
  - `400 Bad Request` for validation errors

#### Login
- **Input:**  
  - `email` (string, required)
  - `password` (string, required)
- **Output:**  
  - `200 OK` with JWT token and user info
  - `401 Unauthorized` for invalid credentials

#### Profile
- **Input:**  
  - JWT token in Authorization header
- **Output:**  
  - `200 OK` with user profile
  - `401 Unauthorized` for invalid token

### Validation Rules
- Email must be unique and valid.
- Password must be at least 8 characters, contain letters and numbers.
- Role must be either "admin", "guest" or "host".
- All fields required.

### Performance Criteria
- Registration and login should complete within 1 second under normal load.
- JWT tokens must be securely generated and validated.
- Rate limiting on login to prevent brute-force attacks.

---

## 2. Property Management

### Overview
Allows hosts to create, update, delete, and view property listings.

### API Endpoints

- **POST /api/v1/properties**
  - Create a new property listing (Host only).
- **GET /api/v1/properties**
  - Retrieve a list of properties (with filters).
- **GET /api/v1/properties/{id}**
  - Retrieve details of a specific property.
- **PUT /api/v1/properties/{id}**
  - Update a property listing (Host only).
- **DELETE /api/v1/properties/{id}**
  - Delete a property listing (Host only).

### Input/Output Specifications

#### Create Property
- **Input:**  
  - `title` (string, required)
  - `description` (string, required)
  - `location` (string, required)
  - `price_per_night` (number, required, >0)
  - `max_guests` (integer, required, >0)
  - `images` (array of file uploads, optional)
- **Output:**  
  - `201 Created` with property object
  - `400 Bad Request` for validation errors
  - `409 Conflict` if property already exists`
#### Get Properties
- **Input:**
  - Query parameters for filtering (location, price, etc.)
- **Output:**
  - `200 OK` with paginated list of properties
  - `400 Bad Request` for invalid query parameters
#### Get Property
- **Input:**
  - Property ID in URL
- **Output:**
  - `200 OK` with property object
  - `404 Not Found` if property does not exist

#### Update Property
- **Input:**  
  - Same as create, all fields optional except at least one must be provided
- **Output:**  
  - `200 OK` with updated property object
  - `404 Not Found` if property does not exist or not owned by host

#### Delete Property
- **Input:**  
  - Property ID in URL
- **Output:**  
  - `204 No Content` on success
  - `404 Not Found` if property does not exist or not owned by host

### Validation Rules
- Only authenticated hosts can create, update, or delete properties.
- Title, description, location, price, and max_guests are required for creation.
- Price and max_guests must be positive numbers.
- Images must be valid image files (jpg, png, etc.).

### Performance Criteria
- Property listing retrieval should support pagination and filtering (by location, price, etc.).
- Image uploads should be handled asynchronously.
- CRUD operations should complete within 1 second under normal load.

---

## 3. Booking System

### Overview
Enables guests to book properties, view bookings, and cancel reservations.

### API Endpoints

- **POST /api/v1/bookings**
  - Create a new booking (Guest only).
- **GET /api/v1/bookings**
  - Retrieve bookings for the authenticated user.
- **DELETE /api/v1/bookings/{id}**
  - Cancel a booking (Guest only).

### Input/Output Specifications

#### Create Booking
- **Input:**  
  - `property_id` (string, required)
  - `check_in` (date, required, future date)
  - `check_out` (date, required, after check_in)
  - `guests` (integer, required, >0)
- **Output:**  
  - `201 Created` with booking object
  - `400 Bad Request` for validation errors
  - `409 Conflict` if property is not available

#### Cancel Booking
- **Input:**  
  - Booking ID in URL
- **Output:**  
  - `204 No Content` on success
  - `404 Not Found` if booking does not exist or not owned by guest

### Validation Rules
- Only authenticated guests can create or cancel bookings.
- Property must be available for the requested dates.
- Check-in must be before check-out and both must be valid dates.
- Number of guests must not exceed property max_guests.

### Performance Criteria
- Booking creation and cancellation should complete within 1 second under normal load.
- System must prevent double-booking (atomic transactions or locking).
- Booking retrieval should support pagination.

---

## General Notes

- All endpoints must require authentication unless explicitly public.
- All input data must be validated server-side.
- API responses must use standard HTTP status codes.
- Error messages should be descriptive but not leak sensitive information.
- System should be horizontally scalable and secure against common web vulnerabilities.