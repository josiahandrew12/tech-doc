# Symptom Tracker MVP - Complete Development Guide

## Project Overview

A simple, intuitive symptom tracker that helps patients see patterns and share insights. Built for individuals with chronic conditions (food sensitivities, autoimmune, dysautonomia) who are frustrated with manual spreadsheet tracking.

### Core MVP Features
- Daily food & symptom logging (simple, fast input)
- Symptom severity scale (1–10) for quick tracking
- Activity logging (exercise, exertion, rest) to correlate with flares
- Timeline view (connects meals to symptom flares)
- Onboarding to capture user information
- Pattern recognition and correlation analysis

---

## Phase 1: Project Foundation & Setup

### 1.1 Xcode Project Creation

#### Step 1: Create New iOS Project
1. Open Xcode 15+
2. Select "Create a new Xcode project"
3. Choose "iOS" → "App"
4. Configure project:
   - Product Name: "SymptomTracker"
   - Interface: SwiftUI
   - Language: Swift
   - Use Core Data: ✅ Checked
   - Include Tests: ✅ Checked
   - Minimum Deployment: iOS 15.0

#### Step 2: Project Structure Setup
Create the following folder structure in your project:
```
SymptomTracker/
├── App/
│   ├── SymptomTrackerApp.swift
│   └── ContentView.swift
├── Models/
│   ├── CoreData/
│   │   ├── SymptomTracker.xcdatamodeld
│   │   └── PersistenceController.swift
│   ├── Entities/
│   └── ViewModels/
├── Views/
│   ├── Onboarding/
│   ├── Today/
│   ├── Timeline/
│   ├── Correlations/
│   └── Components/
├── Services/
│   ├── DataManager.swift
│   ├── CorrelationEngine.swift
│   └── UserDefaultsManager.swift
└── Utils/
    ├── Extensions/
    ├── Constants.swift
    └── Helpers/
```

#### Step 3: Add Dependencies
1. In Xcode, go to File → Add Package Dependencies
2. Add Alamofire: `https://github.com/Alamofire/Alamofire.git`
3. Select "Up to Next Major Version" with minimum 5.8.0

### 1.2 Core Data Model Design

#### Step 1: Open SymptomTracker.xcdatamodeld
Double-click the Core Data model file to open the visual editor.

#### Step 2: Create User Entity
1. Click "+" to add new entity, name it "User"
2. Add attributes:
   - `id`: UUID (primary key)
   - `onboardingCompleted`: Boolean (default: NO)
   - `primaryConditions`: Transformable (NSArray)
   - `mainSymptoms`: Transformable (NSArray)
   - `severityImpact`: String
   - `flarePatterns`: String
   - `currentTreatments`: String
   - `createdAt`: Date
   - `updatedAt`: Date

#### Step 3: Create DailyLog Entity
1. Add new entity "DailyLog"
2. Add attributes:
   - `id`: UUID (primary key)
   - `date`: Date (indexed)
   - `overallMood`: Integer 16 (1-10 scale)
   - `energyLevel`: Integer 16 (1-10 scale)
   - `stressLevel`: Integer 16 (1-10 scale)
   - `sleepHours`: Double
   - `notes`: String (optional, max length 500)
   - `createdAt`: Date
   - `updatedAt`: Date

#### Step 4: Create SymptomEntry Entity
1. Add new entity "SymptomEntry"
2. Add attributes:
   - `id`: UUID (primary key)
   - `symptomName`: String
   - `severity`: Integer 16 (1-10 scale)
   - `timestamp`: Date
   - `notes`: String (optional, max length 200)
   - `createdAt`: Date

#### Step 5: Create FoodEntry Entity
1. Add new entity "FoodEntry"
2. Add attributes:
   - `id`: UUID (primary key)
   - `foodName`: String
   - `mealType`: String (breakfast/lunch/dinner/snack)
   - `timestamp`: Date
   - `notes`: String (optional, max length 200)
   - `createdAt`: Date

#### Step 6: Create ExerciseEntry Entity
1. Add new entity "ExerciseEntry"
2. Add attributes:
   - `id`: UUID (primary key)
   - `exerciseType`: String
   - `duration`: Integer 16 (minutes)
   - `intensity`: Integer 16 (1-10 scale)
   - `timestamp`: Date
   - `notes`: String (optional, max length 200)
   - `createdAt`: Date

#### Step 7: Create ActivityEntry Entity
1. Add new entity "ActivityEntry"
2. Add attributes:
   - `id`: UUID (primary key)
   - `activityType`: String
   - `duration`: Integer 16 (minutes)
   - `timestamp`: Date
   - `notes`: String (optional, max length 200)
   - `createdAt`: Date

#### Step 8: Configure Relationships
1. Select DailyLog entity
2. Add relationships:
   - `symptomEntries`: To-Many → SymptomEntry (Delete Rule: Cascade)
   - `foodEntries`: To-Many → FoodEntry (Delete Rule: Cascade)
   - `exerciseEntries`: To-Many → ExerciseEntry (Delete Rule: Cascade)
   - `activityEntries`: To-Many → ActivityEntry (Delete Rule: Cascade)

3. For each entry entity, add inverse relationship:
   - `dailyLog`: To-One → DailyLog (Delete Rule: Nullify)

