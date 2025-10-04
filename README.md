# 💼 Enterprise Expense Management System

A comprehensive, production-ready expense management system with multi-step approval workflows, OCR receipt processing, multi-currency support, and advanced automation features.

## 🌟 Key Features

### ✅ Complete Expense Lifecycle Management
- **Draft → Submit → Approve → Pay** workflow
- Multi-step approval with complex rules
- Auto-approval for small amounts
- Admin override capabilities
- Expense cancellation and suspension

### 🔄 Advanced Approval Workflows
- **Percentage-based rules** (e.g., 60% of approvers must approve)
- **Specific approver rules** (e.g., CFO approval auto-completes step)
- **Hybrid rules** (percentage OR specific approver)
- **Veto power** (one rejection fails the step)
- **Manager-based routing** (auto-assigns to submitter's manager)
- **Conditional flows** (different flows for categories/amounts)

### ⏰ Smart Escalation & Reminders
- **Auto-escalation** after configurable timeout
- **Email reminders** 24 hours before escalation
- **Manual escalation** to higher authority
- **Escalation tracking** in audit logs

### 💱 Multi-Currency Support
- **Auto-detection** of currency from country
- **Real-time exchange rates** with 1-hour caching
- **Dual amount storage** (original + base currency)
- **Historical rate tracking** for audit compliance

### 📸 OCR Receipt Processing
- **Mock OCR** for development (instant)
- **Google Vision API** integration for production
- **Extracts**: amount, date, merchant, category, line items
- **Confidence scoring** for data quality
- **User confirmation** before submission

### 🔐 Enterprise Security
- **JWT authentication** with refresh tokens
- **Role-based access control** (Admin/Manager/Employee)
- **Company data isolation** (multi-tenant ready)
- **Complete audit trail** (all actions logged)
- **Password hashing** with bcrypt
- **Input validation** on all endpoints

### 📊 Admin Dashboard & Reports
- **Real-time statistics** (users, expenses, financials)
- **Pending approvals** overview
- **Team expense tracking**
- **Date range filtering**
- **Export capabilities** (ready for CSV/PDF)

### 🔔 Notification System
- **In-app notifications**
- **Email notifications** (SMTP ready)
- **Approval requests**
- **Decision notifications**
- **Escalation alerts**
- **Status change updates**

## 🏗️ Architecture

### Backend (Node.js + Express)
```
backend/
├── controllers/       # Request handlers
│   ├── authController.js
│   ├── expenseController.js
│   ├── approvalController.js
│   ├── approvalFlowController.js
│   ├── adminController.js
│   └── companyController.js
├── models/           # Sequelize ORM models
│   ├── User.js
│   ├── Company.js
│   ├── Expense.js
│   ├── Approval.js
│   ├── ApprovalFlow.js
│   ├── ApprovalStep.js
│   ├── ExpenseReceipt.js
│   └── AuditLog.js
├── services/         # Business logic
│   ├── approvalWorkflowService.js
│   ├── currencyService.js
│   ├── ocrService.js
│   ├── notificationService.js
│   └── escalationService.js
├── routes/           # API routes
├── middleware/       # Auth, validation, error handling
└── server.js         # Entry point
```

### Frontend (React + Redux)
```
frontend/
├── src/
│   ├── components/   # Reusable UI components
│   ├── pages/        # Page components
│   ├── features/     # Redux slices
│   ├── services/     # API calls
│   └── App.js
```

### Database (PostgreSQL)
- **8 core tables** with proper relationships
- **Foreign key constraints** for data integrity
- **Indexes** on frequently queried fields
- **JSONB columns** for flexible metadata
- **Audit log** with immutable entries

## 🚀 Quick Start

### Prerequisites
- Node.js v16+
- PostgreSQL v12+
- npm or yarn

### Installation

```bash
# Clone repository
git clone <repository-url>
cd expense-management-system

# Backend setup
cd backend
npm install
cp .env.example .env
# Edit .env with your database credentials
npm run dev

# Frontend setup (new terminal)
cd frontend
npm install
npm start
```

### Default Credentials
After first run, a default admin is created:
- **Email**: admin@example.com
- **Password**: admin123

**⚠️ Change these immediately in production!**

## 📚 Documentation

- **[Quick Start Guide](QUICK_START_GUIDE.md)** - Step-by-step setup and usage
- **[API Documentation](API_DOCUMENTATION.md)** - Complete API reference
- **[Implementation Status](IMPLEMENTATION_STATUS.md)** - Feature completeness checklist

## 🎯 Core Workflows

### 1. Company Registration
```
User registers → Company created → Base currency auto-detected → 
Admin user created → Default approval flow created → Ready to use
```

### 2. Expense Submission
```
Employee creates expense → Uploads receipt → OCR extracts data → 
Employee confirms → Submits → Workflow triggered → Approvals created
```

### 3. Multi-Step Approval
```
Step 1: Manager approves → 
Step 2: Finance team (60% threshold) → 
All steps complete → Expense approved → 
Notification sent → Ready for payment
```

### 4. Auto-Escalation
```
Approval pending → 24h reminder sent → 
Timeout reached → Auto-escalate to manager's manager → 
New approval created → Notification sent
```

### 5. Admin Override
```
Expense stuck → Admin reviews → 
Override decision → All pending approvals updated → 
Notifications sent → Audit logged
```

## 🔧 Configuration

### Environment Variables (.env)

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/expense_db

# Server
PORT=5000
NODE_ENV=development
FRONTEND_URL=http://localhost:3001

# JWT
JWT_SECRET=your_super_secret_key_here
JWT_EXPIRE=30d

# Email (Optional - for production)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password
EMAIL_FROM=noreply@yourcompany.com

# OCR (Optional - for production)
GOOGLE_VISION_API_KEY=your_google_vision_api_key

# File Upload
MAX_FILE_SIZE=10485760
UPLOAD_PATH=./uploads
```

### Company Settings

Configurable per company:
- `autoApproveLimit` - Auto-approve expenses below this amount
- `requireApprovalCategories` - Categories requiring approval
- `allowExpenseSubmission` - Enable/disable expense submission
- `timezone` - Company timezone

## 📊 Database Schema

### Core Tables
1. **Users** - User accounts with roles and permissions
2. **Companies** - Multi-tenant company data
3. **Expenses** - Expense records with amounts and status
4. **Approvals** - Individual approval requests
5. **ApprovalFlows** - Workflow definitions
6. **ApprovalSteps** - Step configurations with rules
7. **ExpenseReceipts** - Receipt files with OCR data
8. **AuditLogs** - Complete audit trail

### Key Relationships
- User → Company (many-to-one)
- User → Manager (self-referencing)
- Expense → User (submitter)
- Expense → ApprovalFlow
- Approval → Expense
- Approval → User (approver)
- Approval → ApprovalStep

## 🧪 Testing

### Manual Testing Scenarios

**Scenario 1: Complete Approval Flow**
1. Register company (creates admin)
2. Create manager and employee users
3. Set up 2-step approval flow
4. Employee submits expense
5. Manager approves (moves to step 2)
6. Finance team approves (60% threshold)
7. Expense approved

**Scenario 2: Auto-Escalation**
1. Submit expense
2. Wait for escalation timeout (or manually trigger)
3. System escalates to next level
4. New approver receives notification
5. New approver makes decision

**Scenario 3: Admin Override**
1. Expense stuck in approval
2. Admin overrides with reason
3. Expense immediately approved/rejected
4. All parties notified

### API Testing

Use the provided Postman collection or test with curl:

```bash
# Register company
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@company.com",
    "password": "test123",
    "companyName": "Test Corp",
    "country": "US"
  }'

