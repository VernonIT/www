---
title: "1 Less Idiot Driver"
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
  <h1>1 Less Idiot Driver</h1>
  <p class="project-tagline">
    A gamified driving-hours tracker for Georgia teens earning their Class D provisional licence —
    because getting to 40 hours should feel like levelling up, not a chore.
  </p>
  <div class="project-tags">
    <span class="project-tag">Mobile App</span>
    <span class="project-tag">React Native</span>
    <span class="project-tag">Expo</span>
    <span class="project-tag">ASP.NET Core 10</span>
    <span class="project-tag">Azure SQL</span>
    <span class="project-tag">Google OAuth</span>
  </div>
</div>

<div class="project-body">

## The Problem

Georgia requires teen drivers to log 40 hours of supervised practice (including 6 at night) before they can sit their road test. Most families track this on paper — or don't track it at all. The result: teens who show up underprepared, and parents who have no idea where things stand.

## What It Does

1LiD replaces the paper log with a mobile app that makes every session count toward something.

<ul class="feature-list">
  <li>Log a driving session in seconds — date, duration, conditions, notes.</li>
  <li>Dashboard shows total hours, night hours, and progress toward the 40-hour goal at a glance.</li>
  <li>Twelve structured session groups guide teens through progressively harder driving scenarios.</li>
  <li>XP and badges reward consistency and milestones — finishing a session group, hitting 20 hours, completing a night drive.</li>
  <li>Shared view lets parents or supervising adults see the same progress without needing their own account.</li>
</ul>

## Stack

<div class="stack-grid">
  <div class="stack-card">
    <h3>Mobile</h3>
    <ul>
      <li>React Native (Expo)</li>
      <li>TypeScript</li>
      <li>Expo Go for dev</li>
      <li>iOS + Android targets</li>
    </ul>
  </div>
  <div class="stack-card">
    <h3>API</h3>
    <ul>
      <li>ASP.NET Core 10</li>
      <li>Entity Framework (EF Core)</li>
      <li>Azure SQL Database</li>
      <li>JWT + Google OAuth</li>
    </ul>
  </div>
  <div class="stack-card">
    <h3>Testing</h3>
    <ul>
      <li>xUnit (API)</li>
      <li>Jest (app)</li>
      <li>In-memory DB for unit tests</li>
      <li>ATDD acceptance suite</li>
    </ul>
  </div>
  <div class="stack-card">
    <h3>Infra</h3>
    <ul>
      <li>Azure (API hosting)</li>
      <li>Azure SQL</li>
      <li>Google Cloud (OAuth client)</li>
      <li>Azure Pipelines CI</li>
    </ul>
  </div>
</div>

## Status

Active development. Core session logging, hour tracking, and XP system are built. The app is currently in TestFlight / internal track ahead of a public launch.

[Privacy Policy](/privacy-1lid)

</div>
