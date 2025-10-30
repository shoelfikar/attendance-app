# Blueprint Aplikasi Absensi Berbasis GPS

## 📋 Overview
Aplikasi absensi mobile yang menggunakan GPS untuk memvalidasi lokasi user saat melakukan absensi. Sistem terdiri dari 3 komponen utama: Mobile App (Flutter), Backend API (Golang), dan Admin Dashboard (Next.js).

---

## 🏗️ Tech Stack

### Mobile App
- **Framework**: Flutter
- **Platform**: Android
- **State Management**: Provider / Riverpod / Bloc (rekomendasi)
- **HTTP Client**: Dio / http
- **GPS**: geolocator package
- **Storage**: shared_preferences / hive

### Backend
- **Language**: Golang
- **Framework**: Gin / Echo / Fiber
- **Database**: PostgreSQL
- **ORM**: GORM
- **Authentication**: JWT
- **Geospatial**: PostGIS extension

### Admin Dashboard
- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Build Tool**: Vite (atau Next.js native)
- **UI Components**: shadcn/ui / Headless UI
- **Maps**: Leaflet / Google Maps / Mapbox
- **State Management**: Zustand / Redux Toolkit
- **HTTP Client**: Axios / Fetch API

---

## 🗄️ Database Schema

### Table: users
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    phone VARCHAR(50),
    role VARCHAR(20) NOT NULL DEFAULT 'user', -- 'admin' or 'user'
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Table: attendance_locations
```sql
CREATE TABLE attendance_locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    radius INTEGER DEFAULT 10, -- in meters
    is_active BOOLEAN DEFAULT true,
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create spatial index for better performance
CREATE INDEX idx_attendance_locations_coords ON attendance_locations USING GIST (
    ST_SetSRID(ST_MakePoint(longitude, latitude), 4326)
);
```

### Table: attendances
```sql
CREATE TABLE attendances (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    location_id INTEGER NOT NULL REFERENCES attendance_locations(id),
    check_in_time TIMESTAMP NOT NULL,
    check_out_time TIMESTAMP,
    check_in_latitude DECIMAL(10, 8) NOT NULL,
    check_in_longitude DECIMAL(11, 8) NOT NULL,
    check_out_latitude DECIMAL(10, 8),
    check_out_longitude DECIMAL(11, 8),
    distance_from_location DECIMAL(10, 2), -- in meters
    status VARCHAR(20) DEFAULT 'present', -- 'present', 'late', 'half_day'
    notes TEXT,
    photo_url VARCHAR(500), -- optional selfie photo
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_attendances_user_date ON attendances(user_id, DATE(check_in_time));
CREATE INDEX idx_attendances_location ON attendances(location_id);
```

