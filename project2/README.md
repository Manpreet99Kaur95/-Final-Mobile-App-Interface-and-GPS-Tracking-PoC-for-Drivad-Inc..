Updated Documentation with Your Exact Implementation

# Drivad Technical Architecture

## 🛠️ Tech Stack (Exact Versions)

| Component | Package | Version |
|-----------|---------|---------|
| Flutter SDK | flutter | ^3.10.7 |
| State Management | provider | ^6.1.1 |
| Firebase Core | firebase_core | ^3.1.0 |
| Firebase Auth | firebase_auth | ^5.1.0 |
| Cloud Firestore | cloud_firestore | ^5.0.0 |
| Mapbox Maps | mapbox_maps_flutter | ^2.4.1 |
| Geolocation | geolocator | ^10.1.0 |
| Charts | fl_chart | ^0.63.0 |
| Localization | flutter_localizations | SDK |

## 🏗️ Application Structure

lib/
├── main.dart                    # Entry point, Mapbox init, routes
├── firebase_options.dart        # Auto-generated Firebase config (FlutterFire CLI)
├── language_provider.dart       # Localization state management
├── auth/
│   ├── auth_gate.dart           # Role-based routing logic
│   ├── auth_service.dart        # Firebase Auth & Firestore operations
│   ├── login_page.dart          # UI for authentication
│   └── register_page.dart       # UI for registration (implied)
├── dashboards/
│   ├── driver_dashboard_page.dart
│   ├── advertiser_dashboard_page.dart
│   └── vendor_dashboard_page.dart
├── campaigns_page.dart
├── applications_page.dart
├── earnings_page.dart
├── analytics_page.dart
└── profile_page.dart



##  Authentication Architecture

### Auth Flow
1. **main.dart** initializes Firebase and Mapbox
2. **AuthGate** (StreamBuilder) listens to `authStateChanges`
3. On auth state change, **FutureBuilder** fetches role via `getOrCreateUserRole()`
4. Role-based switch routes to appropriate dashboard:
   - `'advertiser'` → `AdvertiserDashboardPage`
   - `'vendor'` → `VendorDashboardPage`
   - `'driver'` (default) → `DriverDashboardPage`

### Role Management Strategy
```dart
// From auth_service.dart - Your Implementation:

// 1. Registration saves role to Firestore
await _db.collection('users').doc(uid).set({
  'email': email.trim(),
  'role': role.toLowerCase(),  // Critical: stored as lowercase
  'createdAt': FieldValue.serverTimestamp(),
});

// 2. Role fetch with fallback logic
Future<String> getOrCreateUserRole(String uid) async {
  final doc = await ref.get(const GetOptions(source: Source.serverAndCache));
  
  if (doc.exists && hasRole) return role;
  
  // Auto-creates missing user docs as 'driver'
  await ref.set({
    'email': email,
    'role': 'driver',  // Default fallback
    'createdAt': FieldValue.serverTimestamp(),
  });
  return 'driver';
}

// 3. Real-time role stream for dynamic updates
Stream<String> userRoleStream(String uid) {
  return ref.snapshots().asyncMap((doc) async {
    // Creates default driver if doc missing
  });
}

Key Implementation Details:

    Role is always stored/truncated to lowercase
    10-second timeout on role fetch (defaults to driver)
    Uses Source.serverAndCache for offline resilience
    Auto-creates user documents if missing (defensive programming)

🗺️ Mapbox Integration
Initialization (main.dart)
dart
Copy

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize Mapbox with access token
  MapboxOptions.setAccessToken(
    'pk.eyJ1IjoiY2Fwc3RvbmUyMDI2...', // Your public token
  );
  
  // Edge-to-edge UI configuration
  SystemChrome.setSystemUIOverlayStyle(...);
  SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge);
  
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(...);
}

Your Configuration:

    Uses mapbox_maps_flutter v2.4.1 (native Mapbox SDK)
    Token set programmatically in main.dart (not in AndroidManifest/Info.plist)
    Edge-to-edge enabled for modern Android/iOS look

🌐 Localization Support
From main.dart:
dart
Copy

MultiProvider(
  providers: [ChangeNotifierProvider(create: (_) => LanguageProvider())],
  child: const MyApp(),
),

Implementation:

    Provider-based state management for language switching
    Supports English/French (based on memory of previous conversation)
    flutter_localizations from SDK included in dependencies

📊 Analytics Visualization
Using fl_chart ^0.63.0 for:

    Campaign performance charts
    Earnings history graphs
    Impressions/engagement analytics
    Driver activity statistics

4. USER_GUIDES
📘 Driver Guide
Getting Started:

    Download Drivad app
    Sign up with email or social login
    Select "Driver" role (or defaults to driver)
    Complete profile:
        Vehicle type
        License plate
        Average daily KM
        Upload vehicle photos

Finding Campaigns:

    Browse available campaigns in your area
    Filter by ad type (vinyl, magnetic, full-wrap)
    Check daily rate and campaign duration
    View advertiser ratings

Application Process:

    Tap "Apply" on desired campaign
    Wait for advertiser approval (notification sent)
    Once approved, schedule installation with vendor
    Get ad installed professionally

Earning Money:

    App tracks your location during active campaigns
    Drive your normal routes within target zones
    Earnings calculated based on distance + time
    View real-time earnings in "Earnings" tab
    Payment processed weekly/monthly (based on your settings)

Requirements:

    Keep app running during campaigns (background location)
    Maintain vehicle cleanliness
    Drive minimum required hours per week
    Report any damage immediately via app

📗 Advertiser Guide
Creating Campaigns:

    Select "Advertiser" role during registration
    Complete company profile
    Tap "Create Campaign"
    Set parameters:
        Campaign name & description
        Ad type (vinyl/magnetic/wrap)
        Budget & daily driver rate
        Target location (map selection)
        Target radius (km)
        Required vehicle types
        Campaign dates

Managing Applications:

    View all driver applications in "Applications" tab
    Review driver profiles (vehicle type, ratings, driving history)
    Approve or reject with optional reason
    Approved drivers proceed to installation scheduling

Tracking Performance:

    Real-time map showing active driver locations
    Analytics dashboard with:
        Total impressions (estimated)
        Geographic reach
        Distance covered by fleet
        Engagement metrics
    Export reports for stakeholders

Payment:

    Budget held in escrow when campaign starts
    Released to drivers based on verified driving time
    View all transactions in analytics section

📙 Vendor Guide
Installation Workflow:

    Receive installation job notification
    Contact driver to schedule appointment
    Meet driver at agreed location
    Professional installation of ad materials
    Upload completion photos via app
    Mark installation complete

Managing Schedule:

    View upcoming installations in calendar view
    Reschedule if conflicts arise (notify driver/advertiser)
    Track installation history
    Build portfolio with photo documentation

Quality Standards:

    Follow advertiser specifications exactly
    Ensure no damage to driver vehicles
    Clean, bubble-free application
    Photos required for verification

5. API_REFERENCE.md
Markdown
Copy
Code Preview

# Drivad API Reference

Firebase SDK Integration Patterns (Direct usage, no REST API)

## 🔥 Firebase Services Configuration

### Initialization
```dart
// main.dart
import 'firebase_options.dart';
import 'package:firebase_core/firebase_core.dart';

