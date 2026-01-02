# Firebase Integration Guide - RTA Platform

## ✅ Firebase Has Been Added!

I've integrated Firebase into your ASP.NET Core application. Here's what was done and how to use it.

---

## 🔒 **CRITICAL SECURITY WARNING**

**Your Firebase API keys are now in the codebase.** While Firebase API keys are designed to be public, you **MUST** secure them:

### Immediate Actions Required:

1. **Restrict API Keys in Firebase Console:**
   - Go to: https://console.firebase.google.com/project/be-rta/settings/general
   - Click on your web app
   - Under "API keys", click "Restrict key"
   - Add HTTP referrer restrictions:
     - `https://yourdomain.com/*`
     - `https://*.yourdomain.com/*`
     - For development: `http://localhost:*`

2. **Enable App Check (Recommended):**
   - Go to: Firebase Console > App Check
   - Enable App Check for your web app
   - This adds an extra layer of security

3. **Set Up Firebase Security Rules:**
   - If using Firestore/Realtime Database: Set up proper security rules
   - If using Storage: Configure storage rules

---

## 📁 Files Added/Modified

### 1. Firebase Scripts Partial View
**File:** `Presentation/Orca.Web/Views/Shared/_FirebaseScripts.cshtml`

**What it does:**
- Loads Firebase SDK from CDN
- Initializes Firebase with your config
- Sets up Analytics (disabled in development)
- Tracks page views automatically

### 2. Layout Integration
**File:** `Presentation/Orca.Web/Views/Shared/_Layout.Header.cshtml`

**What changed:**
- Added `@await Html.PartialAsync("_FirebaseScripts")` to include Firebase

### 3. Alternative Config Files (Optional)
**Files:**
- `wwwroot/js/firebase-config.js` (ES6 modules version - if you add a build system)
- `wwwroot/js/firebase-config-legacy.js` (Legacy version)

---

## 🚀 How It Works

### Current Implementation (CDN-based)

Firebase is loaded via CDN and initialized on every page:

```javascript
// Firebase is automatically initialized
firebase.initializeApp(firebaseConfig);

// Analytics tracks page views automatically
analytics.logEvent('page_view', {
    page_path: window.location.pathname,
    page_title: document.title
});
```

### Development vs Production

