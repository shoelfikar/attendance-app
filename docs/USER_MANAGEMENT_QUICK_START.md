# User Management - Quick Start Guide

**Admin Dashboard User Management**
**Version:** 1.0.0
**Last Updated:** October 30, 2025

---

## 🚀 Quick Start (5 Minutes)

### Step 1: Start Backend Server
```bash
cd attendance-backend
go run cmd/api/main.go
```

✅ Wait for: **"Server starting on port 8000"**

### Step 2: Start Frontend Dashboard
```bash
cd attendance-admin
npm run dev
```

✅ Wait for: **"Ready in 2.5s"**

### Step 3: Login to Dashboard
1. Open: http://localhost:3000
2. Login with:
   - **Email:** `admin@attendance.com`
   - **Password:** `admin123`

### Step 4: Access User Management
1. Click **"Users"** in sidebar
2. You're now on the Users Management page!

---

## 📋 Features Overview

### What You Can Do:

| Feature | Description | How To |
|---------|-------------|--------|
| **View Users** | See all registered users | Navigate to Users page |
| **Add User** | Create new user account | Click "Add User" button |
| **Edit User** | Update user information | Click "Edit" on any user |
| **Delete User** | Remove user from system | Click "Delete" on any user |
| **Activate/Deactivate** | Enable or disable user login | Click "Activate"/"Deactivate" |
| **Change Role** | Promote user to admin | Edit user → Change role |

---

## ➕ How to Create a User

### Quick Steps:
1. Click **"Add User"** button (top right)
2. Fill in the form:
   - **Full Name:** User's full name
   - **Email:** Must be unique
   - **Password:** Minimum 6 characters
   - **Phone:** Optional
   - **Role:** Select "User" or "Admin"
3. Click **"Create User"**
4. Done! User appears in table

### Example:
```
Full Name: John Doe
Email: john@example.com
Password: john123
Phone: +1234567890
Role: User
```

---

## ✏️ How to Edit a User

### Quick Steps:
1. Find user in table
2. Click **"Edit"** button
3. Modify any field:
   - Full Name
   - Email
   - Phone
   - Role
4. Click **"Update User"**
5. Done! Changes saved

**Note:** Password field is empty in edit mode for security. To change password, use separate change password feature (if implemented) or recreate the user.

---

## 🔄 How to Toggle User Status

### Deactivate User:
1. Find active user (green "ACTIVE" badge)
2. Click **"Deactivate"** button
3. Status changes to red "INACTIVE"
4. User cannot login anymore

### Activate User:
1. Find inactive user (red "INACTIVE" badge)
2. Click **"Activate"** button
3. Status changes to green "ACTIVE"
4. User can login again

**Use Case:** Temporarily disable user access without deleting the account.

---

## 🗑️ How to Delete a User

### Quick Steps:
1. Find user in table
2. Click **"Delete"** button
3. Confirm deletion in dialog
4. User removed from system

### ⚠️ Important Rules:
- ❌ Cannot delete your own account
- ❌ Cannot delete the last admin user
- ✅ Deletion is permanent (no undo)

---

## 👥 User Roles Explained

### Admin Role
- **Badge Color:** Purple
- **Permissions:**
  - Full access to admin dashboard
  - Manage users, locations, attendances
  - View all reports
  - Change system settings

### User Role
- **Badge Color:** Blue
- **Permissions:**
  - Check-in/check-out attendance (mobile app)
  - View own attendance history
  - View own schedule
  - Update own profile

### How to Promote User to Admin:
1. Click "Edit" on user
2. Change Role to "Admin"
3. Click "Update User"
4. User can now access admin dashboard

---

## 🎨 User Interface Guide

### User Table Columns

| Column | Description |
|--------|-------------|
| **User** | Avatar + Full Name + User ID |
| **Email** | User's email address |
| **Phone** | Phone number (if provided) |
| **Role** | ADMIN (purple) or USER (blue) badge |
| **Status** | ACTIVE (green) or INACTIVE (red) badge |
| **Joined** | Account creation date |
| **Actions** | Edit, Activate/Deactivate, Delete buttons |

### Visual Indicators

**Status Badges:**
- 🟢 **ACTIVE** (Green) - User can login
- 🔴 **INACTIVE** (Red) - User cannot login

**Role Badges:**
- 🟣 **ADMIN** (Purple) - Administrator
- 🔵 **USER** (Blue) - Regular user

**Avatars:**
- Circle with first letter of name
- Blue background
- White text

---

## ⚡ Common Tasks

### Task 1: Add New Employee
```
Action: Create User
Role: User
Example:
  - Name: Jane Smith
  - Email: jane.smith@company.com
  - Password: temp123
  - Phone: +1234567890
```