await Firebase.initializeApp(
  options: DefaultFirebaseOptions.currentPlatform,
);

Platforms Supported:

    Android (API key: AIzaSyDZ9-dOL0oWNtweIqYXLkC8k42ljqkt040)
    iOS (Bundle ID: com.example.drivadApp)
    Web
    macOS/Windows (web config reused)

👤 AuthService API
Class Definition
dart
Copy

class AuthService {
  final FirebaseAuth _auth;
  final FirebaseFirestore _db;
  
  // Getters
  User? get currentUser;
  Stream<User?> get authStateChanges;
}

Methods
login({required String email, required String password})
Returns: Future<UserCredential>
Usage:
dart
Copy

final authService = AuthService();
try {
  final cred = await authService.login(
    email: 'driver@example.com',
    password: 'password123',
  );
  // AuthGate automatically handles navigation
} on FirebaseAuthException catch (e) {
  // Handle invalid email, wrong password, user disabled
}

register({required String email, required String password, required String role})
Returns: Future<UserCredential>
Creates user document:
dart
Copy

// Auto-creates Firestore document at /users/{uid}
{
  'email': 'user@example.com',
  'role': 'driver',  // Lowercase enforced
  'createdAt': FieldValue.serverTimestamp()
}

Usage:
dart
Copy

final cred = await authService.register(
  email: 'advertiser@company.com',
  password: 'securePass',
  role: 'advertiser',  // 'driver', 'advertiser', or 'vendor'
);

getOrCreateUserRole(String uid)
Returns: Future<String> (role string)
Behavior:

    Fetches from Firestore with cache strategy
    Timeout: 10 seconds (defaults to 'driver')
    Auto-creates document if missing
    Always returns lowercase role

Usage:
dart
Copy

final role = await authService.getOrCreateUserRole(user.uid);
// Returns: 'driver', 'advertiser', or 'vendor'

userRoleStream(String uid)
Returns: Stream<String>
Real-time role updates:
dart
Copy

authService.userRoleStream(uid).listen((role) {
  // Triggered when admin changes user role
  updateUI(role);
});

logout()
Returns: Future<void>
Usage:
dart
Copy

await authService.logout();
// AuthGate automatically redirects to LoginPage

🗃️ Firestore Database Operations
User Collection
Create User (Auto on Register):
dart
Copy

await FirebaseFirestore.instance
    .collection('users')
    .doc(uid)
    .set({
      'email': email,
      'role': role.toLowerCase(),
      'createdAt': FieldValue.serverTimestamp(),
    });