#### Step 9: Generate NSManagedObject Subclasses
1. Select each entity
2. Set Codegen to "Category/Extension"
3. In terminal, navigate to project directory
4. Run: `xcodegen generate` (or use Xcode's Editor → Create NSManagedObject Subclass)

---

## Phase 2: Core Data Stack & Persistence

### 2.1 PersistenceController Implementation

Create `Models/CoreData/PersistenceController.swift`:

```swift
import CoreData
import CloudKit

class PersistenceController: ObservableObject {
    static let shared = PersistenceController()
    
    lazy var container: NSPersistentCloudKitContainer = {
        let container = NSPersistentCloudKitContainer(name: "SymptomTracker")
        
        // Configure CloudKit
        let storeDescription = container.persistentStoreDescriptions.first
        storeDescription?.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
        storeDescription?.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
        
        container.loadPersistentStores { _, error in
            if let error = error as NSError? {
                fatalError("Core Data error: \(error), \(error.userInfo)")
            }
        }
        
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        return container
    }()
    
    var viewContext: NSManagedObjectContext {
        container.viewContext
    }
    
    func newBackgroundContext() -> NSManagedObjectContext {
        let context = container.newBackgroundContext()
        context.automaticallyMergesChangesFromParent = true
        context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        return context
    }
    
    func save() {
        let context = container.viewContext
        
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                print("Save error: \(error)")
            }
        }
    }
    
    func saveContext(_ context: NSManagedObjectContext) {
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                print("Context save error: \(error)")
            }
        }
    }
}
```

### 2.2 Data Manager Service

Create `Services/DataManager.swift`:

```swift
import CoreData
import Combine

class DataManager: ObservableObject {
    private let persistenceController = PersistenceController.shared
    private var cancellables = Set<AnyCancellable>()
    
    var viewContext: NSManagedObjectContext {
        persistenceController.viewContext
    }
    
    // MARK: - Generic CRUD Operations
    
    func save() {
        persistenceController.save()
    }
    
    func fetch<T: NSManagedObject>(
        _ type: T.Type,
        predicate: NSPredicate? = nil,
        sortDescriptors: [NSSortDescriptor] = [],
        fetchLimit: Int? = nil
    ) -> [T] {
        let request = NSFetchRequest<T>(entityName: String(describing: type))
        request.predicate = predicate
        request.sortDescriptors = sortDescriptors
        
        if let limit = fetchLimit {
            request.fetchLimit = limit
        }
        
        do {
            return try viewContext.fetch(request)
        } catch {
            print("Fetch error for \(type): \(error)")
            return []
        }
    }
    
    func delete<T: NSManagedObject>(_ object: T) {
        viewContext.delete(object)
        save()
    }
    
    // MARK: - Specific Entity Operations
    
    func createDailyLog(for date: Date) -> DailyLog {
        let dailyLog = DailyLog(context: viewContext)
        dailyLog.id = UUID()
        dailyLog.date = Calendar.current.startOfDay(for: date)
        dailyLog.createdAt = Date()
        dailyLog.updatedAt = Date()
        save()
        return dailyLog
    }
    
    func getDailyLog(for date: Date) -> DailyLog? {
        let startOfDay = Calendar.current.startOfDay(for: date)
        let endOfDay = Calendar.current.date(byAdding: .day, value: 1, to: startOfDay)!
        
        let predicate = NSPredicate(format: "date >= %@ AND date < %@", startOfDay as NSDate, endOfDay as NSDate)
        
        return fetch(DailyLog.self, predicate: predicate).first
    }
    
    func getOrCreateDailyLog(for date: Date) -> DailyLog {
        if let existing = getDailyLog(for: date) {
            return existing
        }
        return createDailyLog(for: date)
    }
    
    func createSymptomEntry(
        name: String,
        severity: Int,
        timestamp: Date,
        notes: String? = nil,
        dailyLog: DailyLog
    ) -> SymptomEntry {
        let entry = SymptomEntry(context: viewContext)
        entry.id = UUID()
        entry.symptomName = name
        entry.severity = Int16(severity)
        entry.timestamp = timestamp
        entry.notes = notes
        entry.createdAt = Date()
        entry.dailyLog = dailyLog
        save()
        return entry
    }
    
    func createFoodEntry(
        name: String,
        mealType: String,
        timestamp: Date,
        notes: String? = nil,
        dailyLog: DailyLog
    ) -> FoodEntry {
        let entry = FoodEntry(context: viewContext)
        entry.id = UUID()
        entry.foodName = name
        entry.mealType = mealType
        entry.timestamp = timestamp
        entry.notes = notes
        entry.createdAt = Date()
        entry.dailyLog = dailyLog
        save()
        return entry
    }
    
    func createExerciseEntry(
        type: String,
        duration: Int,
        intensity: Int,
        timestamp: Date,
        notes: String? = nil,
        dailyLog: DailyLog
    ) -> ExerciseEntry {
        let entry = ExerciseEntry(context: viewContext)
        entry.id = UUID()
        entry.exerciseType = type
        entry.duration = Int16(duration)
        entry.intensity = Int16(intensity)
        entry.timestamp = timestamp
        entry.notes = notes
        entry.createdAt = Date()
        entry.dailyLog = dailyLog
        save()
        return entry
    }
    
    func createActivityEntry(
        type: String,
        duration: Int,
        timestamp: Date,
        notes: String? = nil,
        dailyLog: DailyLog
    ) -> ActivityEntry {
        let entry = ActivityEntry(context: viewContext)
        entry.id = UUID()
        entry.activityType = type
        entry.duration = Int16(duration)
        entry.timestamp = timestamp
        entry.notes = notes
        entry.createdAt = Date()
        entry.dailyLog = dailyLog
        save()
        return entry
    }
}
```

---

## Phase 3: Navigation Architecture & User Defaults

### 3.1 User Defaults Manager

Create `Services/UserDefaultsManager.swift`:

```swift
import Foundation

class UserDefaultsManager: ObservableObject {
    static let shared = UserDefaultsManager()
    
    @Published var onboardingCompleted: Bool {
        didSet {
            UserDefaults.standard.set(onboardingCompleted, forKey: "onboardingCompleted")
        }
    }
    
    private init() {
        onboardingCompleted = UserDefaults.standard.bool(forKey: "onboardingCompleted")
    }
    
    func completeOnboarding() {
        onboardingCompleted = true
    }
    
    func resetOnboarding() {
        onboardingCompleted = false
    }
}
```

### 3.2 Navigation Coordinator

Create `Models/ViewModels/NavigationCoordinator.swift`:

```swift
import SwiftUI

enum TabSelection {
    case today
    case timeline
    case correlations
}

enum OnboardingStep {
    case welcome
    case conditions
    case completion
}

class NavigationCoordinator: ObservableObject {
    @Published var selectedTab: TabSelection = .today
    @Published var currentOnboardingStep: OnboardingStep = .welcome
    
    func selectTab(_ tab: TabSelection) {
        selectedTab = tab
    }
    
    func nextOnboardingStep() {
        switch currentOnboardingStep {
        case .welcome:
            currentOnboardingStep = .conditions
        case .conditions:
            currentOnboardingStep = .completion
        case .completion:
            break
        }
    }
    
    func previousOnboardingStep() {
        switch currentOnboardingStep {
        case .welcome:
            break
        case .conditions:
            currentOnboardingStep = .welcome
        case .completion:
            currentOnboardingStep = .conditions
        }
    }
}
```

### 3.3 Constants File

Create `Utils/Constants.swift`:

```swift
import SwiftUI

struct Constants {
    // MARK: - Colors
    struct Colors {
        static let primary = Color.blue
        static let secondary = Color.gray
        static let accent = Color.green
        
        // Severity colors (1-10 scale)
        static let severityColors: [Color] = [
            .clear, // 0 - not used
            .green,    // 1
            .green,    // 2
            .yellow,   // 3
            .yellow,   // 4
            .orange,   // 5
            .orange,   // 6
            .red,      // 7
            .red,      // 8
            .red,      // 9
            .red       // 10
        ]
    }
    
    // MARK: - Meal Types
    static let mealTypes = ["Breakfast", "Lunch", "Dinner", "Snack"]
    
    // MARK: - Exercise Types
    static let exerciseTypes = [
        "Cardio", "Strength", "Yoga", "Walking", 
        "Swimming", "Cycling", "Running", "Other"
    ]
    
    // MARK: - Activity Types
    static let activityTypes = [
        "Work", "Social", "Travel", "Medical", 
        "Shopping", "Household", "Rest", "Other"
    ]
    
    // MARK: - Common Conditions
    static let commonConditions = [
        "Fibromyalgia", "POTS/Dysautonomia", "Chronic Fatigue Syndrome",
        "Autoimmune Disease", "Food Sensitivities", "IBS/IBD",
        "Migraines", "Chronic Pain", "Anxiety", "Depression", "Other"
    ]
    
    // MARK: - Common Symptoms
    static let commonSymptoms = [
        "Fatigue", "Pain", "Migraines", "Gut Issues", "Brain Fog",
        "Dizziness", "Heart Palpitations", "Joint Pain", "Muscle Pain",
        "Nausea", "Sleep Issues", "Mood Changes", "Other"
    ]
    
    // MARK: - Severity Impact Options
    static let severityImpactOptions = [
        "None - symptoms don't affect daily life",
        "Mild - minor impact on daily activities",
        "Moderate - some limitations on daily activities",
        "Severe - significant impact on daily life"
    ]
    
    // MARK: - Flare Pattern Options
    static let flarePatternOptions = [
        "Episodic - symptoms come and go in flares",
        "Constant - symptoms are always present",
        "Mixed - both episodic flares and constant baseline"
    ]
}
```

---

## Phase 4: Onboarding Flow Implementation

### 4.1 Onboarding View Model

Create `Models/ViewModels/OnboardingViewModel.swift`:

```swift
import SwiftUI
import Combine

class OnboardingViewModel: ObservableObject {
    @Published var selectedConditions: Set<String> = []
    @Published var selectedSymptoms: Set<String> = []
    @Published var severityImpact: String = ""
    @Published var flarePattern: String = ""
    @Published var currentTreatments: String = ""
    @Published var currentStep: OnboardingStep = .welcome
    
    private let dataManager = DataManager()
    private let userDefaultsManager = UserDefaultsManager.shared
    
    func toggleCondition(_ condition: String) {
        if selectedConditions.contains(condition) {
            selectedConditions.remove(condition)
        } else {
            selectedConditions.insert(condition)
        }
    }
    
    func toggleSymptom(_ symptom: String) {
        if selectedSymptoms.contains(symptom) {
            selectedSymptoms.remove(symptom)
        } else {
            selectedSymptoms.insert(symptom)
        }
    }
    
    func canProceedFromConditions() -> Bool {
        return !selectedConditions.isEmpty && !selectedSymptoms.isEmpty && 
               !severityImpact.isEmpty && !flarePattern.isEmpty
    }
    
    func completeOnboarding() {
        // Create User entity
        let user = User(context: dataManager.viewContext)
        user.id = UUID()
        user.primaryConditions = Array(selectedConditions)
        user.mainSymptoms = Array(selectedSymptoms)
        user.severityImpact = severityImpact
        user.flarePatterns = flarePattern
        user.currentTreatments = currentTreatments
        user.onboardingCompleted = true
        user.createdAt = Date()
        user.updatedAt = Date()
        
        dataManager.save()
        userDefaultsManager.completeOnboarding()
    }
}
```

### 4.2 Welcome Screen

Create `Views/Onboarding/WelcomeView.swift`:

```swift
import SwiftUI

struct WelcomeView: View {
    @ObservedObject var onboardingVM: OnboardingViewModel
    @ObservedObject var navigationCoordinator: NavigationCoordinator
    
    var body: some View {
        VStack(spacing: 30) {
            Spacer()
            
            // App Icon/Logo
            Image(systemName: "heart.text.square")
                .font(.system(size: 80))
                .foregroundColor(Constants.Colors.primary)
            
            // Title
            Text("Symptom Tracker")
                .font(.largeTitle)
                .fontWeight(.bold)
            
            // Description
            VStack(spacing: 16) {
                Text("Track your symptoms, food, and activities to discover patterns and share insights with your healthcare team.")
                    .font(.body)
                    .multilineTextAlignment(.center)
                    .foregroundColor(.secondary)
                
                Text("Built for people managing chronic conditions who want to understand their triggers and patterns.")
                    .font(.callout)
                    .multilineTextAlignment(.center)
                    .foregroundColor(.secondary)
            }
            .padding(.horizontal)
            
            Spacer()
            
            // Get Started Button
            Button(action: {
                navigationCoordinator.nextOnboardingStep()
            }) {
                Text("Get Started")
                    .font(.headline)
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Constants.Colors.primary)
                    .cornerRadius(12)
            }
            .padding(.horizontal)
            
            Spacer()
        }
        .padding()
    }
}
```

### 4.3 Conditions Selection Screen

Create `Views/Onboarding/ConditionsView.swift`:

```swift
import SwiftUI

struct ConditionsView: View {
    @ObservedObject var onboardingVM: OnboardingViewModel
    @ObservedObject var navigationCoordinator: NavigationCoordinator
    
    var body: some View {
        VStack(alignment: .leading, spacing: 20) {
            // Progress indicator
            HStack {
                Button(action: {
                    navigationCoordinator.previousOnboardingStep()
                }) {
                    Image(systemName: "chevron.left")
                        .font(.title2)
                }
                
                Spacer()
                
                Text("Step 2 of 3")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            ScrollView {
                VStack(alignment: .leading, spacing: 24) {
                    // Primary Conditions Section
                    VStack(alignment: .leading, spacing: 12) {
                        Text("Primary Condition(s)")
                            .font(.headline)
                        
                        Text("What chronic illness or diagnoses are you managing?")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        
                        LazyVGrid(columns: [
                            GridItem(.flexible()),
                            GridItem(.flexible())
                        ], spacing: 8) {
                            ForEach(Constants.commonConditions, id: \.self) { condition in
                                ConditionToggleButton(
                                    condition: condition,
                                    isSelected: onboardingVM.selectedConditions.contains(condition)
                                ) {
                                    onboardingVM.toggleCondition(condition)
                                }
                            }
                        }
                    }
                    
                    // Main Symptoms Section
                    VStack(alignment: .leading, spacing: 12) {
                        Text("Main Symptoms")
                            .font(.headline)
                        
                        Text("Which symptoms affect you most day-to-day?")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        
                        LazyVGrid(columns: [
                            GridItem(.flexible()),
                            GridItem(.flexible())
                        ], spacing: 8) {
                            ForEach(Constants.commonSymptoms, id: \.self) { symptom in
                                ConditionToggleButton(
                                    condition: symptom,
                                    isSelected: onboardingVM.selectedSymptoms.contains(symptom)
                                ) {
                                    onboardingVM.toggleSymptom(symptom)
                                }
                            }
                        }
                    }
                    
                    // Severity Impact Section
                    VStack(alignment: .leading, spacing: 12) {
                        Text("Severity Impact")
                            .font(.headline)
                        
                        Text("How much do these symptoms affect your daily life?")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        
                        VStack(spacing: 8) {
                            ForEach(Constants.severityImpactOptions, id: \.self) { option in
                                RadioButton(
                                    text: option,
                                    isSelected: onboardingVM.severityImpact == option
                                ) {
                                    onboardingVM.severityImpact = option
                                }
                            }
                        }
                    }
                    
                    // Flare Patterns Section
                    VStack(alignment: .leading, spacing: 12) {
                        Text("Flare Patterns")
                            .font(.headline)
                        
                        Text("Do your symptoms flare up (episodic) or are they constant?")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        
                        VStack(spacing: 8) {
                            ForEach(Constants.flarePatternOptions, id: \.self) { option in
                                RadioButton(
                                    text: option,
                                    isSelected: onboardingVM.flarePattern == option
                                ) {
                                    onboardingVM.flarePattern = option
                                }
                            }
                        }
                    }
                    
                    // Current Treatments Section
                    VStack(alignment: .leading, spacing: 12) {
                        Text("Current Treatments/Supports")
                            .font(.headline)
                        
                        Text("Are you currently using medications, supplements, diet restrictions, physical therapy, etc.? (Optional)")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        
                        TextField("Enter current treatments...", text: $onboardingVM.currentTreatments, axis: .vertical)
                            .textFieldStyle(.roundedBorder)
                            .lineLimit(3...6)
                    }
                }
            }
            
            // Continue Button
            Button(action: {
                navigationCoordinator.nextOnboardingStep()
            }) {
                Text("Continue")
                    .font(.headline)
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(
                        onboardingVM.canProceedFromConditions() ? 
                        Constants.Colors.primary : Color.gray
                    )
                    .cornerRadius(12)
            }
            .disabled(!onboardingVM.canProceedFromConditions())
        }
        .padding()
        .navigationBarHidden(true)
    }
}

// MARK: - Supporting Views

struct ConditionToggleButton: View {
    let condition: String
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(condition)
                .font(.caption)
                .padding(.horizontal, 12)
                .padding(.vertical, 8)
                .background(
                    isSelected ? Constants.Colors.primary : Color.gray.opacity(0.2)
                )
                .foregroundColor(isSelected ? .white : .primary)
                .cornerRadius(20)
        }
    }
}

struct RadioButton: View {
    let text: String
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            HStack {
                Image(systemName: isSelected ? "largecircle.fill.circle" : "circle")
                    .foregroundColor(isSelected ? Constants.Colors.primary : .gray)
                
                Text(text)
                    .font(.body)
                    .multilineTextAlignment(.leading)
                
                Spacer()
            }
        }
        .foregroundColor(.primary)
    }
}
```

### 4.4 Completion Screen

Create `Views/Onboarding/CompletionView.swift`:

```swift
import SwiftUI

struct CompletionView: View {
    @ObservedObject var onboardingVM: OnboardingViewModel
    @ObservedObject var navigationCoordinator: NavigationCoordinator
    
    var body: some View {
        VStack(spacing: 30) {
            Spacer()
            
            // Success Icon
            Image(systemName: "checkmark.circle.fill")
                .font(.system(size: 80))
                .foregroundColor(Constants.Colors.accent)
            
            // Title
            Text("You're All Set!")
                .font(.largeTitle)
                .fontWeight(.bold)
            
            // Description
            VStack(spacing: 16) {
                Text("Your profile has been created. You can now start tracking your symptoms, food, and activities.")
                    .font(.body)
                    .multilineTextAlignment(.center)
                    .foregroundColor(.secondary)
                
                Text("Remember: consistent tracking helps identify patterns more effectively.")
                    .font(.callout)
                    .multilineTextAlignment(.center)
                    .foregroundColor(.secondary)
                    .italic()
            }
            .padding(.horizontal)
            
            Spacer()
            
            // Complete Button
            Button(action: {
                onboardingVM.completeOnboarding()
            }) {
                Text("Start Tracking")
                    .font(.headline)
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Constants.Colors.primary)
                    .cornerRadius(12)
            }
            .padding(.horizontal)
            
            Spacer()
        }
        .padding()
        .navigationBarHidden(true)
    }
}
```

### 4.5 Onboarding Container

Create `Views/Onboarding/OnboardingFlowView.swift`:

```swift
import SwiftUI

struct OnboardingFlowView: View {
    @StateObject private var onboardingVM = OnboardingViewModel()
    @StateObject private var navigationCoordinator = NavigationCoordinator()
    
    var body: some View {
        NavigationView {
            Group {
                switch navigationCoordinator.currentOnboardingStep {
                case .welcome:
                    WelcomeView(
                        onboardingVM: onboardingVM, 
                        navigationCoordinator: navigationCoordinator
                    )
                case .conditions:
                    ConditionsView(
                        onboardingVM: onboardingVM,
                        navigationCoordinator: navigationCoordinator
                    )
                case .completion:
                    CompletionView(
                        onboardingVM: onboardingVM,
                        navigationCoordinator: navigationCoordinator
                    )
                }
            }
        }
        .navigationViewStyle(StackNavigationViewStyle())
    }
}
```

---

## Phase 5: Today View Implementation

### 5.1 Today View Model

Create `Models/ViewModels/TodayViewModel.swift`:

```swift
import SwiftUI
import Combine

class TodayViewModel: ObservableObject {
    @Published var selectedDate = Date()
    @Published var currentDailyLog: DailyLog?
    @Published var isLoading = false
    @Published var showingFoodEntry = false
    @Published var showingExerciseEntry = false
    @Published var showingActivityEntry = false
    @Published var showingSymptomEntry = false
    
    // Daily tracking values
    @Published var overallMood: Double = 5.0
    @Published var energyLevel: Double = 5.0
    @Published var stressLevel: Double = 5.0
    @Published var sleepHours: Double = 8.0
    @Published var notes: String = ""
    
    private let dataManager = DataManager()
    private var cancellables = Set<AnyCancellable>()
    private let autoSaveDebouncer = PassthroughSubject<Void, Never>()
    
    init() {
        setupAutoSave()
        loadDailyLog()
    }
    
    private func setupAutoSave() {
        // Debounce auto-save to prevent excessive Core Data writes
        autoSaveDebouncer
            .debounce(for: .milliseconds(500), scheduler: RunLoop.main)
            .sink { [weak self] in
                self?.saveCurrentValues()
            }
            .store(in: &cancellables)
        
        // Observe changes to trigger auto-save
        Publishers.CombineLatest4($overallMood, $energyLevel, $stressLevel, $sleepHours)
            .dropFirst() // Skip initial values
            .sink { [weak self] _ in
                self?.autoSaveDebouncer.send()
            }
            .store(in: &cancellables)
        
        $notes
            .dropFirst()
            .sink { [weak self] _ in
                self?.autoSaveDebouncer.send()
            }
            .store(in: &cancellables)
    }
    
    func loadDailyLog() {
        isLoading = true
        
        currentDailyLog = dataManager.getOrCreateDailyLog(for: selectedDate)
        
        if let dailyLog = currentDailyLog {
            overallMood = Double(dailyLog.overallMood)
            energyLevel = Double(dailyLog.energyLevel)
            stressLevel = Double(dailyLog.stressLevel)
            sleepHours = dailyLog.sleepHours
            notes = dailyLog.notes ?? ""
        }
        
        isLoading = false
    }
    
    private func saveCurrentValues() {
        guard let dailyLog = currentDailyLog else { return }
        
        dailyLog.overallMood = Int16(overallMood)
        dailyLog.energyLevel = Int16(energyLevel)
        dailyLog.stressLevel = Int16(stressLevel)
        dailyLog.sleepHours = sleepHours
        dailyLog.notes = notes.isEmpty ? nil : notes
        dailyLog.updatedAt = Date()
        
        dataManager.save()
    }
    
    func navigateToDate(_ date: Date) {
        selectedDate = date
        loadDailyLog()
    }
    
    func previousDay() {
        selectedDate = Calendar.current.date(byAdding: .day, value: -1, to: selectedDate) ?? selectedDate
        loadDailyLog()
    }
    
    func nextDay() {
        let tomorrow = Calendar.current.date(byAdding: .day, value: 1, to: selectedDate) ?? selectedDate
        if tomorrow <= Date() {
            selectedDate = tomorrow
            loadDailyLog()
        }
    }
    
    func getTodaysEntries() -> (symptoms: [SymptomEntry], foods: [FoodEntry], exercises: [ExerciseEntry], activities: [ActivityEntry]) {
        guard let dailyLog = currentDailyLog else {
            return ([], [], [], [])
        }
        
        let symptoms = (dailyLog.symptomEntries?.allObjects as? [SymptomEntry] ?? [])
            .sorted { $0.timestamp ?? Date() > $1.timestamp ?? Date() }
        
        let foods = (dailyLog.foodEntries?.allObjects as? [FoodEntry] ?? [])
            .sorted { $0.timestamp ?? Date() > $1.timestamp ?? Date() }
        
        let exercises = (dailyLog.exerciseEntries?.allObjects as? [ExerciseEntry] ?? [])
            .sorted { $0.timestamp ?? Date() > $1.timestamp ?? Date() }
        
        let activities = (dailyLog.activityEntries?.allObjects as? [ActivityEntry] ?? [])
            .sorted { $0.timestamp ?? Date() > $1.timestamp ?? Date() }
        
        return (symptoms, foods, exercises, activities)
    }
    
    func deleteEntry<T: NSManagedObject>(_ entry: T) {
        dataManager.delete(entry)
    }
}
```

### 5.2 Custom Slider Component

Create `Views/Components/SeveritySlider.swift`:

```swift
import SwiftUI

struct SeveritySlider: View {
    @Binding var value: Double
    let title: String
    let range: ClosedRange<Double> = 1...10
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Text(title)
                    .font(.headline)
                
                Spacer()
                
                Text("\(Int(value))")
                    .font(.title2)
                    .fontWeight(.bold)
                    .foregroundColor(colorForValue(value))
            }
            
            Slider(value: $value, in: range, step: 1.0)
                .accentColor(colorForValue(value))
                .onChange(of: value) { _ in
                    // Haptic feedback
                    let impactFeedback = UIImpactFeedbackGenerator(style: .light)
                    impactFeedback.impactOccurred()
                }
            
            HStack {
                Text("Low")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                Spacer()
                
                Text("High")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(color: .black.opacity(0.1), radius: 2, x: 0, y: 1)
    }
    
    private func colorForValue(_ value: Double) -> Color {
        let index = Int(value)
        if index >= 1 && index <= Constants.Colors.severityColors.count - 1 {
            return Constants.Colors.severityColors[index]
        }
        return .gray
    }
}
```

### 5.3 Entry Card Component

Create `Views/Components/EntryCard.swift`:

```swift
import SwiftUI

struct EntryCard: View {
    let title: String
    let subtitle: String
    let timestamp: Date
    let severity: Int?
    let onDelete: () -> Void
    
    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(title)
                    .font(.headline)
                    .lineLimit(1)
                
                Text(subtitle)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
                
                Text(timestamp.formatted(date: .omitted, time: .shortened))
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            if let severity = severity {
                VStack {
                    Text("\(severity)")
                        .font(.title2)
                        .fontWeight(.bold)
                        .foregroundColor(.white)
                        .frame(width: 30, height: 30)
                        .background(colorForSeverity(severity))
                        .clipShape(Circle())
                    
                    Text("Severity")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(color: .black.opacity(0.05), radius: 1, x: 0, y: 1)
        .swipeActions(edge: .trailing) {
            Button(role: .destructive) {
                onDelete()
            } label: {
                Image(systemName: "trash")
            }
        }
    }
    
    private func colorForSeverity(_ severity: Int) -> Color {
        if severity >= 1 && severity <= Constants.Colors.severityColors.count - 1 {
            return Constants.Colors.severityColors[severity]
        }
        return .gray
    }
}
```

### 5.4 Date Navigator Component

Create `Views/Components/DateNavigator.swift`:

```swift
import SwiftUI

struct DateNavigator: View {
    @Binding var selectedDate: Date
    let onPrevious: () -> Void
    let onNext: () -> Void
    
    private var canGoNext: Bool {
        Calendar.current.isDate(selectedDate, inSameDayAs: Date()) == false &&
        selectedDate < Date()
    }
    
    var body: some View {
        HStack {
            Button(action: onPrevious) {
                Image(systemName: "chevron.left")
                    .font(.title2)
                    .foregroundColor(Constants.Colors.primary)
            }
            
            Spacer()
            
            VStack {
                Text(selectedDate.formatted(.dateTime.weekday(.wide)))
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                
                Text(selectedDate.formatted(.dateTime.month().day()))
                    .font(.title2)
                    .fontWeight(.semibold)
            }
            
            Spacer()
            
            Button(action: onNext) {
                Image(systemName: "chevron.right")
                    .font(.title2)
                    .foregroundColor(canGoNext ? Constants.Colors.primary : .gray)
            }
            .disabled(!canGoNext)
        }
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(12)
    }
}
```

### 5.5 Main Today View

Create `Views/Today/TodayView.swift`:

```swift
import SwiftUI

struct TodayView: View {
    @StateObject private var viewModel = TodayViewModel()
    
    var body: some View {
        NavigationView {
            ScrollView {
                LazyVStack(spacing: 16) {
                    // Date Navigator
                    DateNavigator(
                        selectedDate: $viewModel.selectedDate,
                        onPrevious: viewModel.previousDay,
                        onNext: viewModel.nextDay
                    )
                    
                    if viewModel.isLoading {
                        ProgressView("Loading...")
                            .frame(height: 100)
                    } else {
                        // Tracking Sliders Section
                        trackingSlidersSection
                        
                        // Sleep & Notes Section
                        sleepAndNotesSection
                        
                        // Quick Actions Section
                        quickActionsSection
                        
                        // Today's Entries Section
                        todaysEntriesSection
                    }
                }
                .padding()
            }
            .navigationTitle("Today")
            .refreshable {
                viewModel.loadDailyLog()
            }
        }
        .sheet(isPresented: $viewModel.showingFoodEntry) {
            FoodEntryView(
                dailyLog: viewModel.currentDailyLog!,
                isPresented: $viewModel.showingFoodEntry
            )
        }
        .sheet(isPresented: $viewModel.showingExerciseEntry) {
            ExerciseEntryView(
                dailyLog: viewModel.currentDailyLog!,
                isPresented: $viewModel.showingExerciseEntry
            )
        }
        .sheet(isPresented: $viewModel.showingActivityEntry) {
            ActivityEntryView(
                dailyLog: viewModel.currentDailyLog!,
                isPresented: $viewModel.showingActivityEntry
            )
        }
        .sheet(isPresented: $viewModel.showingSymptomEntry) {
            SymptomEntryView(
                dailyLog: viewModel.currentDailyLog!,
                isPresented: $viewModel.showingSymptomEntry
            )
        }
    }
    
    private var trackingSlidersSection: some View {
        VStack(spacing: 12) {
            SeveritySlider(value: $viewModel.overallMood, title: "Overall Mood")
            SeveritySlider(value: $viewModel.energyLevel, title: "Energy Level")
            SeveritySlider(value: $viewModel.stressLevel, title: "Stress Level")
        }
    }
    
    private var sleepAndNotesSection: some View {
        VStack(spacing: 12) {
            // Sleep Hours
            VStack(alignment: .leading, spacing: 8) {
                HStack {
                    Text("Sleep Hours")
                        .font(.headline)
                    
                    Spacer()
                    
                    Text(String(format: "%.1f hrs", viewModel.sleepHours))
                        .font(.title2)
                        .fontWeight(.bold)
                        .foregroundColor(Constants.Colors.primary)
                }
                
                Slider(value: $viewModel.sleepHours, in: 0...12, step: 0.5)
                    .accentColor(Constants.Colors.primary)
                
                HStack {
                    Text("0 hrs")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Spacer()
                    
                    Text("12 hrs")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(color: .black.opacity(0.1), radius: 2, x: 0, y: 1)
            
            // Notes
            VStack(alignment: .leading, spacing: 8) {
                Text("Notes")
                    .font(.headline)
                
                TextField("How are you feeling today? Any observations?", text: $viewModel.notes, axis: .vertical)
                    .textFieldStyle(.roundedBorder)
                    .lineLimit(3...6)
            }
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(color: .black.opacity(0.1), radius: 2, x: 0, y: 1)
        }
    }
    
    private var quickActionsSection: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("Quick Actions")
                .font(.headline)
                .padding(.horizontal)
            
            LazyVGrid(columns: [
                GridItem(.flexible()),
                GridItem(.flexible())
            ], spacing: 12) {
                QuickActionButton(
                    title: "Log Food",
                    icon: "fork.knife",
                    color: .orange
                ) {
                    viewModel.showingFoodEntry = true
                }
                
                QuickActionButton(
                    title: "Log Exercise",
                    icon: "figure.run",
                    color: .green
                ) {
                    viewModel.showingExerciseEntry = true
                }
                
                QuickActionButton(
                    title: "Log Activity",
                    icon: "calendar",
                    color: .blue
                ) {
                    viewModel.showingActivityEntry = true
                }
                
                QuickActionButton(
                    title: "Log Symptom",
                    icon: "heart.text.square",
                    color: .red
                ) {
                    viewModel.showingSymptomEntry = true
                }
            }
            .padding(.horizontal)
        }
    }
    
    private var todaysEntriesSection: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("Today's Entries")
                .font(.headline)
                .padding(.horizontal)
            
            let entries = viewModel.getTodaysEntries()
            
            if entries.symptoms.isEmpty && entries.foods.isEmpty && 
               entries.exercises.isEmpty && entries.activities.isEmpty {
                Text("No entries yet today. Use the quick actions above to start tracking!")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .multilineTextAlignment(.center)
                    .padding()
                    .frame(maxWidth: .infinity)
                    .background(Color(.systemGray6))
                    .cornerRadius(12)
                    .padding(.horizontal)
            } else {
                LazyVStack(spacing: 8) {
                    // Symptoms
                    ForEach(entries.symptoms, id: \.id) { symptom in
                        EntryCard(
                            title: symptom.symptomName ?? "Unknown Symptom",
                            subtitle: symptom.notes ?? "No notes",
                            timestamp: symptom.timestamp ?? Date(),
                            severity: Int(symptom.severity)
                        ) {
                            viewModel.deleteEntry(symptom)
                        }
                    }
                    
                    // Foods
                    ForEach(entries.foods, id: \.id) { food in
                        EntryCard(
                            title: food.foodName ?? "Unknown Food",
                            subtitle: food.mealType ?? "Unknown meal",
                            timestamp: food.timestamp ?? Date(),
                            severity: nil
                        ) {
                            viewModel.deleteEntry(food)
                        }
                    }
                    
                    // Exercises
                    ForEach(entries.exercises, id: \.id) { exercise in
                        EntryCard(
                            title: exercise.exerciseType ?? "Unknown Exercise",
                            subtitle: "\(exercise.duration) minutes",
                            timestamp: exercise.timestamp ?? Date(),
                            severity: Int(exercise.intensity)
                        ) {
                            viewModel.deleteEntry(exercise)
                        }
                    }
                    
                    // Activities
                    ForEach(entries.activities, id: \.id) { activity in
                        EntryCard(
                            title: activity.activityType ?? "Unknown Activity",
                            subtitle: "\(activity.duration) minutes",
                            timestamp: activity.timestamp ?? Date(),
                            severity: nil
                        ) {
                            viewModel.deleteEntry(activity)
                        }
                    }
                }
                .padding(.horizontal)
            }
        }
    }
}

// MARK: - Quick Action Button

struct QuickActionButton: View {
    let title: String
    let icon: String
    let color: Color
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            VStack(spacing: 8) {
                Image(systemName: icon)
                    .font(.title2)
                    .foregroundColor(.white)
                    .frame(width: 40, height: 40)
                    .background(color)
                    .clipShape(Circle())
                
                Text(title)
                    .font(.caption)
                    .fontWeight(.medium)
                    .multilineTextAlignment(.center)
            }
            .frame(maxWidth: .infinity)
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(color: .black.opacity(0.05), radius: 1, x: 0, y: 1)
        }
        .foregroundColor(.primary)
    }
}
```

---

## Phase 6: Entry Forms Implementation

### 6.1 Food Entry View

Create `Views/Today/FoodEntryView.swift`:

```swift
import SwiftUI

struct FoodEntryView: View {
    let dailyLog: DailyLog
    @Binding var isPresented: Bool
    
    @State private var foodName = ""
    @State private var selectedMealType = "Breakfast"
    @State private var selectedTime = Date()
    @State private var notes = ""
    
    @StateObject private var dataManager = DataManager()
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("Food Details")) {
                    TextField("Food name", text: $foodName)
                    
                    Picker("Meal Type", selection: $selectedMealType) {
                        ForEach(Constants.mealTypes, id: \.self) { mealType in
                            Text(mealType).tag(mealType)
                        }
                    }
                    .pickerStyle(.segmented)
                    
                    DatePicker("Time", selection: $selectedTime, displayedComponents: .hourAndMinute)
                }
                
                Section(header: Text("Notes (Optional)")) {
                    TextField("Additional notes...", text: $notes, axis: .vertical)
                        .lineLimit(3...6)
                }
            }
            .navigationTitle("Log Food")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Cancel") {
                        isPresented = false
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Save") {
                        saveFoodEntry()
                    }
                    .disabled(foodName.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty)
                }
            }
        }
    }
    
    private func saveFoodEntry() {
        let _ = dataManager.createFoodEntry(
            name: foodName.trimmingCharacters(in: .whitespacesAndNewlines),
            mealType: selectedMealType,
            timestamp: selectedTime,
            notes: notes.isEmpty ? nil : notes,
            dailyLog: dailyLog
        )
        
        isPresented = false
    }
}
```

### 6.2 Exercise Entry View

Create `Views/Today/ExerciseEntryView.swift`:

```swift
import SwiftUI

struct ExerciseEntryView: View {
    let dailyLog: DailyLog
    @Binding var isPresented: Bool
    
    @State private var selectedExerciseType = "Walking"
    @State private var duration: Double = 30
    @State private var intensity: Double = 5
    @State private var selectedTime = Date()
    @State private var notes = ""
    
    @StateObject private var dataManager = DataManager()
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("Exercise Details")) {
                    Picker("Exercise Type", selection: $selectedExerciseType) {
                        ForEach(Constants.exerciseTypes, id: \.self) { type in
                            Text(type).tag(type)
                        }
                    }
                    
                    VStack(alignment: .leading, spacing: 8) {
                        HStack {
                            Text("Duration")
                            Spacer()
                            Text("\(Int(duration)) minutes")
                                .fontWeight(.semibold)
                        }
                        
                        Slider(value: $duration, in: 5...180, step: 5)
                            .accentColor(Constants.Colors.primary)
                    }
                    
                    VStack(alignment: .leading, spacing: 8) {
                        HStack {
                            Text("Intensity")
                            Spacer()
                            Text("\(Int(intensity))/10")
                                .fontWeight(.semibold)
                                .foregroundColor(colorForIntensity(Int(intensity)))
                        }
                        
                        Slider(value: $intensity, in: 1...10, step: 1)
                            .accentColor(colorForIntensity(Int(intensity)))
                    }
                    
                    DatePicker("Time", selection: $selectedTime, displayedComponents: .hourAndMinute)
                }
                
                Section(header: Text("Notes (Optional)")) {
                    TextField("How did it feel? Any observations?", text: $notes, axis: .vertical)
                        .lineLimit(3...6)
                }
            }
            .navigationTitle("Log Exercise")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Cancel") {
                        isPresented = false
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Save") {
                        saveExerciseEntry()
                    }
                }
            }
        }
    }
    
    private func saveExerciseEntry() {
        let _ = dataManager.createExerciseEntry(
            type: selectedExerciseType,
            duration: Int(duration),
            intensity: Int(intensity),
            timestamp: selectedTime,
            notes: notes.isEmpty ? nil : notes,
            dailyLog: dailyLog
        )
        
        isPresented = false
    }
    
    private func colorForIntensity(_ intensity: Int) -> Color {
        if intensity >= 1 && intensity <= Constants.Colors.severityColors.count - 1 {
            return Constants.Colors.severityColors[intensity]
        }
        return .gray
    }
}
```

### 6.3 Activity Entry View

Create `Views/Today/ActivityEntryView.swift`:

```swift
import SwiftUI

struct ActivityEntryView: View {
    let dailyLog: DailyLog
    @Binding var isPresented: Bool
    
    @State private var selectedActivityType = "Work"
    @State private var duration: Double = 60
    @State private var selectedTime = Date()
    @State private var notes = ""
    
    @StateObject private var dataManager = DataManager()
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("Activity Details")) {
                    Picker("Activity Type", selection: $selectedActivityType) {
                        ForEach(Constants.activityTypes, id: \.self) { type in
                            Text(type).tag(type)
                        }
                    }
                    
                    VStack(alignment: .leading, spacing: 8) {
                        HStack {
                            Text("Duration")
                            Spacer()
                            Text("\(Int(duration)) minutes")
                                .fontWeight(.semibold)
                        }
                        
                        Slider(value: $duration, in: 15...480, step: 15)
                            .accentColor(Constants.Colors.primary)
                    }
                    
                    DatePicker("Time", selection: $selectedTime, displayedComponents: .hourAndMinute)
                }
                
                Section(header: Text("Notes (Optional)")) {
                    TextField("Any observations about this activity?", text: $notes, axis: .vertical)
                        .lineLimit(3...6)
                }
            }
            .navigationTitle("Log Activity")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Cancel") {
                        isPresented = false
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Save") {
                        saveActivityEntry()
                    }
                }
            }
        }
    }
    
    private func saveActivityEntry() {
        let _ = dataManager.createActivityEntry(
            type: selectedActivityType,
            duration: Int(duration),
            timestamp: selectedTime,
            notes: notes.isEmpty ? nil : notes,
            dailyLog: dailyLog
        )
        
        isPresented = false
    }
}
```

### 6.4 Symptom Entry View

Create `Views/Today/SymptomEntryView.swift`:

```swift
import SwiftUI

struct SymptomEntryView: View {
    let dailyLog: DailyLog
    @Binding var isPresented: Bool
    
    @State private var symptomName = ""
    @State private var severity: Double = 5
    @State private var selectedTime = Date()
    @State private var notes = ""
    
    @StateObject private var dataManager = DataManager()
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("Symptom Details")) {
                    TextField("Symptom name (e.g., headache, fatigue)", text: $symptomName)
                    
                    VStack(alignment: .leading, spacing: 8) {
                        HStack {
                            Text("Severity")
                            Spacer()
                            Text("\(Int(severity))/10")
                                .fontWeight(.semibold)
                                .foregroundColor(colorForSeverity(Int(severity)))
                        }
                        
                        Slider(value: $severity, in: 1...10, step: 1)
                            .accentColor(colorForSeverity(Int(severity)))
                            .onChange(of: severity) { _ in
                                let impactFeedback = UIImpactFeedbackGenerator(style: .light)
                                impactFeedback.impactOccurred()
                            }
                        
                        HStack {
                            Text("Mild")
                                .font(.caption)
                                .foregroundColor(.secondary)
                            
                            Spacer()
                            
                            Text("Severe")
                                .font(.caption)
                                .foregroundColor(.secondary)
                        }
                    }
                    
                    DatePicker("Time", selection: $selectedTime, displayedComponents: .hourAndMinute)
                }
                
                Section(header: Text("Notes (Optional)")) {
                    TextField("Additional details about this symptom...", text: $notes, axis: .vertical)
                        .lineLimit(3...6)
                }
            }
            .navigationTitle("Log Symptom")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Cancel") {
                        isPresented = false
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Save") {
                        saveSymptomEntry()
                    }
                    .disabled(symptomName.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty)
                }
            }
        }
    }
    
    private func saveSymptomEntry() {
        let _ = dataManager.createSymptomEntry(
            name: symptomName.trimmingCharacters(in: .whitespacesAndNewlines),
            severity: Int(severity),
            timestamp: selectedTime,
            notes: notes.isEmpty ? nil : notes,
            dailyLog: dailyLog
        )
        
        isPresented = false
    }
    
    private func colorForSeverity(_ severity: Int) -> Color {
        if severity >= 1 && severity <= Constants.Colors.severityColors.count - 1 {
            return Constants.Colors.severityColors[severity]
        }
        return .gray
    }
}
```

---

## Phase 7: Timeline View Implementation

### 7.1 Timeline View Model

Create `Models/ViewModels/TimelineViewModel.swift`:

```swift
import SwiftUI
import Combine
import CoreData

class TimelineViewModel: ObservableObject {
    @Published var dailyLogs: [DailyLog] = []
    @Published var isLoading = false
    @Published var selectedDayLog: DailyLog?
    @Published var showingDayDetail = false
    
    private let dataManager = DataManager()
    private var cancellables = Set<AnyCancellable>()
    private let pageSize = 20
    private var currentPage = 0
    private var hasMoreData = true
    
    init() {
        loadInitialData()
    }
    
    func loadInitialData() {
        isLoading = true
        currentPage = 0
        hasMoreData = true
        
        let sortDescriptor = NSSortDescriptor(key: "date", ascending: false)
        let logs = dataManager.fetch(
            DailyLog.self,
            sortDescriptors: [sortDescriptor],
            fetchLimit: pageSize
        )
        
        dailyLogs = logs
        hasMoreData = logs.count == pageSize
        isLoading = false
    }
    
    func loadMoreData() {
        guard !isLoading && hasMoreData else { return }
        
        isLoading = true
        currentPage += 1
        
        let sortDescriptor = NSSortDescriptor(key: "date", ascending: false)
        let logs = dataManager.fetch(
            DailyLog.self,
            sortDescriptors: [sortDescriptor],
            fetchLimit: pageSize
        )
        
        // Skip already loaded items
        let newLogs = Array(logs.dropFirst(currentPage * pageSize))
        dailyLogs.append(contentsOf: newLogs)
        hasMoreData = newLogs.count == pageSize
        isLoading = false
    }
    
    func dayQuality(for dailyLog: DailyLog) -> DayQuality {
        let avgMood = Double(dailyLog.overallMood + dailyLog.energyLevel) / 2.0
        let stressLevel = Double(dailyLog.stressLevel)
        
        // Calculate overall quality considering mood, energy, and inverse stress
        let qualityScore = (avgMood + (11 - stressLevel)) / 2.0
        
        if qualityScore >= 7.5 {
            return .good
        } else if qualityScore >= 5.0 {
            return .neutral
        } else {
            return .poor
        }
    }
    
    func entryCount(for dailyLog: DailyLog) -> Int {
        let symptomCount = dailyLog.symptomEntries?.count ?? 0
        let foodCount = dailyLog.foodEntries?.count ?? 0
        let exerciseCount = dailyLog.exerciseEntries?.count ?? 0
        let activityCount = dailyLog.activityEntries?.count ?? 0
        
        return symptomCount + foodCount + exerciseCount + activityCount
    }
    
    func selectDay(_ dailyLog: DailyLog) {
        selectedDayLog = dailyLog
        showingDayDetail = true
    }
    
    func refreshData() {
        loadInitialData()
    }
}

enum DayQuality {
    case good, neutral, poor
    
    var color: Color {
        switch self {
        case .good:
            return .green
        case .neutral:
            return .orange
        case .poor:
            return .red
        }
    }
    
    var description: String {
        switch self {
        case .good:
            return "Good Day"
        case .neutral:
            return "Okay Day"
        case .poor:
            return "Difficult Day"
        }
    }
}
```

### 7.2 Day Detail View Model

Create `Models/ViewModels/DayDetailViewModel.swift`:

```swift
import SwiftUI
import Combine

class DayDetailViewModel: ObservableObject {
    @Published var timelineEntries: [TimelineEntry] = []
    @Published var isLoading = false
    
    private let dataManager = DataManager()
    private let dailyLog: DailyLog
    
    init(dailyLog: DailyLog) {
        self.dailyLog = dailyLog
        loadTimelineEntries()
    }
    
    func loadTimelineEntries() {
        isLoading = true
        
        var entries: [TimelineEntry] = []
        
        // Add symptom entries
        if let symptoms = dailyLog.symptomEntries?.allObjects as? [SymptomEntry] {
            entries.append(contentsOf: symptoms.map { symptom in
                TimelineEntry(
                    id: symptom.id?.uuidString ?? UUID().uuidString,
                    type: .symptom,
                    title: symptom.symptomName ?? "Unknown Symptom",
                    subtitle: "Severity: \(symptom.severity)/10",
                    timestamp: symptom.timestamp ?? Date(),
                    severity: Int(symptom.severity),
                    notes: symptom.notes,
                    originalEntry: symptom
                )
            })
        }
        
        // Add food entries
        if let foods = dailyLog.foodEntries?.allObjects as? [FoodEntry] {
            entries.append(contentsOf: foods.map { food in
                TimelineEntry(
                    id: food.id?.uuidString ?? UUID().uuidString,
                    type: .food,
                    title: food.foodName ?? "Unknown Food",
                    subtitle: food.mealType ?? "Unknown meal",
                    timestamp: food.timestamp ?? Date(),
                    severity: nil,
                    notes: food.notes,
                    originalEntry: food
                )
            })
        }
        
        // Add exercise entries
        if let exercises = dailyLog.exerciseEntries?.allObjects as? [ExerciseEntry] {
            entries.append(contentsOf: exercises.map { exercise in
                TimelineEntry(
                    id: exercise.id?.uuidString ?? UUID().uuidString,
                    type: .exercise,
                    title: exercise.exerciseType ?? "Unknown Exercise",
                    subtitle: "\(exercise.duration) min • Intensity: \(exercise.intensity)/10",
                    timestamp: exercise.timestamp ?? Date(),
                    severity: Int(exercise.intensity),
                    notes: exercise.notes,
                    originalEntry: exercise
                )
            })
        }
        
        // Add activity entries
        if let activities = dailyLog.activityEntries?.allObjects as? [ActivityEntry] {
            entries.append(contentsOf: activities.map { activity in
                TimelineEntry(
                    id: activity.id?.uuidString ?? UUID().uuidString,
                    type: .activity,
                    title: activity.activityType ?? "Unknown Activity",
                    subtitle: "\(activity.duration) minutes",
                    timestamp: activity.timestamp ?? Date(),
                    severity: nil,
                    notes: activity.notes,
                    originalEntry: activity
                )
            })
        }
        
        // Sort by timestamp (most recent first)
        timelineEntries = entries.sorted { $0.timestamp > $1.timestamp }
        isLoading = false
    }
    
    func deleteEntry(_ entry: TimelineEntry) {
        if let managedObject = entry.originalEntry as? NSManagedObject {
            dataManager.delete(managedObject)
            loadTimelineEntries()
        }
    }
    
    func groupedEntries() -> [(String, [TimelineEntry])] {
        let calendar = Calendar.current
        let grouped = Dictionary(grouping: timelineEntries) { entry in
            calendar.dateInterval(of: .hour, for: entry.timestamp)?.start ?? entry.timestamp
        }
        
        return grouped.map { (hour, entries) in
            let formatter = DateFormatter()
            formatter.dateFormat = "h:mm a"
            return (formatter.string(from: hour), entries.sorted { $0.timestamp > $1.timestamp })
        }.sorted { $0.0 > $1.0 }
    }
}

struct TimelineEntry: Identifiable {
    let id: String
    let type: EntryType
    let title: String
    let subtitle: String
    let timestamp: Date
    let severity: Int?
    let notes: String?
    let originalEntry: Any
    
    enum EntryType {
        case symptom, food, exercise, activity
        
        var icon: String {
            switch self {
            case .symptom:
                return "heart.text.square"
            case .food:
                return "fork.knife"
            case .exercise:
                return "figure.run"
            case .activity:
                return "calendar"
            }
        }
        
        var color: Color {
            switch self {
            case .symptom:
                return .red
            case .food:
                return .orange
            case .exercise:
                return .green
            case .activity:
                return .blue
            }
        }
    }
}
```

### 7.3 Timeline List View

Create `Views/Timeline/TimelineView.swift`:

```swift
import SwiftUI

struct TimelineView: View {
    @StateObject private var viewModel = TimelineViewModel()
    
    var body: some View {
        NavigationView {
            List {
                ForEach(viewModel.dailyLogs, id: \.id) { dailyLog in
                    DaySummaryCard(
                        dailyLog: dailyLog,
                        quality: viewModel.dayQuality(for: dailyLog),
                        entryCount: viewModel.entryCount(for: dailyLog)
                    ) {
                        viewModel.selectDay(dailyLog)
                    }
                    .listRowSeparator(.hidden)
                    .listRowInsets(EdgeInsets(top: 4, leading: 16, bottom: 4, trailing: 16))
                    .onAppear {
                        // Load more data when approaching the end
                        if dailyLog == viewModel.dailyLogs.last {
                            viewModel.loadMoreData()
                        }
                    }
                }
                
                if viewModel.isLoading {
                    HStack {
                        Spacer()
                        ProgressView()
                        Spacer()
                    }
                    .listRowSeparator(.hidden)
                }
            }
            .listStyle(.plain)
            .navigationTitle("Timeline")
            .refreshable {
                viewModel.refreshData()
            }
            .sheet(isPresented: $viewModel.showingDayDetail) {
                if let selectedDay = viewModel.selectedDayLog {
                    DayDetailView(dailyLog: selectedDay)
                }
            }
        }
    }
}

struct DaySummaryCard: View {
    let dailyLog: DailyLog
    let quality: DayQuality
    let entryCount: Int
    let onTap: () -> Void
    
    var body: some View {
        Button(action: onTap) {
            VStack(spacing: 12) {
                // Header with date and quality
                HStack {
                    VStack(alignment: .leading, spacing: 4) {
                        Text(dailyLog.date?.formatted(.dateTime.weekday(.wide).month().day()) ?? "Unknown Date")
                            .font(.headline)
                            .foregroundColor(.primary)
                        
                        Text(quality.description)
                            .font(.subheadline)
                            .foregroundColor(quality.color)
                    }
                    
                    Spacer()
                    
                    // Quality indicator
                    Circle()
                        .fill(quality.color)
                        .frame(width: 12, height: 12)
                }
                
                // Metrics row
                HStack(spacing: 20) {
                    MetricView(
                        title: "Mood",
                        value: Int(dailyLog.overallMood),
                        color: colorForValue(Int(dailyLog.overallMood))
                    )
                    
                    MetricView(
                        title: "Energy",
                        value: Int(dailyLog.energyLevel),
                        color: colorForValue(Int(dailyLog.energyLevel))
                    )
                    
                    MetricView(
                        title: "Stress",
                        value: Int(dailyLog.stressLevel),
                        color: colorForValue(11 - Int(dailyLog.stressLevel)) // Invert stress for color
                    )
                    
                    Spacer()
                    
                    VStack {
                        Text("\(entryCount)")
                            .font(.title2)
                            .fontWeight(.bold)
                            .foregroundColor(.secondary)
                        
                        Text("entries")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
            }
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(color: .black.opacity(0.05), radius: 2, x: 0, y: 1)
        }
        .foregroundColor(.primary)
    }
    
    private func colorForValue(_ value: Int) -> Color {
        if value >= 1 && value <= Constants.Colors.severityColors.count - 1 {
            return Constants.Colors.severityColors[value]
        }
        return .gray
    }
}

struct MetricView: View {
    let title: String
    let value: Int
    let color: Color
    
    var body: some View {
        VStack(spacing: 2) {
            Text("\(value)")
                .font(.title3)
                .fontWeight(.semibold)
                .foregroundColor(color)
            
            Text(title)
                .font(.caption)
                .foregroundColor(.secondary)
        }
    }
}
```

### 7.4 Day Detail View

Create `Views/Timeline/DayDetailView.swift`:

```swift
import SwiftUI

struct DayDetailView: View {
    let dailyLog: DailyLog
    @StateObject private var viewModel: DayDetailViewModel
    @Environment(\.dismiss) private var dismiss
    
    init(dailyLog: DailyLog) {
        self.dailyLog = dailyLog
        self._viewModel = StateObject(wrappedValue: DayDetailViewModel(dailyLog: dailyLog))
    }
    
    var body: some View {
        NavigationView {
            ScrollView {
                LazyVStack(alignment: .leading, spacing: 16) {
                    // Day overview header
                    dayOverviewSection
                    
                    // Timeline entries
                    if viewModel.isLoading {
                        ProgressView("Loading timeline...")
                            .frame(maxWidth: .infinity)
                            .padding()
                    } else if viewModel.timelineEntries.isEmpty {
                        emptyStateView
                    } else {
                        timelineSection
                    }
                }
                .padding()
            }
            .navigationTitle(dailyLog.date?.formatted(.dateTime.weekday(.wide).month().day()) ?? "Day Detail")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Done") {
                        dismiss()
                    }
                }
            }
        }
    }
    
    private var dayOverviewSection: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("Day Overview")
                .font(.headline)
            
            VStack(spacing: 12) {
                HStack {
                    MetricOverviewCard(
                        title: "Mood",
                        value: Int(dailyLog.overallMood),
                        maxValue: 10,
                        color: colorForValue(Int(dailyLog.overallMood))
                    )
                    
                    MetricOverviewCard(
                        title: "Energy",
                        value: Int(dailyLog.energyLevel),
                        maxValue: 10,
                        color: colorForValue(Int(dailyLog.energyLevel))
                    )
                }
                
                HStack {
                    MetricOverviewCard(
                        title: "Stress",
                        value: Int(dailyLog.stressLevel),
                        maxValue: 10,
                        color: colorForValue(11 - Int(dailyLog.stressLevel))
                    )
                    
                    MetricOverviewCard(
                        title: "Sleep",
                        value: dailyLog.sleepHours,
                        maxValue: 12,
                        color: Constants.Colors.primary,
                        suffix: "hrs"
                    )
                }
            }
            
            if let notes = dailyLog.notes, !notes.isEmpty {
                VStack(alignment: .leading, spacing: 8) {
                    Text("Notes")
                        .font(.subheadline)
                        .fontWeight(.semibold)
                    
                    Text(notes)
                        .font(.body)
                        .foregroundColor(.secondary)
                        .padding()
                        .background(Color(.systemGray6))
                        .cornerRadius(8)
                }
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(color: .black.opacity(0.05), radius: 2, x: 0, y: 1)
    }
    
    private var timelineSection: some View {
        VStack(alignment: .leading, spacing: 16) {
            Text("Timeline")
                .font(.headline)
                .padding(.horizontal)
            
            ForEach(viewModel.groupedEntries(), id: \.0) { timeGroup, entries in
                TimelineGroupView(timeLabel: timeGroup, entries: entries) { entry in
                    viewModel.deleteEntry(entry)
                }
            }
        }
    }
    
    private var emptyStateView: some View {
        VStack(spacing: 12) {
            Image(systemName: "clock")
                .font(.system(size: 48))
                .foregroundColor(.secondary)
            
            Text("No entries for this day")
                .font(.headline)
                .foregroundColor(.secondary)
            
            Text("Entries you log will appear here in chronological order.")
                .font(.subheadline)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
        }
        .padding()
        .frame(maxWidth: .infinity)
    }
    
    private func colorForValue(_ value: Int) -> Color {
        if value >= 1 && value <= Constants.Colors.severityColors.count - 1 {
            return Constants.Colors.severityColors[value]
        }
        return .gray
    }
}

struct MetricOverviewCard: View {
    let title: String
    let value: Int
    let maxValue: Int
    let color: Color
    let suffix: String
    
    init(title: String, value: Int, maxValue: Int, color: Color, suffix: String = "") {
        self.title = title
        self.value = value
        self.maxValue = maxValue
        self.color = color
        self.suffix = suffix
    }
    
    init(title: String, value: Double, maxValue: Double, color: Color, suffix: String = "") {
        self.title = title
        self.value = Int(value * 10) // Convert to int for display
        self.maxValue = Int(maxValue * 10)
        self.color = color
        self.suffix = suffix
    }
    
    var body: some View {
        VStack(spacing: 8) {
            Text(title)
                .font(.caption)
                .foregroundColor(.secondary)
            
            Text(suffix.isEmpty ? "\(value)" : String(format: "%.1f%@", Double(value)/10.0, suffix))
                .font(.title2)
                .fontWeight(.bold)
                .foregroundColor(color)
            
            // Progress bar
            ProgressView(value: Double(value), total: Double(maxValue))
                .progressViewStyle(LinearProgressViewStyle(tint: color))
                .scaleEffect(x: 1, y: 2, anchor: .center)
        }
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(8)
        .frame(maxWidth: .infinity)
    }
}

struct TimelineGroupView: View {
    let timeLabel: String
    let entries: [TimelineEntry]
    let onDelete: (TimelineEntry) -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // Time label
            HStack {
                Text(timeLabel)
                    .font(.subheadline)
                    .fontWeight(.semibold)
                    .foregroundColor(.secondary)
                
                Rectangle()
                    .fill(Color.secondary.opacity(0.3))
                    .frame(height: 1)
            }
            .padding(.horizontal)
            
            // Entries for this time
            ForEach(entries) { entry in
                TimelineEntryRow(entry: entry) {
                    onDelete(entry)
                }
                .padding(.horizontal)
            }
        }
    }
}

struct TimelineEntryRow: View {
    let entry: TimelineEntry
    let onDelete: () -> Void
    
    var body: some View {
        HStack(spacing: 12) {
            // Type icon
            Image(systemName: entry.type.icon)
                .font(.title3)
                .foregroundColor(.white)
                .frame(width: 32, height: 32)
                .background(entry.type.color)
                .clipShape(Circle())
            
            // Content
            VStack(alignment: .leading, spacing: 4) {
                HStack {
                    Text(entry.title)
                        .font(.headline)
                        .lineLimit(1)
                    
                    Spacer()
                    
                    if let severity = entry.severity {
                        Text("\(severity)")
                            .font(.caption)
                            .fontWeight(.bold)
                            .foregroundColor(.white)
                            .frame(width: 20, height: 20)
                            .background(colorForSeverity(severity))
                            .clipShape(Circle())
                    }
                    
                    Text(entry.timestamp.formatted(date: .omitted, time: .shortened))
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Text(entry.subtitle)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
                
                if let notes = entry.notes, !notes.isEmpty {
                    Text(notes)
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .italic()
                        .lineLimit(3)
                }
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(8)
        .shadow(color: .black.opacity(0.05), radius: 1, x: 0, y: 1)
        .swipeActions(edge: .trailing) {
            Button(role: .destructive) {
                onDelete()
            } label: {
                Image(systemName: "trash")
            }
        }
    }
    
    private func colorForSeverity(_ severity: Int) -> Color {
        if severity >= 1 && severity <= Constants.Colors.severityColors.count - 1 {
            return Constants.Colors.severityColors[severity]
        }
        return .gray
    }
}
```

---

## Phase 8: Correlation Engine Implementation

### 8.1 Correlation Engine Service

Create `Services/CorrelationEngine.swift`:

```swift
import Foundation
import CoreData
import Combine

class CorrelationEngine: ObservableObject {
    @Published var isCalculating = false
    @Published var correlationResults: [CorrelationResult] = []
    @Published var lastCalculationDate: Date?
    
    private let dataManager = DataManager()
    private var cancellables = Set<AnyCancellable>()
    
    // Minimum requirements for correlation analysis
    private let minimumDaysRequired = 7
    private let minimumOccurrences = 3
    private let significanceThreshold = 0.25 // 25% above baseline
    
    func calculateCorrelations() async {
        await MainActor.run {
            isCalculating = true
        }
        
        defer {
            Task { @MainActor in
                isCalculating = false
                lastCalculationDate = Date()
            }
        }
        
        // Get data from last 30 days
        let thirtyDaysAgo = Calendar.current.date(byAdding: .day, value: -30, to: Date()) ?? Date()
        let predicate = NSPredicate(format: "date >= %@", thirtyDaysAgo as NSDate)
        let sortDescriptor = NSSortDescriptor(key: "date", ascending: false)
        
        let dailyLogs = dataManager.fetch(
            DailyLog.self,
            predicate: predicate,
            sortDescriptors: [sortDescriptor]
        )
        
        guard dailyLogs.count >= minimumDaysRequired else {
            await MainActor.run {
                correlationResults = []
            }
            return
        }
        
        let results = await calculateAllCorrelations(from: dailyLogs)
        
        await MainActor.run {
            correlationResults = results
        }
    }
    
    private func calculateAllCorrelations(from dailyLogs: [DailyLog]) async -> [CorrelationResult] {
        var results: [CorrelationResult] = []
        
        // Identify flare days (average symptom severity > 7)
        let flareDays = identifyFlareDays(from: dailyLogs)
        let nonFlareDays = dailyLogs.filter { !flareDays.contains($0) }
        
        guard flareDays.count >= minimumOccurrences else {
            return results
        }
        
        // Analyze food correlations
        results.append(contentsOf: await analyzeFoodCorrelations(flareDays: flareDays, nonFlareDays: nonFlareDays))
        
        // Analyze exercise correlations
        results.append(contentsOf: await analyzeExerciseCorrelations(flareDays: flareDays, nonFlareDays: nonFlareDays))
        
        // Analyze activity correlations
        results.append(contentsOf: await analyzeActivityCorrelations(flareDays: flareDays, nonFlareDays: nonFlareDays))
        
        // Analyze sleep correlations
        results.append(contentsOf: await analyzeSleepCorrelations(flareDays: flareDays, nonFlareDays: nonFlareDays))
        
        // Analyze stress correlations
        results.append(contentsOf: await analyzeStressCorrelations(flareDays: flareDays, nonFlareDays: nonFlareDays))
        
        return results.sorted { $0.strength > $1.strength }
    }
    
    private func identifyFlareDays(from dailyLogs: [DailyLog]) -> [DailyLog] {
        return dailyLogs.filter { dailyLog in
            guard let symptoms = dailyLog.symptomEntries?.allObjects as? [SymptomEntry],
                  !symptoms.isEmpty else {
                return false
            }
            
            let averageSeverity = symptoms.map { Double($0.severity) }.reduce(0, +) / Double(symptoms.count)
            return averageSeverity > 7.0
        }
    }
    
    private func analyzeFoodCorrelations(flareDays: [DailyLog], nonFlareDays: [DailyLog]) async -> [CorrelationResult] {
        var foodCorrelations: [String: CorrelationData] = [:]
        
        // Collect all food items
        let allFoods = Set(
            (flareDays + nonFlareDays).flatMap { dailyLog in
                (dailyLog.foodEntries?.allObjects as? [FoodEntry] ?? []).compactMap { $0.foodName }
            }
        )
        
        for food in allFoods {
            let flareOccurrences = countFoodOccurrences(food: food, in: flareDays, lookbackHours: 6)
            let nonFlareOccurrences = countFoodOccurrences(food: food, in: nonFlareDays, lookbackHours: 6)
            
            guard flareOccurrences >= minimumOccurrences else { continue }
            
            let flareFrequency = Double(flareOccurrences) / Double(flareDays.count)
            let nonFlareFrequency = Double(nonFlareOccurrences) / Double(max(nonFlareDays.count, 1))
            
            let correlation = (flareFrequency - nonFlareFrequency) / max(nonFlareFrequency, 0.1)
            
            if abs(correlation) > significanceThreshold {
                foodCorrelations[food] = CorrelationData(
                    factor: food,
                    strength: abs(correlation),
                    isProtective: correlation < 0,
                    occurrences: flareOccurrences + nonFlareOccurrences,
                    confidence: calculateConfidence(occurrences: flareOccurrences + nonFlareOccurrences)
                )
            }
        }
        
        return foodCorrelations.values.map { data in
            CorrelationResult(
                type: .food,
                factor: data.factor,
                strength: data.strength,
                isProtective: data.isProtective,
                confidence: data.confidence,
                description: generateFoodDescription(data)
            )
        }
    }
    
    private func analyzeExerciseCorrelations(flareDays: [DailyLog], nonFlareDays: [DailyLog]) async -> [CorrelationResult] {
        var results: [CorrelationResult] = []
        
        // Analyze exercise types
        let allExerciseTypes = Set(
            (flareDays + nonFlareDays).flatMap { dailyLog in
                (dailyLog.exerciseEntries?.allObjects as? [ExerciseEntry] ?? []).compactMap { $0.exerciseType }
            }
        )
        
        for exerciseType in allExerciseTypes {
            let flareOccurrences = countExerciseOccurrences(type: exerciseType, in: flareDays)
            let nonFlareOccurrences = countExerciseOccurrences(type: exerciseType, in: nonFlareDays)
            
            guard flareOccurrences >= minimumOccurrences else { continue }
            
            let flareFrequency = Double(flareOccurrences) / Double(flareDays.count)
            let nonFlareFrequency = Double(nonFlareOccurrences) / Double(max(nonFlareDays.count, 1))
            
            let correlation = (flareFrequency - nonFlareFrequency) / max(nonFlareFrequency, 0.1)
            
            if abs(correlation) > significanceThreshold {
                results.append(CorrelationResult(
                    type: .exercise,
                    factor: exerciseType,
                    strength: abs(correlation),
                    isProtective: correlation < 0,
                    confidence: calculateConfidence(occurrences: flareOccurrences + nonFlareOccurrences),
                    description: generateExerciseDescription(exerciseType, correlation < 0)
                ))
            }
        }
        
        // Analyze high-intensity exercise pattern
        let highIntensityFlareOccurrences = countHighIntensityExercise(in: flareDays)
        let highIntensityNonFlareOccurrences = countHighIntensityExercise(in: nonFlareDays)
        
        if highIntensityFlareOccurrences >= minimumOccurrences {
            let flareFreq = Double(highIntensityFlareOccurrences) / Double(flareDays.count)
            let nonFlareFreq = Double(highIntensityNonFlareOccurrences) / Double(max(nonFlareDays.count, 1))
            let correlation = (flareFreq - nonFlareFreq) / max(nonFlareFreq, 0.1)
            
            if abs(correlation) > significanceThreshold {
                results.append(CorrelationResult(
                    type: .exercise,
                    factor: "High-intensity exercise",
                    strength: abs(correlation),
                    isProtective: correlation < 0,
                    confidence: calculateConfidence(occurrences: highIntensityFlareOccurrences + highIntensityNonFlareOccurrences),
                    description: "High-intensity exercise (7+ intensity) appears \(correlation > 0 ? "before flares" : "on better days")"
                ))
            }
        }
        
        return results
    }
    
    private func analyzeActivityCorrelations(flareDays: [DailyLog], nonFlareDays: [DailyLog]) async -> [CorrelationResult] {
        var results: [CorrelationResult] = []
        
        let allActivityTypes = Set(
            (flareDays + nonFlareDays).flatMap { dailyLog in
                (dailyLog.activityEntries?.allObjects as? [ActivityEntry] ?? []).compactMap { $0.activityType }
            }
        )
        
        for activityType in allActivityTypes {
            let flareOccurrences = countActivityOccurrences(type: activityType, in: flareDays)
            let nonFlareOccurrences = countActivityOccurrences(type: activityType, in: nonFlareDays)
            
            guard flareOccurrences >= minimumOccurrences else { continue }
            
            let flareFrequency = Double(flareOccurrences) / Double(flareDays.count)
            let nonFlareFrequency = Double(nonFlareOccurrences) / Double(max(nonFlareDays.count, 1))
            
            let correlation = (flareFrequency - nonFlareFrequency) / max(nonFlareFrequency, 0.1)
            
            if abs(correlation) > significanceThreshold {
                results.append(CorrelationResult(
                    type: .activity,
                    factor: activityType,
                    strength: abs(correlation),
                    isProtective: correlation < 0,
                    confidence: calculateConfidence(occurrences: flareOccurrences + nonFlareOccurrences),
                    description: "\(activityType) activities appear \(correlation > 0 ? "before flares" : "on better days")"
                ))
            }
        }
        
        return results
    }
    
    private func analyzeSleepCorrelations(flareDays: [DailyLog], nonFlareDays: [DailyLog]) async -> [CorrelationResult] {
        var results: [CorrelationResult] = []
        
        // Analyze insufficient sleep (< 6 hours)
        let poorSleepFlareCount = flareDays.filter { $0.sleepHours < 6.0 }.count
        let poorSleepNonFlareCount = nonFlareDays.filter { $0.sleepHours < 6.0 }.count
        
        if poorSleepFlareCount >= minimumOccurrences {
            let flareFreq = Double(poorSleepFlareCount) / Double(flareDays.count)
            let nonFlareFreq = Double(poorSleepNonFlareCount) / Double(max(nonFlareDays.count, 1))
            let correlation = (flareFreq - nonFlareFreq) / max(nonFlareFreq, 0.1)
            
            if abs(correlation) > significanceThreshold {
                results.append(CorrelationResult(
                    type: .sleep,
                    factor: "Poor sleep (< 6 hours)",
                    strength: abs(correlation),
                    isProtective: correlation < 0,
                    confidence: calculateConfidence(occurrences: poorSleepFlareCount + poorSleepNonFlareCount),
                    description: "Sleeping less than 6 hours appears \(correlation > 0 ? "before flares" : "protective")"
                ))
            }
        }
        
        // Analyze excessive sleep (> 10 hours)
        let excessiveSleepFlareCount = flareDays.filter { $0.sleepHours > 10.0 }.count
        let excessiveSleepNonFlareCount = nonFlareDays.filter { $0.sleepHours > 10.0 }.count
        
        if excessiveSleepFlareCount >= minimumOccurrences {
            let flareFreq = Double(excessiveSleepFlareCount) / Double(flareDays.count)
            let nonFlareFreq = Double(excessiveSleepNonFlareCount) / Double(max(nonFlareDays.count, 1))
            let correlation = (flareFreq - nonFlareFreq) / max(nonFlareFreq, 0.1)
            
            if abs(correlation) > significanceThreshold {
                results.append(CorrelationResult(
                    type: .sleep,
                    factor: "Excessive sleep (> 10 hours)",
                    strength: abs(correlation),
                    isProtective: correlation < 0,
                    confidence: calculateConfidence(occurrences: excessiveSleepFlareCount + excessiveSleepNonFlareCount),
                    description: "Sleeping more than 10 hours appears \(correlation > 0 ? "before flares" : "protective")"
                ))
            }
        }
        
        return results
    }
    
    private func analyzeStressCorrelations(flareDays: [DailyLog], nonFlareDays: [DailyLog]) async -> [CorrelationResult] {
        var results: [CorrelationResult] = []
        
        // Analyze high stress (8+ on scale)
        let highStressFlareCount = flareDays.filter { $0.stressLevel >= 8 }.count
        let highStressNonFlareCount = nonFlareDays.filter { $0.stressLevel >= 8 }.count
        
        if highStressFlareCount >= minimumOccurrences {
            let flareFreq = Double(highStressFlareCount) / Double(flareDays.count)
            let nonFlareFreq = Double(highStressNonFlareCount) / Double(max(nonFlareDays.count, 1))
            let correlation = (flareFreq - nonFlareFreq) / max(nonFlareFreq, 0.1)
            
            if abs(correlation) > significanceThreshold {
                results.append(CorrelationResult(
                    type: .stress,
                    factor: "High stress levels",
                    strength: abs(correlation),
                    isProtective: correlation < 0,
                    confidence: calculateConfidence(occurrences: highStressFlareCount + highStressNonFlareCount),
                    description: "High stress (8+ on scale) appears \(correlation > 0 ? "on flare days" : "less on flare days")"
                ))
            }
        }
        
        return results
    }
    
    // MARK: - Helper Methods
    
    private func countFoodOccurrences(food: String, in logs: [DailyLog], lookbackHours: Int) -> Int {
        var count = 0
        
        for log in logs {
            guard let foods = log.foodEntries?.allObjects as? [FoodEntry] else { continue }
            
            let relevantFoods = foods.filter { entry in
                guard let foodName = entry.foodName,
                      foodName.lowercased() == food.lowercased(),
                      let timestamp = entry.timestamp else { return false }
                
                // Check if food was consumed within lookback hours of any symptom
                if let symptoms = log.symptomEntries?.allObjects as? [SymptomEntry] {
                    return symptoms.contains { symptom in
                        guard let symptomTime = symptom.timestamp else { return false }
                        let timeDiff = symptomTime.timeIntervalSince(timestamp)
                        return timeDiff >= 0 && timeDiff <= TimeInterval(lookbackHours * 3600)
                    }
                }
                return false
            }
            
            if !relevantFoods.isEmpty {
                count += 1
            }
        }
        
        return count
    }
    
    private func countExerciseOccurrences(type: String, in logs: [DailyLog]) -> Int {
        return logs.filter { log in
            guard let exercises = log.exerciseEntries?.allObjects as? [ExerciseEntry] else { return false }
            return exercises.contains { $0.exerciseType?.lowercased() == type.lowercased() }
        }.count
    }
    
    private func countActivityOccurrences(type: String, in logs: [DailyLog]) -> Int {
        return logs.filter { log in
            guard let activities = log.activityEntries?.allObjects as? [ActivityEntry] else { return false }
            return activities.contains { $0.activityType?.lowercased() == type.lowercased() }
        }.count
    }
    
    private func countHighIntensityExercise(in logs: [DailyLog]) -> Int {
        return logs.filter { log in
            guard let exercises = log.exerciseEntries?.allObjects as? [ExerciseEntry] else { return false }
            return exercises.contains { $0.intensity >= 7 }
        }.count
    }
    
    private func calculateConfidence(occurrences: Int) -> ConfidenceLevel {
        if occurrences >= 10 {
            return .high
        } else if occurrences >= 5 {
            return .medium
        } else {
            return .low
        }
    }
    
    private func generateFoodDescription(_ data: CorrelationData) -> String {
        if data.isProtective {
            return "\(data.factor) appears on better days (\(Int(data.strength * 100))% correlation)"
        } else {
            return "\(data.factor) appears before flares (\(Int(data.strength * 100))% correlation)"
        }
    }
    
    private func generateExerciseDescription(_ exerciseType: String, _ isProtective: Bool) -> String {
        if isProtective {
            return "\(exerciseType) appears on better days"
        } else {
            return "\(exerciseType) appears before flares"
        }
    }
}

// MARK: - Supporting Data Structures

struct CorrelationResult: Identifiable {
    let id = UUID()
    let type: CorrelationType
    let factor: String
    let strength: Double
    let isProtective: Bool
    let confidence: ConfidenceLevel
    let description: String
}

enum CorrelationType {
    case food, exercise, activity, sleep, stress
    
    var icon: String {
        switch self {
        case .food:
            return "fork.knife"
        case .exercise:
            return "figure.run"
        case .activity:
            return "calendar"
        case .sleep:
            return "bed.double"
        case .stress:
            return "brain.head.profile"
        }
    }
    
    var color: Color {
        switch self {
        case .food:
            return .orange
        case .exercise:
            return .green
        case .activity:
            return .blue
        case .sleep:
            return .purple
        case .stress:
            return .red
        }
    }
}

enum ConfidenceLevel: String, CaseIterable {
    case low = "Low"
    case medium = "Medium"
    case high = "High"
    
    var color: Color {
        switch self {
        case .low:
            return .orange
        case .medium:
            return .yellow
        case .high:
            return .green
        }
    }
}

private struct CorrelationData {
    let factor: String
    let strength: Double
    let isProtective: Bool
    let occurrences: Int
    let confidence: ConfidenceLevel
}
```

### 8.2 Correlations View Model

Create `Models/ViewModels/CorrelationsViewModel.swift`:

```swift
import SwiftUI
import Combine

class CorrelationsViewModel: ObservableObject {
    @Published var correlationResults: [CorrelationResult] = []
    @Published var isLoading = false
    @Published var hasMinimumData = false
    @Published var lastCalculated: Date?
    @Published var selectedFilter: CorrelationFilter = .all
    
    private let correlationEngine = CorrelationEngine()
    private let dataManager = DataManager()
    private var cancellables = Set<AnyCancellable>()
    
    enum CorrelationFilter: String, CaseIterable {
        case all = "All"
        case triggers = "Triggers"
        case protective = "Protective"
        
        var systemImage: String {
            switch self {
            case .all:
                return "list.bullet"
            case .triggers:
                return "exclamationmark.triangle"
            case .protective:
                return "shield"
            }
        }
    }
    
    init() {
        setupBindings()
        checkDataAvailability()
    }
    
    private func setupBindings() {
        correlationEngine.$correlationResults
            .receive(on: DispatchQueue.main)
            .assign(to: \.correlationResults, on: self)
            .store(in: &cancellables)
        
        correlationEngine.$isCalculating
            .receive(on: DispatchQueue.main)
            .assign(to: \.isLoading, on: self)
            .store(in: &cancellables)
        
        correlationEngine.$lastCalculationDate
            .receive(on: DispatchQueue.main)
            .assign(to: \.lastCalculated, on: self)
            .store(in: &cancellables)
    }
    
    func checkDataAvailability() {
        let sevenDaysAgo = Calendar.current.date(byAdding: .day, value: -7, to: Date()) ?? Date()
        let predicate = NSPredicate(format: "date >= %@", sevenDaysAgo as NSDate)
        
        let recentLogs = dataManager.fetch(DailyLog.self, predicate: predicate)
        hasMinimumData = recentLogs.count >= 7
    }
    
    func calculateCorrelations() {
        Task {
            await correlationEngine.calculateCorrelations()
        }
    }
    
    var filteredResults: [CorrelationResult] {
        switch selectedFilter {
        case .all:
            return correlationResults
        case .triggers:
            return correlationResults.filter { !$0.isProtective }
        case .protective:
            return correlationResults.filter { $0.isProtective }
        }
    }
    
    var triggerResults: [CorrelationResult] {
        correlationResults.filter { !$0.isProtective }
    }
    
    var protectiveResults: [CorrelationResult] {
        correlationResults.filter { $0.isProtective }
    }
}
```

### 8.3 Correlations View

Create `Views/Correlations/CorrelationsView.swift`:

```swift
import SwiftUI

struct CorrelationsView: View {
    @StateObject private var viewModel = CorrelationsViewModel()
    
    var body: some View {
        NavigationView {
            Group {
                if !viewModel.hasMinimumData {
                    insufficientDataView
                } else {
                    correlationsContent
                }
            }
            .navigationTitle("Correlations")
            .refreshable {
                viewModel.calculateCorrelations()
            }
        }
    }
    
    private var insufficientDataView: some View {
        VStack(spacing: 20) {
            Spacer()
            
            Image(systemName: "chart.line.uptrend.xyaxis")
                .font(.system(size: 64))
                .foregroundColor(.secondary)
            
            Text("Not Enough Data")
                .font(.title2)
                .fontWeight(.semibold)
            
            Text("Keep tracking for at least 7 days to see patterns and correlations.")
                .font(.body)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal)
            
            Text("We need more data to identify what might trigger your symptoms or help you feel better.")
                .font(.callout)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal)
            
            Spacer()
        }
        .padding()
    }
    
    private var correlationsContent: some View {
        VStack(spacing: 0) {
            if viewModel.correlationResults.isEmpty && !viewModel.isLoading {
                emptyCorrelationsView
            } else {
                correlationsList
            }
        }
    }
    
    private var emptyCorrelationsView: some View {
        VStack(spacing: 20) {
            Spacer()
            
            Image(systemName: "magnifyingglass")
                .font(.system(size: 64))
                .foregroundColor(.secondary)
            
            Text("No Patterns Found")
                .font(.title2)
                .fontWeight(.semibold)
            
            Text("We haven't found any significant correlations yet. Keep tracking consistently to help us identify patterns.")
                .font(.body)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal)
            
            Button("Refresh Analysis") {
                viewModel.calculateCorrelations()
            }
            .buttonStyle(.borderedProminent)
            
            Spacer()
        }
        .padding()
    }
    
    private var correlationsList: some View {
        List {
            if viewModel.isLoading {
                loadingSection
            } else {
                headerSection
                
                if !viewModel.triggerResults.isEmpty {
                    triggersSection
                }
                
                if !viewModel.protectiveResults.isEmpty {
                    protectiveSection
                }
            }
        }
        .listStyle(.insetGrouped)
    }
    
    private var loadingSection: some View {
        Section {
            HStack {
                ProgressView()
                Text("Analyzing patterns...")
                    .foregroundColor(.secondary)
                Spacer()
            }
            .padding()
        }
    }
    
    private var headerSection: some View {
        Section {
            VStack(alignment: .leading, spacing: 12) {
                HStack {
                    Image(systemName: "chart.line.uptrend.xyaxis")
                        .foregroundColor(Constants.Colors.primary)
                    
                    Text("Pattern Analysis")
                        .font(.headline)
                    
                    Spacer()
                    
                    Button("Refresh") {
                        viewModel.calculateCorrelations()
                    }
                    .font(.caption)
                    .foregroundColor(Constants.Colors.primary)
                }
                
                if let lastCalculated = viewModel.lastCalculated {
                    Text("Last updated: \(lastCalculated.formatted(date: .abbreviated, time: .shortened))")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Text("Based on the last 30 days of tracking data. Correlations show factors that appear more frequently before symptom flares or on better days.")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            .padding(.vertical, 4)
        }
    }
    
    private var triggersSection: some View {
        Section(header: sectionHeader(title: "Potential Triggers", 
                                    icon: "exclamationmark.triangle", 
                                    color: .red)) {
            ForEach(viewModel.triggerResults) { result in
                CorrelationRow(result: result)
            }
        }
    }
    
    private var protectiveSection: some View {
        Section(header: sectionHeader(title: "Protective Factors", 
                                    icon: "shield", 
                                    color: .green)) {
            ForEach(viewModel.protectiveResults) { result in
                CorrelationRow(result: result)
            }
        }
    }
    
    private func sectionHeader(title: String, icon: String, color: Color) -> some View {
        HStack {
            Image(systemName: icon)
                .foregroundColor(color)
            Text(title)
                .textCase(nil)
        }
    }
}

struct CorrelationRow: View {
    let result: CorrelationResult
    
    var body: some View {
        HStack(spacing: 12) {
            // Type icon
            Image(systemName: result.type.icon)
                .font(.title3)
                .foregroundColor(.white)
                .frame(width: 32, height: 32)
                .background(result.type.color)
                .clipShape(Circle())
            
            // Content
            VStack(alignment: .leading, spacing: 4) {
                HStack {
                    Text(result.factor)
                        .font(.headline)
                        .lineLimit(1)
                    
                    Spacer()
                    
                    // Confidence badge
                    Text(result.confidence.rawValue)
                        .font(.caption2)
                        .fontWeight(.semibold)
                        .foregroundColor(.white)
                        .padding(.horizontal, 8)
                        .padding(.vertical, 2)
                        .background(result.confidence.color)
                        .clipShape(Capsule())
                }
                
                Text(result.description)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
                
                // Strength indicator
                HStack {
                    Text("Correlation Strength:")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    ProgressView(value: result.strength, total: 1.0)
                        .progressViewStyle(LinearProgressViewStyle(tint: result.isProtective ? .green : .red))
                        .scaleEffect(x: 1, y: 2, anchor: .center)
                        .frame(maxWidth: 100)
                    
                    Text("\(Int(result.strength * 100))%")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
        }
        .padding(.vertical, 4)
    }
}
```

---

## Phase 9: Main App Structure & Tab View

### 9.1 Main Tab View

Create `Views/MainTabView.swift`:

```swift
import SwiftUI

struct MainTabView: View {
    @StateObject private var navigationCoordinator = NavigationCoordinator()
    
    var body: some View {
        TabView(selection: $navigationCoordinator.selectedTab) {
            TodayView()
                .tabItem {
                    Image(systemName: "calendar.day.timeline.left")
                    Text("Today")
                }
                .tag(TabSelection.today)
            
            TimelineView()
                .tabItem {
                    Image(systemName: "clock.arrow.circlepath")
                    Text("Timeline")
                }
                .tag(TabSelection.timeline)
            
            CorrelationsView()
                .tabItem {
                    Image(systemName: "chart.line.uptrend.xyaxis")
                    Text("Insights")
                }
                .tag(TabSelection.correlations)
        }
        .environmentObject(navigationCoordinator)
    }
}
```

### 9.2 Content View (App Entry Point)

Update `App/ContentView.swift`:

```swift
import SwiftUI

struct ContentView: View {
    @StateObject private var userDefaultsManager = UserDefaultsManager.shared
    
    var body: some View {
        Group {
            if userDefaultsManager.onboardingCompleted {
                MainTabView()
            } else {
                OnboardingFlowView()
            }
        }
        .environmentObject(userDefaultsManager)
    }
}

#Preview {
    ContentView()
        .environment(\.managedObjectContext, PersistenceController.shared.viewContext)
}
```

### 9.3 Main App File

Update `App/SymptomTrackerApp.swift`:

```swift
import SwiftUI

@main
struct SymptomTrackerApp: App {
    let persistenceController = PersistenceController.shared
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, persistenceController.viewContext)
                .environmentObject(DataManager())
        }
    }
}
```

---

## Phase 10: Testing & Quality Assurance

### 10.1 Unit Tests Setup

Create `SymptomTrackerTests/DataManagerTests.swift`:

```swift
import XCTest
import CoreData
@testable import SymptomTracker

class DataManagerTests: XCTestCase {
    var dataManager: DataManager!
    var mockContext: NSManagedObjectContext!
    
    override func setUpWithError() throws {
        // Create in-memory store for testing
        let persistentContainer = NSPersistentContainer(name: "SymptomTracker")
        let description = NSPersistentStoreDescription()
        description.type = NSInMemoryStoreType
        persistentContainer.persistentStoreDescriptions = [description]
        
        persistentContainer.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Failed to load store: \(error)")
            }
        }
        
        mockContext = persistentContainer.viewContext
        dataManager = DataManager()
        // Override the context for testing
        // Note: You'll need to modify DataManager to accept a custom context for testing
    }
    
    override func tearDownWithError() throws {
        dataManager = nil
        mockContext = nil
    }
    
    func testCreateDailyLog() throws {
        let testDate = Date()
        let dailyLog = dataManager.createDailyLog(for: testDate)
        
        XCTAssertNotNil(dailyLog.id)
        XCTAssertEqual(Calendar.current.startOfDay(for: dailyLog.date ?? Date()), 
                      Calendar.current.startOfDay(for: testDate))
        XCTAssertNotNil(dailyLog.createdAt)
        XCTAssertNotNil(dailyLog.updatedAt)
    }
    
    func testCreateSymptomEntry() throws {
        let dailyLog = dataManager.createDailyLog(for: Date())
        let symptomEntry = dataManager.createSymptomEntry(
            name: "Headache",
            severity: 7,
            timestamp: Date(),
            notes: "Throbbing pain",
            dailyLog: dailyLog
        )
        
        XCTAssertEqual(symptomEntry.symptomName, "Headache")
        XCTAssertEqual(symptomEntry.severity, 7)
        XCTAssertEqual(symptomEntry.notes, "Throbbing pain")
        XCTAssertEqual(symptomEntry.dailyLog, dailyLog)
    }
    
    func testGetOrCreateDailyLog() throws {
        let testDate = Date()
        
        // First call should create
        let firstLog = dataManager.getOrCreateDailyLog(for: testDate)
        XCTAssertNotNil(firstLog)
        
        // Second call should return existing
        let secondLog = dataManager.getOrCreateDailyLog(for: testDate)
        XCTAssertEqual(firstLog.id, secondLog.id)
    }
}
```

### 10.2 UI Tests Setup

Create `SymptomTrackerUITests/OnboardingUITests.swift`:

```swift
import XCTest

class OnboardingUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        
        // Reset onboarding state for testing
        app.launchArguments.append("--reset-onboarding")
        app.launch()
    }
    
    func testOnboardingFlow() throws {
        // Welcome screen
        XCTAssertTrue(app.staticTexts["Symptom Tracker"].exists)
        XCTAssertTrue(app.buttons["Get Started"].exists)
        
        app.buttons["Get Started"].tap()
        
        // Conditions screen
        XCTAssertTrue(app.staticTexts["Primary Condition(s)"].exists)
        
        // Select a condition
        app.buttons["Fibromyalgia"].tap()
        
        // Select a symptom
        app.buttons["Fatigue"].tap()
        
        // Select severity impact
        app.buttons["Moderate - some limitations on daily activities"].tap()
        
        // Select flare pattern
        app.buttons["Episodic - symptoms come and go in flares"].tap()
        
        app.buttons["Continue"].tap()
        
        // Completion screen
        XCTAssertTrue(app.staticTexts["You're All Set!"].exists)
        
        app.buttons["Start Tracking"].tap()
        
        // Should now be on main app
        XCTAssertTrue(app.tabBars.buttons["Today"].exists)
    }
}
```

### 10.3 Performance Tests

Create `SymptomTrackerTests/PerformanceTests.swift`:

```swift
import XCTest
import CoreData
@testable import SymptomTracker

class PerformanceTests: XCTestCase {
    var dataManager: DataManager!
    
    override func setUpWithError() throws {
        dataManager = DataManager()
    }
    
    func testFetchPerformance() throws {
        // Create sample data
        for i in 0..<100 {
            let date = Calendar.current.date(byAdding: .day, value: -i, to: Date()) ?? Date()
            let dailyLog = dataManager.createDailyLog(for: date)
            
            // Add some entries
            _ = dataManager.createSymptomEntry(
                name: "Test Symptom \(i)",
                severity: Int.random(in: 1...10),
                timestamp: date,
                dailyLog: dailyLog
            )
        }
        
        measure {
            let logs = dataManager.fetch(
                DailyLog.self,
                sortDescriptors: [NSSortDescriptor(key: "date", ascending: false)],
                fetchLimit: 20
            )
            XCTAssertEqual(logs.count, 20)
        }
    }
    
    func testCorrelationPerformance() throws {
        let correlationEngine = CorrelationEngine()
        
        // Create sample data with patterns
        for i in 0..<30 {
            let date = Calendar.current.date(byAdding: .day, value: -i, to: Date()) ?? Date()
            let dailyLog = dataManager.createDailyLog(for: date)
            
            // Create flare pattern every 5 days
            if i % 5 == 0 {
                _ = dataManager.createSymptomEntry(
                    name: "Headache",
                    severity: 8,
                    timestamp: date,
                    dailyLog: dailyLog
                )
                
                // Add potential trigger food 2 hours before
                let triggerTime = Calendar.current.date(byAdding: .hour, value: -2, to: date) ?? date
                _ = dataManager.createFoodEntry(
                    name: "Chocolate",
                    mealType: "Snack",
                    timestamp: triggerTime,
                    dailyLog: dailyLog
                )
            }
        }
        
        measure {
            let expectation = XCTestExpectation(description: "Correlation calculation")
            
            Task {
                await correlationEngine.calculateCorrelations()
                expectation.fulfill()
            }
            
            wait(for: [expectation], timeout: 10.0)
        }
    }
}
```

---

## Phase 11: Build Configuration & Deployment

### 11.1 App Configuration

#### Configure Info.plist
Add the following keys to your `Info.plist`:

```xml
<key>NSHealthShareUsageDescription</key>
<string>This app tracks your symptoms and health data to help identify patterns and triggers.</string>
<key>NSHealthUpdateUsageDescription</key>
<string>This app may update your health data based on your symptom tracking.</string>
<key>UIBackgroundModes</key>
<array>
    <string>background-processing</string>
</array>
```

#### Configure Build Settings
1. Set deployment target to iOS 15.0
2. Enable CloudKit capability
3. Configure App Groups if needed for future extensions
4. Set up proper code signing

### 11.2 CloudKit Configuration

#### Set up CloudKit Schema
1. Open CloudKit Dashboard
2. Create the following Record Types matching your Core Data entities:
   - User
   - DailyLog
   - SymptomEntry
   - FoodEntry
   - ExerciseEntry
   - ActivityEntry

3. Configure relationships and indexes
4. Deploy to production environment

### 11.3 App Store Preparation

#### App Store Connect Setup
1. Create app record in App Store Connect
2. Configure app information:
   - Name: "Symptom Tracker"
   - Subtitle: "Track patterns, find triggers"
   - Category: Health & Fitness
   - Keywords: "symptom, tracking, chronic, illness, health"

3. Prepare screenshots for all device sizes
4. Write app description focusing on chronic illness management
5. Configure pricing (Free recommended for MVP)

#### Privacy Policy
Create a privacy policy covering:
- Data collection practices
- CloudKit sync usage
- No third-party data sharing
- User data control and deletion

---

## Phase 12: Advanced Features (Post-MVP)

### 12.1 Export Functionality

```swift
// Add to DataManager.swift
func exportDataAsCSV() -> String {
    let logs = fetch(DailyLog.self, sortDescriptors: [NSSortDescriptor(key: "date", ascending: true)])
    
    var csvContent = "Date,Mood,Energy,Stress,Sleep,Notes\n"
    
    for log in logs {
        let dateStr = log.date?.formatted(.iso8601.year().month().day()) ?? ""
        let notes = log.notes?.replacingOccurrences(of: ",", with: ";") ?? ""
        
        csvContent += "\(dateStr),\(log.overallMood),\(log.energyLevel),\(log.stressLevel),\(log.sleepHours),\"\(notes)\"\n"
    }
    
    return csvContent
}

func exportSymptomDataAsCSV() -> String {
    let symptoms = fetch(SymptomEntry.self, sortDescriptors: [NSSortDescriptor(key: "timestamp", ascending: true)])
    
    var csvContent = "Date,Time,Symptom,Severity,Notes\n"
    
    for symptom in symptoms {
        let dateStr = symptom.timestamp?.formatted(.iso8601.year().month().day()) ?? ""
        let timeStr = symptom.timestamp?.formatted(.dateTime.hour().minute()) ?? ""
        let notes = symptom.notes?.replacingOccurrences(of: ",", with: ";") ?? ""
        
        csvContent += "\(dateStr),\(timeStr),\(symptom.symptomName ?? ""),\(symptom.severity),\"\(notes)\"\n"
    }
    
    return csvContent
}
```

### 12.2 Notification System

```swift
// Create Services/NotificationManager.swift
import UserNotifications

class NotificationManager: ObservableObject {
    static let shared = NotificationManager()
    
    func requestPermission() {
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { granted, error in
            if granted {
                self.scheduleCheckInReminders()
            }
        }
    }
    
    func scheduleCheckInReminders() {
        // Schedule daily reminder at 8 PM
        let content = UNMutableNotificationContent()
        content.title = "Daily Check-in"
        content.body = "How are you feeling today? Take a moment to log your symptoms."
        content.sound = .default
        
        var dateComponents = DateComponents()
        dateComponents.hour = 20 // 8 PM
        
        let trigger = UNCalendarNotificationTrigger(dateMatching: dateComponents, repeats: true)
        let request = UNNotificationRequest(identifier: "daily-checkin", content: content, trigger: trigger)
        
        UNUserNotificationCenter.current().add(request)
    }
}
```

### 12.3 HealthKit Integration

```swift
// Create Services/HealthKitManager.swift
import HealthKit

class HealthKitManager: ObservableObject {
    private let healthStore = HKHealthStore()
    
    func requestAuthorization() {
        guard HKHealthStore.isHealthDataAvailable() else { return }
        
        let readTypes: Set<HKObjectType> = [
            HKObjectType.quantityType(forIdentifier: .stepCount)!,
            HKObjectType.quantityType(forIdentifier: .heartRate)!,
            HKObjectType.categoryType(forIdentifier: .sleepAnalysis)!
        ]
        
        healthStore.requestAuthorization(toShare: [], read: readTypes) { success, error in
            if success {
                self.startObservingHealthData()
            }
        }
    }
    
    private func startObservingHealthData() {
        // Implement health data observation and correlation
    }
}
```

---

## Phase 13: Final Testing & Launch Checklist

### 13.1 Pre-Launch Testing Checklist

#### Functional Testing
- [ ] Onboarding flow completes successfully
- [ ] All entry forms save data correctly
- [ ] Data syncs across devices via CloudKit
- [ ] Timeline view displays historical data
- [ ] Correlation analysis works with sufficient data
- [ ] Export functionality generates valid CSV
- [ ] App handles offline mode gracefully

#### Usability Testing
- [ ] Navigation is intuitive for chronic illness patients
- [ ] Text is readable with accessibility settings
- [ ] Voice Over works correctly
- [ ] App performs well on older devices (iPhone SE, iPad)
- [ ] Battery usage is reasonable

#### Edge Cases
- [ ] App handles no internet connection
- [ ] App recovers from Core Data errors
- [ ] Large amounts of data don't slow performance
- [ ] CloudKit sync conflicts resolve correctly
- [ ] App works correctly after iOS updates

### 13.2 Launch Strategy

#### Soft Launch
1. Release to small group of beta testers with chronic illnesses
2. Gather feedback on usability and feature requests
3. Monitor crash reports and performance metrics
4. Iterate based on real-world usage

#### Marketing Strategy
1. Partner with chronic illness support groups
2. Reach out to healthcare providers specializing in:
   - Fibromyalgia
   - POTS/Dysautonomia
   - Autoimmune conditions
   - Chronic fatigue syndrome
3. Create educational content about symptom tracking
4. Develop referral program for healthcare professionals

### 13.3 Post-Launch Monitoring

#### Analytics to Track
- Daily active users
- Feature usage patterns
- Correlation calculation success rates
- Data export frequency
- User retention at 7, 30, 90 days

#### Support Preparation
- Create comprehensive FAQ
- Set up user feedback system
- Establish bug reporting process
- Plan regular updates and feature releases

---

## Final MVP Timeline Summary

### Week 1: Foundation (40 hours)
- Project setup and Core Data model
- Basic navigation architecture
- User defaults management
- Onboarding flow implementation

### Week 2: Core Features (40 hours)
- Today view with sliders and entry forms
- Food, exercise, activity, symptom logging
- Timeline view with historical data
- Day detail view with chronological entries

### Week 3: Advanced Features (40 hours)
- Correlation engine implementation
- Pattern analysis and display
- Correlations view with insights
- Performance optimization and testing

### Week 4: Polish & Launch (40 hours)
- Bug fixes and edge cases
- UI/UX improvements
- CloudKit setup and testing
- App Store submission preparation

**Total Estimated Development Time: 160 hours (4 weeks for 1 developer)**

---

## Success Metrics for MVP

### User Engagement
- **Target**: 70% of users complete onboarding
- **Target**: 60% of users log data for 7+ consecutive days
- **Target**: 40% of users generate their first correlation insights

### Technical Performance
- **Target**: App launch time < 3 seconds
- **Target**: Correlation calculation < 10 seconds for 30 days data
- **Target**: Crash rate < 1%
- **Target**: CloudKit sync success rate > 95%

### User Satisfaction
- **Target**: App Store rating > 4.2/5
- **Target**: 85% of surveyed users find correlations helpful
- **Target**: 75% of users would recommend to others with chronic conditions

This comprehensive guide provides everything needed to build a functional MVP symptom tracker app that will genuinely help people with chronic illnesses identify patterns and improve their quality of life.
