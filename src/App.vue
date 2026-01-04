<script setup lang="ts">
import CalendarPage from './pages/CalendarPage.vue'
</script>

<template>
  <CalendarPage />
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;600;700&display=swap');

:root {
  color-scheme: light;
  --bg: #f3f4f8;
  --sidebar-bg: #3b3a4e;
  --sidebar-text: #d7d7e6;
  --sidebar-active: #ffffff;
  --card-bg: #ffffff;
  --border: #e6e9f2;
  --muted: #8b90a7;
  --text: #2d3142;
  --brand: #3d7bfd;
  --shadow: 0 16px 32px rgba(32, 38, 53, 0.08);
  font-family: 'Nunito', sans-serif;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
}

#app {
  min-height: 100vh;
}

button,
input {
  font: inherit;
}
</style>
