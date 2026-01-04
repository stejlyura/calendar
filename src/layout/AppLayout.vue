<script setup lang="ts">
import { ref } from 'vue'
import SidebarNav from '../components/sidebar/SidebarNav.vue'

const isSidebarOpen = ref(false)

const toggleSidebar = () => {
  isSidebarOpen.value = !isSidebarOpen.value
}

const handleSidebarSelect = () => {
  isSidebarOpen.value = false
}
</script>

<template>
  <div class="app-shell">
    <SidebarNav
      :class="{ 'is-open': isSidebarOpen }"
      @select="handleSidebarSelect"
    />
    <button
      class="sidebar-backdrop"
      :class="{ 'is-visible': isSidebarOpen }"
      type="button"
      aria-label="Close menu"
      @click="isSidebarOpen = false"
    ></button>

    <div class="app-main">
      <header class="topbar">
        <div class="topbar__left">
          <button
            class="topbar__menu"
            :class="{ 'is-open': isSidebarOpen }"
            type="button"
            aria-label="Toggle menu"
            aria-controls="sidebar-nav"
            :aria-expanded="isSidebarOpen"
            @click="toggleSidebar"
          >
            <span class="burger" aria-hidden="true">
              <span class="burger__line"></span>
              <span class="burger__line"></span>
              <span class="burger__line"></span>
            </span>
          </button>
          <div class="topbar__search">
            <span class="topbar__search-icon" aria-hidden="true"></span>
            <input
              class="topbar__input"
              type="text"
              placeholder="Search transactions, invoices or help"
            />
          </div>
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

.topbar__left {
  display: flex;
  align-items: center;
  gap: 14px;
  flex: 1;
}

.topbar__menu {
  border: none;
  background: #f5f6fb;
  width: 40px;
  height: 40px;
  border-radius: 12px;
  display: none;
  place-items: center;
  cursor: pointer;
  padding: 0;
}

.burger {
  width: 18px;
  height: 12px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.burger__line {
  height: 2px;
  border-radius: 2px;
  background: #68708f;
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.topbar__menu.is-open .burger__line:nth-child(1) {
  transform: translateY(5px) rotate(45deg);
}

.topbar__menu.is-open .burger__line:nth-child(2) {
  opacity: 0;
}

.topbar__menu.is-open .burger__line:nth-child(3) {
  transform: translateY(-5px) rotate(-45deg);
}

.topbar__search {
  display: flex;
  align-items: center;
  gap: 10px;
  background: #f5f6fb;
  border-radius: 18px;
  padding: 8px 14px;
  min-width: 280px;
  flex: 1;
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

.sidebar-backdrop {
  display: none;
}

@media (max-width: 980px) {
  .app-shell {
    grid-template-columns: 1fr;
  }

  .topbar {
    flex-direction: column;
    align-items: stretch;
  }

  .topbar__menu {
    display: grid;
  }

  .topbar__search {
    min-width: 0;
  }

  .topbar__actions {
    justify-content: space-between;
  }

  .sidebar-backdrop {
    display: block;
    position: fixed;
    inset: 0;
    background: rgba(15, 18, 37, 0.35);
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.2s ease;
    z-index: 15;
  }

  .sidebar-backdrop.is-visible {
    opacity: 1;
    pointer-events: auto;
  }
}
</style>
