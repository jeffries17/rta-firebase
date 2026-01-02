# Score Calculations - Responsible Travel Adviser Platform

## 📊 Overview

The RTA platform uses a sophisticated scoring system to measure and analyze sustainability performance for travel operators. The system calculates ratings from questionnaire responses and generates analytics for the dashboard.

---

## 🎯 Core Calculation Components

### 1. Individual Question Ratings

**Location:** User questionnaire responses stored in `QuestionResult` table

**Data Structure:**
```csharp
public class QuestionResult
{
    public int QuestionId { get; set; }
    public int QuestionnaireId { get; set; }
    public int UserId { get; set; }
    public int CompanyId { get; set; }
    public int ProductId { get; set; }
    public int Rating { get; set; }  // Individual response (typically 1-5 scale)
    public DateTime CreatedOnUtc { get; set; }
}
```

**Rating Range:** Typically 1-5 (configurable per question)

---

### 2. Rating Sum (Aggregated Scores)

**Location:** `QuestionResultRatingSum` table (performance optimization)

**Purpose:** Pre-aggregated running totals to avoid recalculating from all individual responses

**Data Structure:**
```csharp
public class QuestionResultRatingSum
{
    public int QuestionId { get; set; }
    public int CompanyId { get; set; }
    public int ProductId { get; set; }
    public int RatingSum { get; set; }      // Sum of all ratings
    public int TotalResults { get; set; }    // Count of all responses
}
```

**Calculation Logic:**
```csharp
// File: Presentation/Orca.Web/Controllers/QuestionnaireController.cs
// Lines: 218-241

// When a new rating is submitted (NOT for Company Importance questionnaires):
var ratingSum = _questionnaireService.GetQuestionResultRatingSum(questionId, companyId, productId);

if (ratingSum != null)
{
    // Update existing sum
    ratingSum.RatingSum += rating;      // Add new rating to total
    ratingSum.TotalResults += 1;        // Increment count
    _questionnaireService.UpdateQuestionResultRatingSum(ratingSum);
}
else
{
    // Create new sum record
    ratingSum = new QuestionResultRatingSum
    {
        QuestionId = questionId,
        CompanyId = companyId,
        ProductId = productId,
        RatingSum = rating,
        TotalResults = 1
    };
    _questionnaireService.InsertQuestionResultRatingSum(ratingSum);
}
```

**Average Calculation:**
```
Average Rating = RatingSum / TotalResults
```

**Example:**
- 5 users rated a question: 5, 4, 5, 3, 4
- RatingSum = 21
- TotalResults = 5
- Average = 21 / 5 = 4.2

---

## 📈 Questionnaire Rating Results

### 3. Aggregated Question Results

**Location:** `Libraries/Orca.Services/Questionnaires/QuestionnaireService.cs`
**Method:** `GetQuestionnaireRatingResult()`

**Data Structure:**
```csharp
public class QuestionnaireRatingResult
{
    public int TotalValue { get; set; }      // Sum of all ratings
    public int TotalCount { get; set; }      // Total number of responses
    public decimal TotalRating { get; set; } // Average rating
    
    public IList<QuestionnaireResultValue> QuestionnaireResults { get; set; }
}

public class QuestionnaireResultValue
{
    public int QuestionnaireId { get; set; }
    public int TotalValue { get; set; }
    public int TotalCount { get; set; }
    public decimal TotalRating { get; set; }
    
    public IList<QuestionResultValue> QuestionResults { get; set; }
}

public class QuestionResultValue
{
    public int QuestionId { get; set; }
    public int TotalValue { get; set; }
    public int TotalCount { get; set; }
    public decimal TotalRating { get; set; }
}
```

### Calculation Flow (Lines 508-584)

**Step 1: Fetch Data from Stored Procedure**
```csharp
var questionnaireResults = _dbContext.EntityFromSql<QuestionResult>(
    "GetQuestionResult",
    questionnaireTypeId, companyId, productId, genderId, 
    ageGroupId, countryId, nationalityId, totalRecords
).ToList();
```

