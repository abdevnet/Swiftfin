# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Swiftfin is a native SwiftUI video client for Jellyfin media servers, supporting both iOS (16+) and tvOS (17+). The project uses VLCKit for video playback to maximize codec support and direct play capabilities.

## Setup

Install dependencies using Homebrew and Carthage:

```bash
brew install carthage swiftformat swiftgen
carthage update --use-xcframeworks
```

### Development Team Configuration

Create `XcodeConfig/DevelopmentTeam.xcconfig` with your Apple Developer Team ID:

```
DEVELOPMENT_TEAM = "YOUR_TEAM_ID"
PRODUCT_BUNDLE_IDENTIFIER = org.jellyfin.swiftfin
```

This file is gitignored. Find your Team ID in your Apple Developer account membership details or by temporarily setting Development Team in Xcode's Signing & Capabilities tab.

## Build Commands

```bash
# Build iOS app
xcodebuild -scheme "Swiftfin" -destination 'platform=iOS Simulator,name=iPhone 15' build

# Build tvOS app
xcodebuild -scheme "Swiftfin tvOS" -destination 'platform=tvOS Simulator,name=Apple TV' build

# Run SwiftFormat linter (required before PR)
swiftformat .

# Generate localization strings
swiftgen
```

## Architecture

### Platform Structure

- **Shared/**: Core business logic, models, services, and ViewModels shared between iOS and tvOS
  - `SwiftfinStore/`: CoreStore-based data persistence layer with SQLite backend
  - `Coordinators/`: Navigation system using custom Router pattern
  - `Services/`: Business logic including DownloadManager, UserSession, SwiftfinDefaults, Keychain
  - `ViewModels/`: MVVM pattern ViewModels for UI state management
  - `Objects/`: Domain models and business objects including VideoPlayerType
  - `Extensions/`: Swift standard library and SwiftUI extensions
  - `Components/`: Reusable SwiftUI components

- **Swiftfin/**: iOS-specific views and UI
- **Swiftfin tvOS/**: tvOS-specific views and UI optimized for TV interaction

### Navigation

Uses a custom coordinator/router pattern:
- `NavigationCoordinator`: Manages navigation stack and routing
- `Router`: Property wrapper for declarative navigation in views
- `RootCoordinator`: Manages root-level navigation (tab changes, auth flow)

Access routing in views via `@Router` property wrapper:
```swift
@Router var router
router.route(to: .item(item))
```

### Data Layer

- **CoreStore**: Primary persistence framework using SQLite
- **SwiftfinStore**: Manages server and user state with V1→V2 migration chain
- **Factory**: Dependency injection for services and shared instances
- **UserSession**: Manages authenticated user sessions with Jellyfin API

### Video Playback

Two player implementations:
- **Swiftfin Player (VLCKit)**: Primary player with extensive codec support, subtitle handling, custom UI controls (recommended)
- **Native Player (AVKit)**: Alternative using Apple's AVFoundation for framerate matching and PiP support

VideoPlayerType objects define player capabilities and are located in `Shared/Objects/VideoPlayerType/`.

## Code Standards

### SwiftFormat Configuration

The project uses SwiftFormat with specific rules in `.swiftformat`:
- 140 character line width
- 4-space tabs with Xcode indentation
- Custom file header with MPL 2.0 license
- Excludes: `Shared/Strings/Strings.swift` (generated), `fastlane/`

Key enabled rules: `isEmpty`, `leadingDelimiters`, `wrapEnumCases`, `typeSugar`
Key disabled rules: `redundantSelf`, `yodaConditions`, `andOperator`

### Localization

Strings are managed via SwiftGen:
- Source: `Translations/en.lproj/`
- Generated: `Shared/Strings/Strings.swift` (do not edit manually)
- New user-facing strings must be localized (unless experimental features)
- Run `swiftgen` after adding new localizable strings

### File Headers

All Swift files must include the MPL 2.0 header (applied by SwiftFormat):
```swift
//
// Swiftfin is subject to the terms of the Mozilla Public
// License, v2.0. If a copy of the MPL was not distributed with this
// file, you can obtain one at https://mozilla.org/MPL/2.0/.
//
// Copyright (c) {year} Jellyfin & Jellyfin Contributors
//
```

## Key Dependencies

- **VLCKit**: Video playback engine (MobileVLCKit for iOS, TVVLCKit for tvOS)
- **JellyfinAPI**: Auto-generated API client for Jellyfin server communication
- **CoreStore**: Data persistence and migration
- **Factory**: Dependency injection
- **Nuke/NukeUI**: Async image loading and caching
- **Pulse**: Network logging and debugging
- **Defaults**: UserDefaults wrapper with type safety
- **SwiftUIIntrospect**: Access underlying UIKit/AppKit views from SwiftUI

## PR Requirements

Before submitting PRs, ensure:
1. iOS and tvOS builds succeed
2. SwiftFormat linting passes (`swiftformat .`)
3. No development team credentials are committed
4. New strings are localized (non-experimental features)
5. PR has appropriate labels: `enhancement`, `bug`, `crash`, or `ignore-for-release`

## Development Notes

- Use `// MARK:` comments for code organization (helps Xcode minimap navigation)
- Known VLCKit limitation: Audio delay on playback start/unpause (may be fixed in VLCKit v4)
- Both iOS and tvOS targets may need updates when changing shared business logic
- Testing requires physical devices for full VLCKit playback validation