Read User Role:
dart
Copy

final doc = await FirebaseFirestore.instance
    .collection('users')
    .doc(uid)
    .get(const GetOptions(source: Source.serverAndCache));
    
final role = doc.data()?['role'];

Update User:
dart
Copy

await FirebaseFirestore.instance
    .collection('users')
    .doc(uid)
    .update({
      'vehicleType': 'Sedan',
      'licensePlate': 'ABC123',
    });

Merge Update (Safe):
dart
Copy

await ref.set({
  'role': 'driver',
  'updatedAt': FieldValue.serverTimestamp(),
}, SetOptions(merge: true));

Campaign Collection
Query Active Campaigns:
dart
Copy

FirebaseFirestore.instance
    .collection('campaigns')
    .where('status', isEqualTo: 'active')
    .where('startDate', isLessThanOrEqualTo: Timestamp.now())
    .orderBy('createdAt', descending: true)
    .snapshots();

Create Campaign (Advertiser only):
dart
Copy

await FirebaseFirestore.instance.collection('campaigns').add({
  'advertiserId': currentUser.uid,
  'title': 'Summer Sale 2026',
  'description': 'Promote our summer collection',
  'adType': 'vinyl',  // vinyl, magnetic, full-wrap
  'budget': 5000.00,
  'dailyRate': 50.00,
  'startDate': Timestamp.fromDate(startDate),
  'endDate': Timestamp.fromDate(endDate),
  'targetLocation': GeoPoint(latitude, longitude),
  'targetRadius': 25.0,  // km
  'requiredVehicleTypes': ['Sedan', 'SUV'],
  'status': 'active',
  'maxDrivers': 20,
  'createdAt': FieldValue.serverTimestamp(),
});

Applications Collection
Apply to Campaign (Driver):
dart
Copy

await FirebaseFirestore.instance.collection('applications').add({
  'campaignId': campaignId,
  'driverId': currentUser.uid,
  'status': 'pending',
  'appliedAt': FieldValue.serverTimestamp(),
});

Approve Application (Advertiser):
dart
Copy

await FirebaseFirestore.instance
    .collection('applications')
    .doc(applicationId)
    .update({
      'status': 'approved',
      'respondedAt': FieldValue.serverTimestamp(),
    });

Stream Driver Applications:
dart
Copy

FirebaseFirestore.instance
    .collection('applications')
    .where('driverId', isEqualTo: currentUser.uid)
    .orderBy('appliedAt', descending: true)
    .snapshots();

🗺️ Mapbox Maps API
Map Initialization
dart
Copy

import 'package:mapbox_maps_flutter/mapbox_maps_flutter.dart';

MapboxMap(
  onMapCreated: _onMapCreated,
  initialCameraPosition: CameraPosition(
    center: Point(coordinates: Position(longitude, latitude)),
    zoom: 12.0,
  ),
);

Add Marker
dart
Copy

await mapboxMap.annotations.createPointAnnotationManager().then((manager) {
  manager.create(
    PointAnnotationOptions(
      geometry: Point(coordinates: Position(long, lat)),
      iconSize: 1.0,
      iconImage: 'marker-icon',
    ),
  );
});

📍 Geolocator API (Location Tracking)
Check Permissions
dart
Copy

import 'package:geolocator/geolocator.dart';

LocationPermission permission = await Geolocator.checkPermission();
if (permission == LocationPermission.denied) {
  permission = await Geolocator.requestPermission();
}

Get Current Position
dart
Copy

Position position = await Geolocator.getCurrentPosition(
  desiredAccuracy: LocationAccuracy.high,
);
// position.latitude, position.longitude

Location Stream (Background Tracking)
dart
Copy

StreamSubscription<Position> positionStream = Geolocator.getPositionStream(
  locationSettings: const LocationSettings(
    accuracy: LocationAccuracy.high,
    distanceFilter: 100, // meters
  ),
).listen((Position position) {
  // Send to Firestore every 100 meters
  _updateDriverLocation(position);
});

📊 fl_chart API (Analytics)
Bar Chart (Earnings)
dart
Copy

import 'package:fl_chart/fl_chart.dart';

BarChart(
  BarChartData(
    barGroups: [
      BarChartGroupData(
        x: 0,
        barRods: [BarChartRodData(toY: 150, color: Colors.blue)],
      ),
      // ...
    ],
  ),
);

Line Chart (Campaign Trends)
dart
Copy

LineChart(
  LineChartData(
    lineBarsData: [
      LineChartBarData(
        spots: [
          const FlSpot(0, 100),
          const FlSpot(1, 200),
          // ...
        ],
        isCurved: true,
        color: Colors.green,
      ),
    ],
  ),
);

🔄 State Management (Provider)
LanguageProvider Usage
dart
Copy

// In main.dart
ChangeNotifierProvider(create: (_) => LanguageProvider())