- **Development:** Analytics disabled (won't pollute your data)
- **Production:** Analytics enabled automatically

---

## 📊 What You Can Do Now

### 1. Firebase Analytics (Already Set Up)

**Automatic Tracking:**
- Page views (already implemented)
- User engagement
- Custom events

**Add Custom Events:**
```javascript
// Track questionnaire submission
firebase.analytics().logEvent('questionnaire_submitted', {
    questionnaire_type: 'TravelerExpectation',
    company_id: 123
});

// Track tour view
firebase.analytics().logEvent('tour_viewed', {
    tour_id: 456,
    tour_name: 'Amazon Adventure'
});
```

### 2. Firebase Authentication (Optional)

If you want to use Firebase Auth alongside your existing auth:

```javascript
// Import Firebase Auth
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth-compat.js"></script>

// Use in your code
firebase.auth().signInWithEmailAndPassword(email, password)
    .then((userCredential) => {
        // Signed in
    });
```

### 3. Firebase Firestore (Optional)

For real-time data or additional storage:

```javascript
// Import Firestore
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

// Use in your code
const db = firebase.firestore();
db.collection('tours').doc('tour123').get()
    .then((doc) => {
        console.log(doc.data());
    });
```

### 4. Firebase Storage (Optional)

For file uploads:

```javascript
// Import Storage
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-storage-compat.js"></script>

// Upload file
const storage = firebase.storage();
const storageRef = storage.ref();
const fileRef = storageRef.child('images/tour123.jpg');
fileRef.put(file);
```

---

## 🎯 Recommended Next Steps

### For MVP Launch:

1. **Analytics Only (Recommended for Start)**
   - ✅ Already set up
   - Track user behavior
   - Monitor key metrics
   - No additional setup needed

2. **Add Custom Events** (1-2 hours)
   - Track questionnaire submissions
   - Track tour views
   - Track company registrations
   - Track user sign-ups

### For Future Enhancements:

3. **Firebase Authentication** (Optional)
   - Use if you want social login (Google, Facebook)
   - Can complement existing auth system
   - **Note:** Your existing auth works fine, this is optional

4. **Firebase Firestore** (Optional)
   - Real-time data sync
   - Offline support
   - **Note:** You already have SQL Server, this is optional

5. **Firebase Cloud Functions** (Optional)
   - Serverless backend functions
   - Can call from your .NET backend
   - **Note:** You already have .NET backend, this is optional

---

## 📝 Adding Custom Events

### Example: Track Questionnaire Submission

**File:** `Presentation/Orca.Web/Views/Questionnaire/QuestionsDetail.cshtml`

Add after form submission:
```javascript
<script>
    function trackQuestionnaireSubmission(questionnaireId, companyId) {
        if (typeof firebase !== 'undefined' && firebase.analytics) {
            firebase.analytics().logEvent('questionnaire_submitted', {
                questionnaire_id: questionnaireId,
                company_id: companyId,
                timestamp: new Date().toISOString()
            });
        }
    }
</script>
```

### Example: Track Tour View

**File:** `Presentation/Orca.Web/Views/Category/CategoryDetail.cshtml`

Add on page load:
```javascript
<script>
    $(document).ready(function() {
        if (typeof firebase !== 'undefined' && firebase.analytics) {
            firebase.analytics().logEvent('tour_viewed', {
                tour_id: @Model.ProductId,
                tour_name: '@Model.Name',
                company_id: @Model.CompanyId
            });
        }
    });
</script>
```

---

## 🔧 Configuration Options

### Disable Analytics Completely

Edit `_FirebaseScripts.cshtml`:
```csharp
@* Comment out or remove the analytics initialization *@
@* const analytics = firebase.analytics(); *@
```

### Change Analytics Settings

Edit `_FirebaseScripts.cshtml`:
```javascript
// Disable automatic page tracking
// analytics.logEvent('page_view', ...); // Comment this out

// Or customize it
analytics.logEvent('page_view', {
    page_path: window.location.pathname,
    page_title: document.title,
    custom_param: 'value'
});
```

### Use Environment-Specific Config

For different configs per environment:

**File:** `_FirebaseScripts.cshtml`
```csharp
@{
    var firebaseConfig = new
    {
        apiKey = Environment.GetEnvironmentVariable("FIREBASE_API_KEY") ?? "AIzaSyAoCr4uwJkhn_UtM7EIz_EyupkDmERuNHA",
        authDomain = Environment.GetEnvironmentVariable("FIREBASE_AUTH_DOMAIN") ?? "be-rta.firebaseapp.com",
        // ... other config
    };
}
```

---

## 🐛 Troubleshooting

### Firebase Not Loading

**Check:**
1. Internet connection (CDN requires internet)
2. Browser console for errors
3. Ad blockers (may block Firebase)
4. CSP headers (may block external scripts)

**Solution:**
```html
<!-- Add to web.config if CSP is blocking -->
<system.webServer>
    <httpProtocol>
        <customHeaders>
            <add name="Content-Security-Policy" value="script-src 'self' 'unsafe-inline' https://www.gstatic.com;" />
        </customHeaders>
    </httpProtocol>
</system.webServer>
```

### Analytics Not Working

**Check:**
1. Not in development mode (analytics disabled)
2. Firebase Console shows data (may take 24-48 hours)
3. Browser console for errors
4. Ad blockers

### Performance Concerns

**If Firebase slows down your site:**
1. Load Firebase asynchronously
2. Use lazy loading
3. Load only needed Firebase modules

**Example (Async Loading):**
```html
<script async src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
```

---

## 📊 Viewing Analytics Data

1. **Go to Firebase Console:**
   - https://console.firebase.google.com/project/be-rta/analytics

2. **View Reports:**
   - Events: See custom events
   - User engagement: See user behavior
   - Conversions: Track goals

3. **Set Up Conversions:**
   - Mark important events as conversions
   - Track business goals

---

## 💰 Firebase Pricing

### Free Tier (Spark Plan)
- ✅ Analytics: Unlimited
- ✅ Authentication: 50,000 MAU
- ✅ Firestore: 1 GB storage, 50K reads/day
- ✅ Storage: 5 GB storage
- ✅ Functions: 2 million invocations/month

**For MVP:** Free tier is likely sufficient!

### Paid Tier (Blaze Plan)
- Pay as you go
- Scales automatically
- Only pay for what you use

---

## ✅ Integration Checklist

- [x] Firebase SDK added to layout
- [x] Configuration added
- [x] Analytics initialized
- [x] Page view tracking enabled
- [ ] **API keys restricted in Firebase Console** ⚠️ DO THIS
- [ ] **App Check enabled** (recommended)
- [ ] Custom events added (optional)
- [ ] Firebase Console access verified
- [ ] Analytics data visible (may take 24-48 hours)

---

## 🎯 Summary

**What's Done:**
- ✅ Firebase integrated into your ASP.NET Core app
- ✅ Analytics set up and tracking page views
- ✅ Development/production detection working
- ✅ Ready to add custom events

**What You Need to Do:**
1. ⚠️ **Restrict API keys in Firebase Console** (CRITICAL)
2. Enable App Check (recommended)
3. Add custom events for key actions (optional)
4. Monitor analytics in Firebase Console

**Next Steps:**
- Start with Analytics only (already working)
- Add custom events as needed
- Consider other Firebase services if needed
- Monitor usage to stay within free tier

---

**Firebase is now integrated and ready to use!** 🚀

Just remember to secure your API keys in the Firebase Console.

---

*Integration Guide Created: October 14, 2025*

