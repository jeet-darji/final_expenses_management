# 🧪 Complete Testing Guide - Approval Workflow

## ⚠️ Why "Failed to load approvals" or "No pending approvals"?

This happens because:
1. ❌ No expenses have been submitted yet
2. ❌ No approval workflow has been created
3. ❌ Employee doesn't have a manager assigned
4. ❌ Backend error (check console)

---

## ✅ Complete Setup & Testing Steps

### **STEP 1: Create Users (Admin)**

1. **Login as Admin**
2. **Go to User Management** (`/admin/users`)
3. **Create Manager:**
   ```
   Name: Test Manager
   Email: manager@test.com
   Password: manager123
   Role: Manager
   Department: Sales
   Approval Limit: 5000
   ```
4. **Create Employee:**
   ```
   Name: Test Employee
   Email: employee@test.com
   Password: employee123
   Role: Employee
   Department: Sales
   Assign Manager: Test Manager ← IMPORTANT!
   ```

---

### **STEP 2: Create Approval Flow (Admin)**

1. **Go to Approval Flows** (`/admin/approval-flows`)
2. **Click "Create Flow"**
3. **Fill in:**
   ```
   Name: Standard Approval
   Description: Default approval workflow
   Min Amount: 0
   Max Amount: (leave empty)
   Default Flow: ✅ Check this
   ```
4. **Configure Step 1:**
   ```
   Step Name: Manager Approval
   Step Type: Single Approver
   Manager Step: ✅ Check this
   ```
5. **Click "Create Flow"**

---

### **STEP 3: Submit Expense (Employee)**

1. **Logout** (click logout in sidebar)
2. **Login as Employee:**
   ```
   Email: employee@test.com
   Password: employee123
   ```
3. **Go to Expenses** → **New Expense**
4. **Fill in:**
   ```
   Amount: 100
   Currency: USD
   Date: Today's date
   Category: Travel
   Description: Client meeting transportation
   Merchant: Uber
   ```
5. **Click "Submit Expense"**

---

### **STEP 4: Approve Expense (Manager)**

1. **Logout**
2. **Login as Manager:**
   ```
   Email: manager@test.com
   Password: manager123
   ```
3. **Go to Approvals**
4. **You should now see:**
   - ✅ Pending tab with 1 expense
   - Employee name, description, amount
   - Approve/Reject buttons

5. **Click Approve (green checkmark)**
6. **Expense moves to "Approved" tab** ✅

---

## 🔍 Troubleshooting

### **Issue 1: "Failed to load approvals"**

**Check Browser Console (F12):**
```
Look for error messages
Check Network tab for failed requests
```

**Common Causes:**
- Backend not running
- Database connection issue
- No token (need to re-login)

**Fix:**
```bash
# Restart backend
cd backend
npm start

# Refresh frontend
Ctrl+R
```

---

### **Issue 2: "No pending approvals" (but no error)**

**This is NORMAL if:**
- ✅ No expenses submitted yet
- ✅ All expenses already approved/rejected
- ✅ You're not assigned as approver

**To Fix:**
1. Submit an expense as employee
2. Make sure employee has manager assigned
3. Make sure approval flow exists

---

### **Issue 3: Employee can't submit expense**

**Check:**
- Is approval flow created?
- Is it set as default?
- Check backend console for errors

**Fix:**
- Create default approval flow
- Restart backend

---

### **Issue 4: Manager doesn't see expense**

**Check:**
- Is employee's `managerId` set to this manager?
- Is approval flow using `isManagerStep`?
- Check backend logs

**Fix:**
```sql
-- Check in database
SELECT id, name, email, role, manager_id 
FROM users 
WHERE role = 'employee';

-- Update if needed
UPDATE users 
SET manager_id = '<manager-uuid>' 
WHERE email = 'employee@test.com';
```

---

## 📊 Database Verification

### **Check if users exist:**
```sql
SELECT id, name, email, role, manager_id, can_approve_expenses
FROM users
ORDER BY role;
```

### **Check if approval flow exists:**
```sql
SELECT id, name, is_default, is_active
FROM approval_flows;
```

### **Check approval steps:**
```sql
SELECT af.name as flow_name, 
       ast.step_number, 
       ast.name as step_name,
       ast.is_manager_step
FROM approval_flows af
JOIN approval_steps ast ON ast.approval_flow_id = af.id
ORDER BY af.name, ast.step_number;
```

### **Check if expenses exist:**
```sql
SELECT id, submitter_id, amount, status, approval_flow_id
FROM expenses
ORDER BY created_at DESC
LIMIT 10;
```

### **Check if approvals were created:**
```sql
SELECT a.id, a.status, a.approver_id, 
       e.description, e.amount,
       u.name as approver_name
FROM approvals a
JOIN expenses e ON e.id = a.expense_id
JOIN users u ON u.id = a.approver_id
ORDER BY a.created_at DESC;
```

---

## 🎯 Quick Test Script

Run this in order:

```bash
# 1. Check backend is running
curl http://localhost:3000/api/auth/login

# 2. Login as admin (save token)
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@company.com","password":"your-password"}'

# 3. Check if users exist
curl http://localhost:3000/api/companies/{companyId}/users \
  -H "Authorization: Bearer <admin-token>"

# 4. Check if approval flows exist
curl http://localhost:3000/api/companies/{companyId}/approval-flows \
  -H "Authorization: Bearer <admin-token>"
```

---

## ✅ Success Checklist

Before testing approvals, verify:

- [ ] Backend is running (`npm start` in backend folder)
- [ ] Frontend is running (`npm start` in frontend folder)
- [ ] Database is running (PostgreSQL)
- [ ] Admin user exists
- [ ] Manager user created with `canApproveExpenses: true`
- [ ] Employee user created with `managerId` set
- [ ] Approval flow created and set as default
- [ ] Approval flow has at least one step with `isManagerStep: true`

---

## 🚀 Expected Flow

```
1. Employee submits expense
   ↓
2. System finds default approval flow
   ↓
3. System creates approval for Step 1 (Manager)
   ↓
4. Manager logs in
   ↓
5. Manager goes to Approvals page
   ↓
6. Manager sees expense in "Pending" tab ✅
   ↓
7. Manager clicks Approve
   ↓
8. Expense moves to "Approved" tab ✅
   ↓
9. Expense status = "approved" ✅
```

---

## 📞 Still Not Working?

### **Check Backend Console:**
Look for errors when:
- Employee submits expense
- Manager opens approvals page

### **Check Browser Console (F12):**
- Network tab: Check API calls
- Console tab: Check for JavaScript errors

### **Common Error Messages:**

**"No applicable approval flow found"**
- Create a default approval flow
- Set `isDefault: true`
- Set `minAmount: 0`

**"Manager not found"**
- Employee doesn't have managerId set
- Update employee to assign manager

**"Cannot read property 'id' of null"**
- User object not loaded
- Try re-login

---

## 🎉 When It Works

You should see:
- ✅ Pending tab shows submitted expenses
- ✅ Badge count shows number of pending items
- ✅ Approve button works
- ✅ Reject button works
- ✅ Items move between tabs correctly
- ✅ No error messages

---

**Follow these steps in order and it will work!** 🚀