**Step 2: Calculate Overall Totals**
```csharp
var totalQuestionnaireResultValue = questionnaireResults.Sum(q => q.Rating);
var totalQuestionnaireResultCount = questionnaireResults.Count;

result.TotalValue = totalQuestionnaireResultValue;
result.TotalCount = totalQuestionnaireResultCount;

if (totalQuestionnaireResultValue > 0 && totalQuestionnaireResultCount > 0)
    result.TotalRating = Convert.ToDecimal(totalQuestionnaireResultValue) 
                       / Convert.ToDecimal(totalQuestionnaireResultCount);
```

**Step 3: Group by Questionnaire**
```csharp
var groupedQuestionnaireResults = questionnaireResults
    .GroupBy(q => q.QuestionnaireId)
    .Select(grp => grp.ToList())
    .ToList();
```

**Step 4: Calculate Questionnaire-Level Scores**
```csharp
foreach (var questionnaireResult in groupedQuestionnaireResults)
{
    var totalQuestionnaireValue = questionnaireResult.Sum(q => q.Rating);
    var totalQuestionnaireCount = questionnaireResult.Count;
    
    var questionnaireResultModel = new QuestionnaireResultValue
    {
        QuestionnaireId = questionnaire?.QuestionnaireId ?? 0,
        TotalValue = totalQuestionnaireValue,
        TotalCount = totalQuestionnaireCount,
        TotalRating = totalQuestionnaireValue / totalQuestionnaireCount
    };
}
```

**Step 5: Calculate Question-Level Scores**
```csharp
var groupedQuestionResults = questionnaireResult
    .GroupBy(q => q.QuestionId)
    .Select(grp => grp.ToList())
    .ToList();

foreach (var questionResult in groupedQuestionResults)
{
    var totalQuestionResultValue = questionResult.Sum(q => q.Rating);
    var totalQuestionResultCount = questionResult.Count;
    
    questionnaireResultModel.QuestionResults.Add(new QuestionResultValue
    {
        QuestionId = question?.QuestionId ?? 0,
        TotalValue = totalQuestionResultValue,
        TotalCount = totalQuestionResultCount,
        TotalRating = totalQuestionResultValue / totalQuestionResultCount
    });
}
```

---

## 📊 Matrix Calculations (IPA - Importance-Performance Analysis)

### 4. Matrix Rating Results

**Location:** `Libraries/Orca.Services/Questionnaires/QuestionnaireService.cs`
**Method:** `GetQuestionnaireRatingResultMatrix()` (Lines 597-673)

**Purpose:** Calculate statistical measures for IPA matrix visualization

**Additional Metrics:**
```csharp
public class QuestionnaireRatingResult
{
    // Standard metrics
    public int TotalValue { get; set; }
    public int TotalCount { get; set; }
    
    // Matrix-specific metrics
    public int MinValue { get; set; }      // Lowest rating
    public int MaxValue { get; set; }      // Highest rating
    public decimal MeanValue { get; set; } // Average (same as TotalRating but clearer)
}
```

**Calculation (Lines 619-627):**
```csharp
result.TotalCount = questionnaireResults.Count;
result.TotalValue = questionnaireResults.Sum(q => q.Rating);

if (result.TotalValue > 0 && result.TotalCount > 0)
{
    result.MeanValue = Convert.ToDecimal(questionnaireResults.Average(q => q.Rating));
    result.MaxValue = questionnaireResults.Max(q => q.Rating);
    result.MinValue = questionnaireResults.Min(q => q.Rating);
}
```

**Question-Level Matrix Metrics (Lines 658-666):**
```csharp
questionnaireResultModel.QuestionResults.Add(new QuestionResultValue
{
    QuestionId = question?.QuestionId ?? 0,
    TotalValue = questionResult.Sum(q => q.Rating),
    MeanValue = Convert.ToDecimal(questionResult.Average(q => q.Rating)),
    MaxValue = questionResult.Max(q => q.Rating),
    MinValue = questionResult.Min(q => q.Rating)
});
```

