# 🚀 Enterprise Expense Management System Setup Guide

## 📋 Pre-requisites

1. **Node.js** (v16 or higher)
2. **PostgreSQL** (v12 or higher)
3. **Git**
4. **Google Cloud Account** (for Vision API)

## 🔧 Backend Setup

### 1. Install Dependencies
```bash
cd expense-management-system/backend
npm install
```

### 2. Database Configuration
```bash
# Create PostgreSQL database
createdb expense_management_db

# Create user (optional)
createuser expense_user
psql -c "GRANT ALL PRIVILEGES ON DATABASE expense_management_db TO expense_user;"
```

### 3. Environment Configuration
Create `.env` file in backend directory:

```env
# Database Configuration
DB_NAME=expense_management_db
DB_USER=expense_user
DB_PASSWORD=your_secure_password
DB_HOST=localhost
DB_PORT=5432

# JWT Configuration  
JWT_SECRET=your_super_secret_jwt_key_change_this_in_production_minimum_32_characters
JWT_EXPIRE=30d

# Server Configuration
PORT=5000
NODE_ENV=development
FRONTEND_URL=http://localhost:3001

# Email Configuration (for notifications)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password

# Google Vision API (for OCR)
GOOGLE_VISION_API_KEY=your_google_vision_api_key

# Redis Configuration (for caching and queues)
REDIS_URL=redis://localhost:6379

# Security
BCRYPT_ROUNDS=12
AUDIT_ENABLED=true
```

### 4. Start Backend Server
```bash
npm run dev
```

## 🎨 Frontend Setup

### 1. Install Dependencies
```bash
cd expense-management-system/frontend
npm install
```

### 2. Environment Configuration
Create `.env` file in frontend directory:
```env
REACT_APP_API_URL=http://localhost:5000/api
REACT_APP_NAME=Expense Management System
```

### 3. Start Frontend Server
```bash
npm start
```

## 🗄️ Database Schema

The system automatically creates these tables:
- `companies` - Company information and settings
- `users` - User accounts with roles and hierarchy
- `expenses` - Expense claims with multi-currency support
- `approval_flows` - Configurable approval workflow templates
- `approval_steps` - Individual steps within workflows
- `approvals` - Individual approval records
- `expense_receipts` - Receipt files with OCR data
- `audit_logs` - Comprehensive audit trail

## 🏢 Getting Started

### 1. Create Your First Company

**Step 1:** Navigate to `http://localhost:3001/register`

**Step 2:** Fill out the company registration form:
- Company Name: "Your Company Inc"
- Country: Select your country (e.g., "United States")
- Admin Details: Your name, email, and password

**Step 3:** The system automatically:
- ✅ Creates your company
- ✅ Sets base currency based on country
- ✅ Creates default approval workflow
- ✅ Makes you company admin

### 2. Create Team Members (Admin Only)

**Step 1:** Login and go to Admin → Users
**Step 2:** Click "Add User"
**Step 3:** Fill user details:
- Name, Email, Role (Employee/Manager/Admin)
- Department, Employee Level
- Manager Assignment (for hierarchy)
- Approval Permissions

### 3. Configure Approval Workflows (Admin Only)

**Step 1:** Go to Admin → Approval Workflows
**Step 2:** Create workflow:
```
Workflow: High-Value Expenses (>$500)
Rules:
  Step 1: Manager Approval (single approver)
  Step 2: Finance Team (3 approvers, 60% threshold)
  Step 3: Director (specific approver OR 60% threshold)
```

**Step 3:** Set conditions:
- Apply to: Amount > $500
- Default for: All categories

### 4. Submit Your First Expense

**Step 1:** Go to "New Expense"
**Step 2:** Upload receipt(s) - OCR auto-extracts:
- Amount, Date, Merchant, Category
- Review and edit extracted data
**Step 3:** Submit expense:
- System converts to base currency
- Triggers approval workflow
- Sends notifications

## 🎯 Key Features Demo

### Multi-Currency Support
```javascript
// Employee in Germany submits EUR 150 expense
// System automatically converts to company USD base
POST /api/companies/{id}/expenses
{
  "amount": 150.00,
  "currency": "EUR",
  "amountInBaseCurrency": 165.00, // Auto-converted
  "exchangeRate": 1.10,
  "exchangeRateDate": "2024-01-15"
}
```

