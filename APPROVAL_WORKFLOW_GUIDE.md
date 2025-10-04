# Approval Workflow Implementation Guide

## Overview
The expense management system implements a **sequential multi-step approval workflow** where expenses move through defined approval steps in order. Each step must be completed before moving to the next.

---

## ✅ Workflow Features Implemented

### 1. **Sequential Approval Steps**
- Expenses follow a defined sequence of approval steps
- **Step 1 → Manager** (if `isManagerStep` is checked)
- **Step 2 → Finance Team**
- **Step 3 → Director/Admin**
- Each step must be approved before moving to the next

### 2. **Manager Approval (First Step)**
- If `isManagerStep` is enabled, the expense is automatically routed to the submitter's manager
- Manager is identified from the `managerId` field in the User model
- Only users with `canApproveExpenses = true` can approve

### 3. **Multiple Approvers per Step**
- Admin can assign multiple approvers to a single step
- Supports different approval modes:
  - **Single Approver**: Any one approver can approve
  - **Multiple Approvers**: Requires threshold (e.g., 60% approval)
  - **Specific Approvers**: Requires specific person to approve

### 4. **Approval Actions**
Managers/Approvers can:
- ✅ **View pending approvals** - See all expenses waiting for their approval
- ✅ **Approve with comments** - Approve and add optional comments
- ✅ **Reject with comments** - Reject and provide reason (required)
- ✅ **Escalate** - Forward to higher authority if needed

---

## 🔄 Workflow Process Flow

```
┌─────────────────┐
│ Employee        │
│ Submits Expense │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│ Check Auto-Approve      │
│ (Amount < $50)          │
└────────┬────────────────┘
         │
    ┌────┴────┐
    │         │
   YES       NO
    │         │
    │         ▼
    │    ┌────────────────────┐
    │    │ Initialize Workflow│
    │    │ Find Approval Flow │
    │    └────────┬───────────┘
    │             │
    │             ▼
    │    ┌────────────────────┐
    │    │ STEP 1: Manager    │
    │    │ (if isManagerStep) │
    │    └────────┬───────────┘
    │             │
    │        ┌────┴────┐
    │        │         │
    │    APPROVED  REJECTED
    │        │         │
    │        ▼         ▼
    │    ┌────────────────────┐
    │    │ STEP 2: Finance    │
    │    │ (Multiple Approvers│
    │    └────────┬───────────┘
    │             │
    │        ┌────┴────┐
    │        │         │
    │    APPROVED  REJECTED
    │        │         │
    │        ▼         ▼
    │    ┌────────────────────┐
    │    │ STEP 3: Director   │
    │    │ (Final Approval)   │
    │    └────────┬───────────┘
    │             │
    │        ┌────┴────┐
    │        │         │
    ▼        ▼         ▼
┌──────────────┐  ┌──────────────┐
│   APPROVED   │  │   REJECTED   │
└──────────────┘  └──────────────┘
```

---

## 📊 Database Schema

### **Approval Flow Table**
```sql
approval_flows:
- id (UUID)
- company_id (UUID)
- name (VARCHAR) - e.g., "Standard Expense Approval"
- description (TEXT)
- is_default (BOOLEAN)
- min_amount (DECIMAL) - Minimum amount for this flow
- max_amount (DECIMAL) - Maximum amount for this flow
- category (VARCHAR) - Specific category or NULL for all
- is_active (BOOLEAN)
```

### **Approval Steps Table**
```sql
approval_steps:
- id (UUID)
- approval_flow_id (UUID)
- step_number (INTEGER) - 1, 2, 3... (defines sequence)
- name (VARCHAR) - "Manager Approval", "Finance Review"
- step_type (ENUM) - 'single', 'multiple', 'conditional'
- is_manager_step (BOOLEAN) - Auto-route to manager
- assigned_role_ids (UUID[]) - Array of role IDs
- assigned_user_ids (UUID[]) - Array of specific user IDs
- approval_threshold (DECIMAL) - 0.60 = 60% approval needed
- required_approvers (INTEGER) - Minimum approvers needed
- specific_approvers (UUID[]) - Must have these specific approvers
- veto_allowed (BOOLEAN) - One rejection rejects all
- escalation_minutes (INTEGER) - Auto-escalate after X minutes
- is_active (BOOLEAN)
```

### **Approvals Table**
```sql
approvals:
- id (UUID)
- expense_id (UUID)
- approval_step_id (UUID)
- approver_id (UUID)
- status (ENUM) - 'pending', 'approved', 'rejected', 'escalated'
- decision (ENUM) - 'approved', 'rejected'
- comments (TEXT) - Approver's comments
- reviewed_at (TIMESTAMP)
- escalated_at (TIMESTAMP)
- escalated_to (UUID)
- escalation_reason (TEXT)
```

---

## 🎯 API Endpoints

### **1. Get Pending Approvals (Manager/Approver)**
```http
GET /api/approvals/pending
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "count": 5,
  "data": [
    {
      "id": "approval-uuid",
      "expense": {
        "id": "expense-uuid",
        "amount": 250.00,
        "currency": "USD",
        "category": "Travel",
        "description": "Client meeting travel",
        "submitter": {
          "name": "John Doe",
          "email": "john@company.com"
        }
      },
      "approvalStep": {
        "stepNumber": 1,
        "name": "Manager Approval"
      },
      "status": "pending",
      "createdAt": "2025-10-04T10:00:00Z"
    }
  ]
}
```

### **2. Approve/Reject Expense**
```http
POST /api/approvals/:approvalId/decision
Authorization: Bearer <token>
Content-Type: application/json

{
  "decision": "approved",  // or "rejected"
  "comments": "Approved for business travel expenses"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Expense approved successfully",
  "data": {
    "approvalId": "approval-uuid",
    "expenseId": "expense-uuid",
    "decision": "approved",
    "comments": "Approved for business travel expenses",
    "processedBy": "Manager Name",
    "processedAt": "2025-10-04T10:30:00Z"
  }
}
```

