# Organization Management Service (Backend Intern Assignment)

A FastAPI + MongoDB backend service that manages organizations in a multi-tenant style architecture. Each organization gets its own MongoDB collection, while a Master Database stores global metadata and admin credentials (hashed with bcrypt). JWT-based authentication protects all CRUD operations.

## Features

- Create organizations with dynamic MongoDB collections (e.g., `org_srm`)
- Master database storing organization metadata and admin references
- Admin user with bcrypt-hashed password per organization
- JWT-based authentication for protected operations
- REST APIs for create, get, update, and delete organization
- Comprehensive error handling and validation

## Tech Stack

- **Framework**: FastAPI (Python)
- **Database**: MongoDB
- **Driver**: PyMongo
- **Authentication**: JWT (python-jose)
- **Password Hashing**: bcrypt (passlib)
- **Server**: Uvicorn

## Project Structure

```
org-management-service-backend/
  app/
    __init__.py
    main.py
    config.py
    database.py
    models.py
    schemas.py
    auth.py
    crud_org.py
    crud_admin.py
    dependencies.py
    routers/
      __init__.py
      org.py
      admin.py
  requirements.txt
  README.md
  .env.example
```

## Setup and Installation

### Prerequisites
- Python 3.8+
- MongoDB (local or MongoDB Atlas)
- pip (Python package manager)

### Local Setup

1. Clone the repository:
```bash
git clone https://github.com/KODAVATISANJAY/org-management-service-backend.git
cd org-management-service-backend
```

2. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Create `.env` file (copy from `.env.example`):
```bash
MONGODB_URI=mongodb://localhost:27017
MASTER_DB_NAME=org_master_db
JWT_SECRET_KEY=your_secret_key_here_change_in_production
```

5. Start the server:
```bash
uvicorn app.main:app --reload
```

Server will run at `http://localhost:8000`

### API Documentation

Interactive API docs available at:
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

## API Endpoints

### 1. Create Organization
**POST** `/org/create`

Create a new organization with admin credentials.

**Request Body:**
```json
{
  "organization_name": "SRM",
  "email": "admin@srm.com",
  "password": "StrongPass123"
}
```

**Response (201):**
```json
{
  "organization_name": "SRM",
  "collection_name": "org_srm",
  "admin_email": "admin@srm.com"
}
```

---

### 2. Get Organization by Name
**GET** `/org/get?organization_name=SRM`

Fetch organization details by name.

**Query Parameters:**
- `organization_name` (required): Name of the organization

**Response (200):**
```json
{
  "organization_name": "SRM",
  "collection_name": "org_srm",
  "admin_email": "admin@srm.com"
}
```

---

### 3. Admin Login
**POST** `/admin/login`

Authenticate admin and receive JWT token.

**Request Body:**
```json
{
  "email": "admin@srm.com",
  "password": "StrongPass123"
}
```

**Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

---

### 4. Update Organization
**PUT** `/org/update`

Update organization details (requires JWT token).

**Headers:**
```
Authorization: Bearer <access_token>
```

**Request Body:**
```json
{
  "current_name": "SRM",
  "new_name": "SRM University",
  "email": "newadmin@srm.com",
  "password": "NewStrongPass456"
}
```

**Response (200):**
```json
{
  "organization_name": "SRM University",
  "collection_name": "org_srm_university",
  "admin_email": "newadmin@srm.com"
}
```

---

### 5. Delete Organization
**DELETE** `/org/delete`

Delete an organization and all its data (requires JWT token).

**Headers:**
```
Authorization: Bearer <access_token>
```

**Request Body:**
```json
{
  "organization_name": "SRM University"
}
```

**Response (204):** No content

---

## High-Level Architecture

