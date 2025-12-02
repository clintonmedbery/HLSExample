# Understanding Video Controls: Vanilla JS vs HLS.js

## Table of Contents
- [HTML5 Video Element Basics](#html5-video-element-basics)
- [Vanilla JavaScript Video Control](#vanilla-javascript-video-control)
- [HLS.js Integration](#hlsjs-integration)
- [Complete Examples](#complete-examples)

---

## HTML5 Video Element Basics

### The HTMLMediaElement API

The `<video>` tag in HTML5 is an instance of `HTMLMediaElement`, which provides a rich API for controlling video playback.

```html
<video id="myVideo" controls>
  <source src="video.mp4" type="video/mp4">
</video>
```

### Core Properties

| Property | Type | Description |
|----------|------|-------------|
| `currentTime` | number | Current playback position (in seconds) |
| `duration` | number | Total video length (in seconds, read-only) |
| `volume` | number | Volume level (0.0 to 1.0) |
| `muted` | boolean | Whether audio is muted |
| `paused` | boolean | Whether video is paused (read-only) |
| `playbackRate` | number | Playback speed (1.0 = normal, 2.0 = 2x speed) |
| `ended` | boolean | Whether video has finished playing |
| `seeking` | boolean | Whether video is currently seeking |
| `readyState` | number | How much of the video has been loaded |
| `networkState` | number | Current network activity state |
| `buffered` | TimeRanges | Buffered time ranges |

### Core Methods

| Method | Description |
|--------|-------------|
| `play()` | Start playback (returns a Promise) |
| `pause()` | Pause playback |
| `load()` | Reload the video source |
| `canPlayType(type)` | Check if a MIME type is supported |

### Core Events

| Event | When It Fires |
|-------|---------------|
| `play` | Playback starts |
| `pause` | Playback pauses |
| `ended` | Video finishes playing |
| `timeupdate` | Current time changes (~4 times/second) |
| `volumechange` | Volume or mute state changes |
| `seeking` | Seek operation begins |
| `seeked` | Seek operation completes |
| `loadedmetadata` | Metadata (duration, dimensions) loads |
| `loadeddata` | First frame loads |
| `canplay` | Enough data to start playing |
| `canplaythrough` | Can play through without buffering |
| `waiting` | Playback stopped due to buffering |
| `error` | An error occurs |

---

## Vanilla JavaScript Video Control

### Basic Setup

```javascript
// Get reference to video element
const video = document.getElementById('myVideo');

// Or create one programmatically
const video = document.createElement('video');
video.src = 'video.mp4';
video.controls = false; // Hide native controls
document.body.appendChild(video);
```

### Play/Pause Control

```javascript
const playBtn = document.getElementById('play-btn');
const video = document.getElementById('video');

// Method 1: Toggle with promise handling
playBtn.addEventListener('click', () => {
  if (video.paused) {
    video.play()
      .then(() => {
        console.log('Playback started');
        playBtn.textContent = 'Pause';
      })
      .catch(error => {
        console.error('Play failed:', error);
      });
  } else {
    video.pause();
    playBtn.textContent = 'Play';
  }
});

// Method 2: Listen to video state changes
video.addEventListener('play', () => {
  playBtn.textContent = 'Pause';
});

video.addEventListener('pause', () => {
  playBtn.textContent = 'Play';
});
```

### Time Control (Seeking)

```javascript
// Skip forward 10 seconds
const skipForward = () => {
  video.currentTime = Math.min(
    video.currentTime + 10,
    video.duration
  );
};

// Skip backward 10 seconds
const skipBackward = () => {
  video.currentTime = Math.max(
    video.currentTime - 10,
    0
  );
};

// Jump to specific time
const jumpTo = (seconds) => {
  video.currentTime = seconds;
};

// Jump to percentage of video
const jumpToPercent = (percent) => {
  video.currentTime = (percent / 100) * video.duration;
};
```

### Progress Bar Implementation

```javascript
const progressBar = document.getElementById('progress-bar');
const progressFilled = document.getElementById('progress-filled');

// Update progress bar as video plays
video.addEventListener('timeupdate', () => {
  const percent = (video.currentTime / video.duration) * 100;
  progressFilled.style.width = percent + '%';
});

// Click progress bar to seek
progressBar.addEventListener('click', (e) => {
  const rect = progressBar.getBoundingClientRect();
  const clickX = e.clientX - rect.left;
  const percent = clickX / rect.width;
  video.currentTime = percent * video.duration;
});

// Drag to seek
let isDragging = false;

progressBar.addEventListener('mousedown', () => {
  isDragging = true;
});

document.addEventListener('mousemove', (e) => {
  if (isDragging) {
    const rect = progressBar.getBoundingClientRect();
    const clickX = Math.max(0, Math.min(e.clientX - rect.left, rect.width));
    const percent = clickX / rect.width;
    video.currentTime = percent * video.duration;
  }
});

document.addEventListener('mouseup', () => {
  isDragging = false;
});
```

### Volume Control

```javascript
const volumeSlider = document.getElementById('volume-slider');
const muteBtn = document.getElementById('mute-btn');

// Volume slider (0-100 to 0.0-1.0)
volumeSlider.addEventListener('input', (e) => {
  const volume = e.target.value / 100;
  video.volume = volume;
  video.muted = volume === 0;
});

// Mute button
muteBtn.addEventListener('click', () => {
  video.muted = !video.muted;
  volumeSlider.value = video.muted ? 0 : video.volume * 100;
});

// Listen to volume changes (from any source)
video.addEventListener('volumechange', () => {
  volumeSlider.value = video.muted ? 0 : video.volume * 100;
  muteBtn.textContent = video.muted ? 'Unmute' : 'Mute';
});
```

### Display Time

```javascript
const currentTimeDisplay = document.getElementById('current-time');
const durationDisplay = document.getElementById('duration');

// Format seconds to MM:SS
function formatTime(seconds) {
  const mins = Math.floor(seconds / 60);
  const secs = Math.floor(seconds % 60);
  return `${mins}:${secs.toString().padStart(2, '0')}`;
}

// Update current time display
video.addEventListener('timeupdate', () => {
  currentTimeDisplay.textContent = formatTime(video.currentTime);
});

// Set duration when metadata loads
video.addEventListener('loadedmetadata', () => {
  durationDisplay.textContent = formatTime(video.duration);
});
```

### Fullscreen Control

```javascript
const fullscreenBtn = document.getElementById('fullscreen-btn');
const videoContainer = document.getElementById('video-container');

fullscreenBtn.addEventListener('click', () => {
  if (!document.fullscreenElement) {
    // Enter fullscreen
    if (videoContainer.requestFullscreen) {
      videoContainer.requestFullscreen();
    } else if (videoContainer.webkitRequestFullscreen) {
      videoContainer.webkitRequestFullscreen(); // Safari
    } else if (videoContainer.mozRequestFullScreen) {
      videoContainer.mozRequestFullScreen(); // Firefox
    } else if (videoContainer.msRequestFullscreen) {
      videoContainer.msRequestFullscreen(); // IE/Edge
    }
  } else {
    // Exit fullscreen
    if (document.exitFullscreen) {
      document.exitFullscreen();
    }
  }
});

// Listen for fullscreen changes
document.addEventListener('fullscreenchange', () => {
  if (document.fullscreenElement) {
    fullscreenBtn.textContent = 'Exit Fullscreen';
  } else {
    fullscreenBtn.textContent = 'Fullscreen';
  }
});
```

### Playback Speed

```javascript
// Change playback speed
const setSpeed = (speed) => {
  video.playbackRate = speed;
};

// Examples:
setSpeed(0.5);  // Half speed
setSpeed(1.0);  // Normal speed
setSpeed(1.5);  // 1.5x speed
setSpeed(2.0);  // 2x speed

// Listen for rate changes
video.addEventListener('ratechange', () => {
  console.log('Playback rate:', video.playbackRate);
});
```

---

## HLS.js Integration

### What is HLS.js?

HLS.js is a JavaScript library that enables HLS (HTTP Live Streaming) playback in browsers that don't natively support it. It works by:

1. Parsing the `.m3u8` playlist file
2. Downloading video segments
3. Converting them to a format the browser understands
4. Feeding them to the video element via Media Source Extensions (MSE)

### Key Concept: HLS.js + Video Element

**Important:** HLS.js does NOT replace the video element. It **works alongside it**.

```
┌─────────────────┐
│   Your Buttons  │
│   & Controls    │
└────────┬────────┘
         │
         ├──────────────────┐
         │                  │
         ▼                  ▼
┌─────────────────┐  ┌──────────────┐
│  Video Element  │  │   HLS.js     │
│   (playback)    │◄─┤ (source mgr) │
└─────────────────┘  └──────────────┘
                            │
                            ▼
                     ┌──────────────┐
                     │ .m3u8 + .m4s │
                     │   segments   │
                     └──────────────┘
```

### Basic HLS.js Setup

```javascript
import Hls from 'hls.js';

const video = document.getElementById('video');
const videoSrc = 'playlist.m3u8';

if (Hls.isSupported()) {
  // Browser needs HLS.js
  const hls = new Hls();

  // Load the playlist
  hls.loadSource(videoSrc);

  // Attach to video element
  hls.attachMedia(video);

  // Wait for manifest to parse
  hls.on(Hls.Events.MANIFEST_PARSED, () => {
    console.log('HLS playlist loaded');
    video.play();
  });

} else if (video.canPlayType('application/vnd.apple.mpegurl')) {
  // Native HLS support (Safari)
  video.src = videoSrc;
  video.addEventListener('loadedmetadata', () => {
    console.log('Native HLS loaded');
    video.play();
  });
}
```

### HLS.js Events

HLS.js provides its own event system for stream-specific events:

```javascript
const hls = new Hls();

// Manifest loaded and parsed
hls.on(Hls.Events.MANIFEST_PARSED, (event, data) => {
  console.log('Levels:', data.levels); // Available quality levels
  console.log('First level bitrate:', data.levels[0].bitrate);
});

// Level switching (quality change)
hls.on(Hls.Events.LEVEL_SWITCHED, (event, data) => {
  console.log('Switched to level:', data.level);
});

// Fragment (segment) loaded
hls.on(Hls.Events.FRAG_LOADED, (event, data) => {
  console.log('Loaded fragment:', data.frag.sn);
});

// Error handling
hls.on(Hls.Events.ERROR, (event, data) => {
  console.error('HLS Error:', data);

  if (data.fatal) {
    switch(data.type) {
      case Hls.ErrorTypes.NETWORK_ERROR:
        console.log('Network error, trying to recover...');
        hls.startLoad();
        break;
      case Hls.ErrorTypes.MEDIA_ERROR:
        console.log('Media error, trying to recover...');
        hls.recoverMediaError();
        break;
      default:
        console.log('Fatal error, destroying HLS...');
        hls.destroy();
        break;
    }
  }
});
```

### Controlling Video with HLS.js Active

**The video element API works exactly the same!**

```javascript
const video = document.getElementById('video');
const hls = new Hls();

// Setup HLS
hls.loadSource('playlist.m3u8');
hls.attachMedia(video);

// Control video EXACTLY as before
const playBtn = document.getElementById('play-btn');

playBtn.addEventListener('click', () => {
  if (video.paused) {
    video.play(); // ← Same as vanilla!
  } else {
    video.pause(); // ← Same as vanilla!
  }
});

// Seeking works the same
document.getElementById('forward').addEventListener('click', () => {
  video.currentTime += 10; // ← Same as vanilla!
});

// Volume works the same
document.getElementById('volume').addEventListener('input', (e) => {
  video.volume = e.target.value / 100; // ← Same as vanilla!
});

// Progress tracking works the same
video.addEventListener('timeupdate', () => {
  console.log(video.currentTime); // ← Same as vanilla!
});
```

### HLS.js-Specific Features

#### Quality Level Control

```javascript
// Get available quality levels
hls.on(Hls.Events.MANIFEST_PARSED, () => {
  const levels = hls.levels;

  levels.forEach((level, index) => {
    console.log(`Level ${index}:`, {
      height: level.height,
      width: level.width,
      bitrate: level.bitrate
    });
  });
});

// Manually set quality level
const setQuality = (levelIndex) => {
  hls.currentLevel = levelIndex; // -1 for auto
};

// Auto quality (adaptive bitrate)
hls.currentLevel = -1;

// Listen for quality changes
hls.on(Hls.Events.LEVEL_SWITCHED, (event, data) => {
  console.log('Now playing level:', data.level);
});
```

#### Live Stream Specific

```javascript
// Check if stream is live
hls.on(Hls.Events.MANIFEST_PARSED, () => {
  console.log('Is live:', hls.liveSyncPosition !== null);
});

// Jump to live edge
const goToLive = () => {
  if (hls.liveSyncPosition) {
    video.currentTime = hls.liveSyncPosition;
  }
};

// Get latency
const getLatency = () => {
  return hls.latency; // Current latency in seconds
};
```

#### Buffer Control

```javascript
// Configure buffer sizes
const hls = new Hls({
  maxBufferLength: 30,      // Max buffer in seconds
  maxMaxBufferLength: 600,  // Max max buffer
  maxBufferSize: 60 * 1000 * 1000, // Max buffer size in bytes
  maxBufferHole: 0.5,       // Max gap in buffer
});

// Get buffer info
video.addEventListener('timeupdate', () => {
  const buffered = video.buffered;

  if (buffered.length > 0) {
    const bufferedEnd = buffered.end(buffered.length - 1);
    const bufferedSeconds = bufferedEnd - video.currentTime;
    console.log(`${bufferedSeconds}s buffered ahead`);
  }
});
```

---

## Complete Examples

### Example 1: Full Custom Controls (Vanilla JS)

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    .player { max-width: 800px; }
    video { width: 100%; }
    .controls { display: flex; gap: 10px; padding: 10px; background: #333; }
    button { padding: 10px; cursor: pointer; }
    .progress { flex: 1; height: 30px; background: #555; cursor: pointer; }
    .progress-bar { height: 100%; background: #2dd4bf; width: 0%; }
  </style>
</head>
<body>
  <div class="player">
    <video id="video">
      <source src="video.mp4" type="video/mp4">
    </video>

    <div class="controls">
      <button id="play">Play</button>
      <button id="rewind">-10s</button>
      <button id="forward">+10s</button>
      <span id="time">0:00 / 0:00</span>
      <div class="progress" id="progress">
        <div class="progress-bar" id="progress-bar"></div>
      </div>
      <input type="range" id="volume" min="0" max="100" value="100">
      <button id="fullscreen">Fullscreen</button>
    </div>
  </div>

  <script>
    const video = document.getElementById('video');
    const playBtn = document.getElementById('play');
    const rewindBtn = document.getElementById('rewind');
    const forwardBtn = document.getElementById('forward');
    const timeDisplay = document.getElementById('time');
    const progress = document.getElementById('progress');
    const progressBar = document.getElementById('progress-bar');
    const volumeSlider = document.getElementById('volume');
    const fullscreenBtn = document.getElementById('fullscreen');

    // Format time
    const formatTime = (sec) => {
      const mins = Math.floor(sec / 60);
      const secs = Math.floor(sec % 60);
      return `${mins}:${secs.toString().padStart(2, '0')}`;
    };

    // Play/Pause
    playBtn.addEventListener('click', () => {
      video.paused ? video.play() : video.pause();
    });

    video.addEventListener('play', () => playBtn.textContent = 'Pause');
    video.addEventListener('pause', () => playBtn.textContent = 'Play');

    // Rewind/Forward
    rewindBtn.addEventListener('click', () => {
      video.currentTime = Math.max(0, video.currentTime - 10);
    });

    forwardBtn.addEventListener('click', () => {
      video.currentTime = Math.min(video.duration, video.currentTime + 10);
    });

    // Time display
    video.addEventListener('timeupdate', () => {
      timeDisplay.textContent =
        `${formatTime(video.currentTime)} / ${formatTime(video.duration)}`;

      const percent = (video.currentTime / video.duration) * 100;
      progressBar.style.width = percent + '%';
    });

    video.addEventListener('loadedmetadata', () => {
      timeDisplay.textContent =
        `0:00 / ${formatTime(video.duration)}`;
    });

    // Progress bar
    progress.addEventListener('click', (e) => {
      const rect = progress.getBoundingClientRect();
      const percent = (e.clientX - rect.left) / rect.width;
      video.currentTime = percent * video.duration;
    });

    // Volume
    volumeSlider.addEventListener('input', (e) => {
      video.volume = e.target.value / 100;
    });

    // Fullscreen
    fullscreenBtn.addEventListener('click', () => {
      if (!document.fullscreenElement) {
        document.querySelector('.player').requestFullscreen();
      } else {
        document.exitFullscreen();
      }
    });
  </script>
</body>
</html>
```

### Example 2: HLS with Custom Controls

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    .player { max-width: 800px; }
    video { width: 100%; background: #000; }
    .controls {
      display: flex;
      gap: 10px;
      padding: 10px;
      background: #2b2d30;
      color: white;
    }
    button {
      padding: 10px;
      cursor: pointer;
      background: #444;
      border: none;
      color: white;
    }
    button:hover { background: #555; }
    .progress {
      flex: 1;
      height: 30px;
      background: #444;
      cursor: pointer;
      display: flex;
      align-items: center;
      padding: 0 10px;
    }
    .progress-bar {
      height: 4px;
      background: #2dd4bf;
      width: 0%;
    }
    select { padding: 8px; }
    #status { padding: 10px; background: #333; color: white; }
  </style>
</head>
<body>
  <div class="player">
    <video id="video"></video>

    <div class="controls">
      <button id="play">Play</button>
      <button id="rewind">-10s</button>
      <button id="forward">+10s</button>
      <span id="time">0:00 / 0:00</span>
      <div class="progress" id="progress">
        <div class="progress-bar" id="progress-bar"></div>
      </div>
      <select id="quality">
        <option value="-1">Auto</option>
      </select>
      <input type="range" id="volume" min="0" max="100" value="100">
    </div>

    <div id="status">Loading...</div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <script>
    const video = document.getElementById('video');
    const playBtn = document.getElementById('play');
    const rewindBtn = document.getElementById('rewind');
    const forwardBtn = document.getElementById('forward');
    const timeDisplay = document.getElementById('time');
    const progress = document.getElementById('progress');
    const progressBar = document.getElementById('progress-bar');
    const qualitySelect = document.getElementById('quality');
    const volumeSlider = document.getElementById('volume');
    const status = document.getElementById('status');

    let hls;

    // Format time
    const formatTime = (sec) => {
      const mins = Math.floor(sec / 60);
      const secs = Math.floor(sec % 60);
      return `${mins}:${secs.toString().padStart(2, '0')}`;
    };

    // Initialize HLS
    if (Hls.isSupported()) {
      hls = new Hls({
        debug: false,
        enableWorker: true,
      });

      hls.loadSource('output.m3u8');
      hls.attachMedia(video);

      // Manifest parsed
      hls.on(Hls.Events.MANIFEST_PARSED, (event, data) => {
        status.textContent = `Loaded ${data.levels.length} quality level(s)`;

        // Populate quality selector
        data.levels.forEach((level, index) => {
          const option = document.createElement('option');
          option.value = index;
          option.textContent = `${level.height}p (${Math.round(level.bitrate / 1000)}kbps)`;
          qualitySelect.appendChild(option);
        });

        video.play().catch(e => {
          status.textContent = 'Click play to start';
        });
      });

      // Quality change
      qualitySelect.addEventListener('change', (e) => {
        hls.currentLevel = parseInt(e.target.value);
      });

      // Level switched
      hls.on(Hls.Events.LEVEL_SWITCHED, (event, data) => {
        const level = hls.levels[data.level];
        status.textContent = `Playing: ${level.height}p`;
      });

      // Error handling
      hls.on(Hls.Events.ERROR, (event, data) => {
        console.error('HLS Error:', data);
        if (data.fatal) {
          status.textContent = `Error: ${data.type}`;
          switch(data.type) {
            case Hls.ErrorTypes.NETWORK_ERROR:
              hls.startLoad();
              break;
            case Hls.ErrorTypes.MEDIA_ERROR:
              hls.recoverMediaError();
              break;
            default:
              hls.destroy();
              break;
          }
        }
      });
    }

    // Video controls (work the same with or without HLS!)

    // Play/Pause
    playBtn.addEventListener('click', () => {
      video.paused ? video.play() : video.pause();
    });

    video.addEventListener('play', () => playBtn.textContent = 'Pause');
    video.addEventListener('pause', () => playBtn.textContent = 'Play');

    // Rewind/Forward
    rewindBtn.addEventListener('click', () => {
      video.currentTime = Math.max(0, video.currentTime - 10);
    });

    forwardBtn.addEventListener('click', () => {
      video.currentTime = Math.min(video.duration, video.currentTime + 10);
    });

    // Time display & progress
    video.addEventListener('timeupdate', () => {
      timeDisplay.textContent =
        `${formatTime(video.currentTime)} / ${formatTime(video.duration)}`;

      const percent = (video.currentTime / video.duration) * 100;
      progressBar.style.width = percent + '%';
    });

    // Progress bar seeking
    progress.addEventListener('click', (e) => {
      const rect = progress.getBoundingClientRect();
      const percent = (e.clientX - rect.left) / rect.width;
      video.currentTime = percent * video.duration;
    });

    // Volume
    volumeSlider.addEventListener('input', (e) => {
      video.volume = e.target.value / 100;
    });
  </script>
</body>
</html>
```

---

## Key Takeaways

### Vanilla JS with Video Element

✅ **Use native video API directly**
- `video.play()`, `video.pause()`, `video.currentTime`, etc.
- Works with standard video formats (MP4, WebM)
- Simple and straightforward

### HLS.js with Video Element

✅ **HLS.js handles the source, you control the playback**
- HLS.js manages playlist parsing and segment loading
- **The video element API remains unchanged**
- You still use `video.play()`, `video.currentTime`, etc.
- HLS.js provides additional events for stream-specific information

### The Golden Rule

**Your buttons control the video element. HLS.js just feeds it data.**

```javascript
// This works exactly the same whether HLS.js is used or not:
video.play();
video.pause();
video.currentTime = 30;
video.volume = 0.5;
```

The only difference is **where the video data comes from**:
- **Vanilla:** `<source src="video.mp4">`
- **HLS.js:** `hls.loadSource('playlist.m3u8')`

Both methods result in the same video element that responds to the same controls!
