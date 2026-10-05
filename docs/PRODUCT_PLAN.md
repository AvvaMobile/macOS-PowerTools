# macOS Power Tools Product Plan

## Product Goal

macOS Power Tools is a native macOS utility suite that collects small but useful Finder and system productivity tools into one application.

The main application is the control center. Every tool can be enabled or disabled independently.

The product should stay intentionally simple:
- Native macOS implementation
- Small independent tools
- Shared Finder integration
- Minimal permissions
- No unnecessary framework complexity
- No cloud dependency for core features
- No third-party runtime dependencies

## Platform

- macOS 14+
- Swift
- SwiftUI
- AppKit
- FinderSync
- CoreGraphics
- ServiceManagement
- Third-party dependencies: 0

## V1 Scope

### 1. Teleport

Teleport becomes a module inside macOS Power Tools.

Source project:
- AvvaMobile/macos-teleport

Do not rewrite the working Teleport behavior unnecessarily. Preserve the current implementation where practical, including:
- Automatic display detection
- Manual display selection
- Cursor mapping
- Accessibility permission handling
- Display configuration change handling
- Launch at Login
- Debug logging
- Current event-driven behavior

When Teleport is disabled, its mouse event monitoring must not remain active.

Teleport-specific settings remain available from its own section in the main application.

### 2. Finder: Copy Path

Available from the Power Tools Finder menu.

Behavior:
- File selected: copy the full path of the selected file.
- Folder selected: copy the full path of the selected folder.
- No selected item / Finder background context: copy the current Finder folder path.

Example:

`/Users/murat/Projects/AvvaMobile/macOS-PowerTools`

### 3. Finder: Open in Terminal

Behavior:
- File selected: open Terminal in the file's containing folder.
- Folder selected: open Terminal in that folder.
- Finder background context: open Terminal in the current Finder folder.

V1 supports Apple's Terminal.app only.

Possible future terminal providers:
- iTerm2
- Warp
- Ghostty

### 4. Finder: New Folder

Create a new folder in the current Finder location.

Desired behavior:
- Create the folder.
- Use a safe unique default name when necessary.
- Enter Finder rename mode if this can be implemented reliably using supported APIs.

The implementation should follow native Finder behavior as closely as possible.

### 5. Finder: New Text Document

Create an empty text document in the current Finder location.

V1 file type:
- .txt

Naming:
- untitled.txt
- untitled 2.txt
- untitled 3.txt
- etc.

If reliable through supported APIs, enter Finder rename mode after creation.

Possible future defaults:
- .md
- .rtf
- .json
- user-configured default extension

### 6. Finder: Show Full Path of Current Folder

Show the absolute path of the Finder window's current folder.

Example:

`/Users/murat/Documents/Projects/AvvaMobile/macOS-PowerTools`

Recommended UI:
- Small native popover or sheet
- Full path displayed clearly
- Copy Path action
- Open in Terminal action

This is intentionally separate from Copy Path:
- Copy Path performs an immediate clipboard action.
- Show Full Path displays the current location to the user.

## Finder Integration

Use one Finder Sync extension shared by Finder-related tools.

### Context Menu

Use a single submenu:

Power Tools >
- Copy Path
- Show Full Path
- Open in Terminal
- New Folder
- New Text Document

Only enabled tools should appear.

Do not flood Finder's root context menu with independent Power Tools entries.

### Finder Toolbar

Use one Power Tools toolbar item.

The toolbar item opens a Power Tools menu containing the enabled Finder actions.

This avoids trying to create a separate Finder toolbar extension item for every tool.

## Shared Finder Context

Create one shared Finder context abstraction for all Finder tools.

Suggested model:

`FinderContext`
- currentFolder
- selectedItems
- selectedFiles
- selectedFolders

Finder selection and target location should be resolved once and reused by tools.

Do not duplicate Finder path-resolution logic inside each feature.

## Tool Registry

Keep the tool model lightweight.

Initial tool IDs:
- teleport
- copy-path
- open-terminal
- new-folder
- new-text-document
- show-full-path

Minimum metadata:
- id
- name
- category
- enabled

