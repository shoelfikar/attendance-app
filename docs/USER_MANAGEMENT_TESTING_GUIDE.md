# User Management - End-to-End Testing Guide

**Date:** October 30, 2025
**Feature:** User Management CRUD (Admin Dashboard)
**Status:** ✅ Ready for Testing

---

## Overview

Panduan lengkap untuk testing fitur User Management di admin dashboard yang terintegrasi dengan backend API.

---

## Prerequisites

### 1. Backend Running
```bash
cd attendance-backend
go run cmd/api/main.go
```

**Expected Output:**
```
Database connected successfully
🚀 Server starting on port 8000
📝 Environment: debug
💾 Database: attendance_db
```

**Verify:** http://localhost:8000/health should return:
```json
{
  "status": "success",
  "message": "Attendance API is running",
  "version": "1.0.0"
}
```

### 2. Frontend Running
```bash
cd attendance-admin
npm run dev
```

**Expected Output:**
```
  ▲ Next.js 15.5.6
  - Local:        http://localhost:3000
  - Ready in 2.5s
```

### 3. Admin Account
**Login Credentials:**
- Email: `admin@attendance.com`
- Password: `admin123`

---

## Test Scenarios

### Scenario 1: Login & Access Users Page

#### Steps:
1. Open browser: http://localhost:3000
2. Should redirect to: http://localhost:3000/login
3. Enter credentials:
   - Email: `admin@attendance.com`
   - Password: `admin123`
4. Click "Sign in"
5. Should redirect to: http://localhost:3000/dashboard
6. Click "Users" in sidebar
7. Should navigate to: http://localhost:3000/dashboard/users

#### Expected Result:
- ✅ Login successful
- ✅ Dashboard loads
- ✅ Users page displays
- ✅ User list table shows existing users
- ✅ "Add User" button visible

#### Screenshot Points:
- [ ] Login page
- [ ] Dashboard overview
- [ ] Users page with table

---

### Scenario 2: View All Users

#### Steps:
1. Navigate to Users page
2. Observe the table

#### Expected Result:
- ✅ Table shows all users
- ✅ Columns displayed:
  - User (with avatar)
  - Email
  - Phone
  - Role (ADMIN/USER badge)
  - Status (ACTIVE/INACTIVE badge)
  - Joined date
  - Actions (Edit, Activate/Deactivate, Delete)
- ✅ Default admin user visible

#### Verification:
- User count matches backend data
- All user information displayed correctly
- Avatar shows first letter of name
- Badges have correct colors

---

### Scenario 3: Create New User

#### Steps:
1. Click "Add User" button
2. Modal opens with form
3. Fill in details:
   - **Full Name:** "John Doe"
   - **Email:** "john@example.com"
   - **Password:** "john123"
   - **Phone:** "+1234567890"
   - **Role:** Select "User"
4. Click "Create User"

#### Expected Result:
- ✅ Modal opens smoothly
- ✅ Form fields are empty
- ✅ All fields accept input
- ✅ On submit:
  - Modal closes
  - Success message (if implemented)
  - Table refreshes
  - New user appears in table
- ✅ User created with:
  - ID auto-generated
  - is_active = true (default)
  - created_at = current timestamp

#### Backend Verification:
Check backend logs for:
```
[GIN] 2025/10/30 - 12:00:00 | 201 | POST "/api/v1/admin/users"
```

#### Database Verification:
```sql
SELECT * FROM users WHERE email = 'john@example.com';
```

Should return the newly created user.

#### Error Cases to Test:
- [ ] Empty required fields → Should show validation error
- [ ] Invalid email format → Should show error
- [ ] Password < 6 characters → Should show error
- [ ] Duplicate email → Should show "email already exists"

---

### Scenario 4: View User Details

#### Steps:
1. Find user "John Doe" in table
2. Click user row or view icon (if implemented)

#### Expected Result:
- ✅ User details displayed:
  - Full name
  - Email
  - Phone
  - Role
  - Status
  - Created date
  - Updated date

---

### Scenario 5: Edit User

#### Steps:
1. Click "Edit" button on user "John Doe"
2. Modal opens with pre-filled data
3. Modify fields:
   - **Full Name:** "John Doe Updated"
   - **Phone:** "+0987654321"
4. Click "Update User"

