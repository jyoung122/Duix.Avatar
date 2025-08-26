# Lip-Sync Animation Component

This document describes the new lip-sync animation functionality added to the Duix.Avatar application.

## Overview

The lip-sync animation component allows users to animate existing avatars with uploaded audio files, creating synchronized lip movements that match the audio content.

## Features

- **Avatar Selection**: Choose from existing digital avatars/models
- **Audio Upload**: Support for MP3, WAV, FLAC, and M4A audio formats
- **Real-time Preview**: Preview the avatar with audio playback
- **Lip-sync Settings**: Adjustable sensitivity and smoothing parameters
- **Integration**: Seamlessly integrates with existing video synthesis pipeline

## Usage

1. **Access the Component**: Click the "Lip-Sync" banner on the home page or navigate to `/lipsync`

2. **Select an Avatar**: 
   - Browse available avatars in the left panel
   - Use the search function to find specific avatars
   - Click on an avatar to select it

3. **Upload Audio**:
   - Drag and drop an audio file into the upload area, or click to select
   - Supported formats: MP3, WAV, FLAC, M4A
   - Maximum duration: 30 minutes
   - Preview the audio with the built-in player

4. **Adjust Settings**:
   - **Sensitivity**: Controls how responsive the lip movements are to audio
   - **Smoothing**: Controls how smooth the transitions between lip positions are

5. **Preview**: Use the preview panel to see how the avatar will look with the audio

6. **Generate**: Click "Create Animation" to submit the lip-sync animation for processing

## Technical Details

### Components

- **LipSyncView.vue**: Main component container
- **HeaderView.vue**: Navigation and action buttons
- **AvatarSelectView.vue**: Avatar selection interface
- **AudioUploadView.vue**: Audio file upload and settings
- **PreviewView.vue**: Real-time preview with audio synchronization

### Integration

The component integrates with the existing video synthesis pipeline:

1. Creates a video record with the selected avatar and audio
2. Uses the existing `saveVideo()` and `makeVideo()` API functions
3. Results appear in the "My Works" section of the home page

### Internationalization

Full support for both English and Chinese languages with the `common.lipSync` translation namespace.

### Navigation

- Accessible via route: `/lipsync`
- New banner added to home page
- Back navigation returns to home page

## Files Added

```
src/renderer/src/views/lipsync-animation/
├── LipSyncView.vue                    # Main component
└── components/
    ├── HeaderView.vue                 # Header with actions
    ├── AvatarSelectView.vue          # Avatar selection
    ├── AudioUploadView.vue           # Audio upload
    └── PreviewView.vue               # Preview panel
```

### Modified Files

- `src/renderer/src/router/index.js` - Added `/lipsync` route
- `src/renderer/src/i18n/components/common.js` - Added translations
- `src/renderer/src/views/home/components/bannerList.vue` - Added lip-sync banner

## Future Enhancements

- Audio waveform visualization
- Advanced lip-sync parameter controls
- Batch processing for multiple audio files
- Custom avatar expressions during speech
- Audio preprocessing and noise reduction