<script lang="ts">
  import { onDestroy } from 'svelte';

  // --- State ---
  let seconds = $state(0);
  let running = $state(false);
  let finished = $state(false);
  let inputMinutes = $state(1);
  let totalSeconds = $state(60);
  let remaining = $state(60);
  let interval: ReturnType<typeof setInterval> | null = null;

  // --- Emoji mood based on time left ---
  const moods = ['😴', '🥱', '😐', '🙂', '😊', '😄', '🤩', '🔥', '💥', '⚡'];
  let mood = $derived(
    finished
      ? '🎉'
      : moods[Math.floor(((totalSeconds - remaining) / totalSeconds) * moods.length)]
  );

  // --- Progress ---
  let progress = $derived(totalSeconds > 0 ? ((totalSeconds - remaining) / totalSeconds) * 100 : 0);
  let circumference = 2 * Math.PI * 90; // radius 90
  let dashOffset = $derived(circumference - (progress / 100) * circumference);

  // --- Time display ---
  let displayMin = $derived(Math.floor(remaining / 60));
  let displaySec = $derived(remaining % 60);

  // --- Color based on progress ---
  let ringColor = $derived(
    progress < 50
      ? '#6ee7f7'
      : progress < 80
      ? '#f7c76e'
      : '#f76e6e'
  );

  function start() {
    if (running || remaining === 0) return;
    running = true;
    finished = false;
    interval = setInterval(() => {
      remaining -= 1;
      if (remaining <= 0) {
        remaining = 0;
        running = false;
        finished = true;
        clearInterval(interval!);
        triggerConfetti();
      }
    }, 1000);
  }

  function pause() {
    running = false;
    if (interval) clearInterval(interval);
  }

  function reset() {
    running = false;
    finished = false;
    if (interval) clearInterval(interval);
    totalSeconds = inputMinutes * 60;
    remaining = totalSeconds;
  }

  function setTimer() {
    if (running) return;
    totalSeconds = inputMinutes * 60;
    remaining = totalSeconds;
    finished = false;
  }

  // --- Confetti burst on finish ---
  function triggerConfetti() {
    const container = document.getElementById('confetti-container');
    if (!container) return;
    container.innerHTML = '';
    const colors = ['#f76e6e', '#6ee7f7', '#f7c76e', '#a78bfa', '#34d399'];
    for (let i = 0; i < 60; i++) {
      const el = document.createElement('div');
      el.className = 'confetti-piece';
      el.style.cssText = `
        left: ${Math.random() * 100}%;
        background: ${colors[Math.floor(Math.random() * colors.length)]};
        animation-delay: ${Math.random() * 0.5}s;
        animation-duration: ${0.8 + Math.random() * 1.2}s;
        width: ${6 + Math.random() * 8}px;
        height: ${6 + Math.random() * 8}px;
        border-radius: ${Math.random() > 0.5 ? '50%' : '2px'};
      `;
      container.appendChild(el);
    }
    setTimeout(() => { if (container) container.innerHTML = ''; }, 3000);
  }

  onDestroy(() => {
    if (interval) clearInterval(interval);
  });
</script>

<!-- Confetti container -->
<div id="confetti-container" class="confetti-container"></div>