### Smart Approval Workflows
```javascript
// Multi-step approval with hybrid rules
Step 1: Manager (single approver)
Step 2: Finance Team (60% OR CFO approves)
Step 3: Director (specific approval OR majority)
```

### OCR Receipt Processing
```javascript
// Upload receipt → Auto-extract data
{
  "fileName": "receipt_20240115.pdf",
  "ocrData": {
    "amount": 87.50,
    "currency": "USD", 
    "date": "2024-01-15",
    "merchant": "Starbucks Coffee",
    "category": "Meals",
    "confidence": 0.95
  }
}
```

## 🔐 Security & Permissions

### Company Isolation
- Users only see their company's data
- Automatic scoping by `companyId`
- Cross-company access blocked

### Role-Based Access
- **Employee**: Own expenses only, submit/receipt upload
- **Manager**: Team expenses + approvals, view team reports
- **Admin**: All expenses, user management, workflow config

### Audit Logging
Every action is logged:
- User operations (create, update, delete)
- Expense submissions and approvals
- User management actions
- System security events

## 🚨 Important Security Notes

1. **Change Default Secrets**: Update JWT_SECRET immediately
2. **SSL/TLS**: Use HTTPS in production
3. **Database Security**: Restrict database access
4. **API Keys**: Never commit API keys to git
5. **Environment Variables**: Keep `.env` files secure

## 📊 API Endpoints Reference

### Authentication
- `POST /api/auth/register` - Create company + admin
- `POST /api/auth/login` - User login

### Companies
- `GET /api/companies/:id` - Get company details
- `PUT /api/companies/:id/currency` - Update currency settings

### Expenses  
- `POST /api/companies/:id/expenses` - Submit expense
- `GET /api/companies/:id/expenses` - List expenses
- `GET /api/companies/:id/expenses/:expenseId` - Get expense details

### Approvals
- `GET /api/companies/:id/approvals/pending` - Pending approvals
- `POST /api/companies/:id/expenses/:expenseId/approve` - Approve/reject
- `POST /api/companies/:id/expenses/:expenseId/escalate` - Escalate

### Admin Functions
- `POST /api/companies/:id/users` - Create user
- `POST /api/companies/:id/approval-flows` - Create workflow
- `POST /api/companies/:id/override` - Admin override

### OCR/Receipts
- `POST /api/uploads/receipt` - Upload receipt
- `POST /api/companies/:id/expenses/:expenseId/receipts` - Attach receipt

## 🧪 Testing

### Run Unit Tests
```bash
cd backend
npm test
```

### Test Approval Rules
```bash
npm test approvalRule.test.js
```

### Integration Testing
```bash
npm run test:integration
```

## 🚀 Deployment Checklist

### Backend Deployment
1. Set `NODE_ENV=production`
2. Configure production database
3. Set up Redis for caching
4. Configure email service
5. Set up Google Vision API
6. Enable SSL/TLS
7. Configure reverse proxy (nginx)

### Frontend Deployment  
1. Set production API URL
2. Build optimized bundle
3. Deploy to CDN/web server
4. Configure caching headers

### Monitoring
1. Set up application monitoring
2. Configure log aggregation
3. Set up alerting
4. Monitor API performance

## 🆘 Troubleshooting

### Common Issues

**Database Connection Failed**
```
Error: Connection to PostgreSQL failed
Solution: Check DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD
```

**OCR Not Working**
```
Error: OCR processing failed
Solution: Verify GOOGLE_VISION_API_KEY is set and valid
```

**JWT Token Issues**
```
Error: Invalid token
Solution: Ensure JWT_SECRET is properly set and never changes
```

**Email Notifications Failed**
```
Error: Email sending failed  
Solution: Check EMAIL_* configuration and app passwords
```

## 📞 Support

For additional help:
1. Check the console logs for detailed error messages
2. Verify all environment variables are set
3. Ensure all dependencies are installed
4. Check database connectivity
5. Review the audit logs for operation history

---

🎉 **Congratulations!** You now have a production-ready, enterprise-grade expense management system with multi-currency support, intelligent approval workflows, OCR receipt processing, and comprehensive audit logging!
