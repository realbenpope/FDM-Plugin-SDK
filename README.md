# FDM Plugin SDK
Create custom plugins for Free Download Manager 6

## Overview
The FDM Plugin SDK allows developers to extend Free Download Manager's capabilities through custom JavaScript plugins. This SDK gives you everything you need to create plugins that enable FDM to download content from websites that would otherwise be difficult to access.

## Key Features

- JavaScript-based Plugin System - Create plugins using standard JavaScript
- Media Content Support - Download single media files or entire playlists
- Python Integration - Launch Python scripts from your plugins for advanced functionality
- Simple Update Mechanism - Automatically deliver updates to your plugins

## Getting Started

Follow our [Quick Start Guide](./docs/getting-started.md) to create your first plugin in minutes.

```javascript
// A simple plugin that downloads audio from Example.com
var exampleParser = (function() {
    function ExampleParser() {}
    
    ExampleParser.prototype = {
        isSupportedSource: function(url) {
            return /^https?:\/\/(www\.)?example\.com\/audio\//.test(url);
        },
        
        parse: function(obj) {
            return downloadUrlAsUtf8Text(obj.url, obj.cookie)
            .then(this.parseContent);
        },
        
        // Additional implementation...
    };
    
    return new ExampleParser();
}());
```

## Documentation

- [Getting Started](./docs/getting-started.md)
- [Plugin Architecture](./docs/architecture.md)
- [API Reference](./docs/api-reference.md)
- [Examples](./examples/)
- [Manifest File Format](./docs/manifest-format.md)

## Requirements

Free Download Manager 6.x or higher
Basic JavaScript knowledge
Python 3.x (optional, for advanced functionality)

## Contributing
We welcome contributions to improve the SDK documentation. Please feel free to submit issues or pull requests.
