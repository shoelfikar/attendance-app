# Attendance App - GPS-Based Attendance System

A comprehensive attendance management system using GPS validation, consisting of three main components:
- **Mobile App** (Flutter) - For employees to check-in/check-out
- **Backend API** (Golang) - RESTful API with GPS validation
- **Admin Dashboard** (Next.js) - For administrators to manage the system

## 📋 Project Structure

```
attendance-app/
├── attendance-mobile/      # Flutter mobile app
├── attendance-backend/     # Golang API backend
├── attendance-admin/       # Next.js admin dashboard
├── docker-compose.yml      # PostgreSQL + PostGIS setup
└── ATTENDANCE_APP_BLUEPRINT.md
```

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- Go 1.22+
- Node.js 20+
- Flutter SDK
- PostgreSQL client (optional)

### 1. Start Database

```bash
# Copy environment file
cp .env.example .env

# Start PostgreSQL with PostGIS
docker-compose up -d postgres

# Check if database is ready
docker-compose ps
```

### 2. Start Backend API

```bash
cd attendance-backend

# Copy environment file
cp .env.example .env

# Run the API
go run cmd/api/main.go
```

API will be available at `http://localhost:8000`

### 3. Start Admin Dashboard

```bash
cd attendance-admin

# Install dependencies
npm install

# Copy environment file
cp .env.example .env.local

# Run development server
npm run dev
```

Dashboard will be available at `http://localhost:3000`

### 4. Start Mobile App

```bash
cd attendance-mobile

# Get dependencies
flutter pub get

# Run on emulator/device
flutter run
```

## 🗄️ Database Management

### Using Docker Compose

PostgreSQL with PostGIS extension is available via Docker:

```bash
# Start database
docker-compose up -d postgres

# Stop database
docker-compose down

# View logs
docker-compose logs -f postgres
```

### Using pgAdmin (Optional)

```bash
# Start pgAdmin
docker-compose up -d pgadmin

# Access at http://localhost:5050
# Email: admin@attendance.com
# Password: admin123
```

### Manual Migration

```bash
# Connect to PostgreSQL
docker exec -it attendance_postgres psql -U postgres -d attendance_db

# Or run migration manually
docker exec -i attendance_postgres psql -U postgres -d attendance_db < attendance-backend/migrations/001_init_schema.sql
```

## 📚 Documentation

For detailed information, see:
- [Blueprint](./ATTENDANCE_APP_BLUEPRINT.md) - Complete system design
- [Backend README](./attendance-backend/README.md) - API documentation
- [Admin Dashboard README](./attendance-admin/README.md) - Dashboard guide
- [Mobile App README](./attendance-mobile/README.md) - Mobile app guide

## 🔐 Default Credentials

### Admin User
- Email: `admin@attendance.com`
- Password: `admin123`

**⚠️ Change these credentials in production!**

## 🧪 Testing

### Backend
```bash
cd attendance-backend
go test ./...
```

### Admin Dashboard
```bash
cd attendance-admin
npm test
```

### Mobile App
```bash
cd attendance-mobile
flutter test
```

## 📊 Tech Stack

| Component | Technology |
|-----------|-----------|
| Mobile | Flutter |
| Backend | Golang + Gin |
| Frontend | Next.js + TypeScript + Tailwind CSS |
| Database | PostgreSQL + PostGIS |
| Auth | JWT |
| Maps | Leaflet / Google Maps |

## 🔧 Development Status

- [x] Phase 1: Setup & Foundation
- [ ] Phase 2: Backend Development
- [ ] Phase 3: Mobile App Development
- [ ] Phase 4: Admin Dashboard Development
- [ ] Phase 5: Integration & Testing
- [ ] Phase 6: Deployment

## 📝 License

This project is proprietary software.

## 👥 Team

Development Team - Attendance Management System

---

**Version:** 1.0.0
**Last Updated:** 2025-10-29
