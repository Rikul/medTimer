# MedTimer Architecture Documentation

## Table of Contents
1. [Overview](#overview)
2. [Technology Stack](#technology-stack)
3. [High-Level Architecture](#high-level-architecture)
4. [Core Components](#core-components)
5. [Data Layer](#data-layer)
6. [User Interface Layer](#user-interface-layer)
7. [Background Processing](#background-processing)
8. [Reminder System](#reminder-system)
9. [Data Flow](#data-flow)
10. [Security & Privacy](#security--privacy)
11. [Testing Strategy](#testing-strategy)
12. [Build & Deployment](#build--deployment)

## Overview

MedTimer is a medication reminder Android application built with privacy and offline-first principles. The app allows users to manage unlimited medications with flexible, customizable reminders while keeping all data stored locally on the device.

**Key Characteristics:**
- **Platform:** Native Android (minimum SDK 28, target SDK 36)
- **Architecture Pattern:** MVVM (Model-View-ViewModel) with Repository pattern
- **Database:** Room (SQLite)
- **Language:** Java and Kotlin (mixed codebase)
- **Privacy:** Fully offline, no internet access, all data stored locally

## Technology Stack

### Core Android Components
- **AndroidX Libraries:** AppCompat, Material Design Components, ConstraintLayout
- **Navigation:** AndroidX Navigation Component with Safe Args
- **Lifecycle:** AndroidX Lifecycle-aware components (ViewModel, LiveData)
- **Work Manager:** For background task scheduling
- **Room Database:** SQLite ORM with version 2.8.4
- **Preferences:** AndroidX Preference with extended components

### UI Libraries
- **Material Design 3:** com.google.android.material
- **Calendar View:** kizitonwose Calendar for medicine calendar view
- **Color Picker:** HSV-Alpha Color Picker for medicine customization
- **Icon Dialog:** maltaisn IconDialog for icon selection
- **TableView:** evrencoskun TableView for tabular data display
- **AndroidPlot:** Charts and graphs for statistics
- **FlexBox:** Google FlexBox for dynamic layouts
- **AppIntro:** Onboarding slides

### Export & Utilities
- **SimplyPDF:** PDF generation for exports
- **Gson:** JSON serialization for backup/restore
- **Biometric:** AndroidX Biometric for app authentication

### Testing
- **Unit Testing:** JUnit 5 (Jupiter), Mockito, Robolectric
- **UI Testing:** Espresso, UI Automator, Barista
- **Code Coverage:** Jacoco
- **Fuzzing:** Jazzer for fuzz testing

### Build & Quality
- **Build System:** Gradle with Kotlin DSL
- **Code Quality:** SonarQube, Android Lint
- **CI/CD:** GitHub Actions
- **Play Store:** Gradle Play Publisher plugin

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Presentation Layer                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Overview    │  │  Medicines   │  │  Statistics  │      │
│  │  Fragment    │  │  Fragment    │  │  Fragment    │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │              │
│         └──────────────────┼──────────────────┘              │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────────┐ │
│  │              ViewModels (ViewModel Layer)              │ │
│  └─────────────────────────┬──────────────────────────────┘ │
└────────────────────────────┼────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────┐
│                     Business Logic Layer                     │
│  ┌──────────────────────────────────────────────────────┐   │
│  │             Medicine Repository                      │   │
│  └──────────────────────────┬───────────────────────────┘   │
│                             │                               │
│  ┌──────────────────────────▼───────────────────────────┐   │
│  │          Reminder Processor & Schedulers             │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────┐
│                         Data Layer                           │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Room Database (SQLite)                  │   │
│  │  - Medicine    - Reminder    - ReminderEvent        │   │
│  │  - Tag         - MedicineToTag                       │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    Background Services                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Android    │  │  WorkManager │  │   Alarm      │      │
│  │  AlarmMgr    │  │   Workers    │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. MainActivity
**Location:** `com.futsch1.medtimer.MainActivity`

The main entry point of the application that:
- Manages the navigation between fragments using Bottom Navigation
- Handles biometric authentication if enabled
- Initializes notification channels
- Manages theme selection
- Controls secure window flag for privacy

### 2. Medicine Repository
**Location:** `com.futsch1.medtimer.database.MedicineRepository`

Central data access layer that:
- Provides LiveData observables for UI updates
- Abstracts database operations from ViewModels
- Executes database operations on background threads
- Manages relationships between medicines, reminders, and events

### 3. Reminder Processor
**Location:** `com.futsch1.medtimer.reminders.ReminderProcessor`

Coordinates all reminder-related activities:
- Enqueues WorkManager workers for reminder operations
- Processes reminder notifications
- Handles user actions (taken, skipped, snoozed)
- Manages notification lifecycle

### 4. Reminder Scheduler Service
**Location:** `com.futsch1.medtimer.ReminderSchedulerService`

A LifecycleService that:
- Observes medicine changes from the database
- Triggers rescheduling when medicines are modified
- Maintains service lifecycle across app restarts

## Data Layer

### Database Schema

The app uses Room Database with 5 main entities:

#### 1. Medicine
**Table:** `medicine`
- Stores medication details (name, description, color, icon)
- Tracks medication stock levels
- Links to reminders and tags
- Supports active/inactive state

#### 2. Reminder
**Table:** `reminder`
- Defines when and how often to remind
- Supports multiple reminder types:
  - Daily reminders
  - Interval-based reminders
  - Cyclic reminders (e.g., birth control pills)
  - Custom day-of-week patterns
- Contains dosage information
- Links to a parent Medicine

#### 3. ReminderEvent
**Table:** `reminder_event`
- Records actual reminder occurrences
- Tracks status: raised, taken, or skipped
- Stores timestamp and dosage amount
- Links to parent Reminder
- Used for statistics and history

#### 4. Tag
**Table:** `tag`
- Custom labels for organizing medicines
- Supports filtering in the UI

#### 5. MedicineToTag
**Table:** `medicine_to_tag`
- Junction table for many-to-many relationship
- Links medicines to tags

### Database Versioning
- **Current Version:** 21
- **Migration Strategy:** AutoMigration with custom specs
- **Schema Location:** `app/schemas/`

### Data Access Objects (DAOs)
**Location:** `com.futsch1.medtimer.database.MedicineDao`

Provides typed queries for:
- CRUD operations on all entities
- Complex queries joining multiple tables
- LiveData observables for reactive UI updates
- Flow-based queries for Kotlin coroutines

## User Interface Layer

### Navigation Structure
The app uses AndroidX Navigation Component with bottom navigation:

```
Main Activity
├── Overview Fragment (Home)
│   ├── Next Reminders View
│   ├── Manual Dose Entry
│   └── Edit Event Side Sheet
├── Medicines Fragment
│   ├── Medicine List
│   └── Edit Medicine Fragment
│       ├── Reminder List
│       ├── Advanced Settings
│       ├── Medicine Calendar
│       └── Stock Management
├── Statistics Fragment
│   ├── Charts View
│   ├── Calendar View
│   └── Table View
└── Settings Fragment
    ├── Notification Settings
    ├── Display Settings
    ├── Privacy Settings
    ├── Weekend Mode
    └── Alarm Settings
```

### Key Fragment Packages

#### Overview Package
**Location:** `com.futsch1.medtimer.overview`
- **OverviewFragment:** Main screen showing upcoming and recent reminders
- **NextReminders:** Displays next scheduled reminders
- **ManualDose:** Allows logging additional doses
- **EditEventSideSheetDialog:** Edit reminder events

#### Medicine Package
**Location:** `com.futsch1.medtimer.medicine`
- **MedicinesFragment:** List of all medications
- **EditMedicineFragment:** Add/edit medicine details
- **Advanced Settings:** Complex reminder configuration
- **Tags Management:** Organize medicines with tags
- **Stock Tracking:** Monitor medication inventory

#### Statistics Package
**Location:** `com.futsch1.medtimer.statistics`
- **ChartsFragment:** Visual analytics (charts, graphs)
- **CalendarFragment:** Calendar view of medication history
- **StatisticsFragment:** Tabular view with filtering

#### Preferences Package
**Location:** `com.futsch1.medtimer.preferences`
- **PreferencesFragment:** Main settings screen
- **NotificationSettingsFragment:** Notification customization
- **AlarmSettingsFragment:** Alarm-type notification settings
- **PrivacyPreferencesFragment:** Security settings

### ViewModel Architecture

ViewModels follow the MVVM pattern:
- Survive configuration changes
- Hold UI-related data
- Expose LiveData for observation
- Don't hold references to Views/Contexts
- Use Repository for data access

**Key ViewModels:**
- `MedicineViewModel`: Shared across multiple fragments
- `OverviewViewModel`: Overview screen state
- `CalendarEventsViewModel`: Calendar data

## Background Processing

### Work Manager Workers

The app uses WorkManager for all background tasks to ensure reliable execution:

#### Reminder Workers
**Location:** `com.futsch1.medtimer.reminders`

1. **RescheduleWorker**
   - Identifies next due reminders
   - Schedules reminders via AlarmManager
   - Runs when medicines or reminders change

2. **ReminderWorker**
   - Triggered at scheduled time by AlarmManager
   - Creates ReminderEvent entries in database
   - Generates and displays notification
   - Attaches PendingIntents for user actions

3. **TakenWorker**
   - Processes "taken" button clicks
   - Updates ReminderEvent status
   - Updates/closes notification
   - Handles stock decrements

4. **SkippedWorker**
   - Processes "skipped" button clicks
   - Updates ReminderEvent status
   - Updates/closes notification

5. **SnoozeWorker**
   - Cancels current notification
   - Reschedules reminder for later
   - Keeps ReminderEvent in "raised" state

6. **RepeatWorker**
   - Re-raises notification after delay
   - Used for repeat reminder feature
   - Keeps ReminderEvent in "raised" state

7. **StockHandlingWorker**
   - Monitors medication stock levels
   - Triggers low stock notifications

8. **ScheduleWorker**
   - Generic scheduler for delayed tasks

### AlarmManager Integration

For precise timing, the app uses AlarmManager:
- **Permission:** `SCHEDULE_EXACT_ALARM`
- **Purpose:** Ensures reminders fire at exact times
- **Integration:** Works with WorkManager via PendingIntent
- **Reliability:** Survives device reboots via `Autostart` receiver

### Notification System

**Location:** `com.futsch1.medtimer.reminders.Notifications`

The notification system provides:

#### Notification Channels
- **Default Priority:** Standard medication reminders
- **High Priority:** Important/urgent medications
- **Alarm Channel:** Alarm-type notifications (bypass silent mode)

#### Notification Features
- Action buttons: Taken, Skipped, Snooze
- Grouped notifications for multiple medicines
- Persistent notifications until acknowledged
- Custom sounds per priority level
- Full-screen intent for alarm notifications
- Repeat reminders at configurable intervals

#### Notification Data Flow
1. **Scheduling:** RescheduleWorker → AlarmManager
2. **Triggering:** AlarmManager → ReminderWorker
3. **Display:** ReminderWorker → NotificationManager
4. **User Action:** PendingIntent → Action Worker (Taken/Skipped/Snooze)
5. **Update:** Action Worker → NotificationManager (update/dismiss)

See [reminder flow documentation](reminder_flow.md) for detailed flow diagrams.

## Reminder System

### Reminder Types

#### 1. Simple Daily Reminders
- Fixed time each day
- Optional day-of-week selection
- Most common use case

#### 2. Interval Reminders
- Repeat every N hours/minutes
- Optional daily time window constraints
- Useful for frequent medications

#### 3. Cyclic Reminders
- Active days followed by pause days
- Perfect for birth control pills
- Supports arbitrary cycle lengths

#### 4. Linked Reminders
- Multiple reminders in sequence
- Each reminder triggers after the previous
- Useful for complex dosing schedules

#### 5. Time Period Reminders
- Active only during specific date ranges
- Useful for tapering medications
- Multiple reminders can be scheduled with adjacent periods

### Reminder Scheduling Algorithm
**Location:** `com.futsch1.medtimer.reminders.scheduling.ReminderScheduler`

1. Query all active reminders from database
2. Calculate next occurrence for each reminder type
3. Apply constraints (day of week, time windows, cycles)
4. Group reminders with same timestamp
5. Schedule earliest group via AlarmManager
6. Store notification data for later processing

### Weekend Mode
**Location:** `com.futsch1.medtimer.preferences.WeekendModePreferencesFragment`

Special feature that:
- Delays reminders on selected days (typically weekends)
- Reschedules to a user-defined time
- Applies globally to all reminders
- Useful for sleeping in on days off

## Data Flow

### Complete Reminder Lifecycle

```
1. User Creates Medicine & Reminder
   └─> MedicineRepository.insert()
       └─> Room Database INSERT
           └─> LiveData update triggers observers

2. Scheduler Observes Changes
   └─> ReminderSchedulerService.updateMedicine()
       └─> ReminderProcessor.requestReschedule()
           └─> Enqueue RescheduleWorker

3. RescheduleWorker Calculates Next Reminder
   └─> ReminderScheduler.scheduleNextReminder()
       └─> Calculate timestamps for all active reminders
           └─> AlarmManager.setExactAndAllowWhileIdle()
               └─> Schedule PendingIntent for ReminderWorker

4. AlarmManager Fires at Scheduled Time
   └─> ReminderWorker.doWork()
       └─> Create ReminderEvent entries in DB
           └─> Build notification with actions
               └─> NotificationManager.notify()
                   └─> Display notification to user

5. User Interacts with Notification
   ├─> "Taken" button
   │   └─> TakenWorker.doWork()
   │       └─> Update ReminderEvent.status = TAKEN
   │           └─> Decrement medicine stock
   │               └─> Update/dismiss notification
   │
   ├─> "Skipped" button
   │   └─> SkippedWorker.doWork()
   │       └─> Update ReminderEvent.status = SKIPPED
   │           └─> Update/dismiss notification
   │
   └─> "Snooze" button
       └─> SnoozeWorker.doWork()
           └─> Cancel notification
               └─> Reschedule reminder for later
                   └─> Keep ReminderEvent.status = RAISED

6. Statistics & History
   └─> ReminderEvent entries persist in database
       └─> Available for charts, calendar, exports
```

### Data Export Flow

**Location:** `com.futsch1.medtimer.exporters`

The app supports multiple export formats:

#### CSV Export
- **Medicine List:** All medicines with details
- **Event History:** All reminder events with timestamps
- **Classes:** `CSVMedicineExport`, `CSVEventExport`

#### PDF Export
- Visual summary of medications
- Event history in formatted tables
- **Classes:** `PDFMedicineExport`, `PDFEventExport`

#### JSON Backup
- Complete database backup
- Medicines, reminders, events, tags
- Restore functionality
- **Class:** `BackupManager`

## Security & Privacy

### Privacy-First Design

1. **No Internet Access**
   - Permissions explicitly removed in manifest
   - All processing happens locally
   - No telemetry or analytics

2. **Local Storage Only**
   - Room database stored in app's private storage
   - Android system-level encryption when device is encrypted
   - No cloud sync or backup (user-controlled exports only)

3. **Secure Window**
   - Optional feature prevents screenshots/screen recording
   - Controlled via `FLAG_SECURE`
   - Protects sensitive medication information

4. **Biometric Authentication**
   - Optional app-level authentication
   - Uses AndroidX Biometric library
   - Supports fingerprint, face, etc.

### Permissions

**Required Permissions:**
- `POST_NOTIFICATIONS`: Display reminder notifications
- `SCHEDULE_EXACT_ALARM`: Precise reminder timing
- `RECEIVE_BOOT_COMPLETED`: Reschedule reminders after reboot
- `ACCESS_NOTIFICATION_POLICY`: Manage Do Not Disturb
- `USE_FULL_SCREEN_INTENT`: Alarm-type notifications
- `VIBRATE`: Notification vibration
- `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`: Prevent background restrictions

**Explicitly Removed:**
- `INTERNET`: No network access
- `ACCESS_NETWORK_STATE`: No network state checks

### Data Extraction Rules
**Location:** `app/src/main/res/xml/data_extraction_rules.xml`

- Backup disabled by default (`allowBackup="false"`)
- User controls all data export via in-app features

## Testing Strategy

### Unit Tests
**Location:** `app/src/test/`

- **Framework:** JUnit 5 (Jupiter)
- **Mocking:** Mockito for dependency mocking
- **Android Testing:** Robolectric for Android components
- **Fuzzing:** Jazzer for fuzz testing
- **Coverage Target:** High coverage for business logic

**Key Test Areas:**
- Database operations and migrations
- Reminder scheduling algorithms
- Date/time calculations
- Export/import functionality
- ViewModel logic

### Instrumented Tests
**Location:** `app/src/androidTest/`

- **Framework:** AndroidX Test, Espresso
- **UI Testing:** Espresso for UI interactions
- **Additional Tools:** Barista for simpler Espresso tests
- **Screenshots:** Fastlane Screengrab for automated screenshots
- **Orchestrator:** Test orchestration for isolation

**Key Test Areas:**
- End-to-end user flows
- Fragment navigation
- Notification interactions
- Database integration
- UI rendering

### Test Execution
```bash
# Unit tests
./gradlew testDebugUnitTest

# Instrumented tests
./gradlew connectedDebugAndroidTest

# Code coverage
./gradlew JacocoDebugCodeCoverage

# Fuzzing (specific tests)
./gradlew test -Dfuzzing=true
```

## Build & Deployment

### Build Configuration

**Build Variants:**
- **Debug:** Development build with coverage enabled
- **Release:** Production build for distribution

**Build Settings:**
- **Min SDK:** 28 (Android 9.0)
- **Target SDK:** 36 (Android 15)
- **Compile SDK:** 36
- **Java Version:** 17
- **Desugaring:** Enabled for Java 8+ APIs on older devices

### Gradle Structure

```
Root Project
├── build.gradle.kts (root configuration)
└── app/
    └── build.gradle.kts (app module configuration)
```

**Key Gradle Features:**
- Kotlin DSL for build scripts
- AndroidX Room with schema export
- Navigation Safe Args code generation
- Jacoco for code coverage
- SonarQube integration
- Play Store publishing plugin

### Code Quality Tools

1. **Android Lint**
   - Configuration: `lint.xml`
   - Enforces: Warnings as errors, abort on error
   - Runs on every build

2. **SonarQube**
   - Cloud-hosted analysis
   - Tracks code quality metrics
   - Coverage reporting integration

3. **Jacoco**
   - Combined unit and instrumented test coverage
   - XML and HTML reports
   - Exclusions for generated code

### Continuous Integration

**GitHub Actions Workflows:**

1. **Build Workflow** (`.github/workflows/build.yml`)
   - Builds debug and release APKs
   - Runs linting
   - Uploads build artifacts

2. **Test Workflow** (`.github/workflows/test.yml`)
   - Executes unit tests
   - Runs instrumented tests on emulator
   - Generates coverage reports
   - Uploads to SonarQube

3. **Monkey Testing** (`monkeyTest.sh`)
   - Random UI interaction testing
   - Stress testing for crashes

### Release Process

1. **Version Management**
   - `versionCode`: Integer incremented per release
   - `versionName`: Semantic versioning (e.g., 1.21.3)
   - Defined in `app/build.gradle.kts`

2. **Release Channels**
   - F-Droid: Open-source app store
   - Google Play: Internal/Beta/Production tracks
   - GitHub Releases: Direct APK downloads
   - IzzyOnDroid: Alternative F-Droid repository

3. **Fastlane Integration**
   - Automated screenshot generation
   - Play Store metadata management
   - Release deployment automation

## Project Structure

```
medTimer/
├── app/
│   ├── schemas/              # Room database schemas
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/futsch1/medtimer/
│   │   │   │   ├── alarm/              # Alarm activity
│   │   │   │   ├── database/           # Room database entities & DAOs
│   │   │   │   ├── exporters/          # CSV, PDF, JSON exporters
│   │   │   │   ├── helpers/            # Utility classes
│   │   │   │   ├── medicine/           # Medicine management UI
│   │   │   │   │   ├── advancedSettings/
│   │   │   │   │   ├── dialogs/
│   │   │   │   │   ├── editMedicine/
│   │   │   │   │   ├── editors/
│   │   │   │   │   └── tags/
│   │   │   │   ├── overview/           # Main overview UI
│   │   │   │   │   └── actions/
│   │   │   │   ├── preferences/        # Settings UI
│   │   │   │   ├── reminders/          # Reminder system
│   │   │   │   │   ├── notificationData/
│   │   │   │   │   ├── notificationFactory/
│   │   │   │   │   └── scheduling/
│   │   │   │   ├── remindertable/      # Table view for events
│   │   │   │   ├── statistics/         # Charts & analytics
│   │   │   │   └── widgets/            # Home screen widgets
│   │   │   │   ├── MainActivity.java
│   │   │   │   ├── ReminderSchedulerService.java
│   │   │   │   └── ... (other top-level classes)
│   │   │   ├── res/                    # Android resources
│   │   │   │   ├── layout/             # XML layouts
│   │   │   │   ├── navigation/         # Navigation graphs
│   │   │   │   ├── values/             # Strings, colors, themes
│   │   │   │   └── xml/                # Preferences, data rules
│   │   │   └── AndroidManifest.xml
│   │   ├── test/                       # Unit tests
│   │   └── androidTest/                # Instrumented tests
│   └── build.gradle.kts                # App module build config
├── doc/                                 # Documentation
│   ├── Architecture.md                  # This document
│   ├── UseCases.md                      # User scenarios
│   └── reminder_flow.md                 # Reminder flow diagrams
├── fastlane/                            # Release automation
├── gradle/                              # Gradle wrapper
├── .github/                             # CI/CD workflows
├── build.gradle.kts                     # Root build config
├── settings.gradle.kts                  # Gradle settings
├── README.md                            # Project overview
├── CONTRIBUTING.md                      # Contribution guidelines
├── LICENSE                              # Open-source license
└── SECURITY.md                          # Security policy
```

## Key Design Patterns

### 1. Repository Pattern
- **MedicineRepository** abstracts data access
- ViewModels depend on repository, not direct database access
- Enables easy testing with mock repositories

### 2. Observer Pattern
- LiveData for reactive UI updates
- Database changes automatically propagate to UI
- Lifecycle-aware to prevent memory leaks

### 3. Dependency Injection (Manual)
- Repositories passed to ViewModels via factories
- Context dependencies minimized
- Easier unit testing

### 4. Worker Pattern
- All background tasks use WorkManager Workers
- Ensures reliable execution
- Handles process death and recovery

### 5. Single Activity Architecture
- One MainActivity hosts all fragments
- Navigation Component handles fragment transactions
- Shared ViewModels for inter-fragment communication

## Development Guidelines

### Code Style
- **Java:** Follows standard Java conventions
- **Kotlin:** Follows Kotlin coding conventions
- **Lint:** Enforced via Android Lint (warnings as errors)
- **Comments:** Minimal unless explaining complex logic

### Database Migrations
- **Prefer:** AutoMigration with Room
- **Complex Changes:** Custom AutoMigrationSpec
- **Testing:** Schema validation tests
- **Schema Export:** Enabled for version tracking

### Adding New Features

1. **New Reminder Type:**
   - Update `Reminder` entity with new fields
   - Add migration if schema changes
   - Update `ReminderScheduler` calculation logic
   - Add UI in advanced settings
   - Add tests

2. **New Export Format:**
   - Create new class in `exporters` package
   - Implement `Export` interface
   - Add to OptionsMenu
   - Test export/import cycle

3. **New UI Screen:**
   - Create Fragment in appropriate package
   - Add to `navigation.xml`
   - Create ViewModel if needed
   - Add to bottom navigation if top-level

### Testing Requirements
- Unit tests for business logic
- Instrumented tests for UI flows
- Maintain >80% code coverage
- Test edge cases and error paths

## Performance Considerations

### Database Optimization
- Indexed foreign keys for fast joins
- Pagination for large datasets
- Background thread execution
- LiveData for efficient updates

### Memory Management
- ViewModels scope data to lifecycle
- Proper cleanup in `onDestroy()`
- Avoid Context leaks in workers
- Efficient bitmap handling for icons

### Battery Optimization
- WorkManager respects Doze mode
- AlarmManager for precise timing only
- Notification batching when possible
- Efficient scheduling algorithms

## Accessibility

- Content descriptions on all interactive elements
- Support for large text sizes
- High contrast themes available
- Screen reader compatible
- Touch targets meet minimum size requirements

## Localization

**Supported Languages:** 20+ languages including:
- English, German, Spanish, French, Italian, Portuguese
- Arabic, Chinese (Simplified & Traditional), Russian
- And more

**Translation Management:**
- Weblate integration for community translations
- String resources in `res/values-{locale}/`
- Locale config auto-generated
- RTL support enabled

## Future Architecture Considerations

### Potential Improvements

1. **Kotlin Migration**
   - Gradually migrate Java classes to Kotlin
   - Leverage coroutines instead of callbacks
   - Use Kotlin Flow for reactive streams

2. **Dependency Injection Framework**
   - Consider Hilt for automated DI
   - Reduce boilerplate factory code
   - Easier testing infrastructure

3. **Modularization**
   - Separate feature modules
   - Improve build times
   - Enable dynamic delivery

4. **Jetpack Compose**
   - Migrate UI to Compose declarative UI
   - Reduce XML layout complexity
   - Modern Android development

5. **Backup Service**
   - Encrypted cloud backup option
   - Maintain privacy-first approach
   - User-controlled, end-to-end encrypted

## Resources & References

- [Android Developer Guide](https://developer.android.com/)
- [Room Database](https://developer.android.com/training/data-storage/room)
- [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager)
- [Navigation Component](https://developer.android.com/guide/navigation)
- [Material Design](https://material.io/develop/android)

## Conclusion

MedTimer is a well-architected Android application following modern Android development best practices. It leverages MVVM architecture, Room database, WorkManager for background processing, and maintains a strict privacy-first approach. The codebase is organized into clear layers with separation of concerns, making it maintainable and extensible.

The reminder system is the core of the application, using a sophisticated scheduling algorithm that supports multiple reminder types while ensuring reliable, timely notifications through AlarmManager integration. All data remains local to the device, and the app functions completely offline, ensuring user privacy and data security.

The testing strategy includes both unit and instrumented tests with high coverage, and the CI/CD pipeline ensures code quality through automated linting, testing, and analysis. The project is well-documented with clear contribution guidelines and active community translation support.
