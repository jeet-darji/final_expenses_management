# Approval Workflow Testing Guide

## Quick Start Testing

### Prerequisites
1. ✅ Database constraint fixed (run the SQL fix)
2. ✅ Backend server running
3. ✅ At least 3 users created:
   - Admin (role: 'admin')
   - Manager (role: 'manager', canApproveExpenses: true)
   - Employee (role: 'employee', managerId: <manager-id>)

---

## Test Scenario 1: Simple Manager Approval

### Step 1: Create Default Approval Flow (Admin)
```http
POST http://localhost:3000/api/companies/{companyId}/approval-flows
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "name": "Standard Expense Approval",
  "description": "Default workflow for all expenses",
  "isDefault": true,
  "minAmount": 0,
  "maxAmount": null,
  "category": null,
  "steps": [
    {
      "stepNumber": 1,
      "name": "Manager Approval",
      "stepType": "single",
      "isManagerStep": true,
      "escalationMinutes": 4320
    }
  ]
}
```

### Step 2: Employee Submits Expense
```http
POST http://localhost:3000/api/companies/{companyId}/expenses
Authorization: Bearer <employee-token>
Content-Type: application/json

{
  "amount": 150.00,
  "currency": "USD",
  "dateOfExpense": "2025-10-04",
  "category": "Travel",
  "description": "Client meeting transportation costs",
  "merchant": "Uber",
  "location": "New York"
}
```

**Expected Result:**
- Expense created with status: "pending"
- Approval created for manager
- Manager receives notification (if implemented)

### Step 3: Manager Views Pending Approvals
```http
GET http://localhost:3000/api/approvals/pending
Authorization: Bearer <manager-token>
```

**Expected Response:**
```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "id": "approval-uuid",
      "expense": {
        "amount": 150.00,
        "description": "Client meeting transportation costs",
        "submitter": {
          "name": "Employee Name"
        }
      },
      "approvalStep": {
        "stepNumber": 1,
        "name": "Manager Approval"
      },
      "status": "pending"
    }
  ]
}
```

### Step 4: Manager Approves Expense
```http
POST http://localhost:3000/api/approvals/{approvalId}/decision
Authorization: Bearer <manager-token>
Content-Type: application/json

{
  "decision": "approved",
  "comments": "Legitimate business expense, approved."
}
```

**Expected Result:**
- Approval status: "approved"
- Expense status: "approved" (no more steps)
- Employee notified

---

## Test Scenario 2: Multi-Step Approval (Manager → Finance → Director)

### Step 1: Create Multi-Step Flow (Admin)
```http
POST http://localhost:3000/api/companies/{companyId}/approval-flows
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "name": "High-Value Expense Approval",
  "description": "For expenses above $500",
  "minAmount": 500.00,
  "maxAmount": null,
  "category": null,
  "steps": [
    {
      "stepNumber": 1,
      "name": "Manager Approval",
      "stepType": "single",
      "isManagerStep": true,
      "escalationMinutes": 4320
    },
    {
      "stepNumber": 2,
      "name": "Finance Review",
      "stepType": "single",
      "assignedUserIds": ["<finance-user-id>"],
      "escalationMinutes": 2880
    },
    {
      "stepNumber": 3,
      "name": "Director Approval",
      "stepType": "single",
      "assignedUserIds": ["<director-user-id>"],
      "escalationMinutes": 1440
    }
  ]
}
```

### Step 2: Employee Submits High-Value Expense
```http
POST http://localhost:3000/api/companies/{companyId}/expenses
Authorization: Bearer <employee-token>
Content-Type: application/json

{
  "amount": 750.00,
  "currency": "USD",
  "dateOfExpense": "2025-10-04",
  "category": "Travel",
  "description": "Conference attendance and accommodation",
  "merchant": "Hilton Hotel",
  "location": "San Francisco"
}
```

**Expected Result:**
- Expense at Step 1 (Manager Approval)
- Only manager sees it in pending approvals

### Step 3: Manager Approves (Step 1)
```http
POST http://localhost:3000/api/approvals/{approval-step1-id}/decision
Authorization: Bearer <manager-token>
Content-Type: application/json

{
  "decision": "approved",
  "comments": "Conference is relevant to business goals."
}
```

**Expected Result:**
- Step 1 completed
- Expense moves to Step 2 (Finance Review)
- Finance user now sees it in pending approvals
- Manager no longer sees it

### Step 4: Finance Approves (Step 2)
```http
POST http://localhost:3000/api/approvals/{approval-step2-id}/decision
Authorization: Bearer <finance-token>
Content-Type: application/json

{
  "decision": "approved",
  "comments": "Budget available, approved."
}
```

**Expected Result:**
- Step 2 completed
- Expense moves to Step 3 (Director Approval)
- Director now sees it in pending approvals

### Step 5: Director Approves (Step 3)
```http
POST http://localhost:3000/api/approvals/{approval-step3-id}/decision
Authorization: Bearer <director-token>
Content-Type: application/json

{
  "decision": "approved",
  "comments": "Final approval granted."
}
```

**Expected Result:**
- Step 3 completed
- No more steps
- Expense status: "approved"
- Employee notified

