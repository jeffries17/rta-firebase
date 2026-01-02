# 16 Sustainability Qualifiers - RTA Platform

## 📊 Overview

The RTA platform uses **16 sustainability qualifiers** (also called "indicators" or "criteria") to assess and measure the sustainability performance of travel operators. These qualifiers are displayed in various dashboard visualizations.

---

## 🎯 Source of the 16 Qualifiers

### Questionnaire Type
**Source:** `TravelerExpectationQuestionnaire`

The 16 qualifiers come from questions in the **Traveler Expectation Questionnaire**. This questionnaire asks travelers to rate how important various sustainability factors are to them.

### Code Location

**File:** `Presentation/Orca.Web/Factories/TravelOperatorModelFactory.cs`  
**Lines:** 1113-1147

```csharp
//Cas des 16 questions (Case of the 16 questions)
var travelerExpectationResult16 =
    _questionnaireService.GetQuestionnaireRatingResult(
        QuestionnaireType.TravelerExpectationQuestionnaire, 
        companyId: company.Id, 
        productId: productId,
        genderId: genderId, 
        ageGroupId: ageGroupId, 
        countryId: countryId, 
        nationalityId: nationalityId);

var questionnaireCount16 = 0;
foreach (var questionnaireResult in travelerExpectationResult16.QuestionnaireResults)
{
    foreach (var questionResult in questionnaireResult.QuestionResults)
    {
        // Color coding by questionnaire group
        switch (questionnaireCount16)
        {
            case 0:
                model.ChartColours16.Add("#099EC8"); // Blue
                break;
            case 1:
                model.ChartColours16.Add("#84BC41"); // Green
                break;
            case 2:
                model.ChartColours16.Add("#F9C416"); // Yellow
                break;
            default:
                model.ChartColours16.Add("#9CD8E9"); // Light blue
                break;
        }

        var question = _questionnaireService.GetQuestionById(questionResult.QuestionId);
        
        // Add the question title as the qualifier label
        model.ChartLabels16.Add(_localizationService.GetLocalized(question, x => x.Title));
        
        // Add the average rating for this qualifier
        model.ChartDataPoints16.Add(questionResult.TotalRating);
        model.HasData16 = true;
    }
    
    questionnaireCount16++;
}
```

**Also in:** `Presentation/Orca.Web/Areas/TravelOperator/Factories/CompanyModelFactory.cs`  
**Lines:** 404-439 (same logic)

---

## 📋 Data Structure

### Model Properties

**File:** `Presentation/Orca.Web/Models/TravelOperator/DashboardCustomersExpectationModel.cs`

```csharp
public class DashboardCustomersExpectationModel
{
    public bool HasData16 { get; set; }  // Flag indicating if 16-qualifier data exists
    
    public IList<string> ChartColours16 { get; set; }      // Colors for visualization
    public IList<string> ChartLabels16 { get; set; }       // Names of the 16 qualifiers
    public IList<decimal> ChartDataPoints16 { get; set; } // Average ratings (1-5 scale)
}
```

### How It's Populated

1. **Questions are retrieved** from `TravelerExpectationQuestionnaire` type
2. **Each question becomes a qualifier** (up to 16 total)
3. **Question titles** are stored in `ChartLabels16` (localized/multi-language)
4. **Average ratings** are stored in `ChartDataPoints16`
5. **Colors** are assigned based on which questionnaire group they belong to

---

## 🎨 Color Coding

The 16 qualifiers are grouped into **4 color categories**:

| Group | Color | Hex Code | Meaning |
|-------|-------|----------|---------|
| Group 0 | Blue | `#099EC8` | First questionnaire group |
| Group 1 | Green | `#84BC41` | Second questionnaire group |
| Group 2 | Yellow | `#F9C416` | Third questionnaire group |
| Group 3+ | Light Blue | `#9CD8E9` | Additional questionnaire groups |

**Note:** The grouping is based on which questionnaire the question belongs to, not necessarily the qualifier itself.

---

## 📊 Where They're Displayed

### 1. Customer Expectations Dashboard

**File:** `Presentation/Orca.Web/Views/Operator/CustomersExpectation.cshtml`