### Table: work_schedules
```sql
CREATE TABLE work_schedules (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    check_in_start TIME NOT NULL, -- e.g., 08:00:00
    check_in_end TIME NOT NULL,   -- e.g., 09:00:00 (late after this)
    check_out_start TIME NOT NULL, -- e.g., 17:00:00
    work_days INTEGER[], -- [1,2,3,4,5] for Mon-Fri
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Table: user_schedules
```sql
CREATE TABLE user_schedules (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    schedule_id INTEGER NOT NULL REFERENCES work_schedules(id),
    location_id INTEGER NOT NULL REFERENCES attendance_locations(id),
    effective_from DATE NOT NULL,
    effective_to DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, effective_from)
);
```

---

## 🔌 API Endpoints

### Authentication
```
POST   /api/v1/auth/register          - Register user baru
POST   /api/v1/auth/login             - Login user
POST   /api/v1/auth/refresh-token     - Refresh JWT token
POST   /api/v1/auth/logout            - Logout user
GET    /api/v1/auth/me                - Get current user info
```

### Attendance (User)
```
GET    /api/v1/attendance/locations              - Get nearby attendance locations
POST   /api/v1/attendance/check-in                - Check-in attendance
POST   /api/v1/attendance/check-out               - Check-out attendance
GET    /api/v1/attendance/history                 - Get user attendance history
GET    /api/v1/attendance/today                   - Get today's attendance
GET    /api/v1/attendance/status                  - Check current attendance status
POST   /api/v1/attendance/validate-location      - Validate if user in radius
```

### Admin - Users Management
```
GET    /api/v1/admin/users                - Get all users (paginated)
GET    /api/v1/admin/users/:id            - Get user detail
POST   /api/v1/admin/users                - Create new user
PUT    /api/v1/admin/users/:id            - Update user
DELETE /api/v1/admin/users/:id            - Delete user
PATCH  /api/v1/admin/users/:id/status     - Activate/Deactivate user
```

### Admin - Locations Management
```
GET    /api/v1/admin/locations            - Get all locations
GET    /api/v1/admin/locations/:id        - Get location detail
POST   /api/v1/admin/locations            - Create new location
PUT    /api/v1/admin/locations/:id        - Update location
DELETE /api/v1/admin/locations/:id        - Delete location
```

### Admin - Attendance Reports
```
GET    /api/v1/admin/attendances                    - Get all attendances (filtered)
GET    /api/v1/admin/attendances/:id                - Get attendance detail
GET    /api/v1/admin/attendances/user/:userId       - Get user attendance history
GET    /api/v1/admin/reports/daily                  - Daily attendance report
GET    /api/v1/admin/reports/monthly                - Monthly attendance report
GET    /api/v1/admin/reports/export                 - Export to CSV/Excel
```

### Admin - Schedules Management
```
GET    /api/v1/admin/schedules            - Get all work schedules
POST   /api/v1/admin/schedules            - Create work schedule
PUT    /api/v1/admin/schedules/:id        - Update work schedule
DELETE /api/v1/admin/schedules/:id        - Delete work schedule
POST   /api/v1/admin/schedules/assign     - Assign schedule to user
```

---

## 📱 User Flow (Mobile App)

### 1. Authentication Flow
```
┌─────────────┐
│ Splash      │
│ Screen      │
└──────┬──────┘
       │
       ▼
┌─────────────┐      Login Success
│ Login       ├──────────────────┐
│ Screen      │                  │
└──────┬──────┘                  │
       │                         │
       │ No Account              │
       ▼                         ▼
┌─────────────┐            ┌──────────┐
│ Register    │            │   Home   │
│ Screen      ├───────────►│  Screen  │
└─────────────┘            └──────────┘
```

### 2. Check-In Flow
```
┌──────────────┐
│  Home Screen │
└──────┬───────┘
       │
       │ User taps "Absen"
       ▼
┌──────────────────┐
│ Request GPS      │
│ Permission       │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐         Outside Radius
│ Get Current      ├─────────────────────┐
│ GPS Location     │                     │
└──────┬───────────┘                     │
       │                                 ▼
       │ Within Radius           ┌──────────────┐
       ▼                         │ Show Error   │
┌──────────────────┐             │ "Anda diluar"│
│ Validate Location│             │ "radius"     │
│ with Backend     │             └──────────────┘
└──────┬───────────┘
       │
       │ Valid
       ▼
┌──────────────────┐
│ Show Confirmation│
│ Dialog           │
│ - Time           │
│ - Location Name  │
│ - Distance       │
└──────┬───────────┘
       │
       │ User confirms
       ▼
┌──────────────────┐
│ Optional: Take   │
│ Selfie Photo     │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Submit Check-In  │
│ to Backend       │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Show Success     │
│ Message          │
└──────────────────┘
```

### 3. Check-Out Flow
```
┌──────────────┐
│ Home Screen  │
│ (Checked In) │
└──────┬───────┘
       │
       │ User taps "Check Out"
       ▼
