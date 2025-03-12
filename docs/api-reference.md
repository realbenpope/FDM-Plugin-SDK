# API Reference

This document provides a comprehensive reference for the JavaScript API available to FDM plugins.

## Table of Contents

- [Media Parser Interface](#media-parser-interface)
- [Playlist Parser Interface](#playlist-parser-interface)
- [Parse Result Format](#parse-result-format)
- [Network Functions](#network-functions)
- [Python Integration](#python-integration)
- [System Information](#system-information)
- [Utilities](#utilities)
- [Console API](#console-api)

## Media Parser Interface

A media parser must define a global object named `msParser` with the following methods:

### isSupportedSource(url)

Determines if the plugin can handle a specific URL.

**Parameters:**
- `url` (String): The URL to check

**Returns:**
- `Boolean`: `true` if the plugin supports the URL, `false` otherwise

**Example:**
```javascript
isSupportedSource: function(url) {
    return /^https?:\/\/(www\.)?example\.com\/videos\//.test(url);
}
```

### supportedSourceCheckPriority()

Defines the priority of this plugin compared to others that might handle the same URL.

**Returns:**
- `Number`: An integer from 0 to 2³¹-1 (0x7FFFFFFF). Higher values mean higher priority.

**Example:**
```javascript
supportedSourceCheckPriority: function() {
    return 65535; // High priority
}
```

### isPossiblySupportedSource(obj)

Optional fallback method to determine if a URL might be supported when `isSupportedSource` returns `false` for all plugins.

**Parameters:**
- `obj` (Object): Contains:
  - `url` (String): URL entered by the user
  - `contentType` (String): Content type as reported by the server
  - `resourceSize` (Number): Content length as reported by the server

**Returns:**
- `Boolean`: `true` if the plugin might be able to handle this URL

**Example:**
```javascript
isPossiblySupportedSource: function(obj) {
    return obj.contentType === "text/html" && 
           /example\.com/.test(obj.url);
}
```

### parse(obj)

The main parsing function that extracts download information from a URL.

**Parameters:**
- `obj` (Object): Contains:
  - `url` (String): URL entered by the user
  - `cookies` (Array): Array of cookie objects (since API v3)
  - `browser` (String): Source web browser name (can be empty, since API v3)
  - `userAgent` (String): Source browser's user agent (can be empty, since API v3)
  - `requestId` (Number): Identifier for this request
  - `interactive` (Boolean): Whether this request is being performed in the foreground

**Returns:**
- `Promise`: A promise that resolves to a parse result object or rejects with an error

**Example:**
```javascript
parse: function(obj) {
    return downloadUrlAsUtf8Text(obj.url, obj.cookie)
    .then(this.parseContent);
}
```

### minIntevalBetweenQueryInfoDownloads()

Defines the minimum interval between parse requests to prevent overloading servers.

**Returns:**
- `Number`: Minimum interval in milliseconds (0 for no limit)

**Example:**
```javascript
minIntevalBetweenQueryInfoDownloads: function() {
    return 300; // 300ms between requests
}
```

### overrideUrlPolicy(url)

Optional method to allow downloading from URLs that might be restricted in FDM.

**Parameters:**
- `url` (String): The URL to check

**Returns:**
- `Boolean`: `true` to allow downloading from this URL despite restrictions

**Example:**
```javascript
overrideUrlPolicy: function(url) {
    return /^https?:\/\/trusted-source\.example\.com\//.test(url);
}
```

## Playlist Parser Interface

A playlist parser must define a global object named `msBatchVideoParser` with the same interface as the media parser, but returning playlist data.

## Parse Result Format

### Single Media Result

A JavaScript object with the following properties:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | String | No | Identifier for the media |
| `title` | String | Yes | Title to display to the user |
| `webpage_url` | String | No | Original webpage URL |
| `upload_date` | String | No | Date in ISO 8601 or RFC 2822 format |
| `duration` | Number | No | Duration in seconds |
| `formats` | Array | Yes | Available download formats |
| `subtitles` | Object | No | Available subtitles |
| `thumbnails` | Array | No | Available thumbnails |

#### Formats

Each format object contains:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `url` | String | Yes | URL to the media file |
| `manifestUrl` | String | No | HLS manifest URL for "m3u8_native" protocol |
| `filesize` | Number | No | Size of the file in bytes |
| `protocol` | String | Yes | Must be "http", "https", or "m3u8_native" |
| `ext` | String | No | File extension (e.g., "mp4", "mp3") |
| `video_ext` | String | For videos | Video file extension |
| `audio_ext` | String | For audio | Audio file extension |
| `vcodec` | String | No | Video codec name, "unknown", or "none" |
| `acodec` | String | No | Audio codec name, "unknown", or "none" |
| `container` | String | For DASH | Format like "mp4_dash", "webm_dash" |
| `fps` | Number | No | Frames per second for video |
| `height` | Number | No | Video height |
| `width` | Number | No | Video width |
| `quality` | Number | No | Quality indicator (higher is better) |
| `tbr` | Number | No | Total bitrate (audio + video) |
| `abr` | Number | No | Audio bitrate |
| `preference` | Number | No | Format preference (higher is better) |
| `languagePreference` | Number | No | Language preference (higher is better) |
| `httpHeaders` | Object | No | Custom HTTP headers |

#### Subtitles

An object where keys are language codes and values are arrays of subtitle objects:

```javascript
{
  "en": [
    {
      "name": "English",
      "url": "https://example.com/subtitles/en.vtt",
      "ext": "vtt"
    }
  ],
  "fr": [
    {
      "name": "French",
      "url": "https://example.com/subtitles/fr.vtt",
      "ext": "vtt"
    }
  ]
}
```

#### Thumbnails

An array of thumbnail objects:

```javascript
[
  {
    "url": "https://example.com/thumb/small.jpg",
    "height": 90,
    "width": 120,
    "preference": 10
  },
  {
    "url": "https://example.com/thumb/large.jpg",
    "height": 720,
    "width": 1280,
    "preference": 20
  }
]
```

### Playlist Result

A JavaScript object with the following properties:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `_type` | String | Yes | Must be "playlist" |
| `id` | String | No | Identifier for the playlist |
| `title` | String | Yes | Playlist title |
| `webpage_url` | String | No | Original webpage URL |
| `thumbnails` | Array | No | Playlist thumbnails |
| `entries` | Array | Yes | Media items in the playlist |

#### Entries

Each entry object contains:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `_type` | String | Yes | Must be "url" |
| `url` | String | Yes | URL to the media item |
| `title` | String | Yes | Media item title |
| `duration` | Number | No | Duration in seconds |

## Network Functions

### downloadUrlAsUtf8Text(url, cookies, headers, postData)

Downloads a web page or file as UTF-8 text.

**Parameters:**
- `url` (String): The URL to download
- `cookies` (String): Optional cookies string delimited by '\n' (e.g., "name1=value1\nname2=value2")
- `headers` (Array): Optional array of header objects with `key` and `value` properties
- `postData` (String|ArrayBuffer): Optional data to send in a POST request

**Returns:**
- `Promise`: Resolves to an object with:
  - `url` (String): The requested URL
  - `body` (String): The downloaded content
  - `cookies` (String): Cookies returned by the server

**Example:**
```javascript
downloadUrlAsUtf8Text("https://example.com/page", "session=abc123")
  .then(function(result) {
      console.log("Downloaded content:", result.body.substring(0, 100));
  })
  .catch(function(error) {
      console.error("Download failed:", error.error);
  });
```

## Python Integration

### launchPythonScript(requestId, interactive, script, args)

Executes a Python script with the specified arguments. Requires the "launchPython" permission.

**Parameters:**
- `requestId` (Number): Value passed to parse function, or 0
- `interactive` (Boolean): Whether to execute immediately, ignoring queued processes
- `script` (String): Relative path to the Python script
- `args` (Array): Command-line arguments to pass to the script

**Returns:**
- `Promise`: Resolves to an object with:
  - `exitCode` (Number): Python process exit code
  - `output` (String): Python process console output

**Example:**
```javascript
launchPythonScript(obj.requestId, obj.interactive, "scripts/extract.py", [obj.url])
  .then(function(result) {
      console.log("Python output:", result.output);
      return JSON.parse(result.output);
  })
  .catch(function(error) {
      console.error("Python execution failed:", error.error);
  });
```

## System Information

### qtJsNetworkProxyMgr.proxyForUrl(url)

Returns proxy information for a given URL.

**Parameters:**
- `url` (String): The URL to get proxy information for

**Returns:**
- `String`: Full URL to the proxy, including authorization information

**Example:**
```javascript
var proxyUrl = qtJsNetworkProxyMgr.proxyForUrl("https://example.com");
console.log("Proxy URL:", proxyUrl);
```

### qtJsSystem (API v3+)

Contains system information properties:

- `defaultUserAgent` (String): System's default user agent string
- `defaultWebBrowser` (String): System's default web browser name (e.g., firefox, chrome, edge)

**Example:**
```javascript
console.log("Default browser:", qtJsSystem.defaultWebBrowser);
console.log("Default user agent:", qtJsSystem.defaultUserAgent);
```

## Utilities

### qtJsTools.resolvedUrl(relativeUrl, baseUrl)

Resolves a relative URL against a base URL.

**Parameters:**
- `relativeUrl` (String): Relative URL
- `baseUrl` (String): Base URL

**Returns:**
- `String`: Resolved absolute URL

**Example:**
```javascript
var fullUrl = qtJsTools.resolvedUrl("/images/icon.png", "https://example.com/page/");
// Returns: "https://example.com/images/icon.png"
```

### qtJsTools.createTmpFile(desiredName) (API v3+)

Creates a temporary file.

**Parameters:**
- `desiredName` (String): Desired filename

**Returns:**
- `Object`: Temporary file object with methods:
  - `path()`: Returns the full path to the file
  - `writeText(text)`: Writes text to the file
  - `writeBinary(data)`: Writes binary data to the file

**Example:**
```javascript
var tmpFile = qtJsTools.createTmpFile("data.json");
tmpFile.writeText(JSON.stringify({key: "value"}));
console.log("Created temporary file at:", tmpFile.path());
```

## Console API

FDM plugins have access to standard console functions:

| Function | Description |
|----------|-------------|
| `console.log()`<br>`console.debug()` | Log a debug message |
| `console.info()` | Log an informational message |
| `console.warn()` | Log a warning |
| `console.error()` | Log an error |
| `console.assert()` | Check if an expression is true |
| `console.time(label)` | Start a timer |
| `console.timeEnd(label)` | End a timer and log elapsed time |
| `console.trace()` | Output a stack trace |
| `console.count(label)` | Count number of calls |

Additionally, `print()` is available as an alias for `console.debug()`.
