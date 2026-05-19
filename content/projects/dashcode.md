---
title: "dashCODE"
date: 2026-05-19
draft: false
type: page
---

<style>
.project-header {
  max-width: 720px;
  margin: 0 auto;
  padding: 2.5rem 1rem 0;
}
.project-header h1 {
  font-size: 2.2rem;
  font-weight: 800;
  margin-bottom: 0.5rem;
}
.project-tagline {
  font-size: 1.1rem;
  color: #888;
  margin-bottom: 2rem;
  line-height: 1.6;
}
.project-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-bottom: 2.5rem;
}
.project-tag {
  display: inline-block;
  padding: 0.2rem 0.7rem;
  border-radius: 6px;
  background: rgba(0, 102, 153, 0.12);
  color: #006699;
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 0.04em;
  text-transform: uppercase;
}
.project-body {
  max-width: 720px;
  margin: 0 auto;
  padding: 0 1rem 4rem;
}
.project-body h2 {
  font-size: 1.3rem;
  font-weight: 700;
  margin-top: 2.5rem;
  margin-bottom: 0.75rem;
  border-bottom: 1px solid rgba(0, 102, 153, 0.15);
  padding-bottom: 0.4rem;
}
.feature-list {
  list-style: none;
  padding: 0;
  margin: 0 0 1.5rem;
}
.feature-list li {
  padding: 0.35rem 0 0.35rem 1.5rem;
  position: relative;
  font-size: 0.95rem;
  line-height: 1.6;
  color: inherit;
}
.feature-list li::before {
  content: "→";
  position: absolute;
  left: 0;
  color: #006699;
  font-weight: 700;
}
.stack-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1.5rem;
}
.stack-card {
  border-radius: 10px;
  border: 1px solid rgba(0, 102, 153, 0.2);
  background: rgba(0, 102, 153, 0.04);
  padding: 1rem 1.1rem;
}
.stack-card h3 {
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: #006699;
  font-weight: 700;
  margin: 0 0 0.5rem;
}
.stack-card ul {
  list-style: none;
  padding: 0;
  margin: 0;
  font-size: 0.875rem;
  line-height: 1.7;
  color: #888;
}
@media (max-width: 500px) {
  .stack-grid { grid-template-columns: 1fr; }
}
</style>

<div class="project-header">
  <h1>dashCODE</h1>
  <p class="project-tagline">
    A Wear OS QR-code catalog for Pixel Watch — store your loyalty cards, transit passes,
    and access codes on your wrist and display them full-screen at the tap of a button.
  </p>
  <div class="project-tags">
    <span class="project-tag">Wear OS</span>
    <span class="project-tag">Android</span>
    <span class="project-tag">Kotlin</span>
    <span class="project-tag">Jetpack Compose</span>
    <span class="project-tag">Room</span>
    <span class="project-tag">Hilt</span>
  </div>
</div>

<div class="project-body">

## The Problem

QR codes are everywhere — loyalty programs, gym check-ins, transit passes, event tickets. On a phone that's usually fine. But when your hands are full, or you're at the gym, or it's raining, digging out your phone to scan a code is friction you don't need. If you're already wearing a smartwatch, the answer should be on your wrist.

## What It Does

dashCODE is a two-part system: a minimal phone app where you manage your codes, and a Wear OS app where you use them.

<ul class="feature-list">
  <li>Add QR codes on your phone — name them, paste the content, done.</li>
  <li>Tap Sync and the full list pushes to your watch over the Wearable Data Layer.</li>
  <li>On the watch, a scrollable list shows all your codes by name.</li>
  <li>Tap any code: it fills the screen and keeps the display on until you dismiss it.</li>
  <li>No internet required at scan time — codes live locally in a Room database on the watch.</li>
</ul>

## Stack

<div class="stack-grid">
  <div class="stack-card">
    <h3>Watch App</h3>
    <ul>
      <li>Wear OS (Pixel Watch)</li>
      <li>Wear Compose Material3</li>
      <li>Room (local DB)</li>
      <li>ZXing (QR bitmap)</li>
    </ul>
  </div>
  <div class="stack-card">
    <h3>Phone App</h3>
    <ul>
      <li>Android (Material3)</li>
      <li>Jetpack Compose</li>
      <li>DataStore (JSON)</li>
      <li>Wearable Data Layer</li>
    </ul>
  </div>
  <div class="stack-card">
    <h3>Architecture</h3>
    <ul>
      <li>Kotlin throughout</li>
      <li>Hilt (DI)</li>
      <li>KSP annotation processing</li>
      <li>MVVM + Repository pattern</li>
    </ul>
  </div>
  <div class="stack-card">
    <h3>CI/CD</h3>
    <ul>
      <li>GitHub Actions</li>
      <li>assembleDebug on PRs</li>
      <li>bundleRelease → Play Store</li>
      <li>Internal track on main</li>
    </ul>
  </div>
</div>

## Status

Available on Google Play (internal track). Both watch and phone apps are functional and deployed. Ongoing work focuses on edit/delete flows and improved sync reliability.

</div>