The registry exists to:
- build the main app feature list
- build Finder menus
- persist enable/disable state
- keep future tools easy to add

Do not turn this into a plugin framework.

## Main Application

The main application is the management center.

Suggested navigation:
- Power Tools
- Finder
- Teleport
- Settings
- About

### Main Tool List

Each tool should show:
- Name
- Short description
- ON/OFF control
- Settings button only when the tool actually needs additional configuration

Example:

Copy Path
Copy the full path of the selected file or folder.
ON

Open in Terminal
Open the current Finder location in Terminal.
ON

## Shared Settings

Use an App Group so the main application and Finder extension share tool state.

Recommended shared responsibilities:
- enabled/disabled state
- lightweight tool preferences
- Finder extension menu configuration

Changing a tool state in the main application should be reflected by the Finder extension without requiring an application restart whenever practical.

## Permissions

Request permissions only when a feature needs them.

### Finder Tools

Finder tools should not require Accessibility permission merely because Teleport exists in the same product.

### Teleport

Teleport requires Accessibility permission for its current mouse event monitoring behavior.

The app should clearly show:
- permission required
- permission granted / not granted
- direct route to the correct System Settings location

No silent failure.

## Menu Bar

macOS Power Tools can expose one menu bar item.

Keep the menu minimal.

Suggested menu:

Power Tools

Teleport                    ON

Open Power Tools
Settings
Buy Me a Coffee
Quit

Do not put every Finder action into the menu bar.

## Buy Me a Coffee

Include a Buy Me a Coffee action in V1.

Locations:
- Settings / About
- Menu bar menu

Behavior:
- Open the Avva Mobile / project Buy Me a Coffee page in the user's default browser.

The actual support URL should be configurable in one central constant rather than duplicated across UI code.

## Launch at Login

Provide Launch at Login using ServiceManagement / SMAppService.

The setting belongs in the main application.

## Development Phases

### Phase 1: Foundation

- Create native macOS application shell
- Tool Registry
- Shared App Group preferences
- Finder Sync extension
- Main feature list with ON/OFF state
- Basic Settings / About
- Menu bar shell

### Phase 2: Basic Finder Tools

Implement and verify:
- Copy Path
- Show Full Path
- Open in Terminal

This phase validates the shared Finder context and extension architecture.

### Phase 3: Finder Creation Tools

Implement:
- New Folder
- New Text Document

Verify:
- duplicate-name handling
- permissions / writable folders
- Finder refresh behavior
- rename behavior where supported

### Phase 4: Teleport Integration

Move the existing Teleport functionality into macOS Power Tools as a module.

Verify the existing Teleport behavior remains intact:
- automatic mapping
- manual mapping
- display changes
- accessibility
- mouse transition behavior
- performance
- debug logging

### Phase 5: Distribution

- Release build
- Developer ID signing
- Hardened Runtime as appropriate
- Notarization
- DMG
- Installation test on a clean Mac
- Gatekeeper verification
- First GitHub Release

## V1 Non-Goals

Do not add these unless the V1 scope changes explicitly:
- General plugin architecture
- Plugin marketplace
- Cloud backend
- User accounts
- Analytics
- Telemetry
- Script marketplace
- Complex dependency-injection framework
- Custom keyboard shortcut system
- Mac App Store-specific redesign
- Large preferences framework
- Automatic updater framework

The first objective is to make the initial six tools reliable and fast.

## Future Tool Candidates

Potential later additions:
- Copy File Name
- Copy Folder Name
- Copy as file:// URL
- Copy Relative Path
- Open in VS Code
- Open in Cursor
- Open in Xcode
- Terminal provider selection
- Create Markdown File
- Create JSON File
- Show / Hide Hidden Files
- Copy File Size
- Copy File Hash
- Quick Image Resize
- Image Format Conversion
- Compress to ZIP
- Copy Folder Tree
- Create Symbolic Link

These are backlog candidates only and are not part of V1.

## Product Principle

macOS Power Tools should solve small macOS productivity gaps with small, reliable native tools.

Adding the seventh tool should be cheap because the shared Finder and settings infrastructure already exists, not because the application has become a general-purpose plugin platform.
