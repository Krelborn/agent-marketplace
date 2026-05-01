# Component Patterns

Reusable, polished snippets for common UI patterns. Copy and adapt — these assume the design tokens from `assets/template.html`.

## Top Navigation Bar

```html
<header class="topbar">
  <div class="topbar-inner">
    <a class="brand" href="#">
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
        <rect width="24" height="24" rx="6" fill="var(--accent)" />
      </svg>
      <span>Acme</span>
    </a>
    <nav class="nav">
      <a href="#" class="nav-link nav-link-active">Dashboard</a>
      <a href="#" class="nav-link">Projects</a>
      <a href="#" class="nav-link">Team</a>
    </nav>
    <div class="topbar-actions">
      <button class="btn">Invite</button>
      <button class="btn btn-primary">New project</button>
    </div>
  </div>
</header>
<style>
  .topbar {
    border-bottom: 1px solid var(--border);
    background: var(--surface);
  }
  .topbar-inner {
    max-width: 1280px;
    margin: 0 auto;
    padding: var(--space-3) var(--space-6);
    display: flex;
    align-items: center;
    gap: var(--space-8);
  }
  .brand {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    font-weight: 600;
    color: var(--text);
  }
  .nav {
    display: flex;
    gap: var(--space-1);
    flex: 1;
  }
  .nav-link {
    padding: var(--space-2) var(--space-3);
    border-radius: var(--radius);
    color: var(--text-muted);
    font-size: var(--text-sm);
    font-weight: 500;
  }
  .nav-link:hover {
    background: var(--surface-2);
    text-decoration: none;
  }
  .nav-link-active {
    background: var(--surface-2);
    color: var(--text);
  }
  .topbar-actions {
    display: flex;
    gap: var(--space-2);
  }
</style>
```

## Sidebar Layout

```html
<div class="app-shell">
  <aside class="sidebar">
    <!-- nav items -->
  </aside>
  <main class="main">
    <!-- content -->
  </main>
</div>
<style>
  .app-shell {
    display: grid;
    grid-template-columns: 240px 1fr;
    min-height: 100vh;
  }
  .sidebar {
    border-right: 1px solid var(--border);
    background: var(--surface);
    padding: var(--space-4);
  }
  .main {
    padding: var(--space-8);
    max-width: 1200px;
  }
  @media (max-width: 768px) {
    .app-shell {
      grid-template-columns: 1fr;
    }
    .sidebar {
      display: none;
    }
  }
</style>
```

## Stat Card

```html
<div class="stat-card">
  <div class="stat-label">Monthly recurring revenue</div>
  <div class="stat-value">$48,290</div>
  <div class="stat-delta stat-delta-up">+12.4% vs last month</div>
</div>
<style>
  .stat-card {
    padding: var(--space-5);
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
  }
  .stat-label {
    font-size: var(--text-sm);
    color: var(--text-subtle);
  }
  .stat-value {
    font-size: var(--text-3xl);
    font-weight: 600;
    margin-top: var(--space-2);
    letter-spacing: -0.02em;
  }
  .stat-delta {
    font-size: var(--text-sm);
    margin-top: var(--space-1);
  }
  .stat-delta-up {
    color: var(--success);
  }
  .stat-delta-down {
    color: var(--danger);
  }
</style>
```

## Data Table

```html
<table class="data-table">
  <thead>
    <tr>
      <th>Customer</th>
      <th>Plan</th>
      <th>Status</th>
      <th class="num">MRR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <div class="user-cell">
          <img class="avatar" src="https://i.pravatar.cc/64?u=1" />
          <div>
            <div>Sara Lin</div>
            <div class="muted">sara@northwind.io</div>
          </div>
        </div>
      </td>
      <td>Growth</td>
      <td><span class="badge badge-success">Active</span></td>
      <td class="num">$249</td>
    </tr>
  </tbody>
</table>
<style>
  .data-table {
    width: 100%;
    border-collapse: collapse;
    font-size: var(--text-sm);
  }
  .data-table th,
  .data-table td {
    padding: var(--space-3) var(--space-4);
    text-align: left;
    border-bottom: 1px solid var(--border);
  }
  .data-table th {
    font-weight: 500;
    color: var(--text-subtle);
    font-size: var(--text-xs);
    text-transform: uppercase;
    letter-spacing: 0.04em;
    background: var(--surface-2);
  }
  .data-table .num {
    text-align: right;
    font-variant-numeric: tabular-nums;
  }
  .user-cell {
    display: flex;
    align-items: center;
    gap: var(--space-3);
  }
  .avatar {
    width: 32px;
    height: 32px;
    border-radius: var(--radius-full);
  }
  .muted {
    color: var(--text-subtle);
    font-size: var(--text-xs);
  }
  .badge {
    display: inline-flex;
    align-items: center;
    padding: 2px var(--space-2);
    border-radius: var(--radius-full);
    font-size: var(--text-xs);
    font-weight: 500;
  }
  .badge-success {
    background: var(--success-soft);
    color: var(--success-strong);
  }
  .badge-warning {
    background: var(--warning-soft);
    color: var(--warning-strong);
  }
  .badge-danger {
    background: var(--danger-soft);
    color: var(--danger-strong);
  }
</style>
```

