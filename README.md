# Spotify Widget for Zebar

A minimal Spotify now-playing widget for [Zebar](https://github.com/glzr-io/zebar). Shows the currently playing track with scrolling text and playback controls.

## Features

- Spotify icon with scrolling track title and artist
- Playback controls: previous, play/pause, next
- Auto-hides when Spotify is not running
- Tokyo Night color scheme (easily re-themeable via CSS variables)

## Install (Standalone Floating Widget)

The quickest way to get started. This installs a small floating widget that sits on top of your desktop.

Search for **spotify-widget** in the Zebar marketplace, or clone this repo:

```
git clone https://github.com/johnsideserf/zebar-spotify-widget.git
```

Then add it to your `~/.glzr/zebar/settings.json`:

```json
{
  "startupConfigs": [
    { "pack": "johnsideserf.spotify-widget", "widget": "spotify", "preset": "default" }
  ]
}
```

## Embedding in an Existing Bar

If you'd rather have the Spotify widget inside your existing Zebar bar (e.g. the starter `with-glazewm` bar) instead of as a separate floating window, you'll need to manually add the code to your bar config. The marketplace doesn't support embeddable components, so this requires a few copy-paste steps.

### 1. Add imports

Add `useRef` and `useLayoutEffect` to your React imports:

```js
import React, {
  useState,
  useEffect,
  useRef,
  useLayoutEffect,
} from 'https://esm.sh/react@18?dev';
```

### 2. Add the media provider

Add `media` to your provider group if you don't already have it:

```js
const providers = zebar.createProviderGroup({
  // ...your existing providers
  media: { type: 'media' },
});
```

### 3. Add the scroll logic to your component

Add this near the top of your `App()` function, after the `useState` and `useEffect` calls:

```js
const scrollRef = useRef(null);

const spotifySession = output.media?.allSessions?.find(s =>
  s.sessionId.toLowerCase().includes('spotify'),
);

const trackKey = spotifySession
  ? `${spotifySession.title}\0${spotifySession.artist}`
  : '';

// Set animation duration based on element width for constant scroll speed.
useLayoutEffect(() => {
  const el = scrollRef.current;
  if (el) {
    const pixelsPerSecond = 50; // adjust for desired speed
    const duration = el.scrollWidth / pixelsPerSecond;
    el.style.animationDuration = `${duration}s`;
  }
}, [trackKey]);
```

### 4. Add the JSX

Place this wherever you want the widget to appear in your bar's JSX:

```jsx
{spotifySession && (
  <div className="media">
    <i className="nf nf-md-spotify"></i>
    <div className="media-text">
      <span key={trackKey} ref={scrollRef} className="media-scroll">
        {spotifySession.title}
        {spotifySession.artist && ` \u2013 ${spotifySession.artist}`}
      </span>
    </div>
    <button className="media-btn"
      onClick={() => output.media.previous({ sessionId: spotifySession.sessionId })}>
      <i className="nf nf-md-skip_previous"></i>
    </button>
    <button className="media-btn"
      onClick={() => output.media.togglePlayPause({ sessionId: spotifySession.sessionId })}>
      <i className={`nf ${spotifySession.isPlaying ? 'nf-md-pause' : 'nf-md-play'}`}></i>
    </button>
    <button className="media-btn"
      onClick={() => output.media.next({ sessionId: spotifySession.sessionId })}>
      <i className="nf nf-md-skip_next"></i>
    </button>
  </div>
)}
```

### 5. Add the CSS

Copy these styles into your bar's stylesheet:

```css
.media {
  display: flex;
  align-items: center;
  gap: 2px;
}

.media > i {
  color: var(--tn-green, #9ece6a);
  margin-right: 6px;
}

.media-text {
  width: 180px;
  overflow: hidden;
  margin-right: 4px;
  mask-image: linear-gradient(to right, transparent 0, #fff 6px, #fff calc(100% - 6px), transparent 100%);
  -webkit-mask-image: linear-gradient(to right, transparent 0, #fff 6px, #fff calc(100% - 6px), transparent 100%);
}

.media-scroll {
  display: inline-block;
  white-space: nowrap;
  padding-left: 100%;
  animation: media-scroll 0s linear infinite;
}

@keyframes media-scroll {
  0%   { transform: translateX(0); }
  100% { transform: translateX(-100%); }
}

.media-btn {
  background: none;
  border: none;
  padding: 2px 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.media-btn i {
  color: var(--tn-purple, #bb9af7);
  margin-right: 0;
  transition: color 0.15s ease;
}

.media-btn:hover i {
  color: var(--tn-blue, #7aa2f7);
}
```

## Customization

### Theme colors

For the **standalone widget**, override these CSS variables:

```css
:root {
  --bg: rgba(13 14 22 / 92%);
  --fg: #a9b1d6;
  --accent: #9ece6a;        /* Spotify icon color */
  --controls: #bb9af7;      /* Control button color */
  --controls-hover: #7aa2f7; /* Control button hover color */
}
```

For the **embedded version**, the styles use Tokyo Night variables (`--tn-green`, `--tn-purple`, `--tn-blue`) with fallback defaults. Adjust these to match your bar's theme.

### Scroll speed

Change the `pixelsPerSecond` value in the `useLayoutEffect` block. Higher values = faster scrolling. Default is `50`.