### **3. Get All Approvals (Admin/Manager View)**
```http
GET /api/approvals?status=pending&dateFrom=2025-10-01
Authorization: Bearer <token>
```

### **4. Create Approval Flow (Admin Only)**
```http
POST /api/companies/:companyId/approval-flows
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "High-Value Expense Approval",
  "description": "For expenses above $1000",
  "minAmount": 1000.00,
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
      "stepType": "multiple",
      "assignedUserIds": ["finance-user-1", "finance-user-2"],
      "approvalThreshold": 0.5,
      "escalationMinutes": 2880
    },
    {
      "stepNumber": 3,
      "name": "Director Approval",
      "stepType": "single",
      "assignedUserIds": ["director-uuid"],
      "escalationMinutes": 1440
    }
  ]
}
```

---

## 💻 Code Implementation

### **Service: approvalWorkflowService.js**

Key methods:
- `initializeWorkflow(expenseId)` - Start workflow when expense is submitted
- `moveToNextStep(expense, approvalFlow, transaction)` - Move to next step after approval
- `processApproval(approvalId, decision, comments, approverId)` - Handle approve/reject
- `checkStepCompletion(expense, step, transaction)` - Check if step is complete
- `getApproversForStep(expense, step)` - Get list of approvers for a step

### **Controller: approvalController.js**

Key endpoints:
- `getPendingApprovals()` - Get approvals waiting for current user
- `getAllApprovals()` - Get all approvals (admin/manager view)
- `processApprovalDecision()` - Approve or reject with comments
- `escalateApproval()` - Escalate to higher authority

---

## 🔧 Configuration Examples

### **Example 1: Simple Manager → Finance Flow**
```javascript
{
  name: "Standard Approval",
  steps: [
    {
      stepNumber: 1,
      name: "Manager Approval",
      isManagerStep: true,  // ✅ Routes to submitter's manager
      stepType: "single"
    },
    {
      stepNumber: 2,
      name: "Finance Approval",
      stepType: "single",
      assignedUserIds: ["finance-manager-uuid"]
    }
  ]
}
```

### **Example 2: Multi-Approver Finance Step**
```javascript
{
  stepNumber: 2,
  name: "Finance Team Review",
  stepType: "multiple",
  assignedUserIds: [
    "finance-user-1",
    "finance-user-2",
    "finance-user-3"
  ],
  approvalThreshold: 0.6,  // 60% must approve (2 out of 3)
  requiredApprovers: 2
}
```

### **Example 3: Veto-Enabled Director Step**
```javascript
{
  stepNumber: 3,
  name: "Director Final Approval",
  stepType: "single",
  assignedUserIds: ["director-uuid"],
  vetoAllowed: true  // ✅ Director rejection overrides all
}
```

---

## 📝 Usage Examples

### **Manager Viewing Pending Approvals**
```javascript
// Frontend code
const response = await fetch('/api/approvals/pending', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

const { data: pendingApprovals } = await response.json();

// Display in UI
pendingApprovals.forEach(approval => {
  console.log(`
    Expense: ${approval.expense.description}
    Amount: ${approval.expense.amount} ${approval.expense.currency}
    Submitter: ${approval.expense.submitter.name}
    Step: ${approval.approvalStep.name}
  `);
});
```

### **Approving an Expense**
```javascript
const approveExpense = async (approvalId, comments) => {
  const response = await fetch(`/api/approvals/${approvalId}/decision`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      decision: 'approved',
      comments: comments
    })
  });

  return await response.json();
};
```

### **Rejecting an Expense**
```javascript
const rejectExpense = async (approvalId, reason) => {
  const response = await fetch(`/api/approvals/${approvalId}/decision`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      decision: 'rejected',
      comments: reason  // Required for rejection
    })
  });

  return await response.json();
};
```

---

## ✅ Implementation Status

| Feature | Status | Notes |
|---------|--------|-------|
| Sequential approval steps | ✅ Implemented | Steps execute in order |
| Manager as first approver | ✅ Implemented | `isManagerStep` flag |
| Multiple approvers per step | ✅ Implemented | Configurable threshold |
| Approve with comments | ✅ Implemented | Optional comments |
| Reject with comments | ✅ Implemented | Comments field |
| View pending approvals | ✅ Implemented | Filter by status |
| Auto-escalation | ✅ Implemented | Time-based escalation |
| Audit logging | ✅ Implemented | All actions logged |
| Email notifications | ⚠️ Placeholder | Service methods ready |

---

## 🚀 Next Steps

1. **Fix Database Constraint** (Required)
   - Run: `ALTER TABLE audit_logs DROP CONSTRAINT IF EXISTS audit_logs_entity_id_fkey;`

2. **Test Workflow**
   - Create approval flow via admin panel
   - Submit expense as employee
   - Approve as manager
   - Verify it moves to next step

3. **Add Email Notifications**
   - Implement email service
   - Notify approvers when expense arrives
   - Notify submitter on approval/rejection

4. **Frontend Integration**
   - Build approval dashboard for managers
   - Show pending approvals count
   - Add approve/reject buttons with comment modal

---

## 📞 Support

For questions about the approval workflow implementation, refer to:
- `backend/services/approvalWorkflowService.js` - Core workflow logic
- `backend/controllers/approvalController.js` - API endpoints
- `backend/models/ApprovalStep.js` - Step configuration
- `backend/models/Approval.js` - Approval records

**Status:** ✅ Fully implemented and ready for testing after database fix
