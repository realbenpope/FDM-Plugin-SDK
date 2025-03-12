# Manifest File Format

Every FDM plugin requires a `manifest.json` file that describes the plugin's metadata, capabilities, and requirements. This document provides a detailed reference for the manifest file format.

## Basic Structure

Here's an example of a complete `manifest.json` file:

```json
{
  "uuid": "fdm-example-plugin",
  "author": "Developer Name",
  "name": "Example Plugin",
  "description": "Provides support for downloading media from example.com.",
  "version": "1.0.0",
  "icon": "icon.png",
  "mediaParser": true,
  "mediaListParser": true,
  "scripts": [
    "parser.js",
    "batchparser.js"
  ],
  "updateUrl": "https://example.com/updates/fdm-example-plugin-update.json",
  "dependencies": {
    "Python": {"minVersion": "3.0"}
  },
  "permissions": [
    "launchPython"
  ],
  "minApiVersion": 1,
  "minFeaturesLevel": 1
}
```

## Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `uuid` | String | A unique identifier for your plugin. Must be unique across all plugins. |
| `author` | String | The name of the plugin author or organization. |
| `name` | String | The display name of the plugin. |
| `description` | String | A short description of what the plugin does. |
| `version` | String | The version of the plugin (e.g., "1.0.0"). Used for update checking. |
| `icon` | String | The filename of the plugin's icon. SVG, PNG, and other formats are supported. |
| `scripts` | Array of Strings | List of JavaScript files that contain the plugin code. |
| `minApiVersion` | Number | The minimum API version required by the plugin. |
| `minFeaturesLevel` | Number | The minimum set of required app features. |

## Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `mediaParser` | Boolean | Whether the plugin can handle single media downloads. If `true`, the plugin must implement the `msParser` object. |
| `mediaListParser` | Boolean | Whether the plugin can handle playlist/collection downloads. If `true`, the plugin must implement the `msBatchVideoParser` object. |
| `updateUrl` | String | HTTPS URL to a JSON file with update information. |
| `dependencies` | Object | Additional components the plugin depends on. |
| `permissions` | Array of Strings | Special permissions required by the plugin. |

## Dependencies

The `dependencies` object specifies external components that the plugin needs. Currently, only Python is supported as a dependency:

```json
"dependencies": {
  "Python": {"minVersion": "3.0"}
}
```

When a user installs a plugin with Python dependency, FDM will offer to automatically install Python if it's not already installed. FDM can automatically install:

- Python 3.8.10 (features level 1)
- Python 3.10.11 (features level 2)

You can omit the `minVersion` parameter to accept any Python version:

```json
"dependencies": {
  "Python": {}
}
```

## Permissions

The `permissions` array lists special capabilities that the plugin requires. Currently, only one permission is available:

- `launchPython`: Allows the plugin to execute Python scripts

Example:

```json
"permissions": [
  "launchPython"
]
```

When a user installs a plugin that requires permissions, they will be asked to confirm these permissions.

## Update Mechanism

If you provide an `updateUrl`, FDM will periodically check for updates to your plugin. The URL must point to a JSON file with the following format:

```json
{
  "version": "1.1.0",
  "distributiveUrl": "https://example.com/plugins/fdm-example-plugin-1.1.0.zip"
}
```

For plugins with different versions for different API versions:

```json
{
  "versions": [
    {
      "version": "1.1.0",
      "distributiveUrl": "https://example.com/plugins/fdm-example-plugin-1.1.0.zip",
      "minApiVersion": 1,
      "minFeaturesLevel": 1
    },
    {
      "version": "2.0.0",
      "distributiveUrl": "https://example.com/plugins/fdm-example-plugin-2.0.0.zip",
      "minApiVersion": 2,
      "minFeaturesLevel": 2
    }
  ]
}
```

> **Important**: Since FDM 6.24, plugins must be signed for the auto-update feature to work. Contact support@freedownloadmanager.org to get your plugin signed.

## Version Compatibility

The `minApiVersion` and `minFeaturesLevel` fields specify compatibility requirements:

- `minApiVersion`: The minimum API version your plugin is compatible with
- `minFeaturesLevel`: The minimum feature set required by your plugin

Current supported values:
- API Version: 1-4
- Features Level: 1-2

## Examples

### Simple Audio Downloader

```json
{
  "uuid": "fdm-audio-downloader",
  "author": "Jane Smith",
  "name": "Audio Downloader",
  "description": "Download audio from various sites.",
  "version": "1.0.0",
  "icon": "icon.png",
  "mediaParser": true,
  "scripts": [
    "audio-parser.js"
  ],
  "minApiVersion": 1,
  "minFeaturesLevel": 1
}
```

### Full-Featured Media Plugin with Python

```json
{
  "uuid": "fdm-media-suite",
  "author": "Media Tools, Inc.",
  "name": "Media Download Suite",
  "description": "Comprehensive media downloading support for multiple sites.",
  "version": "2.3.1",
  "icon": "icon.svg",
  "mediaParser": true,
  "mediaListParser": true,
  "scripts": [
    "common.js",
    "media-parser.js",
    "playlist-parser.js",
    "utils.js"
  ],
  "updateUrl": "https://mediatools.example.com/fdm-plugins/updates.json",
  "dependencies": {
    "Python": {"minVersion": "3.8"}
  },
  "permissions": [
    "launchPython"
  ],
  "minApiVersion": 3,
  "minFeaturesLevel": 2
}
```
