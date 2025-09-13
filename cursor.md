 # Symptom Tracker MVP - iOS Development Guide

## Project Overview
A **universal symptom tracking iOS app** designed for chronic illness management. This app is **NOT condition-specific** - it's built as a universal platform that can serve any chronic illness community.

**CRITICAL MVP CONSTRAINT:** We are building for **ONLY ONE chronic illness (Peripheral Neuropathy) in the MVP** purely to validate the concept with a single condition before scaling. The app architecture, naming, and design are **completely condition-agnostic** and ready for multi-condition expansion.

## Core Features for MVP
1. Multi-screen onboarding flow (universal for any chronic illness)
2. Homepage with symptom logging sliders (configurable per condition)
3. Food logging functionality (universal)
4. Exercise logging functionality (universal)
5. Rest/sleep logging functionality (universal)
6. **Predictive flare-up analysis** - NO notifications, pure prediction based on yesterday's data

**Key Principle:** This is **Symptom Tracker**, not "Neuropathy Tracker" - all naming reflects universal chronic illness management.

---

# Complete App Flow & Functionality Summary

## What This App Does (High-Level Overview)

### **Core Purpose**
The app learns your personal chronic illness patterns by analyzing what you did yesterday and correlating it with how you feel today. It then uses today's activities to predict tomorrow's symptom severity and flare risk.

### **Daily User Journey**
1. **Morning:** User opens app, sees prediction for today based on yesterday's logged activities
2. **Reality Check:** User logs actual current symptoms, app learns prediction accuracy  
3. **Activity Logging:** Throughout day, user logs food, exercise, and sleep in under 2 minutes per entry
4. **Intelligence Building:** App analyzes today's complete data to predict tomorrow's flare risk
5. **Next Morning:** Cycle repeats with new prediction, algorithm gets smarter over time

### **No Notifications - Pure Prediction**
- App does NOT send any notifications or alerts
- All intelligence displayed on homepage as risk indicators
- User opens app to see predictions, app doesn't push information

### **Universal Design with Single Condition MVP**
- Built for ANY chronic illness (arthritis, lupus, fibromyalgia, etc.)
- MVP hardcoded for Peripheral Neuropathy to prove concept
- All architecture ready for immediate multi-condition expansion
- Zero condition-specific naming in codebase or UI

---

# Detailed App Flow & User Journey

## App Launch & First Time Setup

### **When user opens app for the first time:**
1. User sees welcome screen: "Welcome to your personal **Symptom Tracker**"
2. App starts 5-screen onboarding (works for any chronic illness)
3. User cannot access main app until setup complete

### **Onboarding Screen 1 - Welcome & Basic Info:**
- "Who are you?" - User enters their name
- App explains: "This app helps track chronic illness symptoms and predict flare patterns"
- **MVP Implementation:** No mention of specific condition, kept universal
- User taps "Continue"

### **Onboarding Screen 2 - Symptom Selection:**
- "Which symptoms affect you most day-to-day?"
- **MVP Shows (but stores generically):**
  - ✓ Burning or shooting pain
  - ✓ Tingling or numbness  
  - ✓ Electric shock sensations
  - ✓ Touch sensitivity
  - ✓ Fatigue
  - ✓ Balance issues
- **Architecture:** Stored as universal symptom data, not condition-specific
- User must select at least one, taps "Continue"

### **Onboarding Screen 3 - Severity Impact:**
- "How much do these symptoms affect your daily life?"
- Universal 1-10 slider (works for any chronic illness)
- Visual descriptions: Minimal impact → Severe impact
- User taps "Continue"

### **Onboarding Screen 4 - Flare Patterns:**
- "How do your symptoms typically occur?"
- Universal options (apply to most chronic illnesses):
  - ○ Episodic flare-ups (symptoms come and go)
  - ○ Constant with varying intensity
  - ○ Unpredictable patterns
- User selects one, taps "Continue"

### **Onboarding Screen 5 - Tracking Preferences:**
- "What would you like to track daily?"
- Universal tracking categories:
  - ✓ Food and meals
  - ✓ Exercise and activity
  - ✓ Sleep and rest
  - ✓ Stress levels
- User taps "Let's Go!"
- Celebration animation, saves profile, enters main app

## Daily Homepage Usage

### **Every time user opens app:**
1. Personalized greeting: "Hello, [Name] 👋"
2. Current date display
3. **TODAY'S FLARE PREDICTION** (based on yesterday's data):
   - 🟢 Green: "Low flare risk today - yesterday's choices look good"
   - 🟡 Yellow: "Moderate flare risk - yesterday's sleep was concerning"
   - 🔴 Red: "High flare risk today - yesterday's combination suggests caution"

