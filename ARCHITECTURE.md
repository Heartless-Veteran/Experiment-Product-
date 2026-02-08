# Architecture Documentation

## Overview

This project implements Clean Architecture principles with MVVM pattern, following modern Android development best practices.

## Architecture Principles

### 1. Separation of Concerns
Each layer has a specific responsibility and doesn't depend on implementation details of other layers.

### 2. Dependency Rule
Dependencies only point inward:
- Presentation → Domain → Data
- Inner layers know nothing about outer layers

### 3. Testability
Each layer can be tested independently with mocked dependencies.

## Layer Details

### Data Layer (`data/`)

**Responsibility**: Manages data from various sources (local database, network, etc.)

**Components**:
- **Entities**: Room database entities (e.g., `LocationPinEntity`)
- **DAOs**: Data Access Objects with Flow-based reactive queries
- **Repositories**: Implement data access logic, abstract data sources

**Key Features**:
- Room database for local persistence
- Flow for reactive data streams
- Repository pattern for data abstraction

**Example**:
```kotlin
@Entity(tableName = "location_pins")
data class LocationPinEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val name: String,
    val latitude: Double,
    val longitude: Double
)

@Dao
interface LocationPinDao {
    @Query("SELECT * FROM location_pins")
    fun getAllPins(): Flow<List<LocationPinEntity>>
}
```

### Domain Layer (`domain/`)

**Responsibility**: Contains business logic, independent of frameworks

**Components**:
- **Models**: Pure Kotlin data classes (e.g., `LocationPin`)
- **Use Cases**: Encapsulate specific business operations

**Key Features**:
- No Android dependencies
- Pure Kotlin
- Single Responsibility Principle (each use case does one thing)

**Example**:
```kotlin
class GetLocationPinsUseCase @Inject constructor(
    private val repository: LocationPinRepository
) {
    operator fun invoke(): Flow<List<LocationPin>> {
        return repository.getAllPins().map { entities ->
            entities.map { it.toDomainModel() }
        }
    }
}
```

### Presentation Layer (`presentation/`)

**Responsibility**: Handles UI and user interactions

**Components**:
- **ViewModels**: Manage UI state with StateFlow/SharedFlow
- **UI**: Jetpack Compose screens and components
- **Theme**: Material 3 theme configuration

**Key Features**:
- MVVM architecture
- Jetpack Compose for declarative UI
- StateFlow for unidirectional data flow
- Lifecycle-aware components

**Example**:
```kotlin
@HiltViewModel
class MapViewModel @Inject constructor(
    getLocationPinsUseCase: GetLocationPinsUseCase
) : ViewModel() {
    val locationPins: StateFlow<List<LocationPin>> = 
        getLocationPinsUseCase()
            .stateIn(
                scope = viewModelScope,
                started = SharingStarted.WhileSubscribed(5000),
                initialValue = emptyList()
            )
}
```

## Dependency Injection with Hilt

### Application Setup
```kotlin
@HiltAndroidApp
class ExperimentProductApp : Application()
```

### Activity Integration
```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity()
```

### ViewModel Injection
```kotlin
@HiltViewModel
class MapViewModel @Inject constructor(
    private val useCase: GetLocationPinsUseCase
) : ViewModel()
```

### Modules
Modules provide dependencies to the DI graph:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    @Provides
    @Singleton
    fun provideAppDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "experiment_product_database"
        ).build()
    }
}
```

## Data Flow

### Read Operation Flow:
1. **UI** (Compose) collects StateFlow from ViewModel
2. **ViewModel** exposes StateFlow from Use Case
3. **Use Case** transforms data from Repository
4. **Repository** queries DAO
5. **DAO** returns Flow from Room database
6. Data flows back up, triggering UI recomposition

### Write Operation Flow:
1. **UI** calls ViewModel method
2. **ViewModel** launches coroutine
3. **Use Case** (if needed) processes data
4. **Repository** calls DAO suspend function
5. **DAO** inserts/updates in Room database
6. Flow automatically emits new data

## Async Operations

### Coroutines
- Used for all async operations
- ViewModelScope for ViewModel operations
- Structured concurrency for safe cancellation

### Flow
- Reactive data streams
- StateFlow for state management
- Cold flows for one-shot operations

**Example**:
```kotlin
// In ViewModel
val locationPins: StateFlow<List<LocationPin>> = 
    getLocationPinsUseCase()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )

// In Composable
val pins by viewModel.locationPins.collectAsState()
```

## Jetpack Compose Integration

### Key Concepts:
- **Stateless Composables**: UI components receive data as parameters
- **State Hoisting**: State managed in ViewModel
- **Recomposition**: UI updates automatically when StateFlow emits

**Example**:
```kotlin
@Composable
fun MapScreen(modifier: Modifier = Modifier) {
    val context = LocalContext.current
    val mapView = remember { MapView(context) }
    
    AndroidView(
        factory = { mapView },
        modifier = Modifier.fillMaxSize()
    )
}
```

## Mapbox Integration

### Setup:
1. Token configured in `gradle.properties`
2. Maven repository in `settings.gradle.kts`
3. Dependencies in `app/build.gradle.kts`

### Usage:
- AndroidView wraps native MapView
- DisposableEffect handles lifecycle
- Style loaded from Mapbox Streets

## Testing Strategy

### Unit Tests:
- **Domain Layer**: Test use cases with mocked repositories
- **ViewModels**: Test with fake use cases

### Integration Tests:
- **Repository**: Test with in-memory Room database
- **Database**: Test DAOs with actual database

### UI Tests:
- **Compose**: Use Compose testing framework
- Test user interactions and state changes

## Best Practices Followed

1. **Null Safety**: Strict Kotlin null safety, no `!!` operators
2. **Immutability**: Data classes with `val` properties
3. **Single Responsibility**: Each class has one clear purpose
4. **Dependency Inversion**: Depend on abstractions, not implementations
5. **Flow over LiveData**: Modern reactive streams
6. **Kotlin Coroutines**: Structured concurrency for async work
7. **Jetpack Compose**: Declarative UI approach
8. **Material 3**: Latest Material Design guidelines

## Security Considerations

1. **Permissions**: Runtime permission checks required for location
2. **API Keys**: Tokens stored in `gradle.properties`, not in code
3. **Database**: Room uses SQLCipher for encryption (can be added)
4. **Network**: HTTPS only for API calls

## Performance Optimizations

1. **StateFlow**: Only recomposes when state changes
2. **WhileSubscribed(5000)**: Stops flow collection when no subscribers
3. **Room**: Indexed queries for fast lookups
4. **Lazy Loading**: Data loaded on demand
5. **Coroutines**: Non-blocking async operations

## Code Organization Rules

1. **Packages by Layer**: Separate packages for data, domain, presentation
2. **Package by Feature**: Within layers, organize by feature
3. **Naming Conventions**:
   - Entities: `*Entity`
   - ViewModels: `*ViewModel`
   - Use Cases: `*UseCase`
   - Repositories: `*Repository`

## Extending the Architecture

### Adding a New Feature:

1. **Data Layer**:
   - Create entity (if needed)
   - Create/update DAO
   - Create/update repository

2. **Domain Layer**:
   - Create domain model
   - Create use case

3. **Presentation Layer**:
   - Create ViewModel
   - Create Composable screen
   - Wire up navigation (when implemented)

### Adding a New Data Source:

1. Create data source interface in domain
2. Implement in data layer
3. Inject via Hilt module
4. Use in repository

This architecture provides a solid foundation for building a scalable, maintainable, and testable Android application.
