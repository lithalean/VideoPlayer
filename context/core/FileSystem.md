# VideoPlayer Filesystem Documentation

**Purpose**: Complete guide to filesystem browsing, security, and platform handling  
**Version**: 1.0  
**Component**: Core/FileSystem  
**Last Updated**: September 2025

## Overview

VideoPlayer v2.0 implements direct filesystem browsing, eliminating the need to copy files into an app library. This document details the three-tier filesystem implementation and platform-specific handling.

## Filesystem Architecture

### Three-Tier Implementation

```
┌──────────────────────────────────────┐
│ Layer 1: FileBrowser.swift           │ ← Media-specific, simplified
│ (Videos in Documents/VideoPlayer)    │
├──────────────────────────────────────┤
│ Layer 2: FileSystem.swift            │ ← General purpose browser
│ (Full filesystem operations)         │
├──────────────────────────────────────┤
│ Layer 3: FileSystemModel.swift       │ ← Platform abstractions
│ (iOS/tvOS/macOS differences)         │
└──────────────────────────────────────┘
```

## Component Details

### FileBrowser.swift (Media Layer)

**Purpose**: Simplified video browsing focused on Documents/VideoPlayer

```swift
struct FileBrowser: View {
    @StateObject private var fileManager = MediaFileManager()
    @Binding var currentMedia: URL?
    
    enum Destination {
        case movies  // Navigate to MoviesView
        case shows   // Navigate to ShowsView
        case file(URL)  // Play specific file
    }
}

class MediaFileManager: ObservableObject {
    @Published var rootItems: [MediaFileItem] = []
    
    private var documentsURL: URL {
        FileManager.default.urls(for: .documentDirectory, 
                                in: .userDomainMask).first!
    }
    
    private func setupMediaStructure() {
        // Auto-creates Movies/ and TV Shows/ folders
        let defaultFolders = ["Movies", "TV Shows"]
        for folder in defaultFolders {
            let folderURL = documentsURL.appendingPathComponent(folder)
            if !FileManager.default.fileExists(atPath: folderURL.path) {
                try? FileManager.default.createDirectory(
                    at: folderURL, 
                    withIntermediateDirectories: true
                )
            }
        }
    }
}
```

**Key Features**:
- Auto-creates media folder structure
- Filters to show only .m4v files and folders
- Recognizes Movies/TV Shows folders for navigation
- Simplified interface for media browsing

### FileSystem.swift (General Browser)

**Purpose**: Full-featured file browser with create/delete/rename operations

```swift
struct FileSystemView: View {
    @StateObject private var viewModel = FileSystemViewModel()
    @State private var showHiddenFiles = false
    @State private var sortOrder = SortOrder.nameAscending
    
    // Complete file operations
    var contextMenuItems: some View {
        Button("Open") { viewModel.openFile(item) }
        Button("Copy Path") { copyPath() }
        Button("Get Info") { showInfo() }
        Divider()
        Button("Delete", role: .destructive) { 
            viewModel.deleteItem(item) 
        }
    }
}
```

**Key Features**:
- Full CRUD operations
- Hidden files toggle
- Multiple sort options
- Context menus
- File info display

### FileSystemModel.swift (Platform Layer)

**Purpose**: Abstract platform differences, provide unified API

```swift
@MainActor
class FileSystemViewModel: ObservableObject {
    @Published var rootItems: [FileSystemItem] = []
    @Published var selectedItem: FileSystemItem?
    @Published var expandedFolders: Set<URL> = []
    
    private var rootLocations: [(name: String, url: URL)] {
        #if os(macOS)
        return [
            ("Home", homeURL),
            ("Desktop", desktopURL),
            ("Documents", documentsURL),
            ("Downloads", downloadsURL),
            ("Applications", applicationsURL)
        ]
        #elseif os(iOS)
        return [
            ("Documents", documentsURL),
            ("VideoPlayer", videoPlayerURL),
            ("iCloud Drive", iCloudURL)  // if available
        ]
        #else  // tvOS
        return [
            ("Documents", documentsURL),
            ("VideoPlayer", videoPlayerURL)
        ]
        #endif
    }
}
```

## iOS Security-Scoped Resources

### Document Picker Flow

```swift
// 1. User initiates folder selection
struct AdvancedDocumentPicker: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIDocumentPickerViewController {
        let picker = UIDocumentPickerViewController(
            forOpeningContentTypes: [.folder]
        )
        picker.delegate = context.coordinator
        return picker
    }
}

// 2. Handle selection with security-scoped access
func documentPicker(_ controller: UIDocumentPickerViewController, 
                   didPickDocumentsAt urls: [URL]) {
    guard let url = urls.first else { return }
    
    // Start accessing
    guard url.startAccessingSecurityScopedResource() else {
        print("Failed to access security-scoped resource")
        return
    }
    
    // Create bookmark for persistence
    saveBookmark(for: url)
    
    // Use the URL
    processFolder(url)
    
    // Stop accessing
    url.stopAccessingSecurityScopedResource()
}
```

