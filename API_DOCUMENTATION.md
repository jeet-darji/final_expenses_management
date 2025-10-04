# Expense Management System - API Documentation

## Base URL
```
http://localhost:5000/api
```

## Authentication
Most endpoints require JWT authentication. Include the token in the Authorization header:
```
Authorization: Bearer YOUR_JWT_TOKEN
```

---

## 📁 Authentication Endpoints

### POST /auth/register
Register a new company with admin user.

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@company.com",
  "password": "securepass123",
  "companyName": "Acme Corp",
  "country": "US",
  "timezone": "America/New_York"
}
```

**Response:** `201 Created`
```json
{
  "success": true,
  "message": "Company and admin user created successfully",
  "data": {
    "user": {
      "id": "uuid",
      "name": "John Doe",
      "email": "john@company.com",
      "role": "admin",
      "department": "Administration"
    },
    "company": {
      "id": "uuid",
      "name": "Acme Corp",
      "country": "US",
      "baseCurrency": "USD",
      "currencySymbol": "$",
      "timezone": "America/New_York"
    },
    "token": "jwt_token_here"
  }
}
```

### POST /auth/login
Authenticate user and get JWT token.

**Request Body:**
```json
{
  "email": "john@company.com",
  "password": "securepass123"
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "name": "John Doe",
      "email": "john@company.com",
      "role": "admin",
      "department": "Administration",
      "canApproveExpenses": true,
      "approveLimit": null,
      "company": { /* company details */ }
    },
    "token": "jwt_token_here"
  }
}
```

### GET /auth/me
Get current user profile.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "user": { /* full user details */ },
    "recentActivity": [ /* last 10 actions */ ]
  }
}
```

### PUT /auth/updatedetails
Update user profile.

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "name": "John Updated",
  "department": "Finance",
  "employeeId": "EMP001"
}
```

**Response:** `200 OK`

### PUT /auth/updatepassword
Change password.

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "currentPassword": "oldpass123",
  "newPassword": "newpass456"
}
```

**Response:** `200 OK`

---

## 👥 User Management Endpoints

### POST /companies/:companyId/users
Create a new user (Admin only).

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "name": "Jane Smith",
  "email": "jane@company.com",
  "password": "password123",
  "role": "employee",
  "department": "Sales",
  "employeeId": "EMP002",
  "employeeLevel": 2,
  "managerId": "manager_uuid",
  "canApproveExpenses": false,
  "approveLimit": null
}
```

**Response:** `201 Created`

### PATCH /companies/:companyId/users/:userId
Update user details (Admin only).

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "role": "manager",
  "canApproveExpenses": true,
  "approveLimit": 5000.00
}
```

**Response:** `200 OK`

### GET /companies/:companyId/users
Get all company users.

**Headers:** `Authorization: Bearer TOKEN`

**Query Parameters:**
- `role` - Filter by role (admin/manager/employee)
- `department` - Filter by department
- `isActive` - Filter by active status (true/false)
- `limit` - Results per page (default: 100)
- `offset` - Pagination offset

**Response:** `200 OK`
```json
{
  "success": true,
  "count": 25,
  "total": 25,
  "data": [ /* array of users */ ]
}
```

### PATCH /companies/:companyId/users/:userId/deactivate
Deactivate a user account (Admin only).

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`

---

## 💰 Expense Endpoints

### GET /companies/:companyId/expenses
Get all expenses (filtered by role).

**Headers:** `Authorization: Bearer TOKEN`

**Query Parameters:**
- `status` - Filter by status (draft/submitted/pending/approved/rejected/paid)
- `category` - Filter by category
- `userId` - Filter by user (admin only)
- `dateFrom` - Start date (YYYY-MM-DD)
- `dateTo` - End date (YYYY-MM-DD)
- `limit` - Results per page (default: 50)
- `offset` - Pagination offset

**Response:** `200 OK`
```json
{
  "success": true,
  "count": 10,
  "total": 45,
  "data": [
    {
      "id": "uuid",
      "amount": 125.50,
      "currency": "USD",
      "amountInBaseCurrency": 125.50,
      "category": "Meals",
      "description": "Client lunch",
      "dateOfExpense": "2025-01-15",
      "status": "approved",
      "submitter": { /* user details */ },
      "receipts": [ /* receipt files */ ],
      "approvals": [ /* approval history */ ]
    }
  ]
}
```

### POST /companies/:companyId/expenses
Create a new expense.

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "amount": 125.50,
  "currency": "USD",
  "category": "Meals",
  "description": "Client lunch meeting",
  "dateOfExpense": "2025-01-15",
  "merchant": "The Steakhouse",
  "location": "New York, NY",
  "subCategory": "Fine Dining",
  "tags": ["client", "important"],
  "isUrgent": false
}
```