// In widget
final language = context.watch<LanguageProvider>();
// or
final language = Provider.of<LanguageProvider>(context);

Auth State in UI
dart
Copy

// StreamBuilder pattern (used in AuthGate)
StreamBuilder<User?>(
  stream: authService.authStateChanges,
  builder: (context, snapshot) {
    if (snapshot.hasData) return Dashboard();
    return LoginPage();
  },
);

🔒 Security Rules Reference
Firestore Rules (Recommended)
JavaScript
Copy

rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can read their own data, create on register
    match /users/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow create: if request.auth != null && request.auth.uid == userId;
      allow update: if request.auth != null && request.auth.uid == userId;
    }
    
    // Campaigns: public read, advertiser write
    match /campaigns/{campaignId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'advertiser';
      allow update, delete: if request.auth != null && 
        request.auth.uid == resource.data.advertiserId;
    }
    
    // Applications: involved parties only
    match /applications/{appId} {
      allow read: if request.auth != null && (
        request.auth.uid == resource.data.driverId || 
        request.auth.uid == get(/databases/$(database)/documents/campaigns/$(resource.data.campaignId)).data.advertiserId
      );
      allow create: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'driver';
      allow update: if request.auth != null && 
        request.auth.uid == get(/databases/$(database)/documents/campaigns/$(resource.data.campaignId)).data.advertiserId;
    }
  }
}

API Version: 1.0.0+1 | Flutter: ^3.10.7 | Last Updated: April 2026
plain
Copy


---

## 📋 Quick Setup Checklist for Your Teammates

**Immediate Actions:**
1. [ ] Share `google-services.json` (Android) and `GoogleService-Info.plist` (iOS)
2. [ ] Share Mapbox access token (public token starts with `pk.`)
3. [ ] Run `flutter pub get` to install exact versions from pubspec.yaml
4. [ ] Configure Firestore rules (copy from API_REFERENCE.md)
5. [ ] Test on emulator: `flutter run`

**Code Organization:**
- Your app uses **Provider** (not Bloc/Riverpod) - remember this when adding new features
- Role logic defaults to **'driver'** if anything fails - safe fallback
- **AuthGate** handles all navigation - don't use `Navigator.pushReplacement` for auth changes
- Mapbox token is in **main.dart** - change there if token expires

**Common Issues:**
- If map doesn't load: Check Mapbox token quota (free tier has limits)
- If role stuck: Clear app data (role cached with 10s timeout)
- If location fails: Check `AndroidManifest.xml` has location permissions

---

**Want me to create any additional specific documentation?** For example:
- **Testing Guide** (unit tests, widget tests, integration tests)
- **Deployment Guide** (Play Store, App Store, Firebase Hosting)
- **Admin Dashboard Guide** (if you have a web admin panel)
- **API Integration Guide** (if you add Stripe for payments later)

I NEEED THEM ALL IN A SINGLE README FILE?
Edit
Copy
Share
Combine Multiple Docs into One Structured Markdown File
The user wants all the documentation consolidated into a single README.md file instead of multiple separate files. I need to merge everything I've created into one comprehensive README.md with proper structure, table of contents, and formatting.
The user provided:

    pubspec.yaml details
    firebase_options.dart (with actual API keys - I should warn about security)
    main.dart structure
    auth_gate.dart logic
    auth_service.dart implementation

I need to combine:

    README overview (from first response)
    SETUP.md content (adapted to single file format)
    ARCHITECTURE.md content (with their actual code)
    API_REFERENCE.md content (Firebase SDK patterns)
    USER_GUIDES (condensed into sections)

It needs to be:

    A single markdown file
    Well-structured with TOC
    Security warnings about exposed keys
    All the technical details from their code
    Professional formatting

Let me create this comprehensive single-file README.md
I'll create one comprehensive README.md that includes everything - your setup instructions, architecture, API reference, and user guides all in a single file.
Markdown
Copy
Code Preview

# Drivad 🚗📢

**Mobile Vehicle Advertising Platform MVP**  
Connecting advertisers with car owners through geo-targeted mobile advertising

