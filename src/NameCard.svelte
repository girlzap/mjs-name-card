<script>
  import { onMount } from 'svelte';
  import { fly, fade } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';

  let mounted = false;
  let pathEl;
  let flipped = false;

  // Paint flecks — random but seeded-ish for consistency
  const flecks = Array.from({ length: 14 }, (_, i) => {
    const seed = (i * 137.508) % 1;
    const seed2 = ((i + 3) * 97.123) % 1;
    const seed3 = ((i + 7) * 61.803) % 1;
    const colors = [
      'rgba(212, 172, 110, 0.7)',
      'rgba(192, 57, 43, 0.6)',
      'rgba(26, 82, 118, 0.8)',
      'rgba(108, 52, 131, 0.7)',
      'rgba(14, 102, 85, 0.65)',
      'rgba(232, 147, 122, 0.6)',
      'rgba(183, 119, 13, 0.65)',
    ];
    return {
      x: seed * 100,
      y: seed2 * 100,
      size: 2 + seed3 * 5,
      color: colors[i % colors.length],
      delay: seed * 4000,
      duration: 6000 + seed2 * 8000,
      driftX: (seed - 0.5) * 30,
      driftY: (seed2 - 0.5) * 30,
    };
  });

  const asciiArt =
`⠀⠀⠀⠀⠀⠀⠀⢀⣀⡤⠴⠶⠶⠒⠲⠦⢤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⢀⡠⠞⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠲⠤⣄⡀⠀⠀⠀⠀⠀
⠀⠀⣀⡴⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣤⡿⠀⠀⠀⠀⠀
⠀⢾⣅⡀⠀⠀⠀⠀⣀⠀⠀⠀⠀⠀⠀⢀⡦⠤⠄⠀⠀⢻⡀⠀⠀⠀⠀⠀
⠀⠈⢹⡏⠀⠀⠐⠋⠉⠁⠀⠻⢿⠟⠁⠀⠀⢤⠀⠀⠠⠤⢷⣤⣤⢤⡄⠀
⠀⠀⣼⡤⠤⠀⠀⠘⣆⡀⠀⣀⡼⠦⣄⣀⡤⠊⠀⠀⠀⠤⣼⠟⠀⠀⢹⡂
⠀⠊⣿⡠⠆⠀⠀⠀⠈⠉⠉⠙⠤⠤⠋⠀⠀⠀⠀⠀⠀⡰⠋⠀⠀⠀⡼⠁
⠀⢀⡾⣧⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠜⠁⠀⠀⠀⣸⠁⠀
⠀⠀⠀⡼⠙⠢⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⠃⠀⠀
⠀⢀⡞⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⠃⠀⠀⠀
⠀⡼⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀
⣾⠁⠀⢀⣠⡴⠆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀
⠈⠛⠻⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀`;

  const paletteColors = [
    { hex: '#c0392b', label: 'rose'  },
    { hex: '#6c3483', label: 'violet'},
    { hex: '#1a5276', label: 'cerulean'},
    { hex: '#0e6655', label: 'teal'  },
    { hex: '#b7770d', label: 'amber' },
    { hex: '#e8937a', label: 'peach' },
    { hex: '#d4ac6e', label: 'gold'  },
  ];

  onMount(() => {
    mounted = true;
  });

  // 3D tilt on hover — disabled while flipped
  let sceneEl;
  let tiltX = 0;
  let tiltY = 0;
  let lifted = false;

  function handleMouseMove(e) {
    if (flipped) return;
    const rect = sceneEl.getBoundingClientRect();
    const cx = rect.left + rect.width / 2;
    const cy = rect.top + rect.height / 2;
    const dx = (e.clientX - cx) / (rect.width / 2);
    const dy = (e.clientY - cy) / (rect.height / 2);
    tiltX = dy * -10;
    tiltY = dx * 10;
  }

  function handleMouseLeave() {
    tiltX = 0;
    tiltY = 0;
    lifted = false;
  }

  function handleMouseEnter() {
    if (!flipped) lifted = true;
  }

  // Floating hearts
  let particles = [];

  const emojiSet = ['✨', '🌟', '⭐', '💫', '🌸', '🌺', '🌼', '🌻'];

  function spawnParticles() {
    const count = 14;
    const batch = Array.from({ length: count }, (_, i) => {
      const size = 32 + Math.random() * 40;         // big: 32–72px
      const duration = 2800 + Math.random() * 2200; // 2.8–5s
      const delay = i * 120 + Math.random() * 200;  // staggered cascade
      const startX = (Math.random() - 0.5) * 700;   // spread across card width
      const startY = -220 - Math.random() * 40;     // just below the top edge of the card (~270px above center)
      const rise = 600 + Math.random() * 300;       // rise well above the card
      const sway = (Math.random() - 0.5) * 100;     // gentle horizontal drift
      const emoji = emojiSet[Math.floor(Math.random() * emojiSet.length)];
      return { id: Date.now() + i, startX, startY, rise, sway, size, duration, delay, emoji };
    });
    particles = [...particles, ...batch];
    const maxWait = Math.max(...batch.map(p => p.duration + p.delay)) + 100;
    setTimeout(() => {
      const ids = new Set(batch.map(p => p.id));
      particles = particles.filter(p => !ids.has(p.id));
    }, maxWait);
  }

  function handleClick() {
    flipped = !flipped;
    tiltX = 0;
    tiltY = 0;
    lifted = false;
    spawnParticles();
  }

  $: sceneTransform = `rotateX(${tiltX}deg) rotateY(${tiltY}deg) translateY(${lifted ? '-6px' : '0'})`;
  $: flipTransform  = flipped ? 'rotateY(180deg)' : 'rotateY(0deg)';