**Features:**
- Polar area chart (`polar-chart16`)
- Table showing all 16 qualifiers with values and colors
- Dynamic sorting table

**Code:**
```cshtml
@for (int i = 0; i < 16; i++)
{
    <tr>
        <td>@Model.ChartLabels16[i]</td>
        <td>@Model.ChartDataPoints16[i]</td>
        <td style="background-color:@Model.ChartColours16[i];"></td>
    </tr>
}
```

### 2. Perception of Sustainability Dashboard

**File:** `Presentation/Orca.Web/Views/Operator/PerceptionOfSustainability.cshtml`

**Features:**
- Bar chart
- Table with all 16 qualifiers

**Code:**
```cshtml
@for (int i = 0; i < 16; i++)
{
    <tr>
        <td>@Model.ChartLabels[i]</td>
        <td>@Model.ChartDataPoints[i]</td>
        <td style="background-color:@Model.ChartColours[i];"></td>
    </tr>
}
```

### 3. IPA Matrix (Areas of Improvement)

**File:** `Presentation/Orca.Web/Views/Operator/Areas41.cshtml`

**Features:**
- Bubble chart with 16 points
- Each point represents one qualifier
- X-axis: Satisfaction (Performance)
- Y-axis: Expectation (Importance)

**Code:**
```javascript
for (let i = 0; i < 16; i++) {
    ind = Math.floor(i / 4);  // Group into 4 sectors
    elt = {
        label: sectors[ind],
        backgroundColor: colors[ind],
        title: titles[i],  // Qualifier name
        data: [{ x: coords[i][0], y: coords[i][1], r: 10 }]
    };
    datas.push(elt);
}
```

---

## 🔍 How to Find the Actual Qualifier Names

The qualifier names are **stored in the database** as question titles. They're retrieved dynamically and localized.

### Database Query

To see the actual 16 qualifiers, query the database:

```sql
-- Find questions in Traveler Expectation Questionnaires
SELECT 
    q.Id AS QuestionId,
    q.Title,
    q.QuestionText,
    qn.Id AS QuestionnaireId,
    qn.Title AS QuestionnaireTitle,
    qn.QuestionnaireTypeId
FROM Question q
INNER JOIN Questionnaire qn ON q.QuestionnaireId = qn.Id
WHERE qn.QuestionnaireTypeId = 5  -- TravelerExpectationQuestionnaire
  AND q.Deleted = 0
  AND qn.Deleted = 0
ORDER BY qn.Id, q.DisplayOrder
```

**Expected Result:** Up to 16 questions across one or more questionnaires of type `TravelerExpectationQuestionnaire`

### Through Admin Panel

1. Login to Admin: `/Admin`
2. Navigate to: **Questionnaires** → **List**
3. Filter by: **Traveler Expectation Questionnaire**
4. View questions in each questionnaire
5. The questions are your 16 qualifiers

### Through Code

```csharp
// Get all Traveler Expectation Questionnaires
var questionnaires = _questionnaireService
    .GetAllQuestionnaires()
    .Where(q => q.QuestionnaireType == QuestionnaireType.TravelerExpectationQuestionnaire);

// Get all questions from these questionnaires
var allQuestions = new List<Question>();
foreach (var questionnaire in questionnaires)
{
    var questions = _questionnaireService.GetQuestionsByQuestionnaireId(questionnaire.Id);
    allQuestions.AddRange(questions);
}

// Display qualifier names
foreach (var question in allQuestions.Take(16))
{
    var title = _localizationService.GetLocalized(question, x => x.Title);
    Console.WriteLine($"Qualifier: {title}");
}
```

---

## 📐 Typical Sustainability Qualifiers (Expected)

Based on sustainable travel industry standards, the 16 qualifiers likely include categories such as:

### Environmental Qualifiers
1. **Carbon Footprint** - Minimizing greenhouse gas emissions
2. **Waste Management** - Reducing and properly disposing of waste
3. **Water Conservation** - Efficient water use
4. **Energy Efficiency** - Renewable energy and conservation
5. **Biodiversity Protection** - Protecting local ecosystems
6. **Wildlife Conservation** - Ethical wildlife interactions

