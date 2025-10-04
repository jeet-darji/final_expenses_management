# Admin User Management Guide

## Overview
Admins have full control over user management including creating employees, managers, assigning roles, and defining manager-employee relationships.

---

## ✅ Admin Capabilities

### 1. **Create Users**
- ✅ Create Employees
- ✅ Create Managers
- ✅ Create Additional Admins

### 2. **Manage Roles**
- ✅ Assign roles: Employee, Manager, Admin
- ✅ Change user roles
- ✅ Set approval permissions

### 3. **Define Relationships**
- ✅ Assign managers to employees
- ✅ Change manager assignments
- ✅ View organizational hierarchy

### 4. **User Administration**
- ✅ Activate/Deactivate users
- ✅ Update user details
- ✅ Set approval limits for managers

---

## 🎯 API Endpoints for Admin

### **1. Create Employee**

```http
POST http://localhost:3000/api/companies/{companyId}/users
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john.doe@company.com",
  "password": "secure123",
  "role": "employee",
  "department": "Sales",
  "employeeId": "EMP001",
  "employeeLevel": 2,
  "managerId": "<manager-user-id>",
  "canApproveExpenses": false
}
```

**Response:**
```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "user-uuid",
    "name": "John Doe",
    "email": "john.doe@company.com",
    "role": "employee",
    "department": "Sales",
    "employeeId": "EMP001",
    "managerId": "manager-uuid",
    "canApproveExpenses": false
  }
}
```

---

### **2. Create Manager**

```http
POST http://localhost:3000/api/companies/{companyId}/users
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "name": "Jane Smith",
  "email": "jane.smith@company.com",
  "password": "manager123",
  "role": "manager",
  "department": "Sales",
  "employeeId": "MGR001",
  "employeeLevel": 4,
  "managerId": null,
  "canApproveExpenses": true,
  "approveLimit": 5000.00
}
```

**Response:**
```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "manager-uuid",
    "name": "Jane Smith",
    "email": "jane.smith@company.com",
    "role": "manager",
    "department": "Sales",
    "canApproveExpenses": true,
    "approveLimit": 5000.00
  }
}
```

---

### **3. Create Additional Admin**

```http
POST http://localhost:3000/api/companies/{companyId}/users
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "name": "Admin User",
  "email": "admin2@company.com",
  "password": "admin123",
  "role": "admin",
  "department": "Administration",
  "employeeId": "ADM002",
  "employeeLevel": 5,
  "canApproveExpenses": true,
  "approveLimit": null
}
```

**Note:** Admin has unlimited approval limit (null = unlimited)

---

### **4. Get All Company Users**

```http
GET http://localhost:3000/api/companies/{companyId}/users
Authorization: Bearer <admin-token>
```

**With Filters:**
```http
GET http://localhost:3000/api/companies/{companyId}/users?role=employee&department=Sales&isActive=true
```

**Response:**
```json
{
  "success": true,
  "count": 25,
  "total": 25,
  "data": [
    {
      "id": "user-uuid",
      "name": "John Doe",
      "email": "john.doe@company.com",
      "role": "employee",
      "department": "Sales",
      "employeeId": "EMP001",
      "managerId": "manager-uuid",
      "manager": {
        "id": "manager-uuid",
        "name": "Jane Smith",
        "email": "jane.smith@company.com"
      },
      "isActive": true
    }
  ]
}
```

---

### **5. Update User Role**

```http
PATCH http://localhost:3000/api/companies/{companyId}/users/{userId}
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "role": "manager",
  "canApproveExpenses": true,
  "approveLimit": 3000.00
}
```

**Example: Promote Employee to Manager**
```json
{
  "role": "manager",
  "canApproveExpenses": true,
  "approveLimit": 5000.00,
  "employeeLevel": 4
}
```

**Example: Demote Manager to Employee**
```json
{
  "role": "employee",
  "canApproveExpenses": false,
  "approveLimit": null,
  "employeeLevel": 2
}
```

---

### **6. Assign/Change Manager**

```http
PATCH http://localhost:3000/api/companies/{companyId}/users/{employeeId}
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "managerId": "<new-manager-user-id>"
}
```

**Remove Manager Assignment:**
```json
{
  "managerId": null
}
```

---

### **7. Update User Details**

```http
PATCH http://localhost:3000/api/companies/{companyId}/users/{userId}
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "name": "Updated Name",
  "department": "New Department",
  "employeeLevel": 3,
  "position": "Senior Developer"
}
```

---

### **8. Deactivate User**

```http
PATCH http://localhost:3000/api/companies/{companyId}/users/{userId}/deactivate
Authorization: Bearer <admin-token>
```

**Response:**
```json
{
  "success": true,
  "message": "User account deactivated successfully"
}
```

**Note:** Deactivated users cannot login but data is preserved.

---

### **9. Reactivate User**

```http
PATCH http://localhost:3000/api/companies/{companyId}/users/{userId}
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "isActive": true
}
```