**Response:** `201 Created`

### GET /companies/:companyId/expenses/:expenseId
Get single expense details.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`

### PUT /companies/:companyId/expenses/:expenseId
Update expense (only if status is draft).

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:** (same as create, partial updates allowed)

**Response:** `200 OK`

### DELETE /companies/:companyId/expenses/:expenseId
Delete expense (only if status is draft).

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`

### POST /companies/:companyId/expenses/:expenseId/submit
Submit expense for approval.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`
```json
{
  "success": true,
  "message": "Expense submitted for approval",
  "data": {
    "expenseId": "uuid",
    "status": "pending",
    "approvalFlow": { /* flow details */ },
    "currentStep": { /* step details */ },
    "approvers": [ /* list of approvers */ ]
  }
}
```

### POST /companies/:companyId/expenses/:expenseId/cancel
Cancel submitted expense.

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "reason": "Submitted by mistake"
}
```

**Response:** `200 OK`

### POST /companies/:companyId/expenses/:expenseId/receipts
Upload receipt with OCR processing.

**Headers:** 
- `Authorization: Bearer TOKEN`
- `Content-Type: multipart/form-data`

**Form Data:**
- `receipt` - File (image/pdf)

**Response:** `201 Created`
```json
{
  "success": true,
  "data": {
    "receiptId": "uuid",
    "fileName": "receipt_123.jpg",
    "ocrData": {
      "amount": 125.50,
      "currency": "USD",
      "date": "2025-01-15",
      "merchant": "The Steakhouse",
      "category": "Meals",
      "confidence": 0.95
    },
    "extractedFields": {
      "amount": 125.50,
      "date": "2025-01-15",
      "merchant": "The Steakhouse"
    }
  }
}
```

---

## ✅ Approval Endpoints

### GET /approvals/pending
Get pending approvals for current user.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`
```json
{
  "success": true,
  "count": 5,
  "data": [
    {
      "id": "uuid",
      "status": "pending",
      "expense": {
        "id": "uuid",
        "amount": 125.50,
        "currency": "USD",
        "category": "Meals",
        "submitter": { /* user details */ }
      },
      "approvalStep": {
        "stepNumber": 1,
        "name": "Manager Approval"
      },
      "createdAt": "2025-01-15T10:00:00Z"
    }
  ]
}
```

### GET /approvals
Get all approvals (Admin/Manager).

**Headers:** `Authorization: Bearer TOKEN`

**Query Parameters:**
- `status` - Filter by status
- `dateFrom` - Start date
- `dateTo` - End date
- `userId` - Filter by approver

**Response:** `200 OK`

### GET /approvals/:approvalId
Get single approval details.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`

### POST /approvals/:approvalId/decision
Make approval decision.

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "decision": "approved",
  "comments": "Approved - valid business expense"
}
```

**Decisions:** `approved` | `rejected`

**Response:** `200 OK`
```json
{
  "success": true,
  "message": "Expense approved successfully",
  "data": {
    "approvalId": "uuid",
    "expenseId": "uuid",
    "decision": "approved",
    "comments": "Approved - valid business expense",
    "processedBy": "John Doe",
    "processedAt": "2025-01-15T14:30:00Z",
    "nextStep": { /* next step details or null if complete */ }
  }
}
```

### POST /approvals/:approvalId/escalate
Escalate approval to higher authority.

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "escalatedTo": "user_uuid",
  "reason": "Requires senior management approval"
}
```

**Response:** `200 OK`

### POST /approvals/bulk-approve
Bulk approve multiple expenses (Admin only).

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "approvalIds": ["uuid1", "uuid2", "uuid3"],
  "decision": "approved",
  "comments": "Batch approval - Q1 expenses"
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "message": "Bulk approval completed",
  "data": {
    "successful": 2,
    "failed": 1,
    "results": [
      { "approvalId": "uuid1", "success": true },
      { "approvalId": "uuid2", "success": true },
      { "approvalId": "uuid3", "success": false, "error": "Already processed" }
    ]
  }
}
```

---

## 🔄 Approval Flow Endpoints

### POST /companies/:companyId/approval-flows
Create approval flow (Admin only).

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "name": "Standard Expense Approval",
  "description": "Default flow for all expenses",
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
      "escalationMinutes": 4320,
      "approvalThreshold": 1.0
    },
    {
      "stepNumber": 2,
      "name": "Finance Review",
      "stepType": "multiple",
      "assignedUserIds": ["user1", "user2", "user3"],
      "approvalThreshold": 0.6,
      "specificApprovers": [],
      "vetoAllowed": false,
      "escalationMinutes": 2880
    }
  ]
}
```

