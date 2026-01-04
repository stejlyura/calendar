<script setup lang="ts">
import AppLayout from '../layout/AppLayout.vue'
import CalendarShell from '../components/calendar/CalendarShell.vue'
</script>

<template>
  <AppLayout>
    <section class="calendar-page">
      <div class="calendar-page__header">
        <h1>Calendar</h1>
      </div>
      <CalendarShell />
    </section>
  </AppLayout>
</template>

<style scoped>
.calendar-page {
  display: grid;
  gap: 16px;
}

.calendar-page__header {
  margin-bottom: 16px;
}

.calendar-page__header h1 {
  margin: 0;
  font-size: 22px;
  font-weight: 700;
}
</style>
