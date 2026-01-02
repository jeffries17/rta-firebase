# Quick Start Checklist - Get Running in 2-3 Hours

## ✅ Pre-Flight Check

- [ ] .NET Core 2.2 SDK installed? (`dotnet --version` should show 2.2.x)
- [ ] SQL Server installed? (SQL Server Express is fine)
- [ ] Visual Studio 2019+ or VS Code installed?
- [ ] Text editor ready? (for editing config files)

---

## 🔴 Step 1: Fix Connection Strings (15 min)

### File 1: `Presentation/Orca.Web/appsettings.Development.json`

**FIND THIS:**
```json
"DefaultConnection": "Server=136.144.214.127,1433;..."
```

**REPLACE WITH:**
```json
"DefaultConnection": "Server=localhost;Database=SustainableTravelDb;Integrated Security=True;MultipleActiveResultSets=True;TrustServerCertificate=True"
```

### File 2: `Presentation/Orca.Web/App_Data/dataSettings.json`

**FIND THIS:**
```json
"DataConnectionString": "Data Source=LFATY\\SQLEXPRESS;..."
```

**REPLACE WITH:**
```json
"DataConnectionString": "Server=localhost;Database=SustainableTravelDb;Integrated Security=True;Persist Security Info=False;MultipleActiveResultSets=True;TrustServerCertificate=True"
```

**OR if using SQL Server Express:**
```json
"DataConnectionString": "Server=localhost\\SQLEXPRESS;Database=SustainableTravelDb;Integrated Security=True;Persist Security Info=False;MultipleActiveResultSets=True;TrustServerCertificate=True"
```

---

## 🔴 Step 2: Fix JWT Secret (5 min)

### File: `Presentation/Orca.Api/appsettings.json`

**FIND:**
```json
"Secret": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
```

**REPLACE WITH:**
```json
"Secret": "YourRandomSecretKeyHereMinimum32CharactersLong123456"
```

**Generate one:**
- Mac/Linux: `openssl rand -base64 32`
- Or use: https://www.grc.com/passwords.htm (generate 64 character password)

---

## 🔴 Step 3: Create Database (5 min)

**Open SQL Server Management Studio (SSMS) or use command line:**

```sql
CREATE DATABASE SustainableTravelDb;
GO
```

**OR command line:**
```bash
sqlcmd -S localhost -E -Q "CREATE DATABASE SustainableTravelDb"
```

---

## 🔴 Step 4: Install EF Core Tools (5 min)

```bash
dotnet tool install --global dotnet-ef --version 2.2.0
```

**Verify:**
```bash
dotnet ef --version
```

---

## 🔴 Step 5: Run Migrations (10 min)

```bash
cd /Users/alexjeffries/Downloads/SustainableTravel-master
cd Presentation/Orca.Web
dotnet ef database update --project ../../Libraries/Orca.Data
```

**Expected:** Database tables created

**If error:** Check connection string again

---

## 🔴 Step 6: Build Solution (10 min)

```bash
cd /Users/alexjeffries/Downloads/SustainableTravel-master
dotnet restore
dotnet build SustainableTravel.sln
```

**Expected Output:**
```
Build succeeded.
    0 Warning(s)
    0 Error(s)
```

**If errors:** 
- Check .NET Core 2.2 is installed
- Check all packages restored
- Read error messages carefully

---

## 🔴 Step 7: Run Application (5 min)

```bash
cd Presentation/Orca.Web
dotnet run
```

**Expected:**
```
Now listening on: https://localhost:5001
Now listening on: http://localhost:5000
Application started.
```

**Open browser:**
- http://localhost:5000

---

## 🔴 Step 8: Complete Installation (15 min)

1. **You'll see installation wizard**
2. **Database:** Should auto-detect (click Next)
3. **Admin Account:**
   - Email: `admin@test.com`
   - Password: `Test123!` (or your choice)
   - **SAVE THIS!**
4. **Sample Data:** Choose "Yes" for testing
5. **Click Install**

---

## 🔴 Step 9: Test Login (2 min)

1. Go to: http://localhost:5000/login
2. Use admin credentials from Step 8
3. Should see admin dashboard

---

## ✅ You're Done! Start Testing

### Quick Tests:

- [ ] Homepage: http://localhost:5000
- [ ] Admin: http://localhost:5000/Admin
- [ ] Register User: http://localhost:5000/register
- [ ] Register Company: http://localhost:5000/company/register

---

## 🐛 Troubleshooting

### "Cannot connect to database"
- Check SQL Server is running
- Verify connection string
- Test: `sqlcmd -S localhost -E -Q "SELECT 1"`

### "dotnet ef not found"
- Run: `dotnet tool install --global dotnet-ef --version 2.2.0`

### "Port in use"
- Change port in `Properties/launchSettings.json`
- Or kill process: `lsof -i :5000` then `kill -9 <PID>`

### "Build fails"
- Run: `dotnet clean && dotnet restore && dotnet build`

---

## 📞 Need Help?

1. Check `GET_TO_TESTABLE_VERSION.md` for detailed steps
2. Check `QUICK_START_GUIDE.md` for setup instructions
3. Check browser console for errors
4. Check application logs in `Presentation/Orca.Web/Logs/`

---

**Total Time: 2-3 hours** ⏱️

**You can do this!** 🚀

