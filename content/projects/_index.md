---
title: Projects
type: page
---

<style>
.projects-hero {
  text-align: center;
  padding: 2rem 0 3rem;
}
.projects-hero h1 {
  font-size: 2.5rem;
  font-weight: 800;
  letter-spacing: 0.03em;
  background: linear-gradient(135deg, #006699, #00aacc);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 0.5rem;
}
.projects-hero p {
  color: #888;
  font-size: 1rem;
}
.projects-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
  max-width: 960px;
  margin: 0 auto;
  padding: 0 1rem 4rem;
}
.project-card {
  display: block;
  text-decoration: none;
  border-radius: 16px;
  overflow: hidden;
  border: 1px solid rgba(0, 102, 153, 0.2);
  background: rgba(0, 102, 153, 0.04);
  transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease;
}
.project-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 32px rgba(0, 102, 153, 0.15);
  border-color: rgba(0, 102, 153, 0.5);
  text-decoration: none;
}
.project-card-thumb {
  width: 100%;
  aspect-ratio: 16/9;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 5rem;
  background: #0a0a12;
}
.project-card-body {
  padding: 1.25rem 1.25rem 1.5rem;
}
.project-card-title {
  font-size: 1.1rem;
  font-weight: 700;
  margin: 0 0 0.4rem;
  color: inherit;
  letter-spacing: 0.03em;
}
.project-card-desc {
  font-size: 0.875rem;
  color: #888;
  margin: 0 0 1rem;
  line-height: 1.5;
}
.project-card-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
}
.project-tag {
  display: inline-block;
  padding: 0.2rem 0.6rem;
  border-radius: 6px;
  background: rgba(0, 102, 153, 0.12);
  color: #006699;
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 0.04em;
  text-transform: uppercase;
}
</style>

<div class="projects-hero">
  <h1>Projects</h1>
  <p>Apps built end-to-end — from idea to production.</p>
</div>

<div class="projects-grid">

  <a class="project-card" href="/projects/1lid/">
    <div class="project-card-thumb">🚗</div>
    <div class="project-card-body">
      <div class="project-card-title">1 Less Idiot Driver</div>
      <p class="project-card-desc">Gamified driving-hours tracker for Georgia teens earning their provisional licence. Log sessions, earn XP, hit your 40-hour goal.</p>
      <div class="project-card-tags">
        <span class="project-tag">React Native</span>
        <span class="project-tag">ASP.NET Core</span>
        <span class="project-tag">Azure SQL</span>
      </div>
    </div>
  </a>

  <a class="project-card" href="/projects/dashcode/">
    <div class="project-card-thumb">⌚</div>
    <div class="project-card-body">
      <div class="project-card-title">dashCODE</div>
      <p class="project-card-desc">Wear OS QR-code catalog for Pixel Watch. Store codes on your wrist, display them full-screen at the tap of a button.</p>
      <div class="project-card-tags">
        <span class="project-tag">Wear OS</span>
        <span class="project-tag">Kotlin</span>
        <span class="project-tag">Jetpack Compose</span>
      </div>
    </div>
  </a>

</div>