```
┌─────────────────────────────────┐
│   Client (Postman/Frontend)     │
└──────────────┬──────────────────┘
               │ HTTP Requests
               ▼
┌─────────────────────────────────┐
│    FastAPI Application          │
│  - Route Handlers               │
│  - Input Validation (Pydantic)  │
│  - JWT Authentication           │
└──────────────┬──────────────────┘
               │
      ┌────────┴────────┐
      ▼                 ▼
┌────────────────┐  ┌──────────────┐
│  Auth Module   │  │  CRUD Module │
│ - Password     │  │ - Create Org │
│   Hashing      │  │ - Get Org    │
│ - JWT Tokens   │  │ - Update Org │
│                │  │ - Delete Org │
└────────────────┘  └──────────────┘
      │                 │
      └────────┬────────┘
               ▼
┌─────────────────────────────────┐
│    MongoDB Master Database      │
│                                 │
│  - organizations collection     │
│  - admins collection            │
│                                 │
│  - org_srm collection (dynamic) │
│  - org_abc collection (dynamic) │
└─────────────────────────────────┘
```

## Authentication Flow

1. **Admin Registration**: When org is created, admin email/password is stored
2. **Password Hashing**: Passwords are hashed using bcrypt before storage
3. **Login**: Admin provides email/password → Server validates → Returns JWT
4. **JWT Token**: Contains admin_id, org_id, org_name → Expires in 60 minutes
5. **Protected Routes**: `/org/update`, `/org/delete` require valid JWT in Authorization header

## Database Schema

### Master Database Collections

**organizations:**
```json
{
  "_id": ObjectId,
  "name": "SRM",
  "collection_name": "org_srm",
  "admin_id": ObjectId,
  "created_at": ISODate
}
```

**admins:**
```json
{
  "_id": ObjectId,
  "email": "admin@srm.com",
  "password_hash": "$2b$12$...",
  "created_at": ISODate
}
```

## Deployment

### Deploy on Render

1. Push code to GitHub
2. Sign up at [render.com](https://render.com)
3. Create new "Web Service"
4. Connect your GitHub repo
5. Set Build Command: `pip install -r requirements.txt`
6. Set Start Command: `uvicorn app.main:app --host 0.0.0.0 --port 10000`
7. Add Environment Variables:
   - `MONGODB_URI`: Your MongoDB connection string
   - `MASTER_DB_NAME`: org_master_db
   - `JWT_SECRET_KEY`: Your secret key
8. Deploy!

### Deploy on Vercel (with Mangum)

1. Install Mangum: `pip install mangum`
2. Create `api/main.py` with ASGI handler
3. Add `vercel.json` configuration
4. Deploy via Vercel CLI or GitHub integration

## Testing

You can test the API using Postman or cURL. Example:

```bash
# Create organization
curl -X POST http://localhost:8000/org/create \
  -H "Content-Type: application/json" \
  -d '{"organization_name": "SRM", "email": "admin@srm.com", "password": "StrongPass123"}'

# Get organization
curl http://localhost:8000/org/get?organization_name=SRM

# Admin login
curl -X POST http://localhost:8000/admin/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@srm.com", "password": "StrongPass123"}'
```

## Environment Variables

Create `.env` file with:

```
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/?retryWrites=true&w=majority
MASTER_DB_NAME=org_master_db
JWT_SECRET_KEY=your_secret_key_here
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60
```

## Error Handling

The API returns appropriate HTTP status codes:

- `200`: Success
- `201`: Created
- `204`: No Content
- `400`: Bad Request (validation error)
- `401`: Unauthorized (invalid credentials or token)
- `403`: Forbidden (not authorized for this organization)
- `404`: Not Found (organization doesn't exist)
- `500`: Internal Server Error

## Design Decisions

1. **Multi-tenant Architecture**: Each organization has its own MongoDB collection for data isolation
2. **Master DB**: Centralized metadata storage for organization and admin information
3. **JWT Authentication**: Stateless authentication suitable for REST APIs and microservices
4. **Bcrypt Hashing**: Industry-standard password hashing with automatic salt generation
5. **Pydantic Validation**: Type-safe request/response validation
6. **Class-based CRUD**: Modular, maintainable code structure

## Future Enhancements

- Add user roles and permissions
- Implement API rate limiting
- Add comprehensive logging
- Create unit and integration tests
- Add organization member management
- Implement audit trails for operations
- Add GraphQL support

## License

MIT License

## Author

Sanjay Kodavati (@KODAVATISANJAY)

## Support

For issues or questions, please open an issue in the GitHub repository.