┌──────────────────┐
│ Validate Location│
│ (same as check-in)│
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Submit Check-Out │
│ to Backend       │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Show Summary     │
│ - Check-in time  │
│ - Check-out time │
│ - Work duration  │
└──────────────────┘
```

---

## 💻 Admin Dashboard Flow

### 1. Dashboard Overview
```
┌────────────────────────────────────────┐
│  Admin Dashboard - Home                │
├────────────────────────────────────────┤
│                                        │
│  📊 Statistics Cards                   │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐│
│  │Total │ │Hadir │ │Telat │ │Absen ││
│  │ User │ │ Hari │ │ Hari │ │ Hari ││
│  │      │ │  Ini │ │  Ini │ │  Ini ││
│  └──────┘ └──────┘ └──────┘ └──────┘│
│                                        │
│  📈 Attendance Chart (Last 7 Days)    │
│  ┌────────────────────────────────┐  │
│  │     Bar/Line Chart             │  │
│  └────────────────────────────────┘  │
│                                        │
│  📋 Recent Attendances Table          │
│  ┌────────────────────────────────┐  │
│  │ User | Time | Location | Status│  │
│  └────────────────────────────────┘  │
└────────────────────────────────────────┘
```

### 2. Location Management Flow
```
┌─────────────────┐
│  Locations      │
│  List Page      │
└────────┬────────┘
         │
         │ Click "Add Location"
         ▼
┌─────────────────────┐
│  Add Location Form  │
│  ┌───────────────┐  │
│  │ Name          │  │
│  │ Description   │  │
│  │ Radius (m)    │  │
│  │               │  │
│  │ 🗺️  Map       │  │
│  │ (Click to set)│  │
│  │ coordinates)  │  │
│  │               │  │
│  │ Lat: xx.xxx   │  │
│  │ Lng: xx.xxx   │  │
│  └───────────────┘  │
└────────┬────────────┘
         │
         │ Submit
         ▼
┌─────────────────┐
│ Save to DB      │
│ Show Success    │
└─────────────────┘
```

### 3. User Management Flow
```
┌─────────────────┐
│  Users List     │
│  ┌───────────┐  │
│  │ Search    │  │
│  └───────────┘  │
│  ┌───────────┐  │
│  │ Filter    │  │
│  │ - Role    │  │
│  │ - Status  │  │
│  └───────────┘  │
│                 │
│  Users Table    │
│  [Edit] [Delete]│
└────────┬────────┘
         │
         ▼
┌─────────────────────┐
│  User Detail/Edit   │
│  - Personal Info    │
│  - Assign Schedule  │
│  - Assign Location  │
│  - View History     │
└─────────────────────┘
```

### 4. Reports Flow
```
┌─────────────────────┐
│  Reports Page       │
│  ┌───────────────┐  │
│  │ Filters       │  │
│  │ - Date Range  │  │
│  │ - User        │  │
│  │ - Location    │  │
│  │ - Status      │  │
│  └───────────────┘  │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Report Table       │
│  - Sortable columns │
│  - Pagination       │
│                     │
│  [Export to CSV]    │
│  [Export to Excel]  │
│  [Print]            │
└─────────────────────┘
```

---

## 🏛️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Client Layer                         │
├──────────────────────────┬──────────────────────────────────┤
│                          │                                  │
│  📱 Flutter Mobile App   │   💻 Next.js Admin Dashboard    │
│  (Android)               │   (Web Browser)                  │
│  - GPS Location          │   - User Management              │
│  - Camera                │   - Location Management          │
│  - Local Storage         │   - Reports & Analytics          │
│                          │   - Interactive Maps             │
└──────────┬───────────────┴────────────┬─────────────────────┘
           │                            │
           │         HTTPS/JSON         │
           │                            │
┌──────────▼────────────────────────────▼─────────────────────┐
│                   API Gateway / Load Balancer               │
│                         (Optional)                          │
└──────────┬──────────────────────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────┐
│                    Backend Layer (Golang)                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🔐 Authentication Middleware (JWT)                         │
│                                                             │
│  📍 Controllers                                             │
│  ┌──────────────┬──────────────┬──────────────────────┐   │
│  │ Auth         │ Attendance   │ Admin                │   │
│  │ Controller   │ Controller   │ Controller           │   │
│  └──────┬───────┴──────┬───────┴──────┬───────────────┘   │
│         │              │              │                    │
│  ┌──────▼──────────────▼──────────────▼───────────────┐   │
│  │          Business Logic Services                    │   │
│  │  - Location Validation (Haversine Formula)          │   │
│  │  - Attendance Rules Engine                          │   │
│  │  - Schedule Management                              │   │
│  └──────┬──────────────────────────────────────────────┘   │
│         │                                                   │
│  ┌──────▼──────────────────────────────────────────────┐   │
│  │          Data Access Layer (GORM)                   │   │
│  └──────┬──────────────────────────────────────────────┘   │
│         │                                                   │
└─────────┼───────────────────────────────────────────────────┘
          │
┌─────────▼───────────────────────────────────────────────────┐
│              Database Layer (PostgreSQL + PostGIS)          │
├─────────────────────────────────────────────────────────────┤
│  - Users                                                    │
│  - Attendance Locations (with spatial indexing)             │
│  - Attendances                                              │
│  - Work Schedules                                           │
│  - User Schedules                                           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    External Services                        │
├─────────────────────────────────────────────────────────────┤
│  - File Storage (AWS S3 / Google Cloud Storage)            │
│    (for user photos)                                        │
│  - Email Service (SendGrid / AWS SES)                       │
│    (for notifications)                                      │
│  - Push Notifications (FCM)                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧮 GPS Validation Logic (Haversine Formula)

Backend akan menggunakan Haversine Formula untuk menghitung jarak antara koordinat user dengan koordinat lokasi absen:

```go
func CalculateDistance(lat1, lon1, lat2, lon2 float64) float64 {
    const earthRadius = 6371000 // meters

    dLat := toRadians(lat2 - lat1)
    dLon := toRadians(lon2 - lon1)

    a := math.Sin(dLat/2)*math.Sin(dLat/2) +
         math.Cos(toRadians(lat1))*math.Cos(toRadians(lat2))*
         math.Sin(dLon/2)*math.Sin(dLon/2)

    c := 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))

    distance := earthRadius * c
    return distance // in meters
}

