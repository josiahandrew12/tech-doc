TICKET 1: CORE DATA STACK & ENTITY CONFIGURATION
Entity Schema Design
User Entity:
Attributes:
- id: UUID (Primary Key, Not Optional)
- firstName: String (Optional, Max 50 chars)
- lastName: String (Optional, Max 50 chars)
- primaryConditions: String (Optional, JSON array of selected conditions)
- onboardingCompleted: Boolean (Not Optional, Default: false)
- createdAt: Date (Not Optional)
- updatedAt: Date (Not Optional, Auto-update trigger)

Indexes:
- id (Primary)
- onboardingCompleted (for app state queries)
DailyLog Entity:
Attributes:
- id: UUID (Primary Key, Not Optional)
- date: Date (Not Optional, Unique constraint per user)
- sleepHours: Double (Optional, Range: 0-24, Precision: 0.5)
- createdAt: Date (Not Optional)
- updatedAt: Date (Not Optional)

Indexes:
- date (for timeline queries)
- (userId, date) composite index
SymptomEntry Entity:
Attributes:
- id: UUID (Primary Key, Not Optional)
- severity: Int16 (Not Optional, Range: 1-10)
- timeLogged: Date (Not Optional)
- symptomName: String (Not Optional, Max 100 chars)
- bodyLocation: String (Optional, Max 50 chars)
- duration: Int32 (Optional, minutes)
- notes: String (Optional, Max 500 chars)

Relationships:
- dailyLog: To One DailyLog (Delete Rule: Nullify)

Indexes:
- timeLogged (for correlation queries)
- severity (for flare analysis)
- (dailyLogId, timeLogged) composite
FoodEntry Entity:
Attributes:
- id: UUID (Primary Key, Not Optional)
- foodName: String (Not Optional, Max 200 chars)
- mealType: String (Not Optional, Enum: breakfast|lunch|dinner|snack)
- timeConsumed: Date (Not Optional)
- quantity: Double (Optional)
- unit: String (Optional, Default: "serving")
- preparation: String (Optional, Max 100 chars)
- ingredients: String (Optional, JSON array for compound foods)

Relationships:
- dailyLog: To One DailyLog (Delete Rule: Nullify)

Indexes:
- timeConsumed (for correlation analysis)
- foodName (for trigger identification)
- mealType (for meal pattern analysis)
ExerciseEntry Entity:
Attributes:
- id: UUID (Primary Key, Not Optional)
- exerciseType: String (Not Optional, Enum: cardio|strength|yoga|walking|swimming|cycling|other)
- duration: Int32 (Not Optional, minutes, Range: 1-480)
- intensity: Int16 (Not Optional, Range: 1-10)
- timePerformed: Date (Not Optional)
- caloriesBurned: Int32 (Optional)
- heartRateAvg: Int16 (Optional, Range: 40-220)
- notes: String (Optional, Max 300 chars)

Relationships:
- dailyLog: To One DailyLog (Delete Rule: Nullify)

Indexes:
- timePerformed (for correlation timing)
- exerciseType (for pattern analysis)
- intensity (for impact assessment)
ActivityEntry Entity:
Attributes:
- id: UUID (Primary Key, Not Optional)
- activityType: String (Not Optional, Enum: work|social|travel|medical|household|leisure|other)
- duration: Int32 (Optional, minutes)
- timePerformed: Date (Not Optional)
- location: String (Optional, Max 100 chars)
- stressLevel: Int16 (Optional, Range: 1-10)
- socialContext: String (Optional, Enum: alone|small_group|large_group)
- notes: String (Optional, Max 300 chars)

Relationships:
- dailyLog: To One DailyLog (Delete Rule: Nullify)

Indexes:
- timePerformed (for timeline analysis)
- activityType (for pattern recognition)
- stressLevel (for stress correlation)
Core Data Configuration
NSPersistentCloudKitContainer Setup:

Container name: "SymptomTrackerModel"
CloudKit container identifier: "iCloud.com.yourcompany.symptomtracker"
Enable persistent history tracking with NSPersistentHistoryTrackingKey
Configure remote change notifications with NSPersistentStoreRemoteChangeNotificationPostOptionKey
Set automatic lightweight migration with NSMigratePersistentStoresAutomaticallyOption

Context Hierarchy:

