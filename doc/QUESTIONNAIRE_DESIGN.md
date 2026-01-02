# Questionnaire System Design

## 📋 Overview

Based on the existing .NET system, we need to design a questionnaire system that supports the 16 sustainability qualifiers through different questionnaire types.

## 🎯 Questionnaire Types

### 1. Traveler Expectation Questionnaire
**Purpose:** Measures how IMPORTANT each sustainability factor is to travelers
**Rating Scale:** 1-5 (1 = Not Important, 5 = Very Important)
**Usage:** Forms the Y-axis for IPA Matrix analysis

### 2. Traveler Satisfaction Questionnaire  
**Purpose:** Measures how SATISFIED travelers are with each sustainability factor
**Rating Scale:** 1-5 (1 = Very Dissatisfied, 5 = Very Satisfied)
**Usage:** Forms the X-axis for IPA Matrix analysis

### 3. Importance Questionnaire
**Purpose:** Additional importance ratings for specific analysis
**Rating Scale:** 1-5 (1 = Not Important, 5 = Very Important)

### 4. Guest Questionnaire
**Purpose:** General feedback from guests/visitors
**Rating Scale:** 1-5 (1 = Poor, 5 = Excellent)

## 📊 16 Sustainability Qualifiers

Based on industry standards and the existing system, here are the 16 sustainability qualifiers organized by category:

### Environmental Qualifiers (Group 0 - Blue #099EC8)
1. **Carbon Footprint Reduction** - Minimizing greenhouse gas emissions
2. **Waste Management** - Reducing and properly managing waste
3. **Water Conservation** - Efficient use and conservation of water resources
4. **Energy Efficiency** - Use of renewable energy and conservation practices

### Social Qualifiers (Group 1 - Green #84BC41)
5. **Local Community Support** - Economic benefits to local communities
6. **Cultural Respect** - Respecting local cultures and traditions
7. **Fair Employment** - Fair wages and working conditions for employees
8. **Local Sourcing** - Using local products and services

### Economic Qualifiers (Group 2 - Yellow #F9C416)
9. **Local Economic Impact** - Money staying in the local economy
10. **Fair Trade Practices** - Ethical and transparent business practices
11. **Transparent Pricing** - Clear and fair pricing policies
12. **Sustainable Business Model** - Long-term economic sustainability

### Wildlife & Conservation Qualifiers (Group 3 - Light Blue #9CD8E9)
13. **Wildlife Conservation** - Ethical wildlife interactions and protection
14. **Biodiversity Protection** - Protecting local ecosystems and habitats
15. **Cultural Heritage Preservation** - Supporting cultural heritage sites
16. **Community Engagement** - Active involvement with local communities

## 🏗️ Data Structure Design

### Questionnaires Collection
```javascript
questionnaires/{questionnaireId} = {
  title: string,                    // "Environmental Sustainability Expectations"
  type: string,                     // "expectation" | "satisfaction" | "importance" | "guest"
  active: boolean,                  // Whether questionnaire is active
  order: number,                    // Display order
  category: string,                 // "environmental" | "social" | "economic" | "conservation"
  createdAt: timestamp,
  updatedAt: timestamp
}
```

### Questions Subcollection
```javascript
questionnaires/{questionnaireId}/questions/{questionId} = {
  title: string,                    // "Carbon Footprint Reduction"
  text: string,                     // "How important is minimizing carbon footprint?"
  qualifierId: number,              // 1-16 (maps to the 16 qualifiers)
  category: string,                 // "environmental" | "social" | "economic" | "conservation"  
  ratingScale: number,              // 5 (for 1-5 scale)
  order: number,                    // Display order within questionnaire
  required: boolean,                // Whether question is required
  createdAt: timestamp
}
```

### Responses Collection
```javascript
responses/{responseId} = {
  userId: string,                   // User who submitted response
  questionId: string,               // Question being answered
  questionnaireId: string,          // Parent questionnaire
  companyId: string,                // Company being rated
  productId: string,                // Specific product/tour being rated
  qualifierId: number,              // 1-16 (for aggregation)
  rating: number,                   // 1-5 rating value
  questionType: string,             // "expectation" | "satisfaction" | "importance" | "guest"
  demographics: {                   // User demographics for filtering
    age: string,                    // "18-25" | "26-35" | "36-50" | "50+"
    gender: string,                 // "male" | "female" | "other"
    nationality: string,            // Country code
    countryVisited: string          // Destination country
  },
  submittedAt: timestamp,
  ipAddress: string,                // For fraud detection
  sessionId: string                 // For grouping responses
}
```

