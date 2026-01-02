# Get to Testable Version - Step-by-Step Guide

## 🎯 Goal: Get the Application Running & Testable in 2-3 Days

This guide focuses on the **absolute minimum** needed to run and test your application locally.

---

## ✅ What You Already Have

- ✅ **Codebase** - Complete application code
- ✅ **Database Schema** - 75+ migrations ready
- ✅ **Firebase** - Just integrated
- ✅ **Core Features** - All business logic implemented

---

## 🔴 What's BLOCKING You (Must Fix)

### 1. Development Environment Setup (2-4 hours)

**You Need:**
- [ ] .NET Core 2.2 SDK installed
- [ ] SQL Server (or SQL Server Express) installed
- [ ] Visual Studio 2019+ or VS Code
- [ ] Git (optional, for version control)

**Steps:**
```bash
# 1. Check if .NET Core 2.2 is installed
dotnet --version
# Should show: 2.2.x

# If not installed, download from:
# https://dotnet.microsoft.com/download/dotnet/2.2

# 2. Check SQL Server
# Open SQL Server Management Studio (SSMS)
# Or use command line: sqlcmd -S localhost -E
```

---

### 2. Database Configuration (1-2 hours)

**Critical:** Your connection strings point to production servers. You need to change them.

#### Step 1: Create Local Database

```sql
-- Open SQL Server Management Studio
-- Connect to your local SQL Server instance
-- Run this:

CREATE DATABASE SustainableTravelDb;
GO
```

#### Step 2: Update Connection Strings

**File 1:** `Presentation/Orca.Web/appsettings.Development.json`

**REPLACE:**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=136.144.214.127,1433;Initial Catalog=SustainableTravelDb;User ID=sa;Password=Vgij@5081;MultipleActiveResultSets=False;TrustServerCertificate=True"
  }
}
```

**WITH:**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=SustainableTravelDb;Integrated Security=True;MultipleActiveResultSets=True;TrustServerCertificate=True"
  }
}
```

**OR if using SQL Server with username/password:**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=SustainableTravelDb;User Id=sa;Password=YourLocalPassword;MultipleActiveResultSets=True;TrustServerCertificate=True"
  }
}
```

**File 2:** `Presentation/Orca.Web/App_Data/dataSettings.json`

**REPLACE:**
```json
{
  "DataProvider": "sqlserver",
  "DataConnectionString": "Data Source=LFATY\\SQLEXPRESS;Initial Catalog=SustainableTravelDb;Integrated Security=True;Persist Security Info=False;MultipleActiveResultSets=False;TrustServerCertificate=True",
  "RawDataSettings": {}
}
```

**WITH:**
```json
{
  "DataProvider": "sqlserver",
  "DataConnectionString": "Server=localhost;Database=SustainableTravelDb;Integrated Security=True;Persist Security Info=False;MultipleActiveResultSets=True;TrustServerCertificate=True",
  "RawDataSettings": {}
}
```

**OR if using named instance:**
```json
{
  "DataProvider": "sqlserver",
  "DataConnectionString": "Server=localhost\\SQLEXPRESS;Database=SustainableTravelDb;Integrated Security=True;Persist Security Info=False;MultipleActiveResultSets=True;TrustServerCertificate=True",
  "RawDataSettings": {}
}
```

#### Step 3: Run Database Migrations

```bash
# Navigate to project directory
cd /Users/alexjeffries/Downloads/SustainableTravel-master

# Restore NuGet packages
dotnet restore

# Run migrations
cd Presentation/Orca.Web
dotnet ef database update --project ../../Libraries/Orca.Data
```

**If `dotnet ef` command not found:**
```bash
# Install EF Core tools
dotnet tool install --global dotnet-ef --version 2.2.0
```

#### Step 4: Verify Stored Procedures

**Check if stored procedure exists:**
```sql
USE SustainableTravelDb;
GO