Main Context: NSMainQueueConcurrencyType for UI operations
Background Context: NSPrivateQueueConcurrencyType for correlation calculations
Import Context: NSPrivateQueueConcurrencyType for bulk data operations
Configure automaticallyMergesChangesFromParent = true on main context

Performance Optimization:

fetchBatchSize: 25 for list views, 100 for analysis
relationshipKeyPathsForPrefetching for common relationship traversals
NSFetchRequest templates for frequent queries stored in .xcdatamodeld
Background context reset() after heavy operations to free memory


TICKET 2: MVVM ARCHITECTURE & NAVIGATION SYSTEM
Navigation Architecture
NavigationCoordinator (ObservableObject):
Properties:
- @Published isOnboardingComplete: Bool
- @Published selectedTab: TabSelection
- @Published navigationPath: NavigationPath
- @Published presentedSheet: SheetDestination?
- @Published alertState: AlertState?

Methods:
- completeOnboarding()
- navigateToTab(_:)
- presentSheet(_:)
- dismissSheet()
- showAlert(_:)
Tab Configuration:
enum TabSelection: String, CaseIterable {
    case today = "Today"
    case timeline = "Timeline"
    case correlations = "Correlations"
    
    var iconName: String
    var accessibilityLabel: String
}
Onboarding Flow Architecture
OnboardingViewModel:
Properties:
- @Published currentStep: Int (0-2)
- @Published selectedConditions: Set<HealthCondition>
- @Published firstName: String
- @Published isValid: Bool (computed)
- @Published isLoading: Bool

Methods:
- validateCurrentStep() -> Bool
- nextStep()
- previousStep()
- completeOnboarding() async throws
HealthCondition Enum:
enum HealthCondition: String, CaseIterable, Codable {
    case autoimmune = "Autoimmune Disease"
    case foodSensitivities = "Food Sensitivities"
    case ibs = "IBS/IBD"
    case chronicFatigue = "Chronic Fatigue"
    case fibromyalgia = "Fibromyalgia"
    case migraines = "Migraines/Headaches"
    case pots = "POTS/Dysautonomia"
    case mentalHealth = "Mental Health"
    case other = "Other"
}
State Management Pattern
BaseViewModel Protocol:
protocol BaseViewModel: ObservableObject {
    var isLoading: Bool { get set }
    var errorMessage: String? { get set }
    var cancellables: Set<AnyCancellable> { get set }
    
    func handleError(_ error: Error)
    func setLoading(_ loading: Bool)
}
ViewState Enum:
enum ViewState<T> {
    case idle
    case loading
    case loaded(T)
    case error(AppError)
    
    var isLoading: Bool
    var data: T?
    var error: AppError?
}

TICKET 3: TODAY VIEW - DAILY LOGGING INTERFACE
TodayViewModel Architecture
State Management:
@Published var currentDate: Date
@Published var currentDailyLog: DailyLog?
@Published var sleepHours: Double
@Published var todaysEntries: TodaysEntries
@Published var viewState: ViewState<DailyLog>

struct TodaysEntries {
    var symptoms: [SymptomEntry]
    var foods: [FoodEntry]
    var exercises: [ExerciseEntry]
    var activities: [ActivityEntry]
}
Auto-Save Implementation:
Combine Pipeline:
$sleepHours
    .dropFirst()
    .debounce(for: .milliseconds(800), scheduler: RunLoop.main)
    .sink { [weak self] in self?.saveDailyLog() }
Date Navigation Logic:
Methods:
- loadDailyLogForDate(_:) async
- createDailyLogIfNeeded() -> DailyLog
- previousDay() -> Void
- nextDay() -> Void (disabled if future date)
- canNavigateToNextDay() -> Bool
Entry Management System
Entry Creation Flow:
Protocol EntryCreatable {
    associatedtype EntryType: NSManagedObject
    func createEntry(for dailyLog: DailyLog) -> EntryType
    func validateEntry() throws -> Bool
    func saveEntry() async throws
}
Entry ViewModels:
SymptomEntryViewModel:
- @Published severity: Double (1-10)
- @Published symptomName: String
- @Published timeLogged: Date
- @Published bodyLocation: String?
- @Published duration: Int? (minutes)

FoodEntryViewModel:
- @Published foodName: String
- @Published mealType: MealType
- @Published timeConsumed: Date
- @Published quantity: Double?
- @Published unit: String