func ValidateLocation(userLat, userLon, locationLat, locationLon, radius float64) bool {
    distance := CalculateDistance(userLat, userLon, locationLat, locationLon)
    return distance <= radius
}
```

---

## 📋 Feature List

### Mobile App Features
- ✅ User Authentication (Login/Register)
- ✅ GPS Location Detection
- ✅ Check-In/Check-Out
- ✅ Location Validation (within radius)
- ✅ View Attendance History
- ✅ View Today's Attendance Status
- ✅ Optional Selfie Photo
- ✅ Push Notifications
- ✅ Offline Mode (save locally, sync later)
- ✅ Profile Management

### Admin Dashboard Features
- ✅ Admin Authentication
- ✅ User Management (CRUD)
- ✅ Attendance Location Management (CRUD)
- ✅ Interactive Map for Setting Locations
- ✅ View All Attendances (with filters)
- ✅ Attendance Reports (Daily/Monthly)
- ✅ Export Reports (CSV/Excel)
- ✅ Work Schedule Management
- ✅ Assign Schedules to Users
- ✅ Dashboard Analytics
- ✅ User Activity Logs

### Backend Features
- ✅ RESTful API
- ✅ JWT Authentication
- ✅ Role-based Access Control (Admin/User)
- ✅ GPS Validation (Haversine)
- ✅ Spatial Queries (PostGIS)
- ✅ File Upload (Photos)
- ✅ Data Validation
- ✅ Error Handling
- ✅ API Rate Limiting
- ✅ Logging & Monitoring

---

## 🚀 Deployment Architecture

### Development Environment
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Flutter   │    │   Golang    │    │  Next.js    │
│  localhost  │───▶│  localhost  │◀───│  localhost  │
│   :8080     │    │   :8000     │    │   :3000     │
��─────────────┘    └──────┬──────┘    └─────────────┘
                          │
                   ┌──────▼──────┐
                   │ PostgreSQL  │
                   │  localhost  │
                   │   :5432     │
                   └─────────────┘
```