---

## 🎯 IPA Matrix Dashboard

### 5. Importance-Performance Analysis

**Location:** `Presentation/Orca.Web/Factories/TravelOperatorModelFactory.cs`
**Method:** `PrepareMatrixIpaModel()` (Lines 1390-1450+)

**Purpose:** Compare traveler expectations (importance) vs actual satisfaction (performance)

**Two-Dimensional Analysis:**

#### X-Axis: Performance (Satisfaction)
```csharp
var travelerSatisfactionResult = 
    _questionnaireService.GetQuestionnaireRatingResultMatrix(
        QuestionnaireType.TravelerSatisfactionQuestionnaire,
        companyId: company.Id,
        productId: productId,
        genderId: genderId,
        ageGroupId: ageGroupId,
        countryId: countryId,
        nationalityId: nationalityId
    );

// Store satisfaction metrics
model.ChartDataMeanX.Add(travelerSatisfactionResult.MeanValue);
model.ChartDataMaxX.Add(travelerSatisfactionResult.MaxValue);
model.ChartDataMinX.Add(travelerSatisfactionResult.MinValue);

// Individual question satisfaction scores
foreach (var questionResult in questionnaireResult.QuestionResults)
{
    model.TravelerSatisfactionChartDataPoints.Add(questionResult.MeanValue);
}
```

#### Y-Axis: Importance (Expectation)
```csharp
var travelerExpectationResult = 
    _questionnaireService.GetQuestionnaireRatingResultMatrix(
        QuestionnaireType.TravelerExpectationQuestionnaire,
        companyId: company.Id,
        productId: productId,
        // ... same filters
    );

// Store expectation metrics
model.ChartDataMeanY.Add(travelerExpectationResult.MeanValue);
model.ChartDataMaxY.Add(travelerExpectationResult.MaxValue);
model.ChartDataMinY.Add(travelerExpectationResult.MinValue);

// Individual question expectation scores
foreach (var questionResult in questionnaireResult.QuestionResults)
{
    model.TravelerExpectationChartDataPoints.Add(questionResult.MeanValue);
}
```

**Matrix Point Calculation:**
```csharp
// Each question becomes a point on the IPA matrix
Point = (Satisfaction MeanValue, Expectation MeanValue)

// Example:
// Question: "How important is eco-friendly accommodation?"
// Expectation (Importance): 4.8 (Y-axis)
// Satisfaction (Performance): 3.2 (X-axis)
// Point plotted at: (3.2, 4.8)
```

---

## 📐 IPA Matrix Quadrants

The IPA matrix divides into 4 quadrants based on mean values:

```
High Importance (Y)
         ^
         |
    II   |   I
 Keep up | Concentrate
  Good   |    Here
---------|--------- Mean Satisfaction (X)
         |
   III   |   IV
 Low     | Possible
Priority | Overkill
         |
```

### Quadrant Interpretation:

**Quadrant I (High Importance, Low Performance)**
- **Name:** Concentrate Here
- **Action:** These are critical areas needing immediate improvement
- **Formula:** Satisfaction < Mean AND Expectation > Mean

**Quadrant II (High Importance, High Performance)**
- **Name:** Keep Up the Good Work
- **Action:** Maintain current high performance
- **Formula:** Satisfaction > Mean AND Expectation > Mean

**Quadrant III (Low Importance, Low Performance)**
- **Name:** Low Priority
- **Action:** Less critical, but monitor
- **Formula:** Satisfaction < Mean AND Expectation < Mean

**Quadrant IV (Low Importance, High Performance)**
- **Name:** Possible Overkill
- **Action:** Resources could be reallocated
- **Formula:** Satisfaction > Mean AND Expectation < Mean

---

## 🔍 Filtering Capabilities

All calculations support demographic filtering:

```csharp
public virtual QuestionnaireRatingResult GetQuestionnaireRatingResult(
    QuestionnaireType questionnaireType,
    int companyId = 0,          // Filter by company
    int productId = 0,          // Filter by product/tour
    int genderId = 0,           // Filter by respondent gender
    int ageGroupId = 0,         // Filter by respondent age group
    int countryId = 0,          // Filter by country
    int nationalityId = 0       // Filter by nationality
)
```

