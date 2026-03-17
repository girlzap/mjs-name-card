<script>
  import './app.css';
  import NameCard from './NameCard.svelte';
</script>

<!-- Painterly background canvas -->
<div class="canvas">
  <!-- SVG noise filter definition -->
  <svg width="0" height="0" style="position:absolute">
    <defs>
      <filter id="noise" x="0%" y="0%" width="100%" height="100%">
        <feTurbulence
          type="fractalNoise"
          baseFrequency="0.65"
          numOctaves="3"
          stitchTiles="stitch"
        />
        <feColorMatrix type="saturate" values="0" />
        <feBlend in="SourceGraphic" mode="multiply" />
      </filter>
    </defs>
  </svg>

  <!-- Drifting color orbs — the impressionist light -->
  <div class="orb orb-1"></div>
  <div class="orb orb-2"></div>
  <div class="orb orb-3"></div>
  <div class="orb orb-4"></div>
  <div class="orb orb-5"></div>

  <!-- Noise grain overlay -->
  <div class="grain"></div>

  <NameCard />
</div>

<style>
  .canvas {
    position: relative;
    width: 100%;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    background: #0d0f14;
  }

  /* Each orb is a large soft radial gradient blob */
  .orb {
    position: absolute;
    border-radius: 50%;
    filter: blur(80px);
    opacity: 0.55;
    will-change: transform;
  }

  .orb-1 {
    width: 700px;
    height: 700px;
    background: radial-gradient(circle, #1a5276 0%, transparent 70%);
    top: -20%;
    left: -15%;
    animation: drift1 22s ease-in-out infinite alternate;
  }

  .orb-2 {
    width: 600px;
    height: 600px;
    background: radial-gradient(circle, #6c3483 0%, transparent 70%);
    bottom: -10%;
    right: -10%;
    animation: drift2 18s ease-in-out infinite alternate;
  }

  .orb-3 {
    width: 500px;
    height: 500px;
    background: radial-gradient(circle, #b7770d 0%, transparent 70%);
    top: 30%;
    left: 55%;
    opacity: 0.4;
    animation: drift3 26s ease-in-out infinite alternate;
  }

  .orb-4 {
    width: 450px;
    height: 450px;
    background: radial-gradient(circle, #c0392b 0%, transparent 70%);
    bottom: 10%;
    left: 5%;
    opacity: 0.35;
    animation: drift4 20s ease-in-out infinite alternate;
  }

  .orb-5 {
    width: 380px;
    height: 380px;
    background: radial-gradient(circle, #0e6655 0%, transparent 70%);
    top: 5%;
    right: 20%;
    opacity: 0.45;
    animation: drift5 24s ease-in-out infinite alternate;
  }

  .grain {
    position: absolute;
    inset: -50%;
    width: 200%;
    height: 200%;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='1'/%3E%3C/svg%3E");
    background-size: 256px 256px;
    opacity: 0.06;
    pointer-events: none;
    animation: grain-shift 0.5s steps(1) infinite;
  }

  @keyframes drift1 {
    0%   { transform: translate(0px, 0px) scale(1); }
    100% { transform: translate(120px, 80px) scale(1.15); }
  }
  @keyframes drift2 {
    0%   { transform: translate(0px, 0px) scale(1); }
    100% { transform: translate(-100px, -60px) scale(1.1); }
  }
  @keyframes drift3 {
    0%   { transform: translate(0px, 0px) scale(1); }
    100% { transform: translate(-80px, 100px) scale(0.9); }
  }
  @keyframes drift4 {
    0%   { transform: translate(0px, 0px) scale(1); }
    100% { transform: translate(90px, -70px) scale(1.2); }
  }
  @keyframes drift5 {
    0%   { transform: translate(0px, 0px) scale(1); }
    100% { transform: translate(-60px, 90px) scale(0.85); }
  }

  @keyframes grain-shift {
    0%   { transform: translate(0, 0); }
    10%  { transform: translate(-2%, -3%); }
    20%  { transform: translate(3%, 1%); }
    30%  { transform: translate(-1%, 4%); }
    40%  { transform: translate(4%, -2%); }
    50%  { transform: translate(-3%, 3%); }
    60%  { transform: translate(2%, -4%); }
    70%  { transform: translate(-4%, 1%); }
    80%  { transform: translate(1%, 3%); }
    90%  { transform: translate(3%, -1%); }
    100% { transform: translate(0, 0); }
  }
</style>