### Bookmark Persistence

```swift
// Save bookmark
private func saveBookmark(for url: URL) {
    do {
        let bookmarkData = try url.bookmarkData(
            options: .minimalBookmark,
            includingResourceValuesForKeys: nil,
            relativeTo: nil
        )
        
        UserDefaults.standard.set(
            bookmarkData, 
            forKey: "bookmark_\(url.lastPathComponent)"
        )
    } catch {
        print("Failed to create bookmark: \(error)")
    }
}

// Restore bookmark
private func restoreBookmark(key: String) -> URL? {
    guard let bookmarkData = UserDefaults.standard.data(forKey: key) else { 
        return nil 
    }
    
    do {
        var isStale = false
        let url = try URL(
            resolvingBookmarkData: bookmarkData,
            options: [],
            relativeTo: nil,
            bookmarkDataIsStale: &isStale
        )
        
        if !isStale && url.startAccessingSecurityScopedResource() {
            // Remember to stop accessing when done
            return url
        }
    } catch {
        print("Failed to resolve bookmark: \(error)")
    }
    
    return nil
}
```

### iOS File Access Checklist

✅ **Required Info.plist Keys**:
```xml
<key>UIFileSharingEnabled</key>
<true/>
<key>LSSupportsOpeningDocumentsInPlace</key>
<true/>
<key>UISupportsDocumentBrowser</key>
<true/>
```

✅ **Required Entitlements**:
```xml
<key>com.apple.security.app-sandbox</key>
<true/>
<key>com.apple.security.files.user-selected.read-only</key>
<true/>
```

## Platform-Specific Implementations

### iOS

**Available Locations**:
- App's Documents directory (visible in Files app)
- Security-scoped external folders (via picker)
- iCloud Drive (if configured)

**Limitations**:
- No access without user selection
- Must use bookmarks for persistence
- Security-scoped resources required

**Implementation**:
```swift
#if os(iOS)
private func enableDocumentBrowserAccess() {
    // Files app visibility - handled by Info.plist keys
    // Create default structure
    createDefaultMediaStructure(in: documentsURL)
}

private func createDefaultMediaStructure(in parentURL: URL) {
    let defaultFolders = ["Movies", "TV Shows"]
    for folderName in defaultFolders {
        let folderURL = parentURL.appendingPathComponent(folderName)
        createDirectoryIfNeeded(at: folderURL)
    }
}
#endif
```

### tvOS

**Available Locations**:
- App sandbox only
- No external storage access
- No document picker

**Implementation**:
```swift
#if os(tvOS)
// Very limited - sandbox only
if let documents = FileManager.default.urls(for: .documentDirectory, 
                                           in: .userDomainMask).first {
    let videoPlayerFolder = documents.appendingPathComponent("VideoPlayer")
    createDirectoryIfNeeded(at: videoPlayerFolder)
    
    // Auto-create media folders
    for folder in ["Movies", "TV Shows"] {
        let url = videoPlayerFolder.appendingPathComponent(folder)
        createDirectoryIfNeeded(at: url)
    }
}
#endif
```

### macOS

**Available Locations**:
- Full filesystem (with user permission)
- Home, Desktop, Documents, Downloads
- External volumes
- Network shares

**Special Features**:
```swift
#if os(macOS)
func revealInFinder(_ item: FileSystemItem) {
    NSWorkspace.shared.selectFile(
        item.url.path, 
        inFileViewerRootedAtPath: item.url.deletingLastPathComponent().path
    )
}

func openInTerminal(_ item: FileSystemItem) {
    let path = item.isDirectory ? item.url.path : 
               item.url.deletingLastPathComponent().path
    let script = """
    tell application "Terminal"
        activate
        do script "cd \(path.replacingOccurrences(of: " ", with: "\\ "))"
    end tell
    """
    
    if let appleScript = NSAppleScript(source: script) {
        appleScript.executeAndReturnError(nil)
    }
}
#endif
```

## Directory Structure

### Standard Media Organization
```
Documents/
└── VideoPlayer/
    ├── Movies/
    │   ├── Action/
    │   │   ├── Die Hard.m4v
    │   │   └── Mad Max.m4v
    │   └── Comedy/
    │       ├── Airplane.m4v
    │       └── Ghostbusters.m4v
    └── TV Shows/
        ├── Breaking Bad/
        │   ├── S01E01.m4v
        │   └── S01E02.m4v
        └── The Office/
            ├── S01E01.m4v
            └── S01E02.m4v
```

### File Type Detection