**Step Types:**
- `single` - One approver
- `multiple` - Multiple approvers with threshold

**Response:** `201 Created`

### GET /companies/:companyId/approval-flows
Get all approval flows.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`

### PUT /companies/:companyId/approval-flows/:flowId
Update approval flow.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`

### DELETE /companies/:companyId/approval-flows/:flowId
Delete approval flow.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`

---

## 🔧 Admin Endpoints

### POST /companies/:companyId/override
Admin override expense decision.

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "expenseId": "uuid",
  "action": "approved",
  "reason": "Emergency approval - CEO authorized",
  "notifySubmitter": true,
  "notifyApprovers": true
}
```

**Actions:** `approved` | `rejected`

**Response:** `200 OK`
```json
{
  "success": true,
  "message": "Expense approved by admin override",
  "data": {
    "expenseId": "uuid",
    "action": "approved",
    "reason": "Emergency approval - CEO authorized",
    "finalApprover": "Admin Name",
    "overriddenAt": "2025-01-15T16:00:00Z"
  }
}
```

### GET /companies/:companyId/admin/stats
Get admin dashboard statistics.

**Headers:** `Authorization: Bearer TOKEN`

**Query Parameters:**
- `dateFrom` - Start date
- `dateTo` - End date

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "users": {
      "total": 50,
      "active": 48,
      "employees": 40,
      "managers": 8
    },
    "expenses": {
      "total": 250,
      "pending": 15,
      "approved": 200,
      "rejected": 35
    },
    "financial": {
      "totalAmount": 125000.00,
      "approvedAmount": 100000.00,
      "pendingAmount": 25000.00
    },
    "period": {
      "dateFrom": "2025-01-01",
      "dateTo": "2025-01-31"
    }
  }
}
```

---

## 🏢 Company Endpoints

### GET /companies/:companyId
Get company details.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`

### PATCH /companies/:companyId
Update company settings (Admin only).

**Headers:** `Authorization: Bearer TOKEN`

**Request Body:**
```json
{
  "settings": {
    "autoApproveLimit": 50.00,
    "requireApprovalCategories": ["Travel", "Client Entertainment"],
    "allowExpenseSubmission": true,
    "timezone": "America/New_York"
  }
}
```

**Response:** `200 OK`

---

## 📊 Dashboard Endpoints

### GET /dashboard/stats
Get user dashboard statistics.

**Headers:** `Authorization: Bearer TOKEN`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "myExpenses": {
      "total": 25,
      "pending": 3,
      "approved": 20,
      "rejected": 2,
      "totalAmount": 5000.00
    },
    "pendingApprovals": 5,
    "teamExpenses": {
      "total": 100,
      "pending": 10
    }
  }
}
```

---

## 📤 Upload Endpoints

### POST /uploads/receipt
Upload receipt file.

**Headers:** 
- `Authorization: Bearer TOKEN`
- `Content-Type: multipart/form-data`

**Form Data:**
- `file` - Receipt file

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "fileName": "receipt_123.jpg",
    "filePath": "/uploads/receipts/receipt_123.jpg",
    "fileSize": 245678,
    "mimeType": "image/jpeg"
  }
}
```

---

## 🚨 Error Responses

### 400 Bad Request
```json
{
  "success": false,
  "message": "Validation error",
  "errors": [
    {
      "field": "amount",
      "message": "Amount must be a positive number"
    }
  ]
}
```

### 401 Unauthorized
```json
{
  "success": false,
  "message": "Not authorized to access this route"
}
```

### 403 Forbidden
```json
{
  "success": false,
  "message": "Only admins can perform this action"
}
```

### 404 Not Found
```json
{
  "success": false,
  "message": "Resource not found"
}
```

### 500 Internal Server Error
```json
{
  "success": false,
  "message": "Server error",
  "error": "Error details (only in development)"
}
```

---

## 📝 Notes

### Expense Categories (Enum)
- Travel
- Meals
- Accommodation
- Office Supplies
- Transportation
- Utilities
- Marketing
- Training
- Client Entertainment
- Other

### Expense Status (Enum)
- draft
- submitted
- pending
- approved
- rejected
- suspended
- paid
- cancelled

### User Roles (Enum)
- admin
- manager
- employee

### Approval Status (Enum)
- pending
- approved
- rejected
- escalated

### Currency Codes
System supports all major currencies via exchange rate API. Base currency is set per company based on country.

### Date Formats
All dates in ISO 8601 format: `YYYY-MM-DDTHH:mm:ss.sssZ`

### Pagination
Default limit: 50, Max limit: 100

### Rate Limiting
Not currently implemented (add in production)

---

**For more details, see `QUICK_START_GUIDE.md` for usage examples.**