### Social Qualifiers
7. **Local Community Support** - Economic benefits to local communities
8. **Cultural Respect** - Respecting local cultures and traditions
9. **Fair Employment** - Fair wages and working conditions
10. **Local Sourcing** - Using local products and services
11. **Community Engagement** - Involving local communities
12. **Cultural Preservation** - Supporting cultural heritage

### Economic Qualifiers
13. **Local Economic Impact** - Money staying in local economy
14. **Fair Trade Practices** - Ethical business practices
15. **Transparent Pricing** - Clear and fair pricing
16. **Long-term Sustainability** - Sustainable business model

**⚠️ Note:** These are **typical examples**. The actual qualifiers are defined in your database and may differ.

---

## 🎯 How They're Used

### 1. Customer Expectations Analysis

**Purpose:** Understand what travelers value most in sustainable travel

**Display:**
- Polar area chart showing relative importance
- Table with exact values
- Color-coded by category

**Interpretation:**
- Higher ratings = More important to travelers
- Lower ratings = Less important to travelers

### 2. IPA Matrix (Importance-Performance Analysis)

**Purpose:** Compare what travelers expect vs. what they experience

**X-Axis (Performance):** Satisfaction scores from `TravelerSatisfactionQuestionnaire`  
**Y-Axis (Importance):** Expectation scores from `TravelerExpectationQuestionnaire`

**Quadrants:**
- **Quadrant I (High Importance, Low Performance):** Concentrate Here - Critical improvements needed
- **Quadrant II (High Importance, High Performance):** Keep Up Good Work - Maintain excellence
- **Quadrant III (Low Importance, Low Performance):** Low Priority - Monitor but not urgent
- **Quadrant IV (Low Importance, High Performance):** Possible Overkill - Resources could be reallocated

### 3. Perception Analysis

**Purpose:** Track how travelers perceive sustainability efforts

**Display:**
- Bar charts
- Trend analysis
- Comparison across demographics

---

## 🔧 Technical Details

### Data Flow

```
Database (Question Table)
    ↓
QuestionnaireService.GetQuestionnaireRatingResult()
    ↓
TravelOperatorModelFactory.PrepareCustomersExpectationModel()
    ↓
ChartLabels16, ChartDataPoints16, ChartColours16
    ↓
View (CustomersExpectation.cshtml)
    ↓
Charts & Tables (Chart.js, HTML)
```

### Calculation Method

For each qualifier:
1. **Get all responses** for that question from travelers
2. **Filter** by company, product, demographics (optional)
3. **Calculate average** rating (1-5 scale)
4. **Store** in `ChartDataPoints16[i]`

**Formula:**
```
Average Rating = Sum of All Ratings / Total Number of Responses
```

### Localization

Qualifier names are **multi-language**:
- Stored in `LocalizedProperty` table
- Retrieved via `_localizationService.GetLocalized(question, x => x.Title)`
- Supports multiple languages per qualifier

---

## ⚠️ Important Notes

### 1. Not Always Exactly 16

The code assumes **up to 16 qualifiers**, but:
- If fewer than 16 questions exist, only those are displayed
- If more than 16 questions exist, only the first 16 are used
- The loop `for (int i = 0; i < 16; i++)` will fail if fewer than 16 exist

**Potential Bug:** If there are fewer than 16 qualifiers, the view will throw an `IndexOutOfRangeException`.

**Fix Needed:**
```cshtml
@for (int i = 0; i < Model.ChartLabels16.Count && i < 16; i++)
{
    // ...
}
```

### 2. Questionnaire Grouping

The 16 qualifiers may come from **multiple questionnaires** of type `TravelerExpectationQuestionnaire`:
- Each questionnaire can have multiple questions
- Questions are grouped by their parent questionnaire
- Colors are assigned per questionnaire group

### 3. Dynamic Content

The qualifiers are **not hardcoded**:
- They're defined in the database
- Admin can add/edit/delete questions
- Changes reflect immediately in dashboards

---

## 📊 Example Data Structure

### What You'd See in the Database

