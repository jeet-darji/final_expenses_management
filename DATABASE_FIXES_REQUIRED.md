# Database Fixes Required

## ⚠️ Critical Issue: Audit Log Foreign Key Constraint

### Problem
The `audit_logs` table has an incorrect foreign key constraint that forces `entity_id` to reference **only** the `expenses` table. However, the application logs actions for multiple entity types:
- **Users** (login, profile updates, password changes)
- **Companies** (settings, currency changes)
- **Expenses** (create, update, approve, reject)
- **Approvals** (decisions, escalations)
- **Approval Flows** (create, update)

### Error Message
```
SequelizeForeignKeyConstraintError: insert or update on table "audit_logs" violates foreign key constraint "audit_logs_entity_id_fkey"
Key (entity_id)=(user-uuid) is not present in table "expenses"
```

### Solution: Remove the Incorrect Constraint

**Run this SQL in your PostgreSQL database:**

```sql
ALTER TABLE audit_logs DROP CONSTRAINT IF EXISTS audit_logs_entity_id_fkey;
```

### How to Execute

#### Option 1: Using pgAdmin
1. Open **pgAdmin 4**
2. Connect to database: `expense_management`
3. Right-click on the database → **Query Tool**
4. Paste the SQL above
5. Click **Execute** (F5)

#### Option 2: Using psql Command Line
```bash
psql -U postgres -d expense_management -c "ALTER TABLE audit_logs DROP CONSTRAINT IF EXISTS audit_logs_entity_id_fkey;"
```

#### Option 3: Using DBeaver or Other SQL Client
1. Connect to `expense_management` database
2. Open SQL editor
3. Paste and execute the SQL

---

## ✅ Code Fixes Applied

The following controller files have been fixed to properly import Sequelize operators:

### 1. **approvalController.js**
- ✅ Added `const { Op } = require('sequelize');`
- ✅ Removed duplicate `Op` import inside function

### 2. **expenseController.js**
- ✅ Fixed incorrect import: `Op` was imported from `express-validator` instead of `sequelize`
- ✅ Changed to: `const { Op } = require('sequelize');`

### 3. **adminController.js**
- ✅ Added `const { Op } = require('sequelize');`

---

## 🔄 After Fixing

1. **Execute the SQL fix** in your database
2. **Restart your backend server**
3. **Test login** - should work without audit log errors
4. **Test approvals** - queries should work correctly

---

## Verification

After applying the fix, verify with:

```sql
-- Check if constraint is removed
SELECT constraint_name, table_name 
FROM information_schema.table_constraints 
WHERE table_name = 'audit_logs' 
AND constraint_type = 'FOREIGN KEY';

-- Should NOT show audit_logs_entity_id_fkey
```

---

## Why This Design?

The `audit_logs` table uses a **polymorphic association** pattern:
- `entity_type` (ENUM): Specifies what type of entity
- `entity_id` (UUID): The ID of that entity

This allows flexible logging across all entity types without rigid foreign key constraints.

---

**Status:** 🟡 Database fix required, code fixes completed