[![Flutter Version](https://img.shields.io/badge/Flutter-3.10.7-blue.svg)](https://flutter.dev)
[![Firebase](https://img.shields.io/badge/Firebase-Cloud%20Services-orange.svg)](https://firebase.google.com)
[![Mapbox](https://img.shields.io/badge/Mapbox-2.4.1-black.svg)](https://www.mapbox.com)

---

## 📋 Table of Contents
1. [Project Overview](#1-project-overview)
2. [Team Members](#2-team-members)
3. [Tech Stack](#3-tech-stack)
4. [Features](#4-features)
5. [Quick Start](#5-quick-start)
6. [Architecture](#6-architecture)
7. [Setup Instructions](#7-setup-instructions)
8. [API Reference](#8-api-reference)
9. [User Guides](#9-user-guides)
10. [Security Rules](#10-security-rules)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Project Overview

Drivad is a three-sided marketplace mobile application enabling:
- **Drivers** to earn money by displaying advertisements on their vehicles
- **Advertisers** to create geo-targeted campaigns and track performance  
- **Vendors** to manage professional installation of ad materials

**Academic Project** | Cross-platform Mobile App | Real-time Location Tracking

---

## 2. Team Members

- **Balogun Oluwafemi**
- **Sakshi Sakshi**
- **Harleen Kaur**
- **Manpreet Kaur**

---

## 3. Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Frontend** | Flutter SDK | ^3.10.7 |
| **State Management** | Provider | ^6.1.1 |
| **Backend** | Firebase Core | ^3.1.0 |
| **Authentication** | Firebase Auth | ^5.1.0 |
| **Database** | Cloud Firestore | ^5.0.0 |
| **Maps & GPS** | Mapbox Maps Flutter | ^2.4.1 |
| **Geolocation** | Geolocator | ^10.1.0 |
| **Charts** | fl_chart | ^0.63.0 |
| **Localization** | Flutter Localizations | SDK |

---

## 4. Features

### For Drivers
- 📍 Real-time GPS tracking during active campaigns (background location)
- 💰 Earnings dashboard with payment history and analytics
- 🔔 Campaign application system with approval workflow
- 🗺️ Mapbox integration for route visualization and target zones
- 📊 fl_chart visualizations for earnings tracking

### For Advertisers
- 📊 Campaign creation with geo-targeting (Mapbox circle radius)
- 📈 Real-time analytics dashboard (impressions, reach, distance)
- ✅ Driver application review and approval system
- 💳 Campaign budget and daily rate management
- 🗺️ Live fleet tracking on Mapbox maps

### For Vendors
- 📅 Installation scheduling system with calendar
- ✅ Installation completion verification with photo upload
- 📋 Installation history and portfolio management
- 🔔 Real-time job assignment notifications

---

## 5. Quick Start

### Prerequisites
- Flutter SDK ^3.10.7
- Android Studio / VS Code
- Firebase CLI (`npm install -g firebase-tools`)
- Git

### Installation

```bash
# Clone repository
git clone [your-repo-url]
cd drivad_app

# Install dependencies (exact versions from pubspec.yaml)
flutter pub get

# Verify setup
flutter doctor

⚠️ CRITICAL: Security Configuration
The repository currently contains hardcoded API keys. For production:

    Create .env file (add to .gitignore):

env
Copy

MAPBOX_PUBLIC_TOKEN=your_mapbox_token

    Move Firebase config to environment:
        Use --dart-define flags or flutter_dotenv package
        Never commit firebase_options.dart with real keys to public repos
        Rotate keys immediately if this becomes public
    Run with environment:

bash
Copy

flutter run --dart-define=MAPBOX_TOKEN=your_token

Running the App
bash
Copy

# Development
flutter run

# Production build
flutter build apk --release          # Android APK
flutter build appbundle --release    # Android App Bundle (Play Store)
flutter build ios --release          # iOS

6. Architecture
Project Structure
plain
Copy

lib/
├── main.dart                    # Entry point, Mapbox init, routes
├── firebase_options.dart        # Firebase config (REMOVE/HIDE KEYS FOR PROD)
├── language_provider.dart       # Localization state management
├── auth/
│   ├── auth_gate.dart           # Role-based routing logic
│   ├── auth_service.dart        # Firebase Auth & Firestore operations
│   └── login_page.dart          # Authentication UI
├── dashboards/
│   ├── driver_dashboard_page.dart
│   ├── advertiser_dashboard_page.dart
│   └── vendor_dashboard_page.dart
├── campaigns_page.dart
├── applications_page.dart
├── earnings_page.dart
├── analytics_page.dart
└── profile_page.dart

Authentication Flow
dart
Copy

// 1. main.dart initializes Firebase + Mapbox
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  MapboxOptions.setAccessToken('pk.your_token'); // MOVE TO ENV
  
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  
  runApp(
    MultiProvider(
      providers: [ChangeNotifierProvider(create: (_) => LanguageProvider())],
      child: const MyApp(),
    ),
  );
}

// 2. AuthGate handles role-based routing
class AuthGate extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: authService.authStateChanges,
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          return FutureBuilder<String>(
            future: authService.getOrCreateUserRole(snapshot.data!.uid),
            builder: (context, roleSnapshot) {
              final role = roleSnapshot.data ?? 'driver';
              switch (role) {
                case 'advertiser': return AdvertiserDashboardPage();
                case 'vendor': return VendorDashboardPage();
                default: return DriverDashboardPage();
              }
            },
          );
        }
        return LoginPage();
      },
    );
  }
}

State Management (Provider)

    LanguageProvider: Manages app localization (English/French)
    AuthService: Firebase authentication state
    Role-based routing: Automatic dashboard selection based on Firestore role field

Data Models
User Model
dart
Copy

// Stored in Firestore: /users/{uid}
{
  'email': 'user@example.com',
  'role': 'driver',  // 'driver', 'advertiser', or 'vendor' (lowercase)
  'createdAt': FieldValue.serverTimestamp(),
  // Driver-specific:
  'vehicleType': 'Sedan',
  'licensePlate': 'ABC123',
  // Advertiser-specific:
  'companyName': 'Acme Corp',
  // Vendor-specific:
  'businessName': 'WrapPro Installers'
}

Campaign Model
dart
Copy

// Stored in Firestore: /campaigns/{campaignId}
{
  'advertiserId': 'uid',
  'title': 'Summer Sale 2026',
  'description': 'Promote summer collection',
  'adType': 'vinyl',  // vinyl, magnetic, full-wrap
  'budget': 5000.00,
  'dailyRate': 50.00,
  'startDate': Timestamp,
  'endDate': Timestamp,
  'targetLocation': GeoPoint(lat, lng),
  'targetRadius': 25.0,  // km
  'requiredVehicleTypes': ['Sedan', 'SUV'],
  'status': 'active',
  'maxDrivers': 20
}

Application Model
dart
Copy

// Stored in Firestore: /applications/{appId}
{
  'campaignId': 'campaign_id',
  'driverId': 'driver_uid',
  'status': 'pending',  // pending, approved, rejected
  'appliedAt': Timestamp,
  'respondedAt': Timestamp,
  'rejectionReason': 'Vehicle too old'
}

7. Setup Instructions
7.1 Firebase Configuration

    Create Firebase Project: Firebase Console
        Project ID: drivadnew (or create new)
    Register Apps:
        Android: Package name com.example.drivad_app
            Download google-services.json → place in android/app/
        iOS: Bundle ID com.example.drivadApp
            Download GoogleService-Info.plist → place in ios/Runner/
    Enable Services:
        Authentication: Email/Password, Google, Apple, Facebook
        Firestore: Create database in test mode (update rules later)
        Storage: For campaign images
    Generate Firebase Config:
    bash
    Copy

    npm install -g firebase-tools
    flutterfire configure  # Generates firebase_options.dart

7.2 Mapbox Configuration

    Create account at Mapbox
    Get Public Token (starts with pk.)
    Get Secret Token (starts with sk.) with DOWNLOADS:READ scope
    Add to android/app/build.gradle:
    gradle
    Copy

    // Add Mapbox repository and credentials

    Secure Token Storage:
    dart
    Copy

    // main.dart - CURRENTLY HARDCODED (SECURITY RISK)
    MapboxOptions.setAccessToken('pk.your_token_here');

7.3 Social Authentication Setup
Google Sign-In:

    Add SHA-1 fingerprint in Firebase Console
    Download updated google-services.json

Apple Sign-In:

    Configure in Apple Developer Portal
    Enable in Firebase Console → Authentication

Facebook Login:

    Create app at Facebook Developers
    Add Firebase OAuth redirect URI
    Add App ID and Secret to Firebase Console

7.4 Dependencies
Your pubspec.yaml:
yaml
Copy

name: drivad_app
description: "A new Flutter project."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.10.7

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  fl_chart: ^0.63.0
  provider: ^6.1.1
  firebase_core: ^3.1.0
  firebase_auth: ^5.1.0
  cloud_firestore: ^5.0.0
  flutter_localizations:
    sdk: flutter
  mapbox_maps_flutter: ^2.4.1
  geolocator: ^10.1.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true

8. API Reference
AuthService API
dart
Copy

class AuthService {
  // Getters
  User? get currentUser;
  Stream<User?> get authStateChanges;
  
  // Authentication
  Future<UserCredential> login({required String email, required String password});
  Future<UserCredential> register({required String email, required String password, required String role});
  Future<void> logout();
  
  // Role Management
  Future<String> getOrCreateUserRole(String uid);  // 10s timeout, defaults to 'driver'
  Stream<String> userRoleStream(String uid);      // Real-time role updates
}

Usage Examples:
dart
Copy

// Login
final cred = await authService.login(
  email: 'driver@example.com',
  password: 'password123',
);

// Register with role
final cred = await authService.register(
  email: 'advertiser@company.com',
  password: 'securePass',
  role: 'advertiser',  // 'driver', 'advertiser', or 'vendor'
);

// Role stream (for admin role changes)
authService.userRoleStream(uid).listen((role) {
  if (role == 'banned') logout();
});

Firestore Operations
dart
Copy

// Read with cache strategy
final doc = await FirebaseFirestore.instance
    .collection('users')
    .doc(uid)
    .get(const GetOptions(source: Source.serverAndCache));

// Create user document
await FirebaseFirestore.instance
    .collection('users')
    .doc(uid)
    .set({
      'email': email,
      'role': role.toLowerCase(),  // Always lowercase
      'createdAt': FieldValue.serverTimestamp(),
    });

// Merge update (safe)
await docRef.set({
  'vehicleType': 'SUV',
}, SetOptions(merge: true));

// Query campaigns
FirebaseFirestore.instance
    .collection('campaigns')
    .where('status', isEqualTo: 'active')
    .orderBy('createdAt', descending: true)
    .snapshots()
    .listen((snapshot) {
      // Update UI
    });

Mapbox Maps API
dart
Copy

import 'package:mapbox_maps_flutter/mapbox_maps_flutter.dart';

// Initialize (in main.dart)
MapboxOptions.setAccessToken('pk.your_token');

// Create map widget
MapboxMap(
  onMapCreated: _onMapCreated,
  initialCameraPosition: CameraPosition(
    center: Point(coordinates: Position(longitude, latitude)),
    zoom: 12.0,
  ),
);

// Add marker
await mapboxMap.annotations.createPointAnnotationManager().then((manager) {
  manager.create(
    PointAnnotationOptions(
      geometry: Point(coordinates: Position(long, lat)),
      iconSize: 1.0,
    ),
  );
});

Geolocator API (GPS Tracking)
dart
Copy

import 'package:geolocator/geolocator.dart';

// Check permissions
LocationPermission permission = await Geolocator.checkPermission();
if (permission == LocationPermission.denied) {
  permission = await Geolocator.requestPermission();
}

// Get current position
Position position = await Geolocator.getCurrentPosition(
  desiredAccuracy: LocationAccuracy.high,
);

// Real-time tracking (background)
StreamSubscription<Position> positionStream = Geolocator.getPositionStream(
  locationSettings: const LocationSettings(
    accuracy: LocationAccuracy.high,
    distanceFilter: 100, // Update every 100 meters
  ),
).listen((Position position) {
  // Upload to Firestore
  _updateLocation(position.latitude, position.longitude);
});

fl_chart API (Analytics)
dart
Copy

import 'package:fl_chart/fl_chart.dart';

// Bar chart for earnings
BarChart(
  BarChartData(
    barGroups: [
      BarChartGroupData(
        x: 0,
        barRods: [BarChartRodData(toY: 150, color: Colors.blue)],
      ),
    ],
  ),
);

// Line chart for campaign trends
LineChart(
  LineChartData(
    lineBarsData: [
      LineChartBarData(
        spots: [const FlSpot(0, 100), const FlSpot(1, 200)],
        isCurved: true,
        color: Colors.green,
      ),
    ],
  ),
);

9. User Guides
📘 Driver Guide
Getting Started:

    Download Drivad app
    Sign up with email or social login (defaults to Driver role)
    Complete profile: vehicle type, license plate, average daily KM
    Upload vehicle photos

Finding Campaigns:

    Browse available campaigns in your area using Mapbox map
    Filter by ad type (vinyl, magnetic, full-wrap) and daily rate
    Check target zones (highlighted on map)
    Apply with one tap

Application Process:

    Tap "Apply" on desired campaign
    Wait for advertiser approval (push notification)
    Once approved, vendor contacts you for installation scheduling
    Get ad installed professionally
    Start driving - app tracks location automatically

Earning Money:

    Keep app running during campaigns (background location enabled)
    Drive your normal routes within target zones
    View real-time earnings in "Earnings" tab with fl_chart graphs
    Payments processed based on distance + time in zones

Requirements:

    Minimum driving hours per week (varies by campaign)
    Keep vehicle clean
    Report ad damage immediately via app
    Allow location permissions "Always" for background tracking

📗 Advertiser Guide
Creating Campaigns:

    Select "Advertiser" role during registration
    Tap "Create Campaign"
    Set parameters:
        Basic: Name, description, ad type, images
        Budget: Total budget, daily driver rate (e.g., $50/day)
        Targeting: Select center point on Mapbox map, set radius (km)
        Vehicles: Required types (Sedan, SUV, Truck, etc.)
        Duration: Start and end dates
    Publish campaign

Managing Applications:

    View incoming driver applications in real-time
    Review driver profiles: vehicle photos, driving history, ratings
    Approve or reject with optional reason message
    Approved drivers automatically assigned to installation queue

Tracking Performance:

    Live Mapbox view showing all active driver locations
    Analytics dashboard:
        Total distance covered by fleet
        Estimated impressions based on location data
        Geographic heatmap of coverage
        Individual driver performance metrics
    Export CSV reports

Payments:

    Campaign budget held in escrow
    Automatic release to drivers based on verified GPS tracking
    View transaction history in analytics section

📙 Vendor Guide
Installation Workflow:

    Receive job assignment notification
    Contact driver via in-app messaging or phone
    Schedule installation appointment (synced to your calendar)
    Meet driver at agreed location
    Professional installation following advertiser specifications
    Upload completion photos (before/after) via app
    Mark installation complete - triggers driver notification

Quality Standards:

    Follow exact advertiser specifications for ad placement
    Ensure bubble-free, clean application
    No damage to driver vehicle paint/windows
    Photo documentation required for verification

Schedule Management:

    View upcoming installations in calendar view
    Reschedule if conflicts arise (notifies driver & advertiser)
    Track installation history and earnings
    Build reputation through ratings

10. Security Rules
Firestore Security Rules
Add to Firebase Console → Firestore Database → Rules:
JavaScript
Copy

rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
  
    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }
    
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }
    
    function getUserRole() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role;
    }
    
    function isDriver() {
      return getUserRole() == 'driver';
    }
    
    function isAdvertiser() {
      return getUserRole() == 'advertiser';
    }
    
    function isVendor() {
      return getUserRole() == 'vendor';
    }

    // Users collection
    match /users/{userId} {
      allow read: if isOwner(userId);
      allow create: if isOwner(userId);
      allow update: if isOwner(userId);
    }
    
    // Campaigns
    match /campaigns/{campaignId} {
      allow read: if isAuthenticated();
      allow create: if isAuthenticated() && isAdvertiser();
      allow update: if isAuthenticated() && 
        (request.auth.uid == resource.data.advertiserId || isAdmin());
      allow delete: if isAuthenticated() && 
        request.auth.uid == resource.data.advertiserId;
    }
    
    // Applications
    match /applications/{applicationId} {
      allow read: if isAuthenticated() && (
        request.auth.uid == resource.data.driverId || 
        request.auth.uid == get(/databases/$(database)/documents/campaigns/$(resource.data.campaignId)).data.advertiserId
      );
      allow create: if isAuthenticated() && isDriver();
      allow update: if isAuthenticated() && 
        request.auth.uid == get(/databases/$(database)/documents/campaigns/$(resource.data.campaignId)).data.advertiserId;
    }
    
    // Installations
    match /installations/{installationId} {
      allow read: if isAuthenticated() && (
        request.auth.uid == resource.data.vendorId ||
        request.auth.uid == resource.data.driverId ||
        request.auth.uid == get(/databases/$(database)/documents/campaigns/$(resource.data.campaignId)).data.advertiserId
      );
      allow create: if isAuthenticated() && isAdvertiser();
      allow update: if isAuthenticated() && request.auth.uid == resource.data.vendorId;
    }
    
    // Earnings
    match /earnings/{earningId} {
      allow read: if isAuthenticated() && request.auth.uid == resource.data.driverId;
      allow create: if isAuthenticated() && isAdvertiser(); // System creates on behalf
    }
    
    // Location tracking (subcollection under driver)
    match /drivers/{driverId}/location_tracking/{docId} {
      allow read: if isAuthenticated() && (
        isOwner(driverId) || 
        request.auth.uid in get(/databases/$(database)/documents/campaigns/$(resource.data.campaignId)).data.advertiserId
      );
      allow create, update: if isAuthenticated() && isOwner(driverId);
    }
  }
}

Android Permissions
Add to android/app/src/main/AndroidManifest.xml:
xml
Copy

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>

iOS Permissions
Add to ios/Runner/Info.plist:
xml
Copy

<key>NSLocationWhenInUseUsageDescription</key>
<string>This app needs location to track campaign coverage</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>This app needs background location to calculate earnings during campaigns</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>Background location is required to verify driving in target zones</string>

Troubleshooting
Mapbox shows blank/gray map	Token expired or quota exceeded >>Check Mapbox dashboard.
Role always defaults to "driver"	10-second timeout hit >> Check Firestore connection. Clear app data.
Location not updating in background	Android >>Check battery optimization whitelist. iOS: Verify "Always" permission granted.
Firestore permission denied	>> Check security rules match your document paths exactly.
Social login not working	Android >> Verify SHA-1 fingerprints in Firebase. iOS: Check URL schemes in Info.plist.
App crashes on startup	>> Run flutter clean then flutter pub get. Check Firebase initialization.
Charts not showing data	>> Verify data format matches fl_chart requirements (double values).

Debug Commands
Clean build >> flutter clean && flutter pub get

Check Firebase connection >> flutter run --verbose | grep firebase

Build with debug symbols >> flutter build apk --debug

Check dependency tree >> flutter pub deps

Testing Checklist

  Email/password login works
  Social logins (Google/Apple/Facebook) work
  Role selection persists after restart
  Mapbox loads and shows current location
  Can create campaign (as advertiser)
  Can apply to campaign (as driver)
  Location tracking works in background (Android 10+)
  Real-time earnings calculate correctly
  Analytics charts render without errors
  Installation photos upload successfully

License & Academic Use
This is an academic project developed for educational purposes. Not for commercial deployment without significant security hardening.
Built with Flutter 💙 | Powered by Firebase 🔥 | Maps by Mapbox 🗺️
Last Updated: April 14, 2026




