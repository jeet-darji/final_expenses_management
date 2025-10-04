# Expense Management System - Quick Start Guide

## 🚀 Getting Started

### Prerequisites
- Node.js (v16 or higher)
- PostgreSQL (v12 or higher)
- npm or yarn

### 1. Backend Setup

```bash
cd backend

# Install dependencies
npm install

# Create .env file
cp .env.example .env

# Edit .env with your configuration:
# - DATABASE_URL (PostgreSQL connection string)
# - JWT_SECRET (random secure string)
# - FRONTEND_URL (default: http://localhost:3001)
# - Optional: GOOGLE_VISION_API_KEY for production OCR

# Start the server
npm run dev
```

The backend will:
- ✅ Auto-sync database schema
- ✅ Create default admin user (email: admin@example.com, password: admin123)
- ✅ Start escalation service (runs hourly)
- ✅ Listen on port 5000 (or PORT from .env)

### 2. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start the development server
npm start
```

The frontend will open at `http://localhost:3001`

## 📋 First Steps

### 1. Register Your Company

**Option A: Via Frontend**
1. Go to `http://localhost:3001/register`
2. Fill in:
   - Company Name
   - Country (will auto-detect currency)
   - Admin Name & Email
   - Password
3. Click "Register"

**Option B: Via API**
```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@company.com",
    "password": "securepass123",
    "companyName": "Acme Corp",
    "country": "US",
    "timezone": "America/New_York"
  }'
```

Response includes:
- User details
- Company details with base currency (auto-detected from country)
- JWT token

### 2. Login

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@company.com",
    "password": "securepass123"
  }'
```

Save the `token` from response for authenticated requests.

### 3. Create Users (Admin Only)

```bash
curl -X POST http://localhost:5000/api/companies/{companyId}/users \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Jane Smith",
    "email": "jane@company.com",
    "password": "password123",
    "role": "employee",
    "department": "Sales",
    "employeeId": "EMP001",
    "employeeLevel": 2,
    "managerId": "MANAGER_USER_ID",
    "canApproveExpenses": false
  }'
```

**User Roles:**
- `admin` - Full system access, can override decisions
- `manager` - Can approve expenses, manage team
- `employee` - Can submit expenses

### 4. Create Approval Flow (Admin Only)

```bash
curl -X POST http://localhost:5000/api/companies/{companyId}/approval-flows \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
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
        "escalationMinutes": 4320
      },
      {
        "stepNumber": 2,
        "name": "Finance Review",
        "stepType": "multiple",
        "assignedUserIds": ["FINANCE_USER_1", "FINANCE_USER_2", "FINANCE_USER_3"],
        "approvalThreshold": 0.6,
        "escalationMinutes": 2880
      }
    ]
  }'
```

**Approval Step Types:**
- `single` - One approver (usually manager)
- `multiple` - Multiple approvers with threshold rules

**Approval Rules:**
- `approvalThreshold` - Percentage needed (0.6 = 60%)
- `specificApprovers` - If any of these approve, step auto-approves
- `vetoAllowed` - If true, one rejection fails the step
- `isManagerStep` - Auto-assigns to submitter's manager

### 5. Submit an Expense

```bash
curl -X POST http://localhost:5000/api/companies/{companyId}/expenses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 125.50,
    "currency": "USD",
    "category": "Meals",
    "description": "Client lunch meeting",
    "dateOfExpense": "2025-01-15",
    "merchant": "The Steakhouse",
    "location": "New York, NY"
  }'
```

The system will:
1. Convert amount to company base currency
2. Find applicable approval flow
3. Create approval requests for Step 1
4. Send notifications to approvers

### 6. Upload Receipt with OCR

```bash
curl -X POST http://localhost:5000/api/companies/{companyId}/expenses/{expenseId}/receipts \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "receipt=@/path/to/receipt.jpg"
```

The system will:
1. Store the receipt file
2. Process with OCR (mock in dev, Google Vision in production)
3. Extract: amount, date, merchant, category
4. Return extracted data for user confirmation

### 7. Approve/Reject Expense

**Get Pending Approvals:**
```bash
curl http://localhost:5000/api/approvals/pending \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Make Decision:**
```bash
curl -X POST http://localhost:5000/api/approvals/{approvalId}/decision \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "decision": "approved",
    "comments": "Approved - valid business expense"
  }'
```

The system will:
1. Record the decision
2. Check if step is complete (based on rules)
3. Move to next step OR approve expense
4. Send notifications

### 8. Admin Override (Admin Only)

```bash
curl -X POST http://localhost:5000/api/companies/{companyId}/override \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "expenseId": "EXPENSE_ID",
    "action": "approved",
    "reason": "Emergency approval - CEO authorized",
    "notifySubmitter": true,
    "notifyApprovers": true
  }'
```

## 🔄 Complete Expense Lifecycle

```
1. DRAFT
   ↓ (employee creates expense)
   
2. SUBMITTED
   ↓ (triggers workflow)
   
3. PENDING (Step 1: Manager Approval)
   ↓ (manager approves)
   
4. PENDING (Step 2: Finance Review)
   ↓ (60% of finance team approves)
   
5. APPROVED
   ↓ (admin marks as paid)
   
6. PAID
```

**Alternative Paths:**
- **REJECTED** - Any step rejects (if veto allowed) or threshold not met
- **CANCELLED** - Employee cancels before approval
- **SUSPENDED** - Admin temporarily suspends
- **ESCALATED** - Auto-escalated after timeout or manually escalated