### Production Environment
```
┌─────────────────┐
│   Play Store    │
│  (APK Release)  │
└─────────────────┘

┌──────────────────────────────────────────┐
│          Cloud Platform (AWS/GCP)        │
├──────────────────────────────────────────┤
│                                          │
│  ┌────────────┐      ┌────────────┐     │
│  │  Next.js   │      │   Golang   │     │
│  │  (Vercel/  │◀────▶│   API      │     │
│  │  CloudRun) │      │  (Cloud    │     │
│  │            │      │   Run/EC2) │     │
│  └────────────┘      └──────┬─────┘     │
│                             │            │
│                      ┌──────▼──────┐     │
│                      │ PostgreSQL  │     │
│                      │  (RDS/Cloud │     │
│                      │   SQL)      │     │
│                      └─────────────┘     │
│                                          │
│  ┌────────────┐      ┌────────────┐     │
│  │ Cloud      │      │   Redis    │     │
│  │ Storage    │      │  (Cache)   │     │
│  │ (S3/GCS)   │      │            │     │
│  └────────────┘      └────────────┘     │
└──────────────────────────────────────────┘
```

---

## 🔐 Security Considerations

1. **Authentication & Authorization**
   - JWT tokens with expiration
   - Refresh token mechanism
   - Role-based access control (RBAC)
   - Password hashing (bcrypt)

2. **API Security**
   - HTTPS only
   - CORS configuration
   - Rate limiting
   - Input validation & sanitization
   - SQL injection prevention (using ORM)

3. **Data Privacy**
   - GPS coordinates encryption in transit
   - Minimal data retention policy
   - User consent for location tracking
   - GDPR compliance (if applicable)

4. **Mobile Security**
   - Certificate pinning
   - Secure storage for tokens
   - Jailbreak/root detection
   - Obfuscation

---

## 📦 Project Structure

### Backend (Golang)
```
attendance-backend/
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── config/
│   ├── controller/
│   ├── middleware/
│   ├── model/
│   ├── repository/
│   ├── service/
│   └── utils/
├── pkg/
│   ├── database/
│   ├── jwt/
│   └── validator/
├── migrations/
├── .env
├── go.mod
└── go.sum
```

### Mobile (Flutter)
```
attendance_mobile/
├── lib/
│   ├── core/
│   │   ├── constants/
│   │   ├── theme/
│   │   └── utils/
│   ├── data/
│   │   ├── models/
│   │   ├── repositories/
│   │   └── datasources/
│   ├── domain/
│   │   ├── entities/
│   │   ├── repositories/
│   │   └── usecases/
│   ├── presentation/
│   │   ├── pages/
│   │   ├── widgets/
│   │   └── providers/
│   └── main.dart
├── assets/
├── android/
├── ios/
├── pubspec.yaml
└── README.md
```

### Admin Dashboard (Next.js)
```
attendance-admin/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   └── login/
│   │   ├── (dashboard)/
│   │   │   ├── users/
│   │   │   ├── locations/
│   │   │   ├── attendances/
│   │   │   └── reports/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   ├── ui/
│   │   ├── forms/
│   │   ├── tables/
│   │   └── maps/
│   ├── lib/
│   │   ├── api/
│   │   ├── utils/
│   │   └── hooks/
│   ├── types/
│   └── styles/
├── public/
├── package.json
├── tailwind.config.ts
└── tsconfig.json
```

---

## 📝 Implementation Phases

### Phase 1: Setup & Foundation (Week 1-2)
- [ ] Setup project repositories
- [ ] Initialize Flutter project
- [ ] Initialize Golang project with Gin/Echo
- [ ] Initialize Next.js project with TypeScript
- [ ] Setup PostgreSQL with PostGIS
- [ ] Design and create database schema
- [ ] Setup CI/CD pipelines

### Phase 2: Backend Development (Week 3-4)
- [ ] Implement authentication system
- [ ] Create user management APIs
- [ ] Create attendance location APIs
- [ ] Implement GPS validation logic
- [ ] Create attendance APIs
- [ ] Create work schedule APIs
- [ ] Write unit tests
- [ ] API documentation (Swagger)

### Phase 3: Mobile App Development (Week 5-7)
- [ ] Setup Flutter project structure
- [ ] Implement authentication screens
- [ ] Implement GPS location service
- [ ] Create home screen with attendance status
- [ ] Implement check-in/check-out flow
- [ ] Create attendance history screen
- [ ] Implement offline mode
- [ ] Add push notifications
- [ ] Testing on real devices