---

## 📊 User Role Specifications

### **Employee Role**
```json
{
  "role": "employee",
  "canApproveExpenses": false,
  "approveLimit": null,
  "managerId": "<manager-id>",  // Required for approval workflow
  "employeeLevel": 1-3
}
```

**Permissions:**
- ✅ Submit expenses
- ✅ View own expenses
- ✅ Check approval status
- ❌ Approve expenses
- ❌ View other's expenses
- ❌ Manage users

---

### **Manager Role**
```json
{
  "role": "manager",
  "canApproveExpenses": true,
  "approveLimit": 5000.00,  // Max amount they can approve
  "managerId": null,  // Optional: Manager's manager
  "employeeLevel": 4-5
}
```

**Permissions:**
- ✅ Submit expenses
- ✅ View own expenses
- ✅ **Approve/reject team expenses**
- ✅ **View team expenses**
- ✅ Escalate expenses
- ✅ View company users (read-only)
- ❌ Manage users
- ❌ Configure approval flows

---

### **Admin Role**
```json
{
  "role": "admin",
  "canApproveExpenses": true,
  "approveLimit": null,  // Unlimited
  "employeeLevel": 5
}
```

**Permissions:**
- ✅ **All Manager permissions**
- ✅ **Create/manage users**
- ✅ **Assign/change roles**
- ✅ **Configure approval flows**
- ✅ **Override any approval**
- ✅ **View all expenses**
- ✅ **Access admin dashboard**
- ✅ **View audit logs**

---

## 🔄 Common Workflows

### **Workflow 1: Onboard New Employee**

```javascript
// Step 1: Create employee
POST /api/companies/{companyId}/users
{
  "name": "New Employee",
  "email": "new.employee@company.com",
  "password": "temp123",
  "role": "employee",
  "department": "Engineering",
  "managerId": "<manager-id>"
}

// Step 2: Employee logs in and changes password
POST /api/auth/login
POST /api/auth/updatepassword

// Step 3: Employee can now submit expenses
POST /api/companies/{companyId}/expenses
```

---

### **Workflow 2: Promote Employee to Manager**

```javascript
// Step 1: Update role
PATCH /api/companies/{companyId}/users/{userId}
{
  "role": "manager",
  "canApproveExpenses": true,
  "approveLimit": 5000.00,
  "employeeLevel": 4
}

// Step 2: Assign employees to this manager
PATCH /api/companies/{companyId}/users/{employee1Id}
{
  "managerId": "<newly-promoted-manager-id>"
}

PATCH /api/companies/{companyId}/users/{employee2Id}
{
  "managerId": "<newly-promoted-manager-id>"
}

// Step 3: Manager can now approve team expenses
GET /api/approvals/pending
```

---

### **Workflow 3: Reorganize Team Structure**

```javascript
// Scenario: Move employees from Manager A to Manager B

// Step 1: Get all employees under Manager A
GET /api/companies/{companyId}/users?managerId=<manager-a-id>

// Step 2: Reassign each employee to Manager B
PATCH /api/companies/{companyId}/users/{employee1Id}
{ "managerId": "<manager-b-id>" }

PATCH /api/companies/{companyId}/users/{employee2Id}
{ "managerId": "<manager-b-id>" }

// Step 3: Verify new structure
GET /api/companies/{companyId}/users?managerId=<manager-b-id>
```

---

### **Workflow 4: Create Department with Manager and Employees**

```javascript
// Step 1: Create Manager
POST /api/companies/{companyId}/users
{
  "name": "Department Manager",
  "email": "dept.manager@company.com",
  "password": "manager123",
  "role": "manager",
  "department": "Marketing",
  "canApproveExpenses": true,
  "approveLimit": 10000.00
}
// Save the manager ID from response

// Step 2: Create Employees under this manager
POST /api/companies/{companyId}/users
{
  "name": "Employee 1",
  "email": "emp1@company.com",
  "password": "emp123",
  "role": "employee",
  "department": "Marketing",
  "managerId": "<manager-id-from-step1>"
}

POST /api/companies/{companyId}/users
{
  "name": "Employee 2",
  "email": "emp2@company.com",
  "password": "emp123",
  "role": "employee",
  "department": "Marketing",
  "managerId": "<manager-id-from-step1>"
}

// Step 3: Verify department structure
GET /api/companies/{companyId}/users?department=Marketing
```

---

## 🎨 Frontend Implementation Examples

### **Admin User Management Dashboard**

```javascript
// Fetch all users
const fetchUsers = async () => {
  const response = await fetch(
    `/api/companies/${companyId}/users`,
    {
      headers: { 'Authorization': `Bearer ${adminToken}` }
    }
  );
  const { data: users } = await response.json();
  return users;
};

// Create new user
const createUser = async (userData) => {
  const response = await fetch(
    `/api/companies/${companyId}/users`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${adminToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(userData)
    }
  );
  return await response.json();
};

// Update user role
const updateUserRole = async (userId, newRole) => {
  const response = await fetch(
    `/api/companies/${companyId}/users/${userId}`,
    {
      method: 'PATCH',
      headers: {
        'Authorization': `Bearer ${adminToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        role: newRole,
        canApproveExpenses: newRole !== 'employee',
        approveLimit: newRole === 'manager' ? 5000 : null
      })
    }
  );
  return await response.json();
};