#### Expected Result:
- ✅ Modal opens with existing data
- ✅ Password field empty (security)
- ✅ On submit:
  - Modal closes
  - Table refreshes
  - User data updated in table
- ✅ Updated fields show new values
- ✅ updated_at timestamp changed

#### Backend Verification:
```
[GIN] 2025/10/30 - 12:05:00 | 200 | PUT "/api/v1/admin/users/2"
```

#### Error Cases to Test:
- [ ] Change email to existing email → "email already exists"
- [ ] Invalid email format → Validation error

---

### Scenario 6: Toggle User Active Status

#### Steps:
1. Find user "John Doe" in table
2. Current status: ACTIVE (green badge)
3. Click "Deactivate" button
4. Wait for update

#### Expected Result:
- ✅ Button changes from "Deactivate" to "Activate"
- ✅ Status badge changes from green to red
- ✅ Badge text changes to "INACTIVE"
- ✅ Table refreshes
- ✅ is_active = false in database

#### Reactivate:
1. Click "Activate" button
2. Status changes back to ACTIVE

---

### Scenario 7: Change User Role

#### Steps:
1. Click "Edit" on user "John Doe"
2. Change **Role** from "User" to "Admin"
3. Click "Update User"

#### Expected Result:
- ✅ Role dropdown shows both options
- ✅ On submit:
  - User role updated
  - Badge changes from "USER" (blue) to "ADMIN" (purple)
  - Table refreshes

#### Backend Verification:
```sql
SELECT role FROM users WHERE email = 'john@example.com';
-- Should return: 'admin'
```

---

### Scenario 8: Delete User

#### Steps:
1. Click "Delete" button on user "John Doe"
2. Confirmation dialog appears
3. Confirm deletion

#### Expected Result:
- ✅ Confirmation dialog shows
- ✅ Dialog message: "Are you sure you want to delete this user?"
- ✅ On confirm:
  - User removed from table
  - Table refreshes
  - User deleted from database

#### Backend Verification:
```
[GIN] 2025/10/30 - 12:10:00 | 200 | DELETE "/api/v1/admin/users/2"
```

#### Database Verification:
```sql
SELECT * FROM users WHERE email = 'john@example.com';
-- Should return: 0 rows
```

#### Error Cases to Test:
- [ ] Try to delete own account → "Cannot delete your own account"
- [ ] Try to delete last admin → "cannot delete the last admin user"

---

### Scenario 9: Create Multiple Users

#### Steps:
Create 5 users with following data:

**User 1:**
- Full Name: "Alice Smith"
- Email: "alice@example.com"
- Password: "alice123"
- Phone: "+1111111111"
- Role: "User"

**User 2:**
- Full Name: "Bob Johnson"
- Email: "bob@example.com"
- Password: "bob123"
- Phone: "+2222222222"
- Role: "User"

**User 3:**
- Full Name: "Carol White"
- Email: "carol@example.com"
- Password: "carol123"
- Phone: "+3333333333"
- Role: "Admin"

**User 4:**
- Full Name: "David Brown"
- Email: "david@example.com"
- Password: "david123"
- Phone: "+4444444444"
- Role: "User"

**User 5:**
- Full Name: "Eve Davis"
- Email: "eve@example.com"
- Password: "eve123"
- Phone: "+5555555555"
- Role: "User"

#### Expected Result:
- ✅ All 5 users created successfully
- ✅ Table shows all users
- ✅ Users sorted by created_at (newest first)

---

### Scenario 10: Bulk Testing

#### Actions to Test:
1. ✅ Create 10 users rapidly
2. ✅ Edit multiple users
3. ✅ Toggle status on multiple users
4. ✅ Delete multiple users

#### Expected Result:
- No errors
- All operations complete successfully
- Table updates correctly after each action
- No data corruption

---

### Scenario 11: Error Handling

#### Test Cases:

**1. Network Error**
- Stop backend server
- Try to load users page
- **Expected:** Error message displayed
- Restart backend
- Refresh page
- **Expected:** Users load successfully

**2. Invalid Token**
- Clear localStorage
- Try to access users page
- **Expected:** Redirect to login

**3. Token Expired**
- Wait for token to expire (15 minutes)
- Try to perform action
- **Expected:**
  - Auto token refresh
  - Action completes successfully
  - OR redirect to login if refresh fails