ExerciseEntryViewModel:
- @Published exerciseType: ExerciseType
- @Published duration: Int (minutes)
- @Published intensity: Double (1-10)
- @Published timePerformed: Date

ActivityEntryViewModel:
- @Published activityType: ActivityType
- @Published duration: Int? (minutes)
- @Published timePerformed: Date
- @Published stressLevel: Double? (1-10)
UI Component Architecture
Custom Slider Component:
SeveritySlider: UIViewRepresentable {
    Configuration:
    - Gradient colors: Red (1) → Yellow (5) → Green (10)
    - Haptic feedback on value change (UIImpactFeedbackGenerator)
    - Accessibility: .adjustable trait with voice descriptions
    - Step value: 1.0 with snap-to-integer behavior
}
Entry Card Components:
EntryCard<T: Entry>: View {
    Properties:
    - entry: T
    - onEdit: () -> Void
    - onDelete: () -> Void
    
    Layout:
    - SwipeActions for edit/delete
    - Time badge
    - Entry-specific content view
    - Visual indicators for entry type
}
Modal Presentation System:
Sheet Management:
- NavigationStack for complex entry forms
- Form validation before save
- Dismiss on successful save
- Cancel confirmation for unsaved changes

TICKET 4: TIMELINE VIEW & HISTORICAL DATA
TimelineViewModel Architecture
Data Fetching Strategy:
@Published var dailyLogs: [DailyLogSummary]
@Published var selectedDateRange: DateRange
@Published var isLoadingMore: Bool
@Published var hasMoreData: Bool

struct DailyLogSummary {
    let date: Date
    let sleepHours: Double?
    let entryCount: EntryCount
    let dayQuality: DayQuality (computed from symptom severity)
}

struct EntryCount {
    let symptoms: Int
    let foods: Int
    let exercises: Int
    let activities: Int
    let total: Int
}
Pagination Implementation:
NSFetchRequest Configuration:
- fetchLimit: 20
- fetchOffset: currentPage * 20
- sortDescriptors: [NSSortDescriptor(keyPath: \DailyLog.date, ascending: false)]
- predicate: Date range filtering

Infinite Scroll:
- onAppear trigger for last 3 items
- Background loading with loading indicators
- Error handling for fetch failures
Day Detail View Architecture
Chronological Timeline Algorithm:
Timeline Creation Process:
1. Fetch all entries for selected date
2. Extract timestamps and create TimelineEvent structs
3. Sort by timestamp ascending
4. Group events within 30-minute windows
5. Apply visual layout with proportional spacing

struct TimelineEvent {
    let id: UUID
    let timestamp: Date
    let type: EntryType
    let content: Any
    let severity: Int? (for symptoms)
}
Edit Functionality:
In-Place Editing:
- Tap to edit any historical entry
- Modal presentation of appropriate entry form
- Pre-populated with existing data
- Validation before saving changes
- Real-time UI updates after successful edit
Performance Optimization
NSFetchedResultsController Integration:
Configuration:
- sectionNameKeyPath: "date" (grouped by day)
- cacheName: "TimelineCache"
- Automatic UI updates via NSFetchedResultsControllerDelegate
- Background context for heavy operations
Memory Management:
Strategies:
- Release unused DailyLog objects using context.refreshObject(_:mergeChanges:)
- Implement fault fulfillment optimization
- Use autoreleasepool for batch operations
- Configure NSManagedObjectContext.undoManager = nil for background contexts

TICKET 5: CORRELATION ENGINE & PATTERN DISCOVERY
Statistical Analysis Engine
Correlation Calculation Algorithm:
Time Window Analysis:
1. Query last 30-90 days of complete DailyLog entries
2. Identify "flare days": days with symptom severity ≥ 7
3. Create temporal windows: [0-2h, 2-6h, 6-24h, 24-48h] before flare
4. Extract factors present in each window
5. Calculate correlation strength using formula:
   
   Correlation = (FlareFrequency - BaselineFrequency) / BaselineFrequency
   
   Where:
   - FlareFrequency = Factor occurrences before flares / Total flare days
   - BaselineFrequency = Factor occurrences on normal days / Total normal days