// Assign manager to employee
const assignManager = async (employeeId, managerId) => {
  const response = await fetch(
    `/api/companies/${companyId}/users/${employeeId}`,
    {
      method: 'PATCH',
      headers: {
        'Authorization': `Bearer ${adminToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ managerId })
    }
  );
  return await response.json();
};
```

---

### **Create User Form Component**

```jsx
const CreateUserForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: '',
    role: 'employee',
    department: '',
    managerId: null,
    canApproveExpenses: false,
    approveLimit: null
  });

  const handleRoleChange = (role) => {
    setFormData({
      ...formData,
      role,
      canApproveExpenses: role !== 'employee',
      approveLimit: role === 'manager' ? 5000 : null
    });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await createUser(formData);
    // Show success message and refresh user list
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="text" 
        placeholder="Name" 
        value={formData.name}
        onChange={(e) => setFormData({...formData, name: e.target.value})}
      />
      
      <input 
        type="email" 
        placeholder="Email" 
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
      />
      
      <input 
        type="password" 
        placeholder="Password" 
        value={formData.password}
        onChange={(e) => setFormData({...formData, password: e.target.value})}
      />
      
      <select 
        value={formData.role}
        onChange={(e) => handleRoleChange(e.target.value)}
      >
        <option value="employee">Employee</option>
        <option value="manager">Manager</option>
        <option value="admin">Admin</option>
      </select>
      
      <input 
        type="text" 
        placeholder="Department" 
        value={formData.department}
        onChange={(e) => setFormData({...formData, department: e.target.value})}
      />
      
      {formData.role === 'employee' && (
        <select 
          value={formData.managerId || ''}
          onChange={(e) => setFormData({...formData, managerId: e.target.value})}
        >
          <option value="">Select Manager</option>
          {managers.map(m => (
            <option key={m.id} value={m.id}>{m.name}</option>
          ))}
        </select>
      )}
      
      {formData.role === 'manager' && (
        <input 
          type="number" 
          placeholder="Approval Limit" 
          value={formData.approveLimit || ''}
          onChange={(e) => setFormData({...formData, approveLimit: e.target.value})}
        />
      )}
      
      <button type="submit">Create User</button>
    </form>
  );
};
```

---

## 📋 Validation Rules

### **Email**
- Must be unique across the company
- Must be valid email format
- Cannot be changed after creation

### **Password**
- Minimum 6 characters
- User can change after first login

### **Role**
- Must be one of: `employee`, `manager`, `admin`
- Determines permissions automatically

### **Manager Assignment**
- Manager must exist in the same company
- Manager must have `role: 'manager'` or `role: 'admin'`
- Manager must be active

### **Approval Limits**
- Employee: `null` or `0` (cannot approve)
- Manager: Positive number (e.g., 5000.00)
- Admin: `null` (unlimited)

---

## 🔒 Security Considerations

### **Only Admins Can:**
- Create users
- Change user roles
- Assign managers
- Deactivate users
- View all user details

### **Managers Can:**
- View users in their company (read-only)
- View their team members
- Cannot modify user data

### **Employees Can:**
- View their own profile
- Update their own profile (limited fields)
- Cannot view other users

---

## ✅ Testing Checklist

- [ ] Admin can create employee
- [ ] Admin can create manager
- [ ] Admin can create additional admin
- [ ] Admin can change employee to manager
- [ ] Admin can change manager to employee
- [ ] Admin can assign manager to employee
- [ ] Admin can change employee's manager
- [ ] Admin can deactivate user
- [ ] Admin can reactivate user
- [ ] Admin can view all users
- [ ] Manager cannot create users
- [ ] Employee cannot create users
- [ ] Email uniqueness is enforced
- [ ] Manager assignment validation works
- [ ] Approval limits are enforced

---

## 📞 API Reference Summary

| Action | Method | Endpoint | Role Required |
|--------|--------|----------|---------------|
| Create user | POST | `/api/companies/:id/users` | Admin |
| Get all users | GET | `/api/companies/:id/users` | Admin, Manager |
| Update user | PATCH | `/api/companies/:id/users/:userId` | Admin |
| Deactivate user | PATCH | `/api/companies/:id/users/:userId/deactivate` | Admin |
| Get user details | GET | `/api/companies/:id/users/:userId` | Admin, Manager |

---

**Status:** ✅ Fully implemented and ready to use!

**Next Steps:**
1. Login as admin
2. Create manager users
3. Create employee users
4. Assign managers to employees
5. Test approval workflow