**4. Validation Errors**
- Try to create user with invalid data
- **Expected:** Clear error messages

**5. Duplicate Email**
- Create user with existing email
- **Expected:** "email already exists" error

---

### Scenario 12: UI/UX Testing

#### Visual Checks:
- [ ] Table is responsive
- [ ] Modals are centered
- [ ] Buttons have hover states
- [ ] Loading spinners show during operations
- [ ] Empty states handled gracefully
- [ ] Badges have correct colors
- [ ] Icons display properly
- [ ] Forms are user-friendly

#### Interaction Checks:
- [ ] Modal opens smoothly
- [ ] Modal closes with X button
- [ ] Modal closes with Cancel button
- [ ] Form submits with Enter key (if applicable)
- [ ] Table rows are hoverable
- [ ] Buttons are clickable
- [ ] Dropdowns work correctly

---

## API Testing (Backend)

### Using cURL

**1. Get All Users**
```bash
# First, login to get token
TOKEN=$(curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@attendance.com","password":"admin123"}' \
  | jq -r '.data.access_token')

# Get all users
curl -X GET http://localhost:8000/api/v1/admin/users \
  -H "Authorization: Bearer $TOKEN" | jq
```

**2. Create User**
```bash
curl -X POST http://localhost:8000/api/v1/admin/users \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "test123",
    "full_name": "Test User",
    "phone": "+1234567890",
    "role": "user"
  }' | jq
```

**3. Update User**
```bash
curl -X PUT http://localhost:8000/api/v1/admin/users/2 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "full_name": "Updated Name"
  }' | jq
```

**4. Delete User**
```bash
curl -X DELETE http://localhost:8000/api/v1/admin/users/2 \
  -H "Authorization: Bearer $TOKEN" | jq
```

**5. Get User Stats**
```bash
curl -X GET http://localhost:8000/api/v1/admin/users/stats \
  -H "Authorization: Bearer $TOKEN" | jq
```

---

## Performance Testing

### Load Test
1. Create 100 users
2. Measure page load time
3. Check table rendering performance
4. Verify no lag when scrolling

### Expected Performance:
- Page load: < 2 seconds
- Create user: < 500ms
- Update user: < 500ms
- Delete user: < 500ms
- Table render (100 users): < 1 second

---

## Browser Compatibility

Test on:
- [ ] Chrome (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest)
- [ ] Edge (latest)

---

## Mobile Responsiveness

Test on:
- [ ] Mobile (375px)
- [ ] Tablet (768px)
- [ ] Desktop (1024px+)

---

## Checklist: Complete Test Coverage

### Basic CRUD ✅
- [ ] Create user
- [ ] Read users (list)
- [ ] Read user (single)
- [ ] Update user
- [ ] Delete user

### Advanced Features ✅
- [ ] Toggle active status
- [ ] Change role
- [ ] View statistics
- [ ] Form validation
- [ ] Error handling

### Security ✅
- [ ] JWT authentication required
- [ ] Admin role required
- [ ] Cannot delete own account
- [ ] Cannot delete last admin
- [ ] Password not shown in responses

### UX/UI ✅
- [ ] Loading states
- [ ] Error messages
- [ ] Success feedback
- [ ] Responsive design
- [ ] Accessibility

---

## Known Issues

None currently identified.

---

## Bug Reporting Template

If you find bugs, report using this format:

```
**Title:** [Short description]

**Severity:** Critical / High / Medium / Low

**Steps to Reproduce:**
1.
2.
3.

**Expected Result:**


**Actual Result:**


**Screenshots:**
[Attach if applicable]

**Environment:**
- Browser:
- OS:
- Backend Version:
- Frontend Version:
```

---

## Success Criteria

All tests must pass:
- ✅ All CRUD operations work
- ✅ No console errors
- ✅ No network errors
- ✅ Proper error handling
- ✅ Good UX/UI
- ✅ Responsive on all devices
- ✅ Security features working

---

## Next Steps After Testing

1. Fix any bugs found
2. Optimize performance if needed
3. Add any missing features
4. Document any edge cases
5. Deploy to production

---

**Tester:** _____________
**Date:** _____________
**Test Result:** ☐ Pass ☐ Fail
**Notes:**
_______________________________________________________

---

**Version:** 1.0.0
**Last Updated:** October 30, 2025
**Author:** Claude Code Assistant
