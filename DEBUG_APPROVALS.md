# 🔍 Debug Guide - Why Approvals Not Showing

## Step-by-Step Debugging

### **Step 1: Check if Approval Flow Exists**

Open pgAdmin and run:
```sql
-- Check if any approval flows exist
SELECT id, name, is_default, is_active, company_id
FROM approval_flows;
```

**Expected Result:** At least 1 row with `is_default = true`

**If NO rows:** You need to create an approval flow!

---

### **Step 2: Check if Approval Steps Exist**

```sql
-- Check approval steps
SELECT 
    af.name as flow_name,
    ast.step_number,
    ast.name as step_name,
    ast.is_manager_step,
    ast.is_active
FROM approval_flows af
JOIN approval_steps ast ON ast.approval_flow_id = af.id
ORDER BY af.name, ast.step_number;
```

**Expected Result:** At least 1 step with `is_manager_step = true`

**If NO rows:** The flow has no steps!

---

### **Step 3: Check if Expenses Exist**

```sql
-- Check if any expenses were submitted
SELECT 
    id,
    submitter_id,
    amount,
    description,
    status,
    approval_flow_id,
    current_step_id,
    created_at
FROM expenses
ORDER BY created_at DESC
LIMIT 10;
```

**Expected Result:** At least 1 expense

**If NO rows:** No expenses have been submitted yet!

---

### **Step 4: Check if Approvals Were Created**

```sql
-- Check if approvals exist
SELECT 
    a.id,
    a.status,
    a.approver_id,
    e.description as expense_description,
    e.amount,
    u.name as approver_name,
    u.email as approver_email
FROM approvals a
JOIN expenses e ON e.id = a.expense_id
LEFT JOIN users u ON u.id = a.approver_id
ORDER BY a.created_at DESC;
```

**Expected Result:** Approvals with `approver_id` matching manager's ID

**If NO rows:** Workflow didn't create approvals!

---

### **Step 5: Check Manager Assignment**

```sql
-- Check if employee has manager assigned
SELECT 
    e.id,
    e.name as employee_name,
    e.email as employee_email,
    e.role,
    e.manager_id,
    m.name as manager_name,
    m.email as manager_email,
    m.can_approve_expenses
FROM users e
LEFT JOIN users m ON m.id = e.manager_id
WHERE e.role = 'employee';
```

**Expected Result:** 
- `manager_id` is NOT NULL
- `manager.can_approve_expenses` = true

**If manager_id is NULL:** Employee has no manager assigned!

---

## 🔧 Quick Fixes

### **Fix 1: Create Approval Flow via SQL**

1. Get your company ID:
```sql
SELECT id, name FROM companies;
```

2. Run the script:
```bash
# Edit create-default-flow.sql and replace YOUR_COMPANY_ID
# Then run it in pgAdmin
```

OR use the UI:
- Login as Admin
- Go to `/admin/approval-flows`
- Click "Create Flow"
- Fill in the form

---

### **Fix 2: Assign Manager to Employee**

```sql
-- Get manager ID
SELECT id, name, email FROM users WHERE role = 'manager';

-- Assign manager to employee
UPDATE users 
SET manager_id = 'MANAGER_UUID_HERE'
WHERE email = 'employee@test.com';
```

OR use the UI:
- Login as Admin
- Go to `/admin/users`
- Edit employee
- Assign manager

---

### **Fix 3: Enable Approval Permissions for Manager**

```sql
UPDATE users 
SET can_approve_expenses = true
WHERE role = 'manager';
```

---

## 🧪 Complete Test Scenario

### **Setup (Do Once):**

1. **Create Approval Flow:**
```sql
-- Use create-default-flow.sql
-- OR create via UI at /admin/approval-flows
```

2. **Verify Users:**
```sql
-- Check users exist
SELECT id, name, email, role, manager_id, can_approve_expenses
FROM users
ORDER BY role;
```

3. **Assign Manager:**
```sql
-- Update employee to have manager
UPDATE users 
SET manager_id = (SELECT id FROM users WHERE role = 'manager' LIMIT 1)
WHERE role = 'employee';
```

---

### **Test Flow:**

1. **Submit Expense (Employee):**
   - Login as employee@test.com
   - Go to Expenses → New Expense
   - Fill form and submit
   - **Check backend console** for workflow initialization logs