```
Questionnaire: "Environmental Sustainability Expectations"
├── Question 1: "How important is carbon footprint reduction?"
├── Question 2: "How important is waste management?"
├── Question 3: "How important is water conservation?"
└── Question 4: "How important is energy efficiency?"

Questionnaire: "Social Sustainability Expectations"
├── Question 5: "How important is local community support?"
├── Question 6: "How important is cultural respect?"
└── ...

Questionnaire: "Economic Sustainability Expectations"
└── ...
```

### What Gets Displayed

```
ChartLabels16 = [
    "Carbon Footprint Reduction",
    "Waste Management",
    "Water Conservation",
    "Energy Efficiency",
    "Local Community Support",
    "Cultural Respect",
    ...
]

ChartDataPoints16 = [
    4.8,  // Average importance rating
    4.6,
    4.2,
    4.5,
    4.9,
    4.7,
    ...
]

ChartColours16 = [
    "#099EC8",  // Blue (Group 0)
    "#099EC8",  // Blue (Group 0)
    "#099EC8",  // Blue (Group 0)
    "#099EC8",  // Blue (Group 0)
    "#84BC41",  // Green (Group 1)
    "#84BC41",  // Green (Group 1)
    ...
]
```

---

## 🔍 Finding the Qualifiers in Your Database

### Step 1: Connect to Database

```sql
USE SustainableTravelDb;
GO
```

### Step 2: Query Questions

```sql
-- Get all 16 qualifiers
SELECT 
    ROW_NUMBER() OVER (ORDER BY qn.Id, q.DisplayOrder) AS QualifierNumber,
    q.Id AS QuestionId,
    q.Title AS QualifierName,
    q.QuestionText AS FullQuestionText,
    qn.Title AS QuestionnaireName,
    q.DisplayOrder
FROM Question q
INNER JOIN Questionnaire qn ON q.QuestionnaireId = qn.Id
WHERE qn.QuestionnaireTypeId = 5  -- TravelerExpectationQuestionnaire
  AND q.Deleted = 0
  AND qn.Deleted = 0
ORDER BY qn.Id, q.DisplayOrder
```

### Step 3: Check Localized Versions

```sql
-- Get localized qualifier names (if multi-language)
SELECT 
    q.Id AS QuestionId,
    lp.LocaleKey,
    lp.LocaleValue AS LocalizedQualifierName,
    l.Name AS LanguageName
FROM Question q
INNER JOIN LocalizedProperty lp ON lp.EntityId = q.Id AND lp.LocaleKey = 'Title'
INNER JOIN Language l ON l.Id = lp.LanguageId
INNER JOIN Questionnaire qn ON q.QuestionnaireId = qn.Id
WHERE qn.QuestionnaireTypeId = 5
  AND q.Deleted = 0
  AND qn.Deleted = 0
ORDER BY q.Id, l.DisplayOrder
```

---

## 📝 Summary

### Key Points

1. ✅ **16 qualifiers exist** - They're questions from `TravelerExpectationQuestionnaire`
2. ✅ **Stored in database** - Not hardcoded, fully configurable
3. ✅ **Multi-language** - Supports localization
4. ✅ **Used in dashboards** - Customer expectations, IPA matrix, perception analysis
5. ✅ **Color-coded** - Grouped by questionnaire (4 color groups)
6. ✅ **Dynamic** - Admin can modify through questionnaire management

### To See Your Actual Qualifiers

1. **Check Admin Panel:** Questionnaires → Traveler Expectation Questionnaire → View Questions
2. **Query Database:** Use SQL queries above
3. **Check Dashboard:** View Customer Expectations page (shows all 16)

### Files to Review

- `Presentation/Orca.Web/Factories/TravelOperatorModelFactory.cs` (Lines 1113-1147)
- `Presentation/Orca.Web/Views/Operator/CustomersExpectation.cshtml` (Lines 170-183)
- `Presentation/Orca.Web/Views/Operator/PerceptionOfSustainability.cshtml` (Lines 113-126)
- `Presentation/Orca.Web/Views/Operator/Areas41.cshtml` (Lines 227-236)

---

**Document Created:** October 14, 2025  
**Codebase Version:** 1.00 (.NET Core 2.2)