**Use Cases:**
- View scores for a specific product
- Compare performance across age groups
- Analyze by traveler nationality
- Gender-based satisfaction analysis

---

## 🗄️ Database Stored Procedure

**Name:** `GetQuestionResult`

**Location:** Referenced but not included in the codebase (should be in database)

**Expected Parameters:**
- `@QuestionnaireTypeId` (int) - Type of questionnaire
- `@CompanyId` (int) - Company filter (0 = all)
- `@ProductId` (int) - Product filter (0 = all)
- `@GenderId` (int) - Gender filter (0 = all)
- `@AgeGroupId` (int) - Age group filter (0 = all)
- `@CountryId` (int) - Country filter (0 = all)
- `@NationalityId` (int) - Nationality filter (0 = all)
- `@TotalRecords` (int OUTPUT) - Returns total count

**Expected Return:** List of `QuestionResult` records matching filters

**⚠️ Note:** This stored procedure needs to be checked in the database. It should exist in the `App_Data/Install/` SQL scripts.

---

## 💡 Practical Examples

### Example 1: Simple Question Average

**Scenario:** Calculate average rating for "Was accommodation eco-friendly?"

**Data:**
- 10 travelers answered
- Ratings: 5, 4, 5, 5, 3, 4, 5, 4, 5, 4

**Calculation:**
```
TotalValue = 5+4+5+5+3+4+5+4+5+4 = 44
TotalCount = 10
TotalRating = 44 / 10 = 4.4
```

**Result:** Average rating is 4.4 out of 5

---

### Example 2: Questionnaire Overall Score

**Scenario:** Calculate overall score for "Traveler Satisfaction Questionnaire"

**Data:**
- Questionnaire has 5 questions
- Each question has 10 responses
- Total responses: 50

**Question Averages:**
1. Accommodation: 4.4
2. Transportation: 3.8
3. Food: 4.6
4. Activities: 4.2
5. Guide: 4.5

**Calculation:**
```
Method 1 (Average of Averages):
Overall = (4.4 + 3.8 + 4.6 + 4.2 + 4.5) / 5 = 4.3

Method 2 (Used in code - Sum all ratings):
TotalValue = Sum of all 50 individual ratings = 215
TotalCount = 50
TotalRating = 215 / 50 = 4.3
```

**Result:** Overall satisfaction score is 4.3 out of 5

---

### Example 3: IPA Matrix Point

**Scenario:** Plot "Sustainable Transportation" on IPA matrix

**Data:**

**Expectation Questionnaire:**
- "How important is sustainable transportation?"
- 20 responses: avg = 4.8

**Satisfaction Questionnaire:**
- "How satisfied were you with sustainable transportation?"
- 15 responses: avg = 3.2

**IPA Coordinates:**
```
X (Performance/Satisfaction) = 3.2
Y (Importance/Expectation) = 4.8

Point plotted at: (3.2, 4.8)
```

**Interpretation:**
- High importance (4.8 > overall mean)
- Low performance (3.2 < overall mean)
- **Quadrant I - CONCENTRATE HERE**
- **Action:** This is a critical area needing immediate improvement

---

## 🔧 Key Files Reference

### Domain Models
- `Libraries/Orca.Core/Domain/Questionnaires/QuestionResult.cs`
- `Libraries/Orca.Core/Domain/Questionnaires/QuestionResultRatingSum.cs`
- `Libraries/Orca.Core/Domain/Questionnaires/QuestionnaireRatingResult.cs`

### Service Layer (Calculation Logic)
- `Libraries/Orca.Services/Questionnaires/QuestionnaireService.cs`
  - Line 508-584: `GetQuestionnaireRatingResult()`
  - Line 597-673: `GetQuestionnaireRatingResultMatrix()`
  - Line 729-741: `GetQuestionResultRatingSum()`

