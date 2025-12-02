# HLS Video Player

A custom HTML5 video player with HLS (HTTP Live Streaming) support, featuring a modern dark UI with custom controls.

## Quick Start

### Running the Player

1. Start the HTTP server:
   ```bash
   python3 -m http.server 8000
   ```

2. Open your browser and navigate to:
   ```
   http://localhost:8000/player.html
   ```

3. The video will automatically load and play.

## How HLS Works

### Overview

HLS (HTTP Live Streaming) is an adaptive bitrate streaming protocol developed by Apple. It works by breaking the video into small chunks (segments) and serving them over standard HTTP.

### Key Components

1. **Master Playlist (.m3u8)**
   - A text file containing metadata about the video segments
   - Lists all video segments in order with their durations
   - Contains playback information like target duration and sequence numbers

2. **Initialization Segment (init.mp4)**
   - Contains codec information and metadata
   - Required for proper decoding of the video segments
   - Downloaded once before any segments

3. **Media Segments (.m4s files)**
   - Small chunks of video data (typically 2-10 seconds each)
   - Can be independently decoded and played
   - Enables adaptive streaming and seeking

### How It Streams

1. Player requests and parses the `.m3u8` playlist
2. Player downloads the initialization segment
3. Player sequentially downloads and plays media segments
4. As segments are downloaded, they're buffered and played seamlessly
5. Player can adapt quality based on network conditions (with multi-bitrate playlists)

### Example Playlist Structure

```
#EXTM3U
#EXT-X-VERSION:7
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-MAP:URI="init.mp4"
#EXTINF:6.013968,
segment_000.m4s
#EXTINF:5.990748,
segment_001.m4s
...
#EXT-X-ENDLIST
```

## Creating HLS Streams with FFmpeg

### Basic Video Conversion

Convert a video file to HLS format:

```bash
ffmpeg -i input.mp4 \
  -c:v libx264 \
  -c:a aac \
  -hls_time 6 \
  -hls_playlist_type vod \
  -hls_segment_type fmp4 \
  -hls_fmp4_init_filename init.mp4 \
  -hls_segment_filename "segment_%03d.m4s" \
  output.m3u8
```

### Audio-Only HLS

Create an HLS stream from an audio file:

```bash
ffmpeg -i input.mp3 \
  -c:a aac \
  -b:a 128k \
  -hls_time 6 \
  -hls_playlist_type vod \
  -hls_segment_type fmp4 \
  -hls_fmp4_init_filename init.mp4 \
  -hls_segment_filename "segment_%03d.m4s" \
  output.m3u8
```

### Multi-Bitrate (Adaptive) Streaming

Create multiple quality levels for adaptive streaming:

```bash
# High quality
ffmpeg -i input.mp4 \
  -c:v libx264 -b:v 2000k -s 1920x1080 \
  -c:a aac -b:a 192k \
  -hls_time 6 \
  -hls_playlist_type vod \
  -hls_segment_type fmp4 \
  -hls_fmp4_init_filename init_1080p.mp4 \
  -hls_segment_filename "1080p_%03d.m4s" \
  playlist_1080p.m3u8

# Medium quality
ffmpeg -i input.mp4 \
  -c:v libx264 -b:v 1000k -s 1280x720 \
  -c:a aac -b:a 128k \
  -hls_time 6 \
  -hls_playlist_type vod \
  -hls_segment_type fmp4 \
  -hls_fmp4_init_filename init_720p.mp4 \
  -hls_segment_filename "720p_%03d.m4s" \
  playlist_720p.m3u8

# Low quality
ffmpeg -i input.mp4 \
  -c:v libx264 -b:v 500k -s 854x480 \
  -c:a aac -b:a 96k \
  -hls_time 6 \
  -hls_playlist_type vod \
  -hls_segment_type fmp4 \
  -hls_fmp4_init_filename init_480p.mp4 \
  -hls_segment_filename "480p_%03d.m4s" \
  playlist_480p.m3u8
```

Then create a master playlist (`master.m3u8`):