### Aggregated Scores Collection
```javascript
aggregatedScores/{scoreId} = {
  qualifierId: number,              // 1-16
  questionId: string,               // Original question
  companyId: string,                // Company being scored
  productId: string,                // Product being scored (optional)
  questionType: string,             // "expectation" | "satisfaction" | "importance" | "guest"
  
  // Aggregated data
  totalRating: number,              // Sum of all ratings
  totalCount: number,               // Number of responses
  avgRating: number,                // Average rating (totalRating / totalCount)
  
  // Statistical measures
  minRating: number,                // Minimum rating received
  maxRating: number,                // Maximum rating received
  standardDeviation: number,        // Standard deviation
  
  // Demographics breakdown (for filtering)
  demographics: {
    age: { [ageGroup]: { total: number, avg: number } },
    gender: { [gender]: { total: number, avg: number } },
    nationality: { [country]: { total: number, avg: number } }
  },
  
  // Metadata
  lastUpdated: timestamp,
  lastResponseId: string            // ID of most recent response
}
```

## 🎨 Color Coding System

| Group | Category | Color | Hex Code | Qualifiers |
|-------|----------|-------|-----------|------------|
| 0 | Environmental | Blue | #099EC8 | 1-4 |
| 1 | Social | Green | #84BC41 | 5-8 |
| 2 | Economic | Yellow | #F9C416 | 9-12 |
| 3 | Conservation | Light Blue | #9CD8E9 | 13-16 |

## 🔄 Question Flow Design

### Multi-Step Questionnaire Flow
1. **Introduction Page** - Explain purpose and time required
2. **Demographics Collection** - Age, gender, nationality, destination
3. **Question Groups** - 4 groups of 4 questions each
4. **Progress Tracking** - Show progress bar (e.g., "Step 2 of 5")
5. **Review & Submit** - Allow review before final submission
6. **Thank You Page** - Confirmation and next steps

### Question Types Support
```javascript
questionTypes = {
  "rating": {
    scale: 5,                       // 1-5 stars
    labels: ["Poor", "Fair", "Good", "Very Good", "Excellent"]
  },
  "importance": {
    scale: 5,                       // 1-5 importance
    labels: ["Not Important", "Slightly Important", "Moderately Important", "Important", "Very Important"]
  },
  "satisfaction": {
    scale: 5,                       // 1-5 satisfaction
    labels: ["Very Dissatisfied", "Dissatisfied", "Neutral", "Satisfied", "Very Satisfied"]
  },
  "multipleChoice": {
    options: ["Option 1", "Option 2", "Option 3"]
  },
  "text": {
    maxLength: 500,                 // Character limit
    placeholder: "Please explain..."
  }
}
```

## 📱 User Experience Design

### Questionnaire UI Components
1. **Rating Stars Component** - Interactive 1-5 star rating
2. **Progress Bar** - Visual progress indicator
3. **Question Card** - Clean card layout for each question
4. **Save & Continue** - Allow partial completion
5. **Responsive Design** - Works on mobile and desktop

### Validation Rules
- All required questions must be answered
- Ratings must be within valid range (1-5)
- Text responses must be within character limits
- Demographics must be selected from valid options
- Prevent duplicate submissions (same user + company + time period)

## 🧪 MVP Features for Phase 2

### Core Features
1. **Dynamic Questionnaire Forms** - Render questions from Firestore
2. **Rating Scale Components** - 1-5 star rating interface
3. **Progress Tracking** - Visual progress through questionnaire
4. **Partial Save** - Save progress and continue later
5. **Real-time Scoring** - Cloud Functions calculate aggregated scores
6. **16 Qualifiers Display** - Polar chart showing all qualifiers

### Testing Scenarios
1. **Complete Flow** - Traveler completes expectation questionnaire
2. **Real-time Updates** - Scores update automatically after submission
3. **Multiple Users** - Different users rating same company
4. **Demographics Filtering** - Filter results by age, gender, nationality
5. **IPA Matrix Data** - Both expectation and satisfaction data available

## 🚀 Implementation Priority

### Phase 2A: Basic Questionnaire System (Week 1)
1. Create sample questionnaire and question data in Firestore
2. Build rating scale components
3. Create basic questionnaire form
4. Implement response submission

### Phase 2B: Aggregation & Scoring (Week 2)  
1. Create Cloud Functions for score calculation
2. Implement real-time aggregated score updates
3. Build 16 qualifiers polar chart display
4. Test complete submission flow

### Phase 2C: Advanced Features (Week 3)
1. Add multi-step navigation and progress tracking
2. Implement partial save functionality
3. Add demographic filtering
4. Create IPA matrix data foundation

## 🎯 Success Criteria

**Phase 2 Complete When:**
- ✅ Users can complete full questionnaire (16 questions)
- ✅ Responses are stored correctly in Firestore
- ✅ Aggregated scores calculate automatically via Cloud Functions
- ✅ 16 qualifiers display in polar chart with correct colors
- ✅ Real-time updates work when new responses submitted
- ✅ Basic demographic filtering works
- ✅ Multiple questionnaire types supported (expectation, satisfaction)

This design provides a solid foundation for building the questionnaire system that matches the existing .NET functionality while leveraging modern React + Firebase architecture.