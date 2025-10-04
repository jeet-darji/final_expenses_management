# 🚀 Quick Start - Approval Workflow

## ⚡ 3-Minute Setup

### Step 1: Fix Database (REQUIRED)
```sql
-- Open pgAdmin and run this:
ALTER TABLE audit_logs DROP CONSTRAINT IF EXISTS audit_logs_entity_id_fkey;
```

### Step 2: Restart Backend
```bash
cd backend
npm start
```

### Step 3: Create Approval Flow (Admin)
```bash
POST http://localhost:3000/api/companies/{companyId}/approval-flows
Authorization: Bearer <admin-token>

{
  "name": "Standard Approval",
  "isDefault": true,
  "minAmount": 0,
  "steps": [
    {
      "stepNumber": 1,
      "name": "Manager Approval",
      "isManagerStep": true,
      "stepType": "single"
    }
  ]
}
```

### Step 4: Test It!

**Employee submits:**
```bash
POST /api/companies/{companyId}/expenses
{
  "amount": 100,
  "currency": "USD",
  "category": "Travel",
  "description": "Client meeting",
  "dateOfExpense": "2025-10-04"
}
```

**Manager views pending:**
```bash
GET /api/approvals/pending
```

**Manager approves:**
```bash
POST /api/approvals/{approvalId}/decision
{
  "decision": "approved",
  "comments": "Approved!"
}
```

**Done!** ✅ Expense is now approved.

---

## 📋 Key Endpoints

| Action | Method | Endpoint |
|--------|--------|----------|
| Get pending approvals | GET | `/api/approvals/pending` |
| Approve/Reject | POST | `/api/approvals/:id/decision` |
| Create flow (admin) | POST | `/api/companies/:id/approval-flows` |
| View expense approvals | GET | `/api/companies/:id/expenses/:id/approvals` |

---

## 🎯 Workflow Flow

```
Employee Submits
      ↓
   Manager
      ↓
   Finance
      ↓
   Director
      ↓
   APPROVED
```

Each step must approve before moving to next!

---

## 📚 Full Documentation

- `APPROVAL_WORKFLOW_GUIDE.md` - Complete guide
- `WORKFLOW_TESTING_GUIDE.md` - Testing scenarios
- `IMPLEMENTATION_SUMMARY.md` - What's implemented
- `DATABASE_FIXES_REQUIRED.md` - Database fix details

---

**Status:** ✅ Ready to use after database fix!