### **Daily Symptom Check-In (Core Feature):**
- 4 universal symptom sliders (work for any chronic illness):
  - **Fatigue:** 1-10 scale, starts with yesterday's value
  - **Pain:** 1-10 scale, starts with yesterday's value
  - **Mood:** 1-10 scale, starts with yesterday's value
  - **Energy:** 1-10 scale, starts with yesterday's value
- User adjusts based on current feeling
- Color coding: Green (1-3) → Yellow (4-6) → Red (7-10)
- Haptic feedback on slider movement
- Auto-saves after 2 seconds, no "Save" button needed
- **Algorithm learns:** Compares predicted vs. actual symptoms

### **Quick Action Cards:**
2x2 grid of logging options:
- 🍽️ "Record Food" (salmon/red card)
- 💪 "Record Exercise" (purple card)
- 😴 "Record Rest" (orange card)
- 📊 "View Trends" (blue card - "Coming Soon" in MVP)

## Food Logging Flow

### **When user taps "Record Food":**
1. Opens food logging screen with today's date
2. **Universal food tracking** (not condition-specific):

### **Food Entry Process:**
1. **Meal Type:** Breakfast | Lunch | Dinner | Snack
2. **Food Items:**
   - Text field: "What did you eat?"
   - Autocomplete suggestions as user types
   - Add button creates list of foods
   - X button removes items from list
3. **Beverages (Optional):**
   - "What did you drink?" text field
   - Same add/remove functionality
4. **Calories (Optional):** Numeric input, skippable
5. **Notes (Optional):** Free text for context

### **Completion:**
- "Save Entry" button
- Success message: "Food logged successfully!"
- Auto-return to homepage
- **Background analysis:** Updates tomorrow's prediction based on today's food choices

## Exercise Logging Flow

### **When user taps "Record Exercise":**
Opens universal exercise logging screen

### **Exercise Entry:**
1. **Exercise Type:** Dropdown selection
   - Walking, Swimming, Yoga, Cycling, Strength Training, Other