SELECT name FROM sys.procedures WHERE name = 'GetQuestionResult';
```

**If missing, check:**
- `Presentation/Orca.Web/App_Data/Install/SqlServer.StoredProcedures.sql`
- Run that SQL file if it exists

---

### 3. Security Fixes (1 hour) ⚠️ CRITICAL

**Even for testing, fix these immediately:**

#### Step 1: Change JWT Secret

**File:** `Presentation/Orca.Api/appsettings.json`

**FIND:**
```json
"JwtSettings": {
  "Secret": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
  "TokenLifetime": "00:00:45"
}
```

**REPLACE WITH:**
```json
"JwtSettings": {
  "Secret": "YOUR-STRONG-SECRET-KEY-HERE-MINIMUM-32-CHARACTERS-LONG",
  "TokenLifetime": "01:00:00"
}
```

**Generate a strong secret:**
```bash
# On Mac/Linux:
openssl rand -base64 32

# Or use online generator:
# https://www.grc.com/passwords.htm
```

#### Step 2: Remove Production Database Credentials

**Already done above** (connection strings), but verify:
- ✅ No production server IPs in config files
- ✅ No production passwords in code

---

### 4. Build & Run (30 minutes)

#### Step 1: Build the Solution

```bash
cd /Users/alexjeffries/Downloads/SustainableTravel-master

# Restore packages
dotnet restore

# Build solution
dotnet build SustainableTravel.sln
```

**Expected Output:**
```
Build succeeded.
    0 Warning(s)
    0 Error(s)
```

**If errors:**
- Check .NET Core 2.2 SDK is installed
- Check all NuGet packages restored
- Check for missing dependencies

#### Step 2: Run the Application

```bash
cd Presentation/Orca.Web
dotnet run
```

**Expected Output:**
```
Now listening on: https://localhost:5001
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

#### Step 3: Access the Application

Open browser:
- **Public Site:** http://localhost:5000
- **Admin Panel:** http://localhost:5000/Admin

**First Time:** You'll be redirected to `/install` for initial setup.

---

### 5. Initial Installation (30 minutes)

#### Step 1: Complete Installation Wizard

When you first run the app, you'll see the installation wizard:

1. **Database Configuration:**
   - Should auto-detect from `dataSettings.json`
   - Click "Next"

2. **Admin Account Setup:**
   - Email: `admin@test.com` (or your email)
   - Password: Choose a strong password
   - **SAVE THIS PASSWORD!**

3. **Sample Data:**
   - Choose "Yes" for testing (adds sample data)
   - Or "No" to start fresh

4. **Click "Install"**

#### Step 2: Login

After installation:
- Go to: http://localhost:5000/login
- Use the admin credentials you just created

---

## 🟡 What You Can SKIP for Testing (Fix Later)

### Skip for Now:
- ❌ **Comprehensive Testing** - Manual testing is fine
- ❌ **Bug Fixes** - Test first, fix bugs you find
- ❌ **Framework Upgrade** - .NET 2.2 works for testing
- ❌ **Complete API** - Web interface is enough
- ❌ **DevOps/CI-CD** - Not needed for local testing
- ❌ **Performance Optimization** - Test functionality first

### Fix After Testing:
- 🟡 **Calculation Bugs** - Test first, then fix issues you discover
- 🟡 **Race Conditions** - Only matters under load
- 🟡 **Input Validation** - Add as you find issues

---

## ✅ Testing Checklist

Once running, test these core features:

### Basic Functionality
- [ ] **Homepage loads** - http://localhost:5000
- [ ] **Admin login works** - http://localhost:5000/Admin
- [ ] **User registration** - http://localhost:5000/register
- [ ] **Company registration** - http://localhost:5000/company/register

### Admin Panel
- [ ] **Dashboard loads** - /Admin
- [ ] **User management** - /Admin/User/List
- [ ] **Company management** - /Admin/Company/List
- [ ] **Questionnaire management** - /Admin/Questionnaire/List
- [ ] **Product management** - /Admin/Category/List

### Travel Operator Portal
- [ ] **Login as company** - Register company, then login
- [ ] **Dashboard** - /TravelOperator
- [ ] **Create tour** - /TravelOperator/Company/Tours
- [ ] **View analytics** - Check dashboard charts

