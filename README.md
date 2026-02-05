
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>You can see Me</title>
<style>
  body {
    margin: 0;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: #0f1724;
    font-family: sans-serif;
    gap: 20px;
    color: #e2e8f0;
  }
  .progress-container {
    width: 90%;
    max-width: 500px;
    height: 4px;
    background: #1e293b;
    border-radius: 2px;
    overflow: hidden;
  }
  .progress-bar {
    height: 100%;
    width: 0%;
    background: #3b82f6;
    transition: width 0.1s linear;
  }
  input {
    width: 90%;
    max-width: 500px;
    padding: 14px 16px;
    border-radius: 12px;
    border: none;
    outline: none;
    font-size: 16px;
    background: #1e293b;
    color: #fff;
  }
  input::placeholder {
    color: #94a3b8;
  }
  #playerContainer {
    width: 90%;
    max-width: 560px;
    aspect-ratio: 16 / 9;
    border-radius: 12px;
    overflow: hidden;
    background: #000;
  }
  iframe {
    width: 100%;
    height: 100%;
    border: none;
  }
  footer {
    position: fixed;
    bottom: 10px;
    width: 100%;
    text-align: center;
    color: #94a3b8;
    font-size: 14px;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 6px;
  }
  .heart {
    color: #f43f5e;
    display: inline-block;
    font-size: 1.2em;
    animation: heartbeat 1.3s infinite ease-in-out;
  }
  @keyframes heartbeat {
    0%   { transform: scale(1.0); }
    14%  { transform: scale(1.35); }
    28%  { transform: scale(1.0); }
    42%  { transform: scale(1.35); }
    70%  { transform: scale(1.0); }
    100% { transform: scale(1.0); }
  }
</style>
<script src="https://www.youtube.com/iframe_api"></script>
</head>
<body>

<input id="ytInput" type="text" placeholder="" />

<div class="progress-container">
  <div class="progress-bar" id="progressBar"></div>
</div>

<div id="playerContainer"></div>

<script>
let ytPlayer = null;

function extractID(url) {
  if (!url) return null;
  const trimmed = url.trim();
  if (/^[a-zA-Z0-9_-]{11}$/.test(trimmed)) return trimmed;

  try {
    const u = new URL(trimmed);
    if (u.hostname.includes('youtu.be')) return u.pathname.slice(1);
    if (u.searchParams.get('v')) return u.searchParams.get('v');
    const parts = u.pathname.split('/');
    const embedIdx = parts.indexOf('embed');
    if (embedIdx >= 0 && parts[embedIdx + 1]) return parts[embedIdx + 1];
  } catch {}

  const match = trimmed.match(/(?:v=|\/embed\/|\.be\/)([a-zA-Z0-9_-]{11})/);
  return match ? match[1] : null;
}

function createPlayer(id) {
  const container = document.getElementById('playerContainer');
  container.innerHTML = '';

  const iframe = document.createElement('iframe');
  iframe.src = `https://www.youtube.com/embed/${id}?autoplay=1&mute=1&controls=1&modestbranding=1&rel=0&playsinline=1&enablejsapi=1`;
  iframe.allow = "autoplay; encrypted-media";
  iframe.allowFullscreen = true;

  container.appendChild(iframe);

  ytPlayer = new YT.Player(iframe, {
    events: {
      'onReady': onPlayerReady,
      'onStateChange': onPlayerStateChange
    }
  });

  // Progress bar start
  requestAnimationFrame(updateProgressBar);
}

function onPlayerReady(event) {
  // Video starts muted automatically
  document.getElementById('ytInput').value = ''; // Clear input
}

function onPlayerStateChange(event) {
  if (event.data === YT.PlayerState.ENDED) {
    document.getElementById('progressBar').style.width = '100%';
  }
}

function updateProgressBar() {
  if (ytPlayer && ytPlayer.getCurrentTime && ytPlayer.getDuration) {
    Promise.all([
      ytPlayer.getCurrentTime(),
      ytPlayer.getDuration()
    ]).then(([time, duration]) => {
      if (duration > 0) {
        const progress = (time / duration) * 100;
        document.getElementById('progressBar').style.width = `${progress}%`;
      }
    }).catch(() => {});
  }
  requestAnimationFrame(updateProgressBar);
}

document.getElementById('ytInput').addEventListener('input', e => {
  const value = e.target.value.trim();
  if (!value) return;

  const id = extractID(value);
  if (id) {
    createPlayer(id);
    e.target.blur(); // Hide keyboard on mobile
  }
});
</script>

<footer>
  Made in <span class="heart">❤️</span> for Pooja
</footer>

</body>
</html>