### Controller (Rating Submission)
- `Presentation/Orca.Web/Controllers/QuestionnaireController.cs`
  - Line 128-307: `QuestionsSubmit()` - Handles rating submission
  - Line 218-241: Rating sum update logic

### Factory (Dashboard Preparation)
- `Presentation/Orca.Web/Factories/TravelOperatorModelFactory.cs`
  - Line 1390-1450+: `PrepareMatrixIpaModel()` - IPA matrix preparation
- `Presentation/Orca.Web/Areas/TravelOperator/Factories/CompanyModelFactory.cs`
  - Line 543-566: Alternative IPA preparation

### View Models
- `Presentation/Orca.Web/Models/TravelOperator/DashboardMatrixIpaModel.cs`
- `Presentation/Orca.Web/Areas/TravelOperator/Models/Company/MatrixIpaModel.cs`

---

## ⚠️ Important Notes

### 1. Company Importance Questionnaires Exception

```csharp
// Line 218 in QuestionnaireController.cs
if (questionnaire.QuestionnaireTypeId != (int)QuestionnaireType.CompanyImportanceQuestionnaire)
{
    // Rating sum is NOT updated for Company Importance questionnaires
    // These are handled differently
}
```

**Reason:** Company importance ratings may need different aggregation logic or may be reference data rather than accumulated scores.

### 2. Stored Procedure Dependency

The calculation heavily relies on `GetQuestionResult` stored procedure. **You must verify this exists in your database.**

**Check:**
```sql
-- Run in SQL Server Management Studio
SELECT name FROM sys.procedures WHERE name = 'GetQuestionResult'
```

If missing, check:
- `Presentation/Orca.Web/App_Data/Install/SqlServer.StoredProcedures.sql`

### 3. Performance Considerations

**QuestionResultRatingSum Table:**
- Acts as a cache for performance
- Reduces need to sum all individual ratings every time
- Updated incrementally when new ratings are submitted

**Trade-off:**
- Faster reads (dashboard loading)
- Slightly slower writes (rating submission)
- Potential for inconsistency if updates fail

### 4. Demographic Filtering

All calculations support filtering by demographics. This enables:
- Age-based analysis
- Gender-based analysis
- Nationality/country-based analysis
- Product-specific analysis

**Use case:** "Show me satisfaction scores for 25-34 year olds from USA for our Amazon tour"

---

## 🔒 Data Integrity

### Potential Issues to Check

1. **Orphaned Rating Sums:**
   - If a QuestionResult is deleted, RatingSum may not be updated
   - **Solution:** Implement cascade updates or periodic reconciliation

2. **Division by Zero:**
   - Code checks `if (totalQuestionnaireResultCount > 0)` before dividing
   - **Safe:** Protected against zero-count scenarios

3. **Stored Procedure Missing:**
   - If `GetQuestionResult` doesn't exist, calculations will fail
   - **Solution:** Verify in database or implement in C# as fallback

4. **Type Mismatches:**
   - Stored procedure must return `QuestionResult` entity structure
   - **Solution:** Ensure schema matches entity definition

---

## 📚 Summary

The RTA scoring system uses a **hierarchical calculation approach**:

1. **Individual Ratings** → Stored per user response (1-5 scale)
2. **Question Aggregates** → Average across all users for a question
3. **Questionnaire Aggregates** → Average across all questions in a questionnaire
4. **Overall Scores** → Average across all questionnaires
5. **IPA Matrix** → Two-dimensional comparison of Importance vs Performance

**Key Formula:**
```
Average Rating = Sum of All Ratings / Count of Ratings
```

**Dashboard Calculations:**
- Use stored procedure for performance
- Support demographic filtering
- Provide Min/Max/Mean for statistical analysis
- Enable IPA matrix visualization for strategic insights

---

**For Questions or Issues:**
- Check the key files listed above
- Verify stored procedure exists in database
- Review controller logic for rating submission
- Test with sample data to understand flow

---

*Document Created: October 14, 2025*
*Codebase Version: 1.00 (.NET Core 2.2)*