# Login
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@company.com",
    "password": "test123"
  }'

# Create expense
curl -X POST http://localhost:5000/api/companies/{companyId}/expenses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 125.50,
    "currency": "USD",
    "category": "Meals",
    "description": "Client lunch",
    "dateOfExpense": "2025-01-15"
  }'
```

## 🔒 Security Best Practices

1. **Change default admin credentials** immediately
2. **Use strong JWT_SECRET** (minimum 32 characters)
3. **Enable HTTPS** in production
4. **Configure CORS** properly
5. **Set up rate limiting** (recommended: express-rate-limit)
6. **Regular security audits**
7. **Keep dependencies updated**
8. **Use environment variables** for sensitive data
9. **Enable database backups**
10. **Monitor audit logs** regularly

## 📈 Production Deployment

### Recommended Stack
- **Hosting**: AWS EC2, DigitalOcean, or Heroku
- **Database**: AWS RDS PostgreSQL or managed PostgreSQL
- **File Storage**: AWS S3 or compatible object storage
- **Email**: SendGrid, AWS SES, or Mailgun
- **OCR**: Google Vision API or Azure Computer Vision
- **Monitoring**: PM2, New Relic, or Datadog
- **Logging**: Winston + CloudWatch or Papertrail

### Pre-Deployment Checklist
- [ ] Change default admin credentials
- [ ] Set strong JWT_SECRET
- [ ] Configure production database
- [ ] Set up SMTP for emails
- [ ] Configure file storage (S3)
- [ ] Set up OCR API (Google Vision)
- [ ] Enable HTTPS/SSL
- [ ] Configure CORS for production domain
- [ ] Set up monitoring and logging
- [ ] Configure automated backups
- [ ] Test all critical workflows
- [ ] Set up CI/CD pipeline
- [ ] Document deployment process

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Update documentation
6. Submit a pull request

## 📝 License

This project is licensed under the MIT License.

## 🆘 Support

For issues, questions, or contributions:
- Create an issue on GitHub
- Check the documentation files
- Review the API documentation
- Check the implementation status

## 🎉 Acknowledgments

Built with:
- **Express.js** - Web framework
- **Sequelize** - ORM
- **PostgreSQL** - Database
- **React** - Frontend framework
- **Redux Toolkit** - State management
- **JWT** - Authentication
- **Multer** - File uploads
- **Axios** - HTTP client
- **node-cron** - Scheduled tasks
- **bcrypt** - Password hashing

## 📊 System Statistics

- **Backend**: ~5,000 lines of code
- **Frontend**: ~3,000 lines of code
- **API Endpoints**: 40+
- **Database Tables**: 8
- **Services**: 5 core services
- **Controllers**: 6 main controllers
- **Models**: 8 Sequelize models
- **Completion**: ~90% (production-ready)

## 🚦 Status

**Current Version**: 1.0.0  
**Status**: Production Ready (with minor enhancements pending)  
**Last Updated**: January 2025

---

**Ready to revolutionize your expense management? Get started with the [Quick Start Guide](QUICK_START_GUIDE.md)!** 🚀