Statistical Significance Testing:
Chi-Square Test Implementation:
- Null hypothesis: Factor and symptoms are independent
- Expected frequency calculation for 2x2 contingency table
- Chi-square statistic: Σ((Observed - Expected)² / Expected)
- p-value calculation using chi-square distribution
- Confidence levels: High (p < 0.01), Medium (p < 0.05), Low (p < 0.1)
Data Structures
Correlation Result Model:
struct CorrelationResult: Codable {
    let id: UUID
    let factor: Factor
    let correlationType: CorrelationType
    let strength: Double (-1.0 to 1.0)
    let confidence: ConfidenceLevel
    let occurrences: Int
    let pValue: Double
    let timeWindow: TimeWindow
    let calculatedAt: Date
}

enum Factor {
    case food(String)
    case exercise(ExerciseType, intensity: IntensityRange?)
    case activity(ActivityType)
    case sleep(hours: DoubleRange)
    case compound([Factor]) // for multi-factor patterns
}

enum CorrelationType {
    case trigger // positive correlation with symptoms
    case protective // negative correlation with symptoms
    case neutral // no significant correlation
}
Time Series Data Structure:
struct TimeSeries<T> {
    let data: [(Date, T)]
    let resolution: TimeResolution
    
    func valuesInWindow(from: Date, duration: TimeInterval) -> [T]
    func correlate<U>(with other: TimeSeries<U>, window: TimeInterval) -> Double
}
Background Processing Architecture
Correlation Service:
actor CorrelationService {
    private let context: NSManagedObjectContext
    private let cache: NSCache<NSString, CorrelationResultSet>
    
    func calculateCorrelations(for userId: UUID) async throws -> CorrelationResultSet
    func invalidateCache(for userId: UUID)
    private func performStatisticalAnalysis(_ data: [DailyLogData]) -> [CorrelationResult]
}
Progress Tracking:
@Published var calculationProgress: Progress
@Published var currentAnalysisStep: AnalysisStep

enum AnalysisStep {
    case fetchingData
    case identifyingFlares
    case analyzingPatterns(factor: String)
    case calculatingSignificance
    case rankingResults
    case complete
}
UI Display Architecture
Correlation Display Components:
CorrelationCard: View {
    Properties:
    - result: CorrelationResult
    - isExpanded: Bool
    
    Layout:
    - Factor name and type
    - Correlation strength indicator (progress bar)
    - Confidence badge (color-coded)
    - Expandable details with statistical information
    - Time window visualization
}

TriggersList: View {
    Features:
    - Sorted by correlation strength (descending)
    - Filtered by minimum confidence level
    - Search functionality by factor name
    - Export capability for healthcare providers
}

ProtectiveFactorsList: View {
    Features:
    - Factors with negative correlation to symptoms
    - Recommendations based on protective patterns
    - Frequency recommendations for beneficial activities
}
Performance Optimization
Caching Strategy:
Cache Configuration:
- NSCache with size limit based on available memory
- Key: "correlations_\(userId)_\(dateRange.hashValue)"
- Automatic eviction based on LRU policy
- Cache invalidation triggers: new data entry, manual refresh

Background Processing:
- DispatchQueue.global(qos: .userInitiated) for calculations
- Task.detached for async/await correlation processing
- Cancellation support using Task.isCancelled checks
- Progress reporting using AsyncSequence for real-time updates
Algorithm Optimization:
Data Structure Optimizations:
- Pre-indexed symptom events by severity for quick flare identification
- Temporal indexing of all entries for efficient window queries
- Sparse matrix representation for correlation calculations
- Vectorized operations where possible using Accelerate framework

Memory Management:
- Stream processing for large datasets to prevent memory spikes
- Incremental correlation updates for new data rather than full recalculation
- Release intermediate calculation results promptly
- Use of autoreleasepool in calculation loops

INTEGRATION REQUIREMENTS
Error Handling Strategy
AppError Hierarchy:
- CoreDataError (save failures, fetch failures)
- ValidationError (invalid input data)
- CorrelationError (insufficient data, calculation failures)
- NetworkError (future API integration)
- SystemError (memory, disk space issues)
Testing Requirements
Unit Tests:
- Core Data entity validation
- Correlation algorithm accuracy with known datasets
- Date/time handling across timezones
- Edge cases (empty data, single entries)

Integration Tests:
- Complete user flows (onboarding → logging → correlation discovery)
- Background processing with UI updates
- Data persistence across app launches
- Memory usage under stress conditions