2. **Check Database:**
```sql
-- Verify expense was created
SELECT id, description, status, approval_flow_id 
FROM expenses 
ORDER BY created_at DESC 
LIMIT 1;

-- Verify approval was created
SELECT a.*, u.name as approver_name
FROM approvals a
JOIN users u ON u.id = a.approver_id
ORDER BY a.created_at DESC 
LIMIT 1;
```

3. **View Approvals (Manager):**
   - Login as manager@test.com
   - Go to Approvals
   - Should see the expense

---

## 🐛 Common Issues

### **Issue 1: "No applicable approval flow found"**

**Cause:** No default approval flow exists

**Fix:**
```sql
-- Check if flow exists
SELECT * FROM approval_flows WHERE is_default = true;

-- If not, create one using create-default-flow.sql
```

---

### **Issue 2: "Manager not found"**

**Cause:** Employee's `manager_id` is NULL

**Fix:**
```sql
-- Assign manager
UPDATE users 
SET manager_id = '<manager-uuid>'
WHERE email = 'employee@test.com';
```

---

### **Issue 3: Manager sees "No pending approvals"**

**Possible Causes:**
1. No expenses submitted
2. Approvals not created (workflow failed)
3. Wrong manager assigned
4. Manager's `can_approve_expenses` is false

**Debug:**
```sql
-- Check if approvals exist for this manager
SELECT COUNT(*) 
FROM approvals 
WHERE approver_id = '<manager-uuid>';

-- Check manager permissions
SELECT id, name, can_approve_expenses 
FROM users 
WHERE id = '<manager-uuid>';
```

---

### **Issue 4: Workflow initialization fails**

**Check backend console for errors:**
```
Error initializing workflow: ...
```

**Common errors:**
- "No applicable approval flow found" → Create default flow
- "Manager not found" → Assign manager to employee
- "Cannot read property 'id'" → Check associations

---

## ✅ Success Checklist

Before submitting an expense, verify:

- [ ] Approval flow exists (`SELECT * FROM approval_flows WHERE is_default = true`)
- [ ] Approval flow has steps (`SELECT * FROM approval_steps`)
- [ ] Manager user exists with `can_approve_expenses = true`
- [ ] Employee has `manager_id` set
- [ ] Backend is running without errors
- [ ] Frontend is connected to backend

After submitting expense, verify:

- [ ] Expense created (`SELECT * FROM expenses ORDER BY created_at DESC LIMIT 1`)
- [ ] Approval created (`SELECT * FROM approvals ORDER BY created_at DESC LIMIT 1`)
- [ ] Approval has correct `approver_id` (should be manager's ID)
- [ ] Manager can see it in UI

---

## 🚀 Quick Setup Script

Run these in order:

```sql
-- 1. Get company ID
SELECT id FROM companies LIMIT 1;

-- 2. Create approval flow (replace company_id)
INSERT INTO approval_flows (id, company_id, name, is_default, min_amount, is_active, created_at, updated_at)
VALUES (gen_random_uuid(), 'YOUR_COMPANY_ID', 'Standard Approval', true, 0, true, NOW(), NOW())
RETURNING id;

-- 3. Create approval step (replace approval_flow_id with ID from step 2)
INSERT INTO approval_steps (id, approval_flow_id, step_number, name, step_type, is_manager_step, approval_threshold, required_approvers, is_active, created_at, updated_at, assigned_role_ids, assigned_user_ids, specific_approvers)
VALUES (gen_random_uuid(), 'FLOW_ID_FROM_STEP_2', 1, 'Manager Approval', 'single', true, 1.0, 1, true, NOW(), NOW(), '{}', '{}', '{}');

-- 4. Get manager ID
SELECT id FROM users WHERE role = 'manager' LIMIT 1;

-- 5. Assign manager to employee (replace IDs)
UPDATE users SET manager_id = 'MANAGER_ID' WHERE role = 'employee';

-- 6. Verify setup
SELECT 
    'Flows' as type, COUNT(*)::text as count FROM approval_flows WHERE is_default = true
UNION ALL
SELECT 'Steps', COUNT(*)::text FROM approval_steps WHERE is_active = true
UNION ALL
SELECT 'Employees with Manager', COUNT(*)::text FROM users WHERE role = 'employee' AND manager_id IS NOT NULL;
```

---

**Follow this guide step by step to find and fix the issue!** 🔍