### Phase 4: Admin Dashboard Development (Week 8-10)
- [ ] Setup Next.js project with Tailwind
- [ ] Implement authentication
- [ ] Create dashboard overview
- [ ] Implement user management module
- [ ] Implement location management with maps
- [ ] Create attendance monitoring module
- [ ] Implement reports and analytics
- [ ] Add export functionality (CSV/Excel)

### Phase 5: Integration & Testing (Week 11-12)
- [ ] Integration testing
- [ ] Performance testing
- [ ] Security audit
- [ ] Bug fixes
- [ ] User acceptance testing (UAT)

### Phase 6: Deployment (Week 13)
- [ ] Deploy backend to cloud
- [ ] Deploy admin dashboard
- [ ] Release mobile app to Play Store
- [ ] Setup monitoring and logging
- [ ] Create user documentation

---

## 🧪 Testing Strategy

### Backend Testing
- Unit tests for services and utilities
- Integration tests for APIs
- Database migration tests
- Load testing (k6 or Apache JMeter)

### Mobile Testing
- Unit tests for business logic
- Widget tests for UI components
- Integration tests for flows
- Manual testing on multiple devices
- GPS accuracy testing in various locations

### Admin Dashboard Testing
- Unit tests for utilities and hooks
- Component tests (React Testing Library)
- E2E tests (Playwright or Cypress)
- Responsive design testing

---

## 📊 Monitoring & Analytics

### Backend Monitoring
- API response times
- Error rates
- Database query performance
- Server resource usage (CPU, Memory)
- Active user sessions

### Mobile Analytics
- User engagement metrics
- GPS accuracy rates
- Crash reports
- Check-in/check-out success rates
- App performance metrics

### Business Metrics
- Daily attendance rate
- Late attendance rate
- Average check-in time
- Location-wise attendance distribution
- User attendance trends

---

## 🔄 Future Enhancements

1. **Advanced Features**
   - Face recognition for check-in
   - QR code alternative (for indoor areas)
   - Leave management system
   - Overtime tracking
   - Shift management
   - Integration with payroll system

2. **Platform Expansion**
   - iOS app
   - Web version for users
   - Desktop app for admin

3. **Improvements**
   - Real-time notifications
   - Geofencing alerts
   - Multiple location support per check-in
   - Attendance forecasting with ML
   - Integration with HR systems

---

## 📚 References & Resources

### Golang
- [Gin Framework](https://gin-gonic.com/)
- [GORM ORM](https://gorm.io/)
- [JWT Go](https://github.com/golang-jwt/jwt)

### Flutter
- [Flutter Documentation](https://flutter.dev/docs)
- [Geolocator Package](https://pub.dev/packages/geolocator)
- [Provider State Management](https://pub.dev/packages/provider)

### Next.js
- [Next.js Documentation](https://nextjs.org/docs)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [shadcn/ui Components](https://ui.shadcn.com/)

### Database
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostGIS Documentation](https://postgis.net/documentation/)

### Maps & GPS
- [Haversine Formula](https://en.wikipedia.org/wiki/Haversine_formula)
- [Leaflet.js](https://leafletjs.com/)
- [Google Maps API](https://developers.google.com/maps)

---

## ✅ Success Criteria

1. ✅ User dapat check-in hanya dalam radius 10 meter dari lokasi yang ditentukan
2. ✅ Akurasi GPS minimal ±5 meter
3. ✅ Response time API < 500ms
4. ✅ Mobile app kompatibel dengan Android 8.0+
5. ✅ Admin dashboard responsive di semua device
6. ✅ System dapat handle 1000+ concurrent users
7. ✅ Uptime 99.9%
8. ✅ Data attendance tersimpan dengan aman dan akurat

---

## 📞 Contact & Support

Untuk pertanyaan lebih lanjut tentang blueprint ini, silakan hubungi tim development.

---

**Version:** 1.0
**Last Updated:** 2025-10-29
**Status:** Blueprint Ready for Development