## 🎯 Key Features Demonstrated

### Multi-Currency Support
```javascript
// Expense in EUR, company base is USD
{
  "amount": 100,
  "currency": "EUR",
  // System auto-converts:
  "amountInBaseCurrency": 108.50,  // Based on current rate
  "exchangeRate": 1.085,
  "rateDate": "2025-01-15T10:30:00Z"
}
```

### Approval Rules Examples

**Example 1: Manager + Finance (60% threshold)**
```javascript
{
  "steps": [
    {
      "stepNumber": 1,
      "isManagerStep": true,  // Auto-assigns to submitter's manager
      "stepType": "single"
    },
    {
      "stepNumber": 2,
      "assignedUserIds": ["user1", "user2", "user3", "user4", "user5"],
      "approvalThreshold": 0.6,  // Need 3 out of 5 approvals
      "stepType": "multiple"
    }
  ]
}
```

**Example 2: Specific Approver (CFO) OR 100% Team**
```javascript
{
  "stepNumber": 1,
  "assignedUserIds": ["finance1", "finance2", "finance3"],
  "specificApprovers": ["CFO_USER_ID"],  // If CFO approves, step auto-completes
  "approvalThreshold": 1.0,  // OR all 3 finance users approve
  "stepType": "multiple"
}
```

**Example 3: Veto Power**
```javascript
{
  "stepNumber": 1,
  "assignedUserIds": ["approver1", "approver2"],
  "vetoAllowed": true,  // Any rejection fails the step
  "stepType": "multiple"
}
```

### Auto-Escalation
```javascript
{
  "stepNumber": 1,
  "escalationMinutes": 4320,  // 3 days (72 hours)
  // If no response in 3 days:
  // - Send reminder at 24 hours before escalation
  // - Auto-escalate to approver's manager
  // - Create new approval for escalated approver
}
```

### Auto-Approval for Small Amounts
```javascript
// In company settings or approval flow:
{
  "autoApproveLimit": 50.00  // Expenses ≤ $50 auto-approve
}
```

## 📊 Admin Dashboard

**Get Statistics:**
```bash
curl http://localhost:5000/api/companies/{companyId}/admin/stats \
  -H "Authorization: Bearer YOUR_TOKEN"
```

Returns:
- User counts (total, active, by role)
- Expense counts (total, pending, approved, rejected)
- Financial totals (total amount, approved amount, pending amount)

## 🔐 Security Features

1. **JWT Authentication** - All protected routes require valid token
2. **Role-Based Access Control** - Admin/Manager/Employee permissions
3. **Company Isolation** - Users only see their company's data
4. **Audit Logging** - All actions logged with user, timestamp, IP
5. **Password Hashing** - bcrypt with salt
6. **Input Validation** - express-validator on all inputs

## 🧪 Testing the System

### Test Scenario 1: Complete Approval Flow

1. **Register company** (creates admin)
2. **Create manager** (admin creates manager user)
3. **Create employee** (admin creates employee with manager assigned)
4. **Create approval flow** (admin sets up 2-step flow)
5. **Employee submits expense** (triggers workflow)
6. **Manager approves** (moves to step 2)
7. **Finance team approves** (60% threshold met)
8. **Expense approved** (workflow complete)

### Test Scenario 2: Escalation

1. **Employee submits expense**
2. **Wait for escalation time** (or manually trigger)
3. **System auto-escalates** to manager's manager
4. **New approver receives notification**
5. **New approver makes decision**

### Test Scenario 3: Admin Override

1. **Expense stuck in approval**
2. **Admin overrides** with reason
3. **Expense immediately approved/rejected**
4. **All parties notified**

## 🐛 Troubleshooting

### Database Connection Issues
```bash
# Check PostgreSQL is running
psql -U postgres

# Verify DATABASE_URL in .env
DATABASE_URL=postgresql://user:password@localhost:5432/expense_db
```

### Port Already in Use
```bash
# Change PORT in backend/.env
PORT=5001

# Or kill process on port 5000
# Windows: netstat -ano | findstr :5000
# Linux/Mac: lsof -ti:5000 | xargs kill
```

### JWT Token Expired
```bash
# Re-login to get new token
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "password"}'
```

### OCR Not Working
```bash
# Development: Uses mock OCR (always works)
# Production: Set GOOGLE_VISION_API_KEY in .env
GOOGLE_VISION_API_KEY=your_api_key_here
```

## 📚 Next Steps

1. **Configure Email** - Set up SMTP for real email notifications
2. **Add More Users** - Create your team structure
3. **Customize Approval Flows** - Set up flows for different categories/amounts
4. **Upload Receipts** - Test OCR extraction
5. **Generate Reports** - Use the reports page for analytics

## 🔗 Useful Links

- Backend API: `http://localhost:5000/api`
- Frontend: `http://localhost:3001`
- API Health Check: `http://localhost:5000/api/health`

## 💡 Tips

1. **Use Postman/Insomnia** - Import the API endpoints for easier testing
2. **Check Logs** - Backend console shows all operations
3. **Audit Trail** - Check audit_logs table for complete history
4. **Test with Small Amounts** - Use auto-approval threshold for quick testing
5. **Multiple Browsers** - Test different user roles simultaneously

---

**Need Help?** Check `IMPLEMENTATION_STATUS.md` for feature completeness or `API_DOCUMENTATION.md` for detailed endpoint specs.