<div class="card">
  <!-- Emoji mood -->
  <div class="mood" class:bounce={running} class:celebrate={finished}>
    {mood}
  </div>

  <h1 class="title">
    {#if finished}
      Tempo scaduto!
    {:else if running}
      Dai, ci siamo quasi...
    {:else}
      Timer Cosmico ✨
    {/if}
  </h1>

  <!-- SVG ring -->
  <div class="ring-wrapper">
    <svg viewBox="0 0 200 200" class="ring-svg">
      <!-- Background circle -->
      <circle
        cx="100" cy="100" r="90"
        fill="none"
        stroke="rgba(255,255,255,0.07)"
        stroke-width="12"
      />
      <!-- Progress circle -->
      <circle
        cx="100" cy="100" r="90"
        fill="none"
        stroke={ringColor}
        stroke-width="12"
        stroke-linecap="round"
        stroke-dasharray={circumference}
        stroke-dashoffset={dashOffset}
        transform="rotate(-90 100 100)"
        style="transition: stroke-dashoffset 0.9s ease, stroke 0.5s ease; filter: drop-shadow(0 0 8px {ringColor});"
      />
    </svg>

    <!-- Time display -->
    <div class="time-display">
      <span class="digits">{String(displayMin).padStart(2, '0')}</span>
      <span class="colon" class:blink={running}>:</span>
      <span class="digits">{String(displaySec).padStart(2, '0')}</span>
    </div>
  </div>

  <!-- Input -->
  <div class="input-row">
    <label for="min-input">Minuti:</label>
    <input
      id="min-input"
      type="number"
      min="1"
      max="99"
      bind:value={inputMinutes}
      on:change={setTimer}
      disabled={running}
    />
  </div>

  <!-- Buttons -->
  <div class="buttons">
    {#if !running}
      <button class="btn btn-start" on:click={start} disabled={remaining === 0}>
        ▶ Start
      </button>
    {:else}
      <button class="btn btn-pause" on:click={pause}>
        ⏸ Pausa
      </button>
    {/if}
    <button class="btn btn-reset" on:click={reset}>
      🔄 Reset
    </button>
  </div>

  <!-- Fun message -->
  {#if finished}
    <p class="fun-msg">🎊 Ottimo lavoro! Prenditi una pausa!</p>
  {:else if running && remaining <= 10}
    <p class="fun-msg urgent">⚠️ Ultimi {remaining} secondi!</p>
  {:else if running}
    <p class="fun-msg">⏳ Concentrati... puoi farcela!</p>
  {:else}
    <p class="fun-msg">👆 Imposta i minuti e premi Start!</p>
  {/if}
</div>

<style>
  .confetti-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 100;
    overflow: hidden;
  }

  :global(.confetti-piece) {
    position: absolute;
    top: -20px;
    animation: confettiFall linear forwards;
  }

  @keyframes confettiFall {
    0%   { transform: translateY(0) rotate(0deg); opacity: 1; }
    100% { transform: translateY(110vh) rotate(720deg); opacity: 0; }
  }

  .card {
    position: relative;
    z-index: 1;
    background: rgba(255, 255, 255, 0.05);
    backdrop-filter: blur(20px);
    border: 1px solid rgba(255, 255, 255, 0.12);
    border-radius: 2rem;
    padding: 2.5rem 2rem;
    width: min(420px, 92vw);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1.4rem;
    box-shadow:
      0 8px 32px rgba(0,0,0,0.4),
      inset 0 1px 0 rgba(255,255,255,0.1);
  }

  .mood {
    font-size: 3.5rem;
    line-height: 1;
    transition: transform 0.3s;
  }

  .mood.bounce {
    animation: moodBounce 1s ease-in-out infinite;
  }

  .mood.celebrate {
    animation: moodCelebrate 0.5s ease-in-out infinite alternate;
  }

  @keyframes moodBounce {
    0%, 100% { transform: translateY(0); }
    50%       { transform: translateY(-8px); }
  }

  @keyframes moodCelebrate {
    from { transform: scale(1) rotate(-10deg); }
    to   { transform: scale(1.2) rotate(10deg); }
  }

  .title {
    color: #fff;
    font-size: 1.3rem;
    font-weight: 700;
    text-align: center;
    letter-spacing: -0.02em;
  }

  .ring-wrapper {
    position: relative;
    width: 200px;
    height: 200px;
  }

  .ring-svg {
    width: 100%;
    height: 100%;
  }

  .time-display {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 2px;
  }

  .digits {
    font-size: 2.8rem;
    font-weight: 700;
    color: #fff;
    letter-spacing: -0.04em;
    font-variant-numeric: tabular-nums;
  }

  .colon {
    font-size: 2.8rem;
    font-weight: 700;
    color: #fff;
    line-height: 1;
    margin-bottom: 4px;
  }

  .colon.blink {
    animation: blink 1s step-end infinite;
  }

  @keyframes blink {
    0%, 100% { opacity: 1; }
    50%       { opacity: 0.2; }
  }

  .input-row {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    color: rgba(255,255,255,0.7);
    font-size: 0.95rem;
  }

  input[type="number"] {
    width: 72px;
    padding: 0.4rem 0.6rem;
    background: rgba(255,255,255,0.1);
    border: 1px solid rgba(255,255,255,0.2);
    border-radius: 0.6rem;
    color: #fff;
    font-family: inherit;
    font-size: 1rem;
    font-weight: 600;
    text-align: center;
    outline: none;
    transition: border-color 0.2s;
  }

  input[type="number"]:focus {
    border-color: #6ee7f7;
  }

  input[type="number"]:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  /* Hide number arrows */
  input[type=number]::-webkit-inner-spin-button,
  input[type=number]::-webkit-outer-spin-button { -webkit-appearance: none; }
  input[type=number] { -moz-appearance: textfield; }

  .buttons {
    display: flex;
    gap: 0.75rem;
  }

  .btn {
    padding: 0.65rem 1.4rem;
    border: none;
    border-radius: 0.8rem;
    font-family: inherit;
    font-size: 0.95rem;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.15s, box-shadow 0.15s, opacity 0.15s;
  }

  .btn:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(0,0,0,0.3);
  }

  .btn:active:not(:disabled) {
    transform: translateY(0);
  }

  .btn:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }

  .btn-start {
    background: linear-gradient(135deg, #6ee7f7, #a78bfa);
    color: #0f0f1a;
  }

  .btn-pause {
    background: linear-gradient(135deg, #f7c76e, #f76e6e);
    color: #0f0f1a;
  }

  .btn-reset {
    background: rgba(255,255,255,0.12);
    color: #fff;
    border: 1px solid rgba(255,255,255,0.2);
  }

  .fun-msg {
    font-size: 0.9rem;
    color: rgba(255,255,255,0.6);
    text-align: center;
  }

  .fun-msg.urgent {
    color: #f76e6e;
    font-weight: 600;
    animation: pulse 0.6s ease-in-out infinite alternate;
  }

  @keyframes pulse {
    from { opacity: 0.6; }
    to   { opacity: 1; }
  }
</style>