2. **Duration:** Time picker (hours/minutes)
3. **Intensity Level:** 
   - Light (could do for hours)
   - Moderate (breathing faster)
   - Intense (can't maintain long)
4. **Steps (Optional):** Number field
5. **Notes (Optional):** Free text

### **Completion:**
- "Save Exercise" button
- Success confirmation
- Return to homepage
- **Background analysis:** Updates tomorrow's flare prediction

## Sleep/Rest Logging Flow

### **When user taps "Record Rest":**
Opens universal sleep tracking screen

### **Sleep Entry:**
1. **Last Night's Sleep:**
   - Sleep start time picker
   - Wake up time picker
   - Auto-calculated duration display
2. **Sleep Quality:** 1-10 slider (Poor → Excellent)
3. **Naps (Optional):**
   - "Did you nap today?" toggle
   - Conditional nap time pickers
   - Auto-calculated nap duration
4. **Overall Restfulness:** 1-10 slider (Exhausted → Fully Rested)

### **Completion:**
- "Save Rest Entry" button
- Success confirmation
- Return to homepage
- **Background analysis:** Most important factor for tomorrow's prediction

## Predictive Intelligence (No Notifications)

### **What Happens Behind the Scenes:**
1. **Yesterday → Today Analysis:**
   - App correlates yesterday's activities with today's reported symptoms
   - Learns user's personal response patterns
   - Builds accuracy score for predictions

2. **Today → Tomorrow Prediction:**
   - Analyzes today's logged food, exercise, sleep
   - Calculates tomorrow's flare risk using learned patterns
   - Updates homepage prediction for next day

3. **Pattern Learning:**
   - Week 1: Establishes baseline (60% accuracy)
   - Week 2-4: Identifies personal triggers (75% accuracy)
   - Month 2+: Highly personalized predictions (85%+ accuracy)

### **Risk Factors (Universal Chronic Illness Principles):**
1. **Sleep Impact (35%):** Poor sleep universally increases next-day symptoms
2. **Exercise Recovery (30%):** Overexertion requires recovery time
3. **Food Triggers (25%):** Inflammatory foods cause delayed symptom response
4. **Cumulative Stress (10%):** Multiple poor choices compound risk

### **No Notifications - Only Homepage Display:**
- All predictions shown when user opens app
- No push notifications, alerts, or interruptions
- User controls when they see information

## Homepage Updates After Logging

### **Real-Time Prediction Updates:**
- **Tomorrow's Risk Indicator:**
  - Green: "Tomorrow looks manageable based on today's choices"
  - Yellow: "Tomorrow may be challenging - consider today's activities"
  - Red: "High flare risk tomorrow - today's combination concerning"

### **Today's Progress:**
- Visual indicators: ✓ Food logged | ✓ Exercise logged | ✓ Rest logged
- Missing items show as empty circles

### **Contextual Insights:**
- "Your pain is higher today - poor sleep last night likely contributed"
- "Energy good today - yesterday's rest day helped"
- "Symptoms match prediction - algorithm learning your patterns"

## Error Handling & Edge Cases

### **Common Scenarios:**
- **Incomplete logging:** Saves partial data, allows completion later
- **Missed days:** Shows "Need more data for accurate predictions"
- **Invalid data:** Friendly validation with clear error messages
- **App crashes:** Auto-save protects data, resume where left off
- **Insufficient history:** "Building your pattern - predictions improve with data"

## Complete Daily Cycle

### **Perfect Usage Pattern:**
1. **Morning:** See yesterday-based prediction, log actual symptoms
2. **Throughout Day:** Log food after meals (2 minutes each)
3. **After Exercise:** Log workout details (1-2 minutes)
4. **Evening:** Log sleep from previous night (2 minutes)
5. **Background:** App calculates tomorrow's prediction
6. **Next Morning:** Cycle repeats with improved accuracy

### **App's Core Promise:**
*"Track your symptoms and activities in under 6 minutes total per day. We'll learn your personal chronic illness patterns and predict tomorrow's flare risk with increasing accuracy."*

### **Key Principles:**
- **Yesterday Predicts Today:** Validates algorithm accuracy
- **Today Predicts Tomorrow:** Uses current data for next-day forecasting
- **Universal Design:** Works for any chronic illness, not just one condition
- **No Interruptions:** Pure prediction, no notifications
- **Personal Learning:** Algorithm adapts to individual patterns

---

## Technical Architecture

### Development Environment Setup
- **Xcode:** Version 15.0 or later
- **iOS Deployment Target:** iOS 16.0+
- **Swift Version:** 5.9+
- **Architecture Pattern:** MVVM (Model-View-ViewModel)
- **Data Persistence:** Core Data
- **UI Framework:** SwiftUI

### Project Structure
```
SymptomTracker/
├── Models/
├── ViewModels/
├── Views/
│   ├── Onboarding/
│   ├── Home/
│   ├── Logging/
│   └── Shared/
├── Services/
├── Utilities/
├── Resources/
└── CoreData/
```

---

## Data Models

### Core Data Entities

#### User Profile Entity
- `userID`: UUID
- `userName`: String
- `primaryCondition`: String (MVP: "Peripheral Neuropathy", Future: selectable)
- `mainSymptoms`: [String] (transformable - universal symptom list)
- `severityImpact`: Int16 (1-10 universal scale)
- `flarePatterns`: String (universal flare categories)
- `currentTreatments`: [String] (transformable - universal)
- `triggers`: [String] (transformable - universal trigger categories)
- `createdAt`: Date

**Universal Design Note:** All fields designed to work with any chronic illness

#### Symptom Entry Entity
- `entryID`: UUID
- `date`: Date
- `fatigue`: Int16 (1-10 scale - universal)
- `pain`: Int16 (1-10 scale - universal)
- `mood`: Int16 (1-10 scale - universal)
- `energy`: Int16 (1-10 scale - universal)
- `customSymptom1`: Int16 (condition-specific, MVP: tingling/numbness)
- `customSymptom2`: Int16 (condition-specific, MVP: burning pain)
- `customSymptom3`: Int16 (condition-specific, MVP: touch sensitivity)
- `customSymptom1Name`: String (MVP: "Tingling/Numbness")
- `customSymptom2Name`: String (MVP: "Burning Pain")
- `customSymptom3Name`: String (MVP: "Touch Sensitivity")
- `createdAt`: Date

**Scalability:** Core symptoms universal, custom fields adapt per condition

#### Food Entry Entity
- `entryID`: UUID
- `date`: Date
- `mealType`: String (breakfast, lunch, dinner, snack)
- `foodItems`: [String] (transformable)
- `calories`: Int32 (optional)
- `drinks`: [String] (transformable)
- `notes`: String (optional)
- `createdAt`: Date

#### Exercise Entry Entity
- `entryID`: UUID
- `date`: Date
- `exerciseType`: String
- `duration`: Int32 (minutes)
- `intensity`: String (light, moderate, intense)
- `steps`: Int32 (optional)
- `notes`: String (optional)
- `createdAt`: Date

#### Rest Entry Entity
- `entryID`: UUID
- `date`: Date
- `sleepStart`: Date
- `sleepEnd`: Date
- `sleepQuality`: Int16 (1-10 scale)
- `napStart`: Date (optional)
- `napEnd`: Date (optional)
- `restfulnessRating`: Int16 (1-10 scale)
- `createdAt`: Date

#### Flare Event Entity
- `flareID`: UUID
- `date`: Date
- `predictedSeverity`: Int16 (algorithm prediction)
- `actualSeverity`: Int16 (user reported)
- `triggerFactors`: [String] (transformable)
- `predictionAccuracy`: Float (for algorithm learning)
- `createdAt`: Date

---

## View Models

### OnboardingViewModel
**Responsibilities:**
- Manage universal onboarding flow (works for any condition)
- Validate user inputs
- Save user profile to Core Data
- Navigate between screens

**Key Properties:**
- `currentStep: Int`
- `userProfile: UserProfile`
- `isOnboardingComplete: Bool`

### HomeViewModel
**Responsibilities:**
- Load symptom data and yesterday's predictions
- Manage symptom slider states with auto-save
- Display prediction accuracy and learning progress
- Handle navigation to logging screens
- Update tomorrow's predictions based on today's data

**Key Properties:**
- `todaysPrediction: FlareRisk`
- `currentSymptoms: SymptomEntry`
- `yesterdaySymptoms: SymptomEntry?`
- `tomorrowsPrediction: FlareRisk`
- `predictionAccuracy: Float`
- `todaysEntries: [LogEntry]`

### FoodLoggingViewModel
**Responsibilities:**
- Manage food entry form (universal for all conditions)
- Save food entries to Core Data
- Validate food data
- Trigger prediction updates

### ExerciseLoggingViewModel
**Responsibilities:**
- Manage exercise entry form
- Track activity metrics
- Save exercise entries
- Update flare risk calculations

### RestLoggingViewModel
**Responsibilities:**
- Manage sleep/rest entry form
- Calculate sleep duration and quality scores
- Save rest entries
- Most critical data for next-day predictions

### FlareAnalysisViewModel
**Responsibilities:**
- Analyze yesterday → today correlations
- Calculate tomorrow's predictions
- Learn personal patterns over time
- Track prediction accuracy
- Universal algorithm adaptable to any chronic illness

---

## Core Services

### CoreDataService
**Responsibilities:**
- Manage Core Data stack
- CRUD operations for all entities
- Data persistence and retrieval
- Universal data management

**Key Methods:**
- `saveContext()`
- `fetchSymptomEntries(for date: Date)`
- `createFoodEntry(_:)`
- `calculatePredictionAccuracy(predicted: FlareRisk, actual: SymptomEntry)`

### FlareAnalysisService
**Responsibilities:**
- Predictive analysis using yesterday → today → tomorrow model
- Personal pattern learning (condition-agnostic)
- Risk calculation without notifications
- Algorithm accuracy improvement over time

**Universal Algorithm Logic:**
1. **Sleep Impact (35%):** < 6 hours increases next-day symptoms (universal)
2. **Exercise Recovery (30%):** Overexertion needs recovery time
3. **Food Triggers (25%):** Inflammatory response patterns
4. **Cumulative Effects (10%):** Multiple factors compound risk

**Key Methods:**
- `predictTomorrowsFlareRisk(basedOnToday: TodayData) -> FlareRisk`
- `analyzeYesterdayTodayCorrelation() -> CorrelationInsight`
- `updatePersonalPatterns(newData: DailyData)`
- `calculatePredictionConfidence() -> Float`

### PredictionDisplayService
**Responsibilities:**
- Format predictions for homepage display
- Generate contextual explanations
- Track and display accuracy metrics
- No notifications - pure display service

---

## Predictive Analysis Algorithm

### Universal Risk Prediction Model

#### Core Concept: Yesterday → Today → Tomorrow
- **Yesterday's Data + Today's Symptoms = Algorithm Learning**
- **Today's Data + Learned Patterns = Tomorrow's Prediction**
- **No Notifications - Pure Predictive Intelligence**

#### Risk Factors (Universal Chronic Illness Framework)
1. **Sleep Quality Impact (35% weight):**
   - < 5 hours sleep → 60% increased next-day symptom risk
   - Poor sleep quality → Compounds all other risk factors
   - Recovery sleep → Reduces accumulated sleep debt

2. **Exercise Recovery Patterns (30% weight):**
   - High intensity + inadequate recovery = High next-day risk
   - Consistent moderate exercise = Protective factor
   - Sedentary periods = Gradual symptom increase risk

3. **Food Trigger Response (25% weight):**
   - Individual trigger foods → 12-24 hour delayed response
   - Large meals + poor sleep = Compounded risk
   - Anti-inflammatory foods = Protective patterns

4. **Cumulative Stress Load (10% weight):**
   - Multiple poor choices exponentially increase risk
   - Single risk factor = Manageable
   - Personal stress tolerance learned over time

### Prediction Accuracy Learning
```
Week 1: Baseline establishment (50-60% accuracy)
Week 2-4: Pattern recognition (70-80% accuracy)  
Month 2+: Personalized model (85%+ accuracy)

Learning Process:
- Compare yesterday's prediction to today's reality
- Adjust algorithm weights based on accuracy
- Identify personal trigger combinations
- Refine timing of symptom responses
```

### Risk Communication (No Notifications)
- **Green (0-30):** "Low risk - patterns look good"
- **Yellow (31-65):** "Moderate risk - some concerning factors"
- **Red (66-100):** "High risk - multiple factors suggest caution"

---

## Technical Implementation Phases

### Phase 1: Foundation & Core Infrastructure

#### Project Architecture Setup
**Xcode & Core Data:**
- Create new iOS project: "SymptomTracker" (universal naming)
- Configure SwiftUI + Core Data stack
- Set up universal data models (condition-agnostic design)
- Implement MVVM architecture with dependency injection
- Create design system with universal components

**Key Deliverables:**
- Universal symptom tracking foundation
- Core Data model supporting any chronic illness
- Reusable UI component library
- Navigation framework

### Phase 2: Universal Onboarding System

#### Multi-Screen Onboarding Flow
**Universal Design Implementation:**
- 5-screen onboarding working for any chronic illness
- Condition-agnostic language and UI
- MVP populated with Peripheral Neuropathy data
- User profile creation with universal data structure
- Smooth animations and validation

**Key Deliverables:**
- Complete onboarding flow
- Universal user profile system
- Form validation and error handling
- Celebration completion flow

### Phase 3: Homepage & Symptom Logging

#### Prediction-Based Dashboard
**Core Features:**
- Homepage displaying today's prediction (no notifications)
- Universal symptom sliders with auto-save
- Yesterday's data correlation with today's reality
- Action cards for logging activities
- Real-time prediction accuracy learning

**Key Deliverables:**
- Predictive homepage dashboard
- Auto-saving symptom tracking
- Navigation to logging screens
- Algorithm accuracy display

### Phase 4: Activity Logging System

#### Universal Logging Interfaces
**Food, Exercise, and Sleep Tracking:**
- Universal food logging (works for any condition)
- Exercise tracking with intensity and recovery analysis
- Sleep logging as primary prediction factor
- All logging optimized for under 2 minutes per entry
- Data validation and error handling

**Key Deliverables:**
- Complete logging system
- Data persistence and retrieval
- Form validation
- Quick, efficient user experience

### Phase 5: Predictive Analysis Engine

#### Yesterday → Today → Tomorrow Intelligence
**Algorithm Development:**
- Personal pattern learning system
- Prediction accuracy tracking and improvement
- Risk calculation without notifications
- Universal chronic illness correlation principles
- Background analysis and prediction updates

**Key Deliverables:**
- Intelligent prediction algorithm
- Personal pattern recognition
- Accuracy tracking and improvement
- Universal risk assessment framework

---

## Testing Strategy

### Comprehensive Testing Approach
- **Unit Tests:** Core Data operations, algorithm accuracy, universal data handling
- **Integration Tests:** Complete user flows, prediction accuracy, multi-condition readiness
- **UI Tests:** Onboarding flow, logging efficiency, accessibility compliance
- **Algorithm Tests:** Prediction accuracy with various chronic illness patterns

---

## Deployment Preparation

### App Store Readiness
- Universal chronic illness positioning
- Privacy policy for health data
- Accessibility compliance
- Performance optimization
- Multi-device testing

### Future Scalability
- Architecture ready for multiple conditions
- Universal data export for healthcare providers
- Algorithm adaptability for different chronic illnesses
- Cloud sync preparation (future feature)

---

## Key Success Metrics

### MVP Validation Goals
1. **User Engagement:** Daily logging completion rate >70%
2. **Prediction Accuracy:** Algorithm accuracy >80% by month 2
3. **Time Efficiency:** Average logging time <6 minutes total per day
4. **Universal Design:** Architecture validated for scalability to other conditions

### Long-term Vision
Transform from single-condition MVP to universal chronic illness management platform, serving multiple conditions with personalized predictive intelligence.

**Bottom Line:** Build Symptom Tracker as a universal platform, prove concept with Peripheral Neuropathy, scale to serve all chronic illness communities.
