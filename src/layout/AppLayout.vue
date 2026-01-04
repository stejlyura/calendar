<script setup lang="ts">
const navItems = [
  'Home',
  'Dashboard',
  'Inbox',
  'Products',
  'Invoices',
  'Customers',
  'Chat Room',
  'Calendar',
  'Help Center',
  'Settings',
]
</script>

<template>
  <div class="app-shell">
    <aside class="sidebar">
      <div class="sidebar__brand">IMPEKABLE</div>
      <nav class="sidebar__nav">
        <button
          v-for="item in navItems"
          :key="item"
          class="sidebar__item"
          :class="{ 'is-active': item === 'Calendar' }"
          type="button"
        >
          <span class="sidebar__dot" aria-hidden="true"></span>
          <span>{{ item }}</span>
        </button>
      </nav>
    </aside>

    <div class="app-main">
      <header class="topbar">
        <div class="topbar__search">
          <span class="topbar__search-icon" aria-hidden="true"></span>
          <input
            class="topbar__input"
            type="text"
            placeholder="Search transactions, invoices or help"
          />
        </div>
        <div class="topbar__actions">
          <button class="topbar__icon-btn" type="button" aria-label="Notifications">
            <span class="topbar__icon-bell" aria-hidden="true"></span>
          </button>
          <button class="topbar__icon-btn" type="button" aria-label="Settings">
            <span class="topbar__icon-gear" aria-hidden="true"></span>
          </button>
          <div class="topbar__user">
            <span class="topbar__user-name">John Doe</span>
            <span class="topbar__avatar">JD</span>
          </div>
        </div>
      </header>

      <main class="page">
        <slot />
      </main>
    </div>
  </div>
</template>

<style scoped>
.app-shell {
  display: grid;
  grid-template-columns: 230px 1fr;
  min-height: 100vh;
}

.sidebar {
  background: var(--sidebar-bg);
  color: var(--sidebar-text);
  padding: 24px 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.sidebar__brand {
  font-weight: 700;
  letter-spacing: 0.16em;
  font-size: 14px;
  color: #f3f3ff;
}

.sidebar__nav {
  display: grid;
  gap: 6px;
}

.sidebar__item {
  background: transparent;
  border: none;
  color: inherit;
  text-align: left;
  padding: 10px 12px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
}

.sidebar__item.is-active {
  color: var(--sidebar-active);
  background: rgba(255, 255, 255, 0.08);
  position: relative;
}

.sidebar__item:hover {
  color: var(--sidebar-active);
  background: rgba(255, 255, 255, 0.08);
}

.sidebar__item.is-active::before {
  content: '';
  position: absolute;
  left: -20px;
  top: 12px;
  bottom: 12px;
  width: 3px;
  border-radius: 2px;
  background: var(--brand);
}

.sidebar__dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: currentColor;
  opacity: 0.6;
}

.app-main {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.topbar {
  background: #ffffff;
  padding: 14px 24px;
  border-bottom: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.topbar__search {
  display: flex;
  align-items: center;
  gap: 10px;
  background: #f5f6fb;
  border-radius: 18px;
  padding: 8px 14px;
  min-width: 280px;
}

.topbar__search-icon {
  width: 14px;
  height: 14px;
  border: 2px solid #b6bbcf;
  border-radius: 50%;
  position: relative;
}

.topbar__search-icon::after {
  content: '';
  position: absolute;
  width: 7px;
  height: 2px;
  background: #b6bbcf;
  right: -5px;
  bottom: -1px;
  transform: rotate(45deg);
}

.topbar__input {
  border: none;
  background: transparent;
  outline: none;
  color: var(--text);
  width: 100%;
}

.topbar__actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.topbar__icon-btn {
  border: none;
  background: #f5f6fb;
  width: 34px;
  height: 34px;
  border-radius: 50%;
  cursor: pointer;
  display: grid;
  place-items: center;
}

.topbar__icon-btn span {
  display: inline-block;
  width: 14px;
  height: 14px;
  position: relative;
}

.topbar__icon-bell {
  border: 2px solid #b6bbcf;
  border-radius: 6px 6px 4px 4px;
}

.topbar__icon-bell::after {
  content: '';
  position: absolute;
  width: 4px;
  height: 4px;
  background: #b6bbcf;
  border-radius: 50%;
  left: 50%;
  bottom: -4px;
  transform: translateX(-50%);
}

.topbar__icon-gear {
  border: 2px solid #b6bbcf;
  border-radius: 50%;
}

.topbar__icon-gear::before,
.topbar__icon-gear::after {
  content: '';
  position: absolute;
  inset: 3px;
  border: 2px solid #b6bbcf;
  border-radius: 50%;
}

.topbar__user {
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 600;
}

.topbar__user-name {
  color: var(--muted);
}

.topbar__avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: var(--brand);
  color: #fff;
  display: grid;
  place-items: center;
  font-size: 12px;
}

.page {
  padding: 28px 32px 40px;
}

@media (max-width: 980px) {
  .app-shell {
    grid-template-columns: 1fr;
  }

  .sidebar {
    flex-direction: row;
    flex-wrap: wrap;
  }

  .topbar {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>