</script>

<!-- Scene: perspective + tilt + sizing -->
<div
  class="card-scene"
  bind:this={sceneEl}
  on:mousemove={handleMouseMove}
  on:mouseleave={handleMouseLeave}
  on:mouseenter={handleMouseEnter}
  on:click={handleClick}
  on:keydown={(e) => e.key === 'Enter' && handleClick()}
  role="button"
  tabindex="0"
  aria-label="Name card for MJ, Design Engineer. Click to flip."
  style="transform: {sceneTransform};"
>
  <!-- Explosion particles — rendered outside the flipper so they escape the card -->
  <div class="particles-layer" aria-hidden="true">
    {#each particles as p (p.id)}
      <span
        class="particle"
        style="
          --sx: {p.startX}px;
          --sy: {p.startY}px;
          --rise: {p.rise}px;
          --sway: {p.sway}px;
          font-size: {p.size}px;
          animation-duration: {p.duration}ms;
          animation-delay: {p.delay}ms;
        "
      >{p.emoji}</span>
    {/each}
  </div>

  <!-- Flipper: rotates 180° on click -->
  <div class="card-flip" style="transform: {flipTransform};">

    <!-- ── FRONT ────────────────────────────────────────── -->
    <div class="card-face card-front">
      <div class="flecks" aria-hidden="true">
        {#each flecks as f}
          <span
            class="fleck"
            style="
              left: {f.x}%;
              top: {f.y}%;
              width: {f.size}px;
              height: {f.size}px;
              background: {f.color};
              animation-delay: {f.delay}ms;
              animation-duration: {f.duration}ms;
              --drift-x: {f.driftX}px;
              --drift-y: {f.driftY}px;
            "
          ></span>
        {/each}
      </div>

      <div class="inner-glow" aria-hidden="true"></div>

      <div class="card-content">
        {#if mounted}
          <div class="name-block" in:fly={{ y: 20, duration: 900, delay: 100, easing: cubicOut }}>
            <h1 class="name">MJ</h1>
          </div>

          <div class="divider-wrap" in:fade={{ duration: 600, delay: 600 }}>
            <svg class="brushstroke" viewBox="0 0 260 18" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path
                bind:this={pathEl}
                d="M4 9 C 30 4, 60 14, 90 9 C 120 4, 150 13, 180 8 C 210 3, 235 12, 256 9"
                stroke="#d4ac6e"
                stroke-width="1.5"
                stroke-linecap="round"
                class="stroke-path"
              />
            </svg>
          </div>

          <div in:fly={{ y: 14, duration: 800, delay: 500, easing: cubicOut }}>
            <p class="title">Design Engineer</p>
          </div>
        {/if}
      </div>

      <div class="corner corner-tl" aria-hidden="true"></div>
      <div class="corner corner-br" aria-hidden="true"></div>
      <div class="flip-hint" aria-hidden="true">click to flip</div>
    </div>

    <!-- ── BACK ─────────────────────────────────────────── -->
    <div class="card-face card-back">
      <div class="inner-glow inner-glow-back" aria-hidden="true"></div>

      <div class="back-content">
        <!-- Magnetic stripe parody -->
        <div class="mag-stripe" aria-hidden="true"></div>

        <div class="back-body">
          <!-- Signature strip -->
          <div class="sig-row">
            <span class="sig-label">AUTHORIZED SIGNATURE</span>
            <div class="sig-strip">
              <span class="sig-name">mj</span>
            </div>
          </div>

          <!-- ASCII art: painter's palette -->
          <pre class="ascii-art" aria-label="ASCII art of a painter's palette">{asciiArt}</pre>

          <!-- Color palette row -->
          <div class="palette-row" aria-label="Color palette">
            {#each paletteColors as c}
              <div class="palette-dot" style="background: {c.hex};" title={c.label}></div>
            {/each}
          </div>

          <p class="valid-thru">VALID THRU: indefinitely &nbsp;·&nbsp; EST. impressionist era</p>
        </div>
      </div>

      <div class="corner corner-tl" aria-hidden="true"></div>
      <div class="corner corner-br" aria-hidden="true"></div>
      <div class="flip-hint" aria-hidden="true">click to flip</div>
    </div>

  </div>
</div>

<style>
  /* ─── Floating hearts ────────────────────────────────────── */
  .particles-layer {
    position: absolute;
    top: 50%;
    left: 50%;
    pointer-events: none;
    z-index: 0; /* behind the card */
  }

  .particle {
    position: absolute;
    animation: float-up ease-in-out both;
    line-height: 1;
    user-select: none;
    will-change: transform, opacity;
  }

  @keyframes float-up {
    0% {
      transform: translate(calc(var(--sx) - 50%), calc(var(--sy) - 50%)) scale(0.7);
      opacity: 0;
    }
    12% {
      opacity: 0.85;
    }
    50% {
      transform:
        translate(
          calc(var(--sx) - 50% + var(--sway) * 0.5),
          calc(var(--sy) - 50% - var(--rise) * 0.5)
        ) scale(1);
      opacity: 0.75;
    }
    85% {
      opacity: 0.5;
    }
    100% {
      transform:
        translate(
          calc(var(--sx) - 50% + var(--sway)),
          calc(var(--sy) - 50% - var(--rise))
        ) scale(0.85);
      opacity: 0;
    }
  }

  /* ─── Scene (perspective + tilt) ────────────────────────── */
  .card-scene {
    width: min(856px, 90vw);
    aspect-ratio: 856 / 540;
    perspective: 1000px;
    cursor: pointer;
    will-change: transform;
    transition: transform 0.15s ease;
  }

  /* ─── Flipper ────────────────────────────────────────────── */
  .card-flip {
    width: 100%;
    height: 100%;
    position: relative;
    z-index: 1; /* above the hearts */
    transform-style: preserve-3d;
    transition: transform 0.7s cubic-bezier(0.4, 0.2, 0.2, 1);
  }

  /* ─── Shared face styles ─────────────────────────────────── */
  .card-face {
    position: absolute;
    inset: 0;
    border-radius: 16px;
    /* overflow: hidden removed — it breaks backface-visibility in Safari
       when combined with backdrop-filter. Clipping is handled by children. */
    backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
    will-change: transform;

    background: linear-gradient(
      135deg,
      rgba(255, 255, 255, 0.07) 0%,
      rgba(255, 255, 255, 0.02) 50%,
      rgba(0, 0, 0, 0.15) 100%
    );
    backdrop-filter: blur(28px) saturate(1.4);
    -webkit-backdrop-filter: blur(28px) saturate(1.4);

    border: 1px solid rgba(255, 255, 255, 0.1);
    box-shadow:
      0 0 0 0.5px rgba(255, 255, 255, 0.05) inset,
      0 8px 60px rgba(0, 0, 0, 0.6),
      0 2px 12px rgba(0, 0, 0, 0.4);
  }

  .card-scene:hover .card-face {
    box-shadow:
      0 0 0 0.5px rgba(255, 255, 255, 0.08) inset,
      0 20px 80px rgba(0, 0, 0, 0.7),
      0 4px 20px rgba(212, 172, 110, 0.12);
  }

  /* Explicit rotateY(0) forces correct GPU layer on front face in Safari */
  .card-front {
    transform: rotateY(0deg);
  }

  /* Back face is pre-rotated 180° */
  .card-back {
    transform: rotateY(180deg);
  }

  /* ─── Inner glow ─────────────────────────────────────────── */
  .inner-glow {
    position: absolute;
    inset: 0;
    background: radial-gradient(ellipse at 30% 40%, rgba(212, 172, 110, 0.08) 0%, transparent 60%);
    pointer-events: none;
    z-index: 1;
  }
  .inner-glow-back {
    background: radial-gradient(ellipse at 70% 60%, rgba(108, 52, 131, 0.1) 0%, transparent 60%);
  }

  /* ─── Floating paint flecks ──────────────────────────────── */
  .flecks {
    position: absolute;
    inset: 0;
    pointer-events: none;
    z-index: 0;
  }
  .fleck {
    position: absolute;
    border-radius: 50%;
    animation: fleck-drift linear infinite;
  }
  @keyframes fleck-drift {
    0%, 100% { transform: translate(0, 0) scale(1); opacity: 0.6; }
    25%       { transform: translate(calc(var(--drift-x) * 0.5), calc(var(--drift-y) * 0.8)) scale(1.2); opacity: 0.9; }
    50%       { transform: translate(var(--drift-x), var(--drift-y)) scale(0.8); opacity: 0.5; }
    75%       { transform: translate(calc(var(--drift-x) * 0.3), calc(var(--drift-y) * 0.4)) scale(1.1); opacity: 0.8; }
  }

  /* ─── Corner accents ─────────────────────────────────────── */
  .corner {
    position: absolute;
    width: 22px;
    height: 22px;
    border-color: rgba(212, 172, 110, 0.35);
    border-style: solid;
    z-index: 3;
    pointer-events: none;
  }
  .corner-tl { top: 20px; left: 20px; border-width: 1px 0 0 1px; border-radius: 3px 0 0 0; }
  .corner-br { bottom: 20px; right: 20px; border-width: 0 1px 1px 0; border-radius: 0 0 3px 0; }

  /* ─── Flip hint ──────────────────────────────────────────── */
  .flip-hint {
    position: absolute;
    bottom: 14px;
    left: 50%;
    transform: translateX(-50%);
    font-family: 'Josefin Sans', system-ui, sans-serif;
    font-size: 9px;
    letter-spacing: 0.3em;
    text-transform: uppercase;
    color: rgba(212, 172, 110, 0.35);
    z-index: 4;
    pointer-events: none;
    user-select: none;
  }

  /* ─── Front content ──────────────────────────────────────── */
  .card-content {
    position: relative;
    z-index: 2;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: clamp(8px, 2%, 18px);
    padding: 10%;
    border-radius: 16px;
    overflow: hidden;
  }

  .name-block { text-align: center; }

  .name {
    font-family: 'Lobster Two', cursive;
    font-size: clamp(52px, 9vw, 110px);
    font-weight: 700;
    letter-spacing: 0.08em;
    color: #f5e6c8;
    line-height: 1;
    text-shadow:
      0 0 40px rgba(212, 172, 110, 0.45),
      0 0 80px rgba(212, 172, 110, 0.18),
      0 2px 8px rgba(0, 0, 0, 0.5);
  }

  .divider-wrap {
    width: 55%;
    min-width: 160px;
    max-width: 300px;
  }
  .brushstroke { width: 100%; height: auto; overflow: visible; }
  .stroke-path {
    stroke-dasharray: 400;
    stroke-dashoffset: 400;
    animation: draw-stroke 1.4s cubic-bezier(0.4, 0, 0.2, 1) 0.7s forwards;
  }
  @keyframes draw-stroke { to { stroke-dashoffset: 0; } }

  .title {
    font-family: 'Josefin Sans', system-ui, sans-serif;
    font-size: clamp(11px, 1.8vw, 17px);
    font-weight: 400;
    letter-spacing: 0.45em;
    text-transform: uppercase;
    color: #d4ac6e;
    opacity: 0.85;
  }

  /* ─── Back content ───────────────────────────────────────── */
  .back-content {
    position: relative;
    z-index: 2;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    border-radius: 16px;
    overflow: hidden;
  }

  .mag-stripe {
    width: 100%;
    height: 13%;
    margin-top: 12%;
    background: linear-gradient(
      180deg,
      rgba(10, 10, 15, 0.95) 0%,
      rgba(20, 15, 25, 0.9) 100%
    );
    border-top: 1px solid rgba(255,255,255,0.04);
    border-bottom: 1px solid rgba(255,255,255,0.04);
  }

  .back-body {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    padding: 4% 8% 10%;
    gap: 4px;
  }

  .sig-row {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }
  .sig-label {
    font-family: 'Josefin Sans', system-ui, sans-serif;
    font-size: clamp(7px, 1vw, 10px);
    letter-spacing: 0.2em;
    text-transform: uppercase;
    color: rgba(212, 172, 110, 0.5);
  }
  .sig-strip {
    background: rgba(245, 230, 200, 0.08);
    border: 1px solid rgba(212, 172, 110, 0.2);
    border-radius: 3px;
    padding: 4px 10px;
    width: 40%;
    min-width: 80px;
  }
  .sig-name {
    font-family: 'Lobster Two', cursive;
    font-size: clamp(16px, 2.5vw, 28px);
    color: #f5e6c8;
    opacity: 0.9;
  }

  .ascii-art {
    font-family: 'Courier New', Courier, monospace;
    font-size: clamp(7px, 1.1vw, 11px);
    line-height: 1.5;
    color: rgba(212, 172, 110, 0.65);
    white-space: pre;
    margin: 0;
    letter-spacing: 0.05em;
  }

  .palette-row {
    display: flex;
    gap: clamp(4px, 1vw, 10px);
    align-items: center;
  }
  .palette-dot {
    width: clamp(8px, 1.4vw, 14px);
    height: clamp(8px, 1.4vw, 14px);
    border-radius: 50%;
    opacity: 0.85;
    box-shadow: 0 0 6px rgba(0,0,0,0.4);
  }

  .valid-thru {
    font-family: 'Josefin Sans', system-ui, sans-serif;
    font-size: clamp(7px, 1vw, 10px);
    letter-spacing: 0.18em;
    text-transform: uppercase;
    color: rgba(212, 172, 110, 0.4);
  }
</style>