### Task 2: Create Another Admin
```
Action: Create User
Role: Admin
Example:
  - Name: Admin Two
  - Email: admin2@company.com
  - Password: admin456
```

### Task 3: Disable Former Employee
```
Action: Deactivate User
Don't Delete: Keep for records
Result: User cannot login but data preserved
```

### Task 4: Reactivate Returning Employee
```
Action: Activate User
Result: User can login again with same credentials
```

### Task 5: Update Employee Information
```
Action: Edit User
Update: Name, Phone, Email as needed
```

---

## 🔒 Security Notes

### Password Security
- ✅ Passwords are hashed (bcrypt)
- ✅ Never shown in responses
- ✅ Minimum 6 characters required
- ✅ Cannot view other users' passwords

### Access Control
- ✅ Only admins can access user management
- ✅ JWT authentication required
- ✅ Session expires after 15 minutes of inactivity
- ✅ Auto-logout on token expiration

### Data Protection
- ✅ Email uniqueness enforced
- ✅ Cannot delete own account (safety)
- ✅ Cannot delete last admin (safety)
- ✅ Audit trail via timestamps

---

## 🐛 Troubleshooting

### Problem: "Email already exists"
**Solution:** Use a different email address. Each user must have unique email.

### Problem: Cannot delete user
**Possible Causes:**
1. Trying to delete your own account → Not allowed
2. Trying to delete last admin → Not allowed
3. Network error → Check backend is running

### Problem: User not appearing in table
**Solution:**
1. Check if creation was successful
2. Refresh the page
3. Check browser console for errors
4. Verify backend is running

### Problem: Cannot login after creating user
**Possible Causes:**
1. User is inactive → Activate user
2. Wrong password → Recreate user or reset password
3. Email typo → Edit user to fix email

---

## 📊 Statistics

View user statistics at the top of the page (if implemented):
- **Total Users:** All users in system
- **Active Users:** Users that can login
- **Inactive Users:** Deactivated accounts
- **Admin Users:** Number of administrators
- **Regular Users:** Number of regular users

Access via: GET `/api/v1/admin/users/stats`

---

## 💡 Best Practices

### ✅ Do's
- ✅ Use descriptive full names
- ✅ Use company email addresses
- ✅ Set strong passwords for admins
- ✅ Deactivate instead of delete when possible
- ✅ Keep at least 2 admin accounts
- ✅ Regularly review active users
- ✅ Update user info when changed

### ❌ Don'ts
- ❌ Don't share admin credentials
- ❌ Don't use weak passwords
- ❌ Don't delete users if you need their data
- ❌ Don't create duplicate accounts
- ❌ Don't leave test accounts active
- ❌ Don't delete the last admin

---

## 🔗 Related Features

### Coming Soon
- 📧 Email notifications on account creation
- 🔑 Password reset functionality
- 📊 User activity logs
- 🔍 Advanced search & filters
- 📄 Pagination for large user lists
- 📤 Export users to CSV/Excel
- 👤 User profile pictures

---

## 📞 Support

### Need Help?
- 📖 Documentation: [API_USER_MANAGEMENT.md](attendance-backend/API_USER_MANAGEMENT.md)
- 🧪 Testing Guide: [USER_MANAGEMENT_TESTING_GUIDE.md](USER_MANAGEMENT_TESTING_GUIDE.md)
- 💻 Implementation Details: [USER_MANAGEMENT_IMPLEMENTATION.md](USER_MANAGEMENT_IMPLEMENTATION.md)

### Found a Bug?
Report issues with:
- Clear description
- Steps to reproduce
- Expected vs actual result
- Screenshots if applicable

---

## 🎯 Quick Reference

### API Endpoints
```
GET    /api/v1/admin/users          - Get all users
GET    /api/v1/admin/users/:id      - Get user by ID
POST   /api/v1/admin/users          - Create user
PUT    /api/v1/admin/users/:id      - Update user
DELETE /api/v1/admin/users/:id      - Delete user
PUT    /api/v1/admin/users/:id/password - Change password
GET    /api/v1/admin/users/stats    - Get statistics
```

### Required Fields
- ✅ Full Name
- ✅ Email (unique)
- ✅ Password (6+ characters)
- ✅ Role (admin/user)
- ⭕ Phone (optional)

### Keyboard Shortcuts (Future)
- `Ctrl + N` - New user
- `Ctrl + F` - Search users
- `Esc` - Close modal

---

**Happy User Managing! 🎉**

If you have questions or need assistance, refer to the documentation or contact your system administrator.

---

**Version:** 1.0.0
**Last Updated:** October 30, 2025
**Author:** Claude Code Assistant