### Questionnaire System
- [ ] **Take questionnaire** - /questionnaire/guestquestions
- [ ] **Submit responses** - Complete a questionnaire
- [ ] **View results** - Check dashboard for results

### Firebase Analytics
- [ ] **Check browser console** - No Firebase errors
- [ ] **Verify Analytics** - Check Firebase Console (may take time)

---

## 🐛 Common Issues & Fixes

### Issue 1: "Cannot connect to database"

**Symptoms:**
- Error on startup
- Installation wizard fails

**Fix:**
1. Verify SQL Server is running
2. Check connection string is correct
3. Test connection:
   ```sql
   sqlcmd -S localhost -E -Q "SELECT @@VERSION"
   ```

### Issue 2: "dotnet ef command not found"

**Symptoms:**
- Error when running migrations

**Fix:**
```bash
dotnet tool install --global dotnet-ef --version 2.2.0
```

### Issue 3: "Build fails with missing packages"

**Symptoms:**
- NuGet restore errors
- Missing references

**Fix:**
```bash
# Clean and restore
dotnet clean
dotnet restore
dotnet build
```

### Issue 4: "Port already in use"

**Symptoms:**
- Can't start application
- Port 5000/5001 in use

**Fix:**
```bash
# Find process using port
lsof -i :5000

# Kill process or change port in launchSettings.json
```

### Issue 5: "Firebase not loading"

**Symptoms:**
- Console errors about Firebase
- Analytics not working

**Fix:**
- Check internet connection (CDN requires internet)
- Check browser console for specific errors
- Verify Firebase scripts are loading in page source

---

## 📋 Quick Start Commands

### Complete Setup (Copy & Paste)

```bash
# 1. Navigate to project
cd /Users/alexjeffries/Downloads/SustainableTravel-master

# 2. Restore packages
dotnet restore

# 3. Install EF Core tools (if needed)
dotnet tool install --global dotnet-ef --version 2.2.0

# 4. Update connection strings (edit files manually first!)

# 5. Create database (in SQL Server Management Studio)
# CREATE DATABASE SustainableTravelDb;

# 6. Run migrations
cd Presentation/Orca.Web
dotnet ef database update --project ../../Libraries/Orca.Data

# 7. Build
cd ../..
dotnet build SustainableTravel.sln

# 8. Run
cd Presentation/Orca.Web
dotnet run
```

---

## 🎯 Success Criteria

**You're ready to test when:**

- ✅ Application starts without errors
- ✅ Can access http://localhost:5000
- ✅ Installation wizard completes
- ✅ Can login to admin panel
- ✅ Database has tables (check in SSMS)
- ✅ No critical errors in browser console

---

## 📝 Next Steps After Testing

Once you have a testable version:

1. **Test All Features** - Use the checklist above
2. **Document Issues** - Create a bug list
3. **Fix Critical Bugs** - See CALCULATION_BUGS_FOUND.md
4. **Plan Improvements** - Based on what you find
5. **Prepare for MVP** - See MVP_LAUNCH_PLAN.md

---

## ⏱️ Time Estimate

| Task | Time | Priority |
|------|------|----------|
| Install .NET Core 2.2 | 15 min | 🔴 Critical |
| Install SQL Server | 30 min | 🔴 Critical |
| Update connection strings | 15 min | 🔴 Critical |
| Create database | 5 min | 🔴 Critical |
| Run migrations | 10 min | 🔴 Critical |
| Fix JWT secret | 5 min | 🔴 Critical |
| Build solution | 10 min | 🔴 Critical |
| Run application | 5 min | 🔴 Critical |
| Complete installation | 15 min | 🔴 Critical |
| **TOTAL** | **2-3 hours** | |

---

## 🚀 You're Ready When...

- [x] Application runs on localhost:5000
- [x] Admin panel accessible
- [x] Can create users
- [x] Can create companies
- [x] Can create tours
- [x] Can take questionnaires
- [x] Dashboard shows data

**Then you can start testing!** 🎉

---

*Guide Created: October 14, 2025*
*Focus: Get to testable version ASAP*

