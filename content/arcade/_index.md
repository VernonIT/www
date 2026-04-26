---
title: Arcade
type: page
---

<style>
.arcade-hero {
  text-align: center;
  padding: 2rem 0 3rem;
}
.arcade-hero h1 {
  font-size: 2.5rem;
  font-weight: 800;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  background: linear-gradient(135deg, #006699, #00aacc);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 0.5rem;
}
.arcade-hero p {
  color: #888;
  font-size: 1rem;
}
.arcade-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
  max-width: 960px;
  margin: 0 auto;
  padding: 0 1rem 4rem;
}
.arcade-card {
  display: block;
  text-decoration: none;
  border-radius: 16px;
  overflow: hidden;
  border: 1px solid rgba(0, 102, 153, 0.2);
  background: rgba(0, 102, 153, 0.04);
  transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease;
}
.arcade-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 32px rgba(0, 102, 153, 0.15);
  border-color: rgba(0, 102, 153, 0.5);
  text-decoration: none;
}
.arcade-card-thumb {
  width: 100%;
  aspect-ratio: 16/9;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 5rem;
  background: #0a0a12;
}
.arcade-card-body {
  padding: 1.25rem 1.25rem 1.5rem;
}
.arcade-card-title {
  font-size: 1.1rem;
  font-weight: 700;
  margin: 0 0 0.4rem;
  color: inherit;
  letter-spacing: 0.03em;
}
.arcade-card-desc {
  font-size: 0.875rem;
  color: #888;
  margin: 0 0 1rem;
  line-height: 1.5;
}
.arcade-card-play {
  display: inline-block;
  padding: 0.4rem 1.1rem;
  border-radius: 8px;
  background: #006699;
  color: #fff !important;
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  text-decoration: none;
}
</style>

<div class="arcade-hero">
  <h1>Arcade</h1>
  <p>Four games. No install. Play in your browser.</p>
</div>

<div class="arcade-grid">

  <a class="arcade-card" href="/arcade/missile_command.html" target="_blank" rel="noopener">
    <div class="arcade-card-thumb">🚀</div>
    <div class="arcade-card-body">
      <div class="arcade-card-title">Missile Command</div>
      <p class="arcade-card-desc">Defend your cities from waves of incoming missiles. Click to fire interceptors before impact.</p>
      <span class="arcade-card-play">Play</span>
    </div>
  </a>

  <a class="arcade-card" href="/arcade/tower-defense.html" target="_blank" rel="noopener">
    <div class="arcade-card-thumb">🏰</div>
    <div class="arcade-card-body">
      <div class="arcade-card-title">Iron Bastion</div>
      <p class="arcade-card-desc">Build and upgrade towers to stop relentless enemy waves from reaching your base.</p>
      <span class="arcade-card-play">Play</span>
    </div>
  </a>

  <a class="arcade-card" href="/arcade/dragon-snake.html" target="_blank" rel="noopener">
    <div class="arcade-card-thumb">🐉</div>
    <div class="arcade-card-body">
      <div class="arcade-card-title">Dragon Snake</div>
      <p class="arcade-card-desc">Pick your spirit animal and hunt exotic fruit. Find the glowing gap to warp through walls.</p>
      <span class="arcade-card-play">Play</span>
    </div>
  </a>

  <a class="arcade-card" href="/arcade/neon-sprint.html" target="_blank" rel="noopener">
    <div class="arcade-card-thumb">🏃</div>
    <div class="arcade-card-body">
      <div class="arcade-card-title">Neon Sprint</div>
      <p class="arcade-card-desc">Run forever through a neon cityscape. Jump, double-jump, and dodge obstacles as the city blurs past.</p>
      <span class="arcade-card-play">Play</span>
    </div>
  </a>

</div>
