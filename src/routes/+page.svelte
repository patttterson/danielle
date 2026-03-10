<script lang="ts">
  import tetrio from '$lib/assets/tetrio-purple.png';
  import taws from '$lib/assets/taws_recolored.png';
  import '@fortawesome/fontawesome-free/css/all.min.css';
  import Schedule from '$lib/Schedule.svelte';

</script>

<div class="banner">
  <div class="banner-left">
    <img src={tetrio} alt="TETR.IO" class="tetrio-logo">
    <div class="subtitle">Amateur Weeklies</div>
  </div>

  <div class="banner-center">
    <img src={taws} alt="icon" class="center-icon">
  </div>

  <div class="banner-right">
    <button class="social-btn" aria-label="TAWS Discord">
      <i class="fab fa-discord social-icon"></i>
      Discord
    </button>
    <button class="social-btn" aria-label="TAWS Twitch">
      <i class="fab fa-twitch social-icon"></i>
      Live
    </button>
    <button class="social-btn" aria-label="TAWS Youtube">
      <i class="fab fa-youtube social-icon"></i>
      VODs
    </button>
  </div>
</div>

<div class="schedule-wrapper">
  <Schedule />
</div>

<style>
  .banner {
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    align-items: center;
    width: 100%;
    box-sizing: border-box;
    margin: 0 auto;
    background: none;
    min-height: 90px;
    padding: 10px 20px;
    position: relative;
    overflow: visible;
    gap: 16px;
  }

  .banner-left {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: 0;
    flex: 0 0 auto;
  }

  .tetrio-logo {
    width: 160px;
    height: auto;
  }

  .subtitle {
    font-family: var(--font-heading);
    font-size: 1.6rem;
    font-weight: 800;
    font-style: italic;
    color: #fff;
    text-transform: uppercase;
    letter-spacing: 0.04em;
    text-shadow: 2px 2px 0 #4f449c;
    white-space: nowrap;
  }

  .banner-center {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .center-icon {
    width: 110px;
    height: 110px;
    object-fit: contain;
    border-radius: 6px;
  }

  .banner-right {
    display: flex;
    gap: 8px;
    align-items: center;
    justify-content: flex-start;
  }

  .social-btn {
    border: none;
    border-radius: 6px;
    background: rgba(255, 255, 255, 0.25);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    border: 1px solid rgba(255, 255, 255, 0.45);
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    padding: 10px 16px;
    gap: 8px;
    box-shadow: 0 2px 12px rgba(0,0,0,0.15);
    transition: filter 0.15s, background 0.15s;
    color: #fff;
    font-family: var(--font-heading);
    font-size: 0.85rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    white-space: nowrap;
    text-shadow: 0 1px 3px rgba(0,0,0,0.3);
  }

  .social-btn:hover {
    background: rgba(255, 255, 255, 0.35);
    filter: brightness(1.05);
  }

  .schedule-wrapper {
    background: rgba(0, 0, 0, 0.45);
    backdrop-filter: blur(2px);
    -webkit-backdrop-filter: blur(2px);
    border-radius: 12px;
    margin-bottom: 20px;
  }

  .social-icon {
    font-size: 28px;
    line-height: 1;
  }
</style>