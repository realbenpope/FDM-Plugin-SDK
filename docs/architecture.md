# FDM Plugin Architecture

This document describes the architecture of the Free Download Manager plugin system (API Version 4).

## Table of Contents

- [Overview](#overview)
- [Plugin Structure](#plugin-structure)
- [Plugin Lifecycle](#plugin-lifecycle)
- [Media Parser Plugins](#media-parser-plugins)
- [Playlist Parser Plugins](#playlist-parser-plugins)
- [JavaScript Environment](#javascript-environment)

## Overview

The FDM plugin system is designed to extend Free Download Manager's capability to download content from websites. Plugins can:

1. Enable downloading resources that cannot be downloaded through standard browser methods
2. Simplify the user experience for obtaining content from supported websites

The current API focuses primarily on extending FDM's media downloading capabilities.

## Plugin Structure

A plugin consists of three main components:

1. **Manifest File**: A JSON file (`manifest.json`) that describes the plugin's metadata and requirements
2. **Icon File**: An image representing your plugin (SVG, PNG, or other supported formats)
3. **JavaScript Files**: One or more JS files containing the plugin's business logic

### Directory Structure

A typical plugin has the following structure:

```
my-plugin/
├── manifest.json
├── icon.png
├── parser.js
└── batchparser.js (optional)
```

## Plugin Lifecycle

1. **Installation**: User installs the plugin through FDM's plugin manager
2. **Registration**: FDM reads the manifest and registers the plugin's capabilities
3. **URL Detection**: When a user attempts to download from a URL, FDM checks if any plugins support it
4. **Parser Selection**: The plugin with the highest priority that supports the URL is selected
5. **Parsing**: The selected plugin's parse function is called to extract download information
6. **Download**: FDM uses the parsed information to download the content

## Media Parser Plugins

A media parser plugin must define a global JavaScript object named `msParser` with the following methods:

| Method | Purpose |
|--------|---------|
| `isSupportedSource(url)` | Returns `true` if the plugin can handle this URL |
| `supportedSourceCheckPriority()` | Returns a number indicating this plugin's priority |
| `isPossiblySupportedSource(obj)` | Optional fallback check for URL support |
| `parse(obj)` | Parses the URL and returns download information |
| `minIntevalBetweenQueryInfoDownloads()` | Specifies rate limiting between parse calls |

### Example Media Parser

```javascript
var msParser = (function() {
    function MsParser() {}
    
    MsParser.prototype = {
        isSupportedSource: function(url) {
            return /^https?:\/\/(www.)?example\.com\/media\/\d+$/.test(url);
        },
        
        supportedSourceCheckPriority: function() {
            return 65535; // High priority
        },
        
        parse: function(obj) {
            return downloadUrlAsUtf8Text(obj.url, obj.cookie)
            .then(this.parseContent);
        },
        
        parseContent: function(obj) {
            return new Promise(function(resolve, reject) {
                try {
                    // Extract media information from the page
                    var title = "Example Media";
                    var mediaUrl = "https://example.com/download/123.mp4";
                    
                    // Return the media information
                    resolve({
                        title: title,
                        webpage_url: obj.url,
                        formats: [{
                            url: mediaUrl,
                            ext: "mp4",
                            protocol: "https",
                            video_ext: "mp4"
                        }]
                    });
                } catch(e) {
                    reject({error: e.message, isParseError: true});
                }
            });
        },
        
        minIntevalBetweenQueryInfoDownloads: function() {
            return 300; // 300ms between queries
        }
    };
    
    return new MsParser();
}());
```

## Playlist Parser Plugins

A playlist parser plugin must define a global JavaScript object named `msBatchVideoParser` with the same interface as a media parser but focused on returning a collection of media items.

### Example Playlist Parser

```javascript
var msBatchVideoParser = (function() {
    function MsBatchVideoParser() {}
    
    MsBatchVideoParser.prototype = {
        isSupportedSource: function(url) {
            return /^https?:\/\/(www.)?example\.com\/playlist\/\d+$/.test(url);
        },
        
        parse: function(obj) {
            return downloadUrlAsUtf8Text(obj.url, obj.cookie)
            .then(this.parseContent);
        },
        
        parseContent: function(obj) {
            return new Promise(function(resolve, reject) {
                try {
                    // Extract playlist items
                    var playlistTitle = "Example Playlist";
                    var entries = [
                        {
                            _type: "url",
                            url: "https://example.com/media/1",
                            title: "Item 1"
                        },
                        {
                            _type: "url",
                            url: "https://example.com/media/2",
                            title: "Item 2"
                        }
                    ];
                    
                    // Return the playlist information
                    resolve({
                        _type: "playlist",
                        title: playlistTitle,
                        webpage_url: obj.url,
                        entries: entries
                    });
                } catch(e) {
                    reject({error: e.message, isParseError: true});
                }
            });
        },
        
        // Other methods similar to msParser
    };
    
    return new MsBatchVideoParser();
}());
```

## JavaScript Environment

The plugin system provides a JavaScript environment with the following features:

- Console API (`console.log`, `console.error`, etc.)
- Timing functions (`setTimeout`, `clearTimeout`)
- HTTP request capability (`downloadUrlAsUtf8Text`)
- Python script execution (with permission)
- Proxy and system information queries

Unlike browser extensions, FDM plugins run in a more limited JavaScript environment. Notable limitations:

- No DOM access
- No XMLHttpRequest (use `downloadUrlAsUtf8Text` instead)
- Limited global objects and browser APIs

For details on available APIs, see the [API Reference](./api-reference.md).