---

## Test Scenario 3: Rejection at Any Step

### Manager Rejects at Step 1
```http
POST http://localhost:3000/api/approvals/{approvalId}/decision
Authorization: Bearer <manager-token>
Content-Type: application/json

{
  "decision": "rejected",
  "comments": "Expense not justified, please provide more details."
}
```

**Expected Result:**
- Expense status: "rejected"
- Workflow stops (no further steps)
- Employee notified with rejection reason

---

## Test Scenario 4: Multiple Approvers (Threshold-Based)

### Step 1: Create Flow with Multiple Approvers
```http
POST http://localhost:3000/api/companies/{companyId}/approval-flows
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "name": "Finance Team Approval",
  "description": "Requires 2 out of 3 finance team members",
  "minAmount": 0,
  "steps": [
    {
      "stepNumber": 1,
      "name": "Finance Team Review",
      "stepType": "multiple",
      "assignedUserIds": [
        "<finance-user-1>",
        "<finance-user-2>",
        "<finance-user-3>"
      ],
      "approvalThreshold": 0.67,
      "requiredApprovers": 2
    }
  ]
}
```

### Step 2: First Finance User Approves
```http
POST http://localhost:3000/api/approvals/{approval-finance1-id}/decision
Authorization: Bearer <finance1-token>
Content-Type: application/json

{
  "decision": "approved",
  "comments": "Looks good to me."
}
```

**Expected Result:**
- 1 out of 3 approved (33%)
- Expense still pending (needs 67%)

### Step 3: Second Finance User Approves
```http
POST http://localhost:3000/api/approvals/{approval-finance2-id}/decision
Authorization: Bearer <finance2-token>
Content-Type: application/json

{
  "decision": "approved",
  "comments": "Approved."
}
```

**Expected Result:**
- 2 out of 3 approved (67%)
- Threshold met!
- Expense approved (or moves to next step)

---

## Verification Queries

### Check Expense Status
```http
GET http://localhost:3000/api/companies/{companyId}/expenses/{expenseId}
Authorization: Bearer <token>
```

### Check All Approvals for an Expense
```http
GET http://localhost:3000/api/companies/{companyId}/expenses/{expenseId}/approvals
Authorization: Bearer <token>
```

### View Audit Log
```sql
SELECT * FROM audit_logs 
WHERE entity_type = 'expense' 
AND entity_id = '<expense-id>'
ORDER BY created_at DESC;
```

---

## Common Issues & Solutions

### Issue 1: "No applicable approval flow found"
**Solution:** Create a default approval flow with `isDefault: true` and `minAmount: 0`, `maxAmount: null`

### Issue 2: "Manager not receiving approvals"
**Check:**
- Employee has `managerId` set
- Manager has `canApproveExpenses: true`
- Manager is active (`isActive: true`)

### Issue 3: "Expense stuck in pending"
**Check:**
- View approvals: `GET /api/companies/{companyId}/expenses/{expenseId}/approvals`
- Verify current step: Check `currentStepId` on expense
- Check if approver exists and is active

### Issue 4: "Approval already processed"
**Cause:** Trying to approve/reject an approval that's already been processed
**Solution:** Check approval status before attempting decision

---

## Testing Checklist

- [ ] Database constraint fixed
- [ ] Default approval flow created
- [ ] Test users created (admin, manager, employee)
- [ ] Employee has managerId set to manager
- [ ] Manager has canApproveExpenses = true
- [ ] Submit expense as employee
- [ ] Verify approval created for manager
- [ ] Manager can see pending approval
- [ ] Manager can approve with comments
- [ ] Expense status changes to approved
- [ ] Test rejection flow
- [ ] Test multi-step workflow
- [ ] Test multiple approvers with threshold
- [ ] Verify audit logs are created

---

## API Testing with Postman/Thunder Client

### Environment Variables
```
BASE_URL=http://localhost:3000
ADMIN_TOKEN=<your-admin-jwt>
MANAGER_TOKEN=<your-manager-jwt>
EMPLOYEE_TOKEN=<your-employee-jwt>
COMPANY_ID=<your-company-uuid>
```

### Collection Structure
```
Expense Management
├── Auth
│   ├── Login as Admin
│   ├── Login as Manager
│   └── Login as Employee
├── Approval Flows (Admin)
│   ├── Create Approval Flow
│   ├── Get All Flows
│   └── Update Flow
├── Expenses (Employee)
│   ├── Create Expense
│   ├── Get My Expenses
│   └── Get Expense Details
└── Approvals (Manager)
    ├── Get Pending Approvals
    ├── Approve Expense
    ├── Reject Expense
    └── Get All Approvals
```

---

## Success Criteria

✅ **Workflow is working correctly if:**
1. Employee can submit expenses
2. Manager sees expense in pending approvals
3. Manager can approve/reject with comments
4. Expense moves to next step after approval
5. Expense is fully approved after all steps
6. Rejection stops the workflow immediately
7. Audit logs are created for all actions

---

**Ready to test!** Start with Scenario 1 (Simple Manager Approval) and work your way up to more complex scenarios.
