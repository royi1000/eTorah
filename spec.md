# Jewish Textbook App Specification

## Overview

A cross-platform application for Jewish religious texts that combines native app infrastructure with web-based UI. The app will store approximately 1GB of textual content locally and provide full offline functionality across iOS, Android, and web platforms.

## Core Architecture

### Application Structure
- **Native Shell**: iOS (Swift) and Android (Kotlin) native applications
- **Web UI**: HTML, CSS, and JavaScript rendered in WebView
- **Content Storage**: Local filesystem with optimized organization
- **Bridge Layer**: Communication between native code and WebView

## Technical Specifications

### 1. Content Management

#### Storage Strategy
- Hierarchical filesystem organization for texts
- Structured JSON format for primary content
- Separate indices for search, navigation, and cross-references
- Compressed storage with on-demand decompression

#### Content Structure
```
/app-content/
  /texts/
    /torah/
      genesis.json
      exodus.json
      ...
    /talmud/
      /bavli/
        berakhot.json
        ...
      /yerushalmi/
        ...
    /commentaries/
      /rashi/
      /ramban/
      ...
  /indices/
    search-index.db
    cross-references.json
    navigation.json
  /user-data/
    bookmarks.json
    notes.json
    settings.json
```

### 2. Native Implementation

#### iOS (Swift)
- WebKit WKWebView implementation
- File system access via native bridge
- Content management in app bundle/documents directory
- ScriptMessageHandler for WebView communication

#### Android (Kotlin)
- WebView with JavaScript interface
- Asset management for content files
- Content provider for file access
- JavascriptInterface for bidirectional communication

### 3. Web UI Implementation

#### Technology Stack
- HTML5 for structure
- CSS3 with responsive design
- JavaScript (ES6+) for interactions
- Optional framework (React/Vue) for component management

#### UI Components
- Text display with proper Hebrew/English rendering
- Navigation system (chapters, verses, commentaries)
- Search interface
- Bookmarking and annotation tools
- Settings panel

### 4. Bridge Implementation

#### Native to WebView
- Content loading functions
- Search execution
- Settings management
- User data persistence

#### JavaScript API
```javascript
// Example bridge API
window.JewishTextApp = {
  // Content access
  getPassage: (book, chapter, verse, commentaries) => {...},
  getChapter: (book, chapter) => {...},
  search: (query, filters) => {...},
  
  // User data
  saveBookmark: (reference, notes) => {...},
  getSavedBookmarks: () => {...},
  
  // Settings
  updateSettings: (settings) => {...}
};
```

## User Experience Features

### Core Features
1. **Text Browser**: Navigate and read texts with original language and translations
2. **Commentary View**: Display commentaries alongside primary texts
3. **Advanced Search**: Search across all texts with filters
4. **Bookmarks & Notes**: Save passages and add personal notes
5. **Offline Access**: Full functionality without internet connection

### Enhanced Features
1. **Cross-References**: Navigate between related passages
2. **Custom Collections**: Create collections of passages by theme
3. **Learning Tools**: Vocabulary assistance, pronunciation guides
4. **Sharing**: Export passages for sharing
5. **Customization**: Font size, language preference, layout options

## Performance Considerations

### Content Loading Optimization
- Lazy loading of content as needed
- Pre-loading of adjacent chapters
- Content compression (GZIP) with on-demand decompression
- Caching strategy for frequently accessed content

### Memory Management
- Limit active content in memory
- Unload unused resources
- Optimize images and media

## Development Phases

### Phase 1: Core Infrastructure
- Native shell implementation
- WebView setup with basic bridge
- Content storage system
- Basic text navigation

### Phase 2: Content & Features
- Complete text import and formatting
- Search functionality
- User data storage (bookmarks, notes)
- UI refinement

### Phase 3: Enhancement & Optimization
- Performance optimization
- Advanced features implementation
- User testing and feedback
- Deployment preparation

## Technical Challenges & Solutions

### Large Content Management
- **Challenge**: Efficiently handling 1GB of text
- **Solution**: Chunked content organization, compression, lazy loading

### Hebrew Text Rendering
- **Challenge**: Proper RTL text rendering with vowels/cantillation
- **Solution**: Use appropriate web fonts, CSS RTL support, dedicated Hebrew text libraries

### Cross-Platform Consistency
- **Challenge**: Maintaining consistent experience across platforms
- **Solution**: Shared WebView code, platform-specific optimizations where necessary

### Search Performance
- **Challenge**: Fast search across large text corpus
- **Solution**: Pre-built search indices, optimized query handling, native search implementation

## Appendix: Sample Implementation Code

### iOS Bridge Example
```swift
class ContentBridge: NSObject, WKScriptMessageHandler {
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        if message.name == "getTextPassage" {
            guard let params = message.body as? [String: Any],
                  let book = params["book"] as? String,
                  let chapter = params["chapter"] as? Int else {
                return
            }
            
            // Load and return content
            let content = loadContent(book: book, chapter: chapter)
            webView.evaluateJavaScript("window.JewishTextApp.receiveContent(\(content))")
        }
    }
}
```

### Android Bridge Example
```kotlin
class ContentBridge(private val context: Context) {
    @JavascriptInterface
    fun getTextPassage(book: String, chapter: Int): String {
        val bookPath = "$contentDir/texts/${getBookCategory(book)}/${book.toLowerCase()}.json"
        return try {
            context.assets.open(bookPath).bufferedReader().use { it.readText() }
        } catch (e: Exception) {
            "{\"error\": \"Failed to load content\"}"
        }
    }
}
```

### WebView JavaScript Example
```javascript
// Initialize bridge on load
document.addEventListener('DOMContentLoaded', () => {
  // Request initial content
  if (window.webkit && window.webkit.messageHandlers.contentBridge) {
    // iOS
    window.webkit.messageHandlers.contentBridge.postMessage({
      method: 'getTextPassage',
      params: { book: 'Genesis', chapter: 1 }
    });
  } else if (window.contentBridge) {
    // Android
    const content = window.contentBridge.getTextPassage('Genesis', 1);
    processContent(JSON.parse(content));
  }
});

// Handle received content
window.JewishTextApp.receiveContent = function(content) {
  renderTextPassage(content);
};
```