```swift
enum FileType {
    static func fromDirectoryName(_ name: String) -> FileType {
        let lowercased = name.lowercased()
        if lowercased.contains("movies") { return .moviesFolder }
        if lowercased.contains("tv shows") { return .tvShowsFolder }
        return .folder
    }
    
    static func fromUTI(_ uti: String, fileName: String) -> FileType {
        let ext = (fileName as NSString).pathExtension.lowercased()
        
        // M4V detection
        if ext == "m4v" { return .videoFile }
        
        // Check UTType
        let type = UTType(uti)
        if type?.conforms(to: .mpeg4Movie) ?? false { 
            return .videoFile 
        }
        
        return .file
    }
}
```

## Performance Considerations

### Directory Enumeration

**Problem**: Large directories block UI

**Solution**: Async enumeration
```swift
func loadItems(at url: URL) async -> [FileSystemItem] {
    await Task.detached(priority: .userInitiated) {
        let items = try? FileManager.default.contentsOfDirectory(
            at: url,
            includingPropertiesForKeys: [
                .isDirectoryKey,
                .fileSizeKey,
                .contentModificationDateKey
            ],
            options: [.skipsHiddenFiles]
        )
        
        return items?.map { FileSystemItem(url: $0) } ?? []
    }.value
}
```

### File Watching

**Current**: Manual refresh only

**Future Enhancement**:
```swift
// FSEvents for macOS
// NSFilePresenter for iOS
// DirectoryMonitor for cross-platform

class DirectoryWatcher {
    private var source: DispatchSourceFileSystemObject?
    
    func watch(url: URL, onChange: @escaping () -> Void) {
        let fd = open(url.path, O_EVTONLY)
        guard fd >= 0 else { return }
        
        source = DispatchSource.makeFileSystemObjectSource(
            fileDescriptor: fd,
            eventMask: [.write, .delete, .rename],
            queue: .main
        )
        
        source?.setEventHandler(handler: onChange)
        source?.resume()
    }
}
```

## Error Handling

### Common Filesystem Errors

```swift
enum FileSystemError: LocalizedError {
    case accessDenied(URL)
    case fileNotFound(URL)
    case diskFull
    case networkUnavailable
    
    var errorDescription: String? {
        switch self {
        case .accessDenied(let url):
            return "Cannot access \(url.lastPathComponent). Permission denied."
        case .fileNotFound(let url):
            return "File not found: \(url.lastPathComponent)"
        case .diskFull:
            return "Not enough disk space"
        case .networkUnavailable:
            return "Network drive unavailable"
        }
    }
}
```

### Error Recovery

```swift
func handleFileError(_ error: Error, for url: URL) {
    if (error as NSError).code == NSFileReadNoPermissionError {
        // Request permission
        #if os(iOS)
        showDocumentPicker()
        #else
        showPermissionAlert()
        #endif
    } else if (error as NSError).code == NSFileReadNoSuchFileError {
        // Remove from recent/bookmarks
        removeBookmark(for: url)
    }
}
```

## Best Practices

### ✅ DO:
- Always use security-scoped resources on iOS
- Create bookmarks for persistent access
- Handle file coordination for concurrent access
- Check file existence before operations
- Use async operations for large directories
- Provide user feedback during long operations

### ❌ DON'T:
- Assume filesystem access on iOS
- Forget to stop accessing security-scoped resources
- Block the main thread with file operations
- Ignore platform differences
- Cache file URLs indefinitely (they can become stale)

## Testing Filesystem Code

### Unit Testing
```swift
class FileSystemTests: XCTestCase {
    func testBookmarkPersistence() {
        let url = URL(fileURLWithPath: "/tmp/test")
        saveBookmark(for: url)
        
        let restored = restoreBookmark(key: "bookmark_test")
        XCTAssertEqual(restored?.path, url.path)
    }
}
```

### UI Testing
```swift
func testFileBrowserNavigation() {
    let app = XCUIApplication()
    
    // Navigate to Movies
    app.buttons["Movies"].tap()
    XCTAssert(app.navigationBars["Movies"].exists)
    
    // Select a video
    app.collectionViews.cells.firstMatch.tap()
    XCTAssert(app.otherElements["VideoPlayer"].exists)
}
```

## Troubleshooting

### iOS: "Cannot access file"
- Check security-scoped resource is active
- Verify bookmark isn't stale
- Ensure Info.plist keys are present

### tvOS: "No files visible"
- tvOS can only access sandbox
- Use Xcode to add test files to app bundle

### macOS: "Permission denied"
- Check app sandbox entitlements
- Request Full Disk Access if needed
- Verify file permissions

## Future Enhancements

1. **File Watching**: Automatic refresh on changes
2. **Thumbnail Cache**: Preview generation for videos
3. **Search**: Full-text search across filenames
4. **Sorting**: Multiple sort options
5. **Filtering**: Show only unwatched, by year, etc.
6. **Network Shares**: SMB/AFP support
7. **Cloud Storage**: Dropbox, Google Drive integration