```
#EXTM3U
#EXT-X-VERSION:7

#EXT-X-STREAM-INF:BANDWIDTH=2192000,RESOLUTION=1920x1080
playlist_1080p.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=1128000,RESOLUTION=1280x720
playlist_720p.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=596000,RESOLUTION=854x480
playlist_480p.m3u8
```

### FFmpeg Options Explained

| Option | Description |
|--------|-------------|
| `-hls_time` | Target duration for each segment (in seconds) |
| `-hls_playlist_type` | `vod` for video-on-demand, `event` for live streaming |
| `-hls_segment_type` | `fmp4` for fragmented MP4 (modern), `mpegts` for MPEG-TS (legacy) |
| `-hls_fmp4_init_filename` | Name of the initialization segment |
| `-hls_segment_filename` | Pattern for naming media segments |
| `-c:v` | Video codec (e.g., `libx264`, `libx265`, `copy`) |
| `-c:a` | Audio codec (e.g., `aac`, `mp3`, `copy`) |
| `-b:v` | Video bitrate |
| `-b:a` | Audio bitrate |

## Player Features

### Custom Controls

- **Play/Pause** - Space bar or click button
- **Rewind 10s** - Left arrow or click button
- **Forward 30s** - Right arrow or click button
- **Volume Control** - Hover to expand slider
- **Progress Bar** - Click to seek, shows current position
- **Fullscreen** - F key or click button
- **Mute/Unmute** - M key or click button

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Space | Play/Pause |
| ← | Rewind 10 seconds |
| → | Forward 30 seconds |
| M | Mute/Unmute |
| F | Toggle Fullscreen |

## Browser Compatibility

- **Chrome/Edge**: Uses hls.js library
- **Firefox**: Uses hls.js library
- **Safari**: Native HLS support (no library needed)
- **Mobile Browsers**: Full support on iOS and Android

## File Structure

```
.
├── player.html           # Custom video player
├── output.m3u8          # HLS playlist/manifest
├── init.mp4             # Initialization segment
├── segment_000.m4s      # Media segments
├── segment_001.m4s
└── ...
```

## Troubleshooting

### Video Won't Play

1. Ensure the server is running on port 8000
2. Check that `init.mp4` exists in the directory
3. Open browser console to check for errors
4. Verify all segment files are present

### CORS Issues

If serving from a different domain, ensure proper CORS headers:

```python
# Python server with CORS
python3 -m http.server 8000 --bind 0.0.0.0
```

### Segment Not Found

Verify the playlist references the correct filenames:
```bash
cat output.m3u8
```

## Advanced Usage

### Live Streaming

For live streaming, use:
```bash
ffmpeg -i rtmp://stream-source \
  -c:v libx264 \
  -c:a aac \
  -hls_time 2 \
  -hls_list_size 5 \
  -hls_flags delete_segments \
  -hls_playlist_type event \
  output.m3u8
```

### Encryption (DRM)

Add AES-128 encryption:
```bash
# Generate encryption key
openssl rand 16 > enc.key

# Create key info file
echo "http://localhost:8000/enc.key" > enc.keyinfo
echo "enc.key" >> enc.keyinfo
openssl rand -hex 16 >> enc.keyinfo

# Encode with encryption
ffmpeg -i input.mp4 \
  -hls_key_info_file enc.keyinfo \
  -hls_segment_type fmp4 \
  -hls_playlist_type vod \
  output.m3u8
```

## Resources

- [HLS Specification (RFC 8216)](https://tools.ietf.org/html/rfc8216)
- [FFmpeg HLS Documentation](https://ffmpeg.org/ffmpeg-formats.html#hls-2)
- [hls.js Library](https://github.com/video-dev/hls.js/)
- [Apple HLS Authoring Guide](https://developer.apple.com/documentation/http_live_streaming)
-[HLS React](https://dev.to/indranilchutia/how-to-implement-hls-video-streaming-in-a-react-app-2cki)

## License

This project is provided as-is for educational and development purposes.