## Modal

```html
<dialog id="example-modal" class="modal">
  <form method="dialog" class="modal-inner">
    <h2>Invite teammate</h2>
    <p class="muted">They'll get an email with a link to join.</p>
    <label class="field">
      <span>Email</span>
      <input class="input" type="email" placeholder="name@company.com" />
    </label>
    <div class="modal-actions">
      <button class="btn" value="cancel">Cancel</button>
      <button class="btn btn-primary" value="confirm">Send invite</button>
    </div>
  </form>
</dialog>
<button class="btn btn-primary" onclick="document.getElementById('example-modal').showModal()">
  Invite
</button>
<style>
  .modal {
    border: 0;
    border-radius: var(--radius-lg);
    padding: 0;
    max-width: 420px;
    box-shadow: var(--shadow-lg);
  }
  .modal::backdrop {
    background: rgba(0, 0, 0, 0.4);
  }
  .modal-inner {
    padding: var(--space-6);
    display: flex;
    flex-direction: column;
    gap: var(--space-4);
  }
  .field {
    display: flex;
    flex-direction: column;
    gap: var(--space-2);
    font-size: var(--text-sm);
  }
  .modal-actions {
    display: flex;
    justify-content: flex-end;
    gap: var(--space-2);
  }
</style>
```

## Tabs (interactive, no JS framework)

```html
<div class="tabs" role="tablist">
  <button role="tab" aria-selected="true" data-tab="overview">Overview</button>
  <button role="tab" aria-selected="false" data-tab="activity">Activity</button>
  <button role="tab" aria-selected="false" data-tab="settings">Settings</button>
</div>
<div class="tab-panel" data-panel="overview">Overview content</div>
<div class="tab-panel" data-panel="activity" hidden>Activity content</div>
<div class="tab-panel" data-panel="settings" hidden>Settings content</div>
<style>
  .tabs {
    display: flex;
    gap: var(--space-1);
    border-bottom: 1px solid var(--border);
  }
  .tabs button {
    padding: var(--space-3) var(--space-4);
    color: var(--text-muted);
    font-size: var(--text-sm);
    font-weight: 500;
    border-bottom: 2px solid transparent;
    margin-bottom: -1px;
  }
  .tabs button[aria-selected='true'] {
    color: var(--text);
    border-color: var(--accent);
  }
  .tab-panel {
    padding: var(--space-6) 0;
  }
</style>
<script>
  document.querySelectorAll('[role="tab"]').forEach((tab) => {
    tab.addEventListener('click', () => {
      const target = tab.dataset.tab;
      document.querySelectorAll('[role="tab"]').forEach((t) => {
        t.setAttribute('aria-selected', t === tab ? 'true' : 'false');
      });
      document.querySelectorAll('.tab-panel').forEach((p) => {
        p.hidden = p.dataset.panel !== target;
      });
    });
  });
</script>
```

## Chart Placeholder (inline SVG sparkline)

For real-feeling dashboards. No chart library — hand-rolled SVG.

```html
<svg viewBox="0 0 200 60" class="sparkline" preserveAspectRatio="none">
  <path
    d="M0,40 L20,38 L40,42 L60,30 L80,32 L100,20 L120,24 L140,14 L160,18 L180,8 L200,12"
    fill="none"
    stroke="var(--accent)"
    stroke-width="2"
  />
  <path
    d="M0,40 L20,38 L40,42 L60,30 L80,32 L100,20 L120,24 L140,14 L160,18 L180,8 L200,12 L200,60 L0,60 Z"
    fill="var(--accent-soft)"
    opacity="0.5"
  />
</svg>
<style>
  .sparkline {
    width: 100%;
    height: 60px;
  }
</style>
```

## Icon Source

Lucide via CDN (recommended — no build step):

```html
<script src="https://unpkg.com/lucide@latest"></script>
<i data-lucide="search"></i>
<i data-lucide="bell"></i>
<script>
  lucide.createIcons();
</script>
```

Or inline SVG when only a few icons are needed (preferred for portability).
