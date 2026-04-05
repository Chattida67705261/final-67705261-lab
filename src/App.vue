<template>
  <div id="app" class="app-shell">
    <div class="ambient ambient-one"></div>
    <div class="ambient ambient-two"></div>

    <header class="topbar">
      <router-link to="/" class="brand-block">
        <span class="brand-mark">C</span>
        <div>
          <p class="brand-kicker">Chattida Workflow Studio</p>
          <h1 class="brand-title">n8n Experience Suite</h1>
        </div>
      </router-link>

      <nav class="nav-panel">
        <router-link to="/" class="nav-link">Overview</router-link>
        <router-link to="/about" class="nav-link">Requests</router-link>
        <router-link to="/register" class="nav-link">Register</router-link>
      </nav>
    </header>

    <main class="page-stage">
      <router-view />
    </main>
  </div>
</template>

<style>
:root {
  --bg: #f6f0e8;
  --bg-deep: #eadfce;
  --surface: rgba(255, 250, 244, 0.82);
  --surface-strong: rgba(255, 252, 248, 0.94);
  --ink: #231815;
  --muted: #6e6259;
  --line: rgba(86, 61, 43, 0.14);
  --gold: #b68a52;
  --gold-soft: #dbc29b;
  --forest: #1f4d43;
  --shadow: 0 28px 80px rgba(85, 55, 29, 0.14);
}

* {
  box-sizing: border-box;
}

html,
body,
#app {
  min-height: 100%;
  margin: 0;
}

body {
  font-family: "Georgia", "Times New Roman", serif;
  background:
    radial-gradient(circle at top left, rgba(194, 153, 102, 0.22), transparent 24%),
    radial-gradient(circle at 80% 10%, rgba(53, 96, 84, 0.16), transparent 18%),
    linear-gradient(180deg, var(--bg) 0%, #f2e8dc 55%, var(--bg-deep) 100%);
  color: var(--ink);
}

a {
  color: inherit;
  text-decoration: none;
}

button,
input,
select {
  font: inherit;
}

.app-shell {
  position: relative;
  min-height: 100vh;
  padding: 24px;
  overflow: hidden;
}

.ambient {
  position: fixed;
  border-radius: 999px;
  filter: blur(20px);
  pointer-events: none;
  opacity: 0.8;
}

.ambient-one {
  top: -60px;
  left: -40px;
  width: 220px;
  height: 220px;
  background: rgba(182, 138, 82, 0.22);
}

.ambient-two {
  right: -80px;
  bottom: 60px;
  width: 280px;
  height: 280px;
  background: rgba(31, 77, 67, 0.18);
}

.topbar {
  position: sticky;
  top: 20px;
  z-index: 10;
  max-width: 1240px;
  margin: 0 auto 28px;
  padding: 18px 22px;
  border: 1px solid var(--line);
  border-radius: 28px;
  background: rgba(255, 248, 240, 0.74);
  backdrop-filter: blur(16px);
  box-shadow: var(--shadow);
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 18px;
}

.brand-block {
  display: flex;
  align-items: center;
  gap: 14px;
}

.brand-mark {
  width: 48px;
  height: 48px;
  display: grid;
  place-items: center;
  border-radius: 16px;
  background: linear-gradient(145deg, #1d4f45, #b68a52);
  color: #fffaf4;
  font-size: 1.45rem;
  font-weight: 700;
  box-shadow: 0 10px 24px rgba(31, 77, 67, 0.22);
}

.brand-kicker {
  margin: 0 0 3px;
  font-size: 0.72rem;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: var(--muted);
}

.brand-title {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 700;
}

.nav-panel {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 8px;
  border-radius: 999px;
  background: rgba(255, 252, 247, 0.88);
  border: 1px solid rgba(92, 69, 49, 0.1);
}

.nav-link {
  padding: 12px 18px;
  border-radius: 999px;
  color: var(--muted);
  font-size: 0.95rem;
  font-weight: 600;
  transition: transform 0.25s ease, background 0.25s ease, color 0.25s ease;
}

.nav-link:hover {
  transform: translateY(-1px);
  background: rgba(182, 138, 82, 0.12);
  color: var(--ink);
}

.nav-link.router-link-exact-active {
  background: linear-gradient(135deg, #204d43, #b68a52);
  color: #fffaf3;
  box-shadow: 0 12px 24px rgba(108, 80, 49, 0.2);
}

.page-stage {
  position: relative;
  z-index: 1;
}

@media (max-width: 860px) {
  .topbar {
    position: static;
    flex-direction: column;
    align-items: stretch;
  }

  .nav-panel {
    justify-content: space-between;
    width: 100%;
    flex-wrap: wrap;
  }

  .nav-link {
    flex: 1 1 120px;
    text-align: center;
  }
}

@media (max-width: 640px) {
  .app-shell {
    padding: 14px;
  }

  .topbar {
    margin-bottom: 18px;
    padding: 16px;
    border-radius: 22px;
  }

  .brand-title {
    font-size: 1rem;
  }
}
</style>
