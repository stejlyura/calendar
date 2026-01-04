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

const activeItem = 'Calendar'

const emit = defineEmits<{
  (event: 'select', item: string): void
}>()

const handleSelect = (item: string) => {
  emit('select', item)
}
</script>

<template>
  <aside id="sidebar-nav" class="sidebar" aria-label="Primary">
    <div class="sidebar__brand">IMPEKABLE</div>
    <nav class="sidebar__nav">
      <button
        v-for="item in navItems"
        :key="item"
        class="sidebar__item"
        :class="{ 'is-active': item === activeItem }"
        type="button"
        @click="handleSelect(item)"
      >
        <span class="sidebar__dot" aria-hidden="true"></span>
        <span>{{ item }}</span>
      </button>
    </nav>
  </aside>
</template>

<style scoped>
.sidebar {
  background: var(--sidebar-bg);
  color: var(--sidebar-text);
  padding: 24px 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  min-height: 100vh;
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

@media (max-width: 980px) {
  .sidebar {
    position: fixed;
    inset: 0 auto 0 0;
    width: 240px;
    min-height: 100%;
    transform: translateX(-100%);
    transition: transform 0.2s ease;
    z-index: 20;
    box-shadow: 18px 0 40px rgba(12, 16, 36, 0.18);
  }

  .sidebar.is-open {
    transform: translateX(0);
  }
}
</style>
