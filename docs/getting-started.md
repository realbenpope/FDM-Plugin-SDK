# Getting Started with FDM Plugins

This guide will walk you through creating your first plugin for Free Download Manager.

## Prerequisites

- Free Download Manager 6 or later installed
- Basic knowledge of JavaScript
- A text editor

## Creating Your First Plugin

Let's create a simple plugin that downloads audio files from a hypothetical website.

### Step 1: Create the Plugin Directory

Create a new directory for your plugin. Name it something descriptive, like `fdm-example-audio`:

```
fdm-example-audio/
```

### Step 2: Create the Manifest File

Create a file named `manifest.json` in your plugin directory with the following content:

```json
{
  "uuid": "fdm-example-audio",
  "author": "Your Name",
  "name": "Example Audio Downloader",
  "description": "Provides support for downloading audio files from example.com.",
  "version": "1.0.0",
  "icon": "icon.png",
  "mediaParser": true,
  "scripts": [
    "parser.js"
  ],
  "minApiVersion": 1,
  "minFeaturesLevel": 1
}
```

### Step 3: Add an Icon

Create or obtain a PNG or SVG icon for your plugin and save it as `icon.png` in your plugin directory.

### Step 4: Create the Parser Script

Create a file named `parser.js` in your plugin directory with the following content:

```javascript
var msParser = (function() {
    function ExampleAudioParser() {
    }

    ExampleAudioParser.prototype = {
        // Check if this plugin supports the given URL
        isSupportedSource: function(url) {
            // Match URLs like "https://example.com/audio/12345"
            return /^https?:\/\/(www\.)?example\.com\/audio\/\d+/.test(url);
        },

        // Define the priority of this plugin (higher numbers = higher priority)
        supportedSourceCheckPriority: function() {
            return 65535;
        },

        // Parse the webpage to extract audio information
        parse: function(obj) {
            return downloadUrlAsUtf8Text(obj.url, obj.cookie)
                .then(this.parseContent);
        },

        // Process the downloaded HTML and extract audio information
        parseContent: function(obj) {
            return new Promise(function(resolve, reject) {
                try {
                    // In a real plugin, you would parse the HTML to find these values
                    // For this example, we'll use hardcoded values
                    
                    // Extract the title (in a real plugin, use regex to extract from HTML)
                    var title = "Example Audio Track";
                    
                    // Extract the audio URL (in a real plugin, use regex to extract from HTML)
                    var audioUrl = "https://example.com/download/audio12345.mp3";
                    
                    // Create a result object with the download information
                    let result = {
                        id: "12345",                 // Optional identifier
                        title: title,                // Required: Title to display to the user
                        webpage_url: obj.url,        // Original webpage URL
                        upload_date: "2023-03-15",   // Date in YYYY-MM-DD format
                        formats: [{                  // Required: Available download formats
                            url: audioUrl,           // Required: URL to download the file
                            ext: "mp3",              // File extension
                            protocol: "https",       // Required: Protocol (http/https)
                            audio_ext: "mp3"         // Required for audio: Audio extension
                        }]
                    };
                    
                    resolve(result);
                } catch (e) {
                    reject({error: e.message, isParseError: true});
                }
            });
        },

        // Define minimum interval between parse requests (rate limiting)
        minIntevalBetweenQueryInfoDownloads: function() {
            return 300; // 300ms
        }
    };

    return new ExampleAudioParser();
}());
```

### Step 5: Package the Plugin

Create a ZIP file containing all the files in your plugin directory:
- `manifest.json`
- `icon.png`
- `parser.js`

### Step 6: Install the Plugin in FDM

1. Open Free Download Manager
2. Go to Tools > Options
3. Select the "Plugins" tab
4. Click "Install Plugin" and select your ZIP file
5. Restart FDM if prompted

### Step 7: Test the Plugin

1. Copy a URL that matches your plugin's pattern (e.g., `https://example.com/audio/12345`)
2. Paste it into FDM's "Add Download" dialog
3. The plugin should recognize the URL and extract the audio information
4. FDM will display the extracted information and allow you to start the download

## Next Steps

Now that you've created a basic plugin, you can:

1. **Enhance your parser**: Implement real HTML parsing to extract actual information
2. **Add playlist support**: Create a `msBatchVideoParser` to handle collections of media
3. **Add subtitles support**: Extract and provide subtitles if available
4. **Implement Python integration**: For more complex parsing needs

## Real-World Example

Here's a more realistic example of parsing HTML content:

```javascript
parseContent: function(obj) {
    return new Promise(function(resolve, reject) {
        try {
            // Extract title using regular expressions
            var titleRegex = /<h1 class="audio-title">(.*?)<\/h1>/;
            var titleMatch = obj.body.match(titleRegex);
            var title = titleMatch ? titleMatch[1] : "Unknown Title";
            
            // Extract audio URL
            var audioRegex = /<a class="download-link" href="(.*?\.mp3)"/;
            var audioMatch = obj.body.match(audioRegex);
            
            if (!audioMatch) {
                throw new Error("Could not find audio URL on page");
            }
            
            var audioUrl = audioMatch[1];
            // Resolve relative URLs
            if (audioUrl.startsWith("/")) {
                var baseUrl = new URL(obj.url);
                audioUrl = baseUrl.origin + audioUrl;
            }
            
            // Extract date
            var dateRegex = /<span class="upload-date">(\d{2})\/(\d{2})\/(\d{4})<\/span>/;
            var dateMatch = obj.body.match(dateRegex);
            var uploadDate = dateMatch ? 
                             dateMatch[3] + "-" + dateMatch[2] + "-" + dateMatch[1] : 
                             null;
            
            let result = {
                title: title,
                webpage_url: obj.url,
                upload_date: uploadDate,
                formats: [{
                    url: audioUrl,
                    ext: "mp3",
                    protocol: "https",
                    audio_ext: "mp3"
                }]
            };
            
            resolve(result);
        } catch (e) {
            reject({error: e.message, isParseError: true});
        }
    });
}
```

## Troubleshooting

If your plugin isn't working:

1. Check the FDM logs for error messages
2. Verify that your URL pattern correctly matches the test URL
3. Make sure your parsing logic correctly extracts information from the page
4. Test your regular expressions using online tools like regex101.com

For more detailed information, see the [Plugin Architecture](./architecture.md) and [API Reference](./api-reference.md) documentation.
