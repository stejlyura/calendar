<script setup lang="ts">
import { computed, reactive, ref, watch } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import listPlugin from '@fullcalendar/list'
import interactionPlugin from '@fullcalendar/interaction'
import type {
  CalendarOptions,
  DatesSetArg,
  DateClickArg,
  EventClickArg,
  EventChangeArg,
  EventApi,
  DateSelectArg,
} from '@fullcalendar/core'

type CalendarEvent = {
  id: string
  title: string
  start: string
  end?: string
  color: string
}

const calendarRef = ref<InstanceType<typeof FullCalendar> | null>(null)
const title = ref('')
const activeView = ref('dayGridMonth')
const isModalOpen = ref(false)
const editingId = ref<string | null>(null)
const formError = ref('')

const viewButtons = [
  { id: 'dayGridMonth', label: 'Month' },
  { id: 'timeGridWeek', label: 'Week' },
  { id: 'timeGridDay', label: 'Day' },
  { id: 'listWeek', label: 'Agenda' },
]

const colorOptions = ['#3d7bfd', '#20c997', '#ff8a65', '#6f52ff', '#f4b23f']
const storageKey = 'calendar-events'
const createId = () =>
  typeof crypto !== 'undefined' && 'randomUUID' in crypto
    ? crypto.randomUUID()
    : `${Date.now()}-${Math.random().toString(16).slice(2)}`

const defaultEvents: CalendarEvent[] = [
  { id: '1', title: 'Event name', start: '2024-01-02T09:00', end: '2024-01-02T10:00', color: '#3d7bfd' },
  { id: '2', title: 'Event name', start: '2024-01-04T09:00', end: '2024-01-04T10:00', color: '#3d7bfd' },
  { id: '3', title: 'Event name', start: '2024-01-04T11:00', end: '2024-01-04T12:00', color: '#3d7bfd' },
  { id: '4', title: 'Event name', start: '2024-01-05T09:00', end: '2024-01-05T10:00', color: '#3d7bfd' },
  { id: '5', title: 'Event name', start: '2024-01-14T09:00', end: '2024-01-14T10:00', color: '#3d7bfd' },
  { id: '6', title: 'Event name', start: '2024-01-20T09:00', end: '2024-01-20T10:00', color: '#3d7bfd' },
  { id: '7', title: 'Event name', start: '2024-01-28T09:00', end: '2024-01-28T10:00', color: '#3d7bfd' },
  { id: '8', title: 'Event name', start: '2024-01-29T09:00', end: '2024-01-29T10:00', color: '#3d7bfd' },
]

const safeParseEvents = () => {
  const stored = localStorage.getItem(storageKey)
  if (!stored) {
    return defaultEvents
  }
  try {
    const parsed = JSON.parse(stored) as CalendarEvent[]
    return Array.isArray(parsed) && parsed.length > 0 ? parsed : defaultEvents
  } catch {
    return defaultEvents
  }
}

const events = ref<CalendarEvent[]>(safeParseEvents())

const formState = reactive({
  title: '',
  date: '',
  startTime: '',
  endTime: '',
  color: colorOptions[0],
})

const options = reactive<CalendarOptions>({
  plugins: [dayGridPlugin, timeGridPlugin, listPlugin, interactionPlugin],
  initialView: 'dayGridMonth',
  editable: true,
  selectable: true,
  selectMirror: true,
  nowIndicator: true,
  eventDisplay: 'block',
  eventOrder: 'start,title',
  slotEventOverlap: true,
  eventMaxStack: 3,
  dayMaxEventRows: 3,
  headerToolbar: false,
  height: 'auto',
  events: events.value,
  datesSet(arg: DatesSetArg) {
    title.value = arg.view.title
    activeView.value = arg.view.type
  },
  dateClick(arg: DateClickArg) {
    openCreateModal(arg.date)
  },
  select(arg: DateSelectArg) {
    openCreateModal(arg.start, arg.end)
  },
  eventClick(arg: EventClickArg) {
    openEditModal(arg.event)
  },
  eventChange(arg: EventChangeArg) {
    syncEventFromApi(arg.event)
  },
})

const isEditing = computed(() => editingId.value !== null)

const handleToday = () => calendarRef.value?.getApi().today()
const handlePrev = () => calendarRef.value?.getApi().prev()
const handleNext = () => calendarRef.value?.getApi().next()
const handleViewChange = (view: string) => calendarRef.value?.getApi().changeView(view)

const pad = (value: number) => value.toString().padStart(2, '0')

const formatDateInput = (date: Date) =>
  `${date.getFullYear()}-${pad(date.getMonth() + 1)}-${pad(date.getDate())}`

const formatTimeInput = (date: Date) => `${pad(date.getHours())}:${pad(date.getMinutes())}`

const addHours = (date: Date, hours: number) =>
  new Date(date.getTime() + hours * 60 * 60 * 1000)

const openCreateModal = (date: Date, end?: Date) => {
  const baseDate = new Date(date)
  if (baseDate.getHours() === 0 && baseDate.getMinutes() === 0) {
    baseDate.setHours(9, 0, 0, 0)
  }
  const computedEnd = end && end > baseDate ? new Date(end) : addHours(baseDate, 1)
  formState.title = ''
  formState.date = formatDateInput(baseDate)
  formState.startTime = formatTimeInput(baseDate)
  formState.endTime = formatTimeInput(computedEnd)
  formState.color = colorOptions[0]
  editingId.value = null
  formError.value = ''
  isModalOpen.value = true
}

const openEditModal = (event: EventApi) => {
  const start = event.start ? new Date(event.start) : new Date()
  const end = event.end ? new Date(event.end) : addHours(start, 1)
  formState.title = event.title ?? ''
  formState.date = formatDateInput(start)
  formState.startTime = formatTimeInput(start)
  formState.endTime = formatTimeInput(end)
  formState.color = event.backgroundColor || event.borderColor || colorOptions[0]
  editingId.value = event.id ?? null
  formError.value = ''
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
}

const buildEventPayload = () => {
  const startDate = new Date(`${formState.date}T${formState.startTime}`)
  let endDate = formState.endTime
    ? new Date(`${formState.date}T${formState.endTime}`)
    : addHours(startDate, 1)
  if (Number.isNaN(startDate.getTime())) {
    return null
  }
  if (Number.isNaN(endDate.getTime()) || endDate <= startDate) {
    endDate = addHours(startDate, 1)
  }
  const start = `${formatDateInput(startDate)}T${formatTimeInput(startDate)}`
  const end = `${formatDateInput(endDate)}T${formatTimeInput(endDate)}`
  return { start, end }
}

const handleSave = () => {
  const titleValue = formState.title.trim().slice(0, 30)
  if (!titleValue) {
    formError.value = 'Title is required.'
    return
  }
  const payload = buildEventPayload()
  if (!payload) {
    formError.value = 'Enter valid date and time.'
    return
  }
  if (editingId.value) {
    const updated = events.value.map((event) =>
      event.id === editingId.value
        ? { ...event, title: titleValue, start: payload.start, end: payload.end, color: formState.color }
        : event,
    )
    events.value = updated
    const api = calendarRef.value?.getApi().getEventById(editingId.value)
    if (api) {
      api.setProp('title', titleValue)
      api.setDates(payload.start, payload.end)
      api.setProp('backgroundColor', formState.color)
      api.setProp('borderColor', formState.color)
    }
  } else {
    const newEvent: CalendarEvent = {
      id: createId(),
      title: titleValue,
      start: payload.start,
      end: payload.end,
      color: formState.color,
    }
    events.value = [...events.value, newEvent]
  }
  closeModal()
}

const handleDelete = () => {
  if (!editingId.value) {
    return
  }
  events.value = events.value.filter((event) => event.id !== editingId.value)
  const api = calendarRef.value?.getApi().getEventById(editingId.value)
  api?.remove()
  closeModal()
}

const syncEventFromApi = (event: EventApi) => {
  if (!event.id) {
    return
  }
  const start = event.start ? formatDateInput(new Date(event.start)) + 'T' + formatTimeInput(new Date(event.start)) : ''
  const end = event.end ? formatDateInput(new Date(event.end)) + 'T' + formatTimeInput(new Date(event.end)) : undefined
  events.value = events.value.map((item) =>
    item.id === event.id
      ? {
          ...item,
          title: event.title ?? item.title,
          start: start || item.start,
          end: end || item.end,
          color: event.backgroundColor || item.color,
        }
      : item,
  )
}

watch(
  events,
  (next) => {
    options.events = [...next]
    localStorage.setItem(storageKey, JSON.stringify(next))
  },
  { deep: true },
)
</script>

<template>
  <section class="calendar-card">
    <div class="calendar-card__label">Calendar View</div>
    <div class="calendar-card__toolbar">
      <div class="calendar-card__nav">
        <button class="ghost-btn is-primary" type="button" @click="handleToday">
          Today
        </button>
        <button class="ghost-btn" type="button" @click="handlePrev">Back</button>
        <button class="ghost-btn" type="button" @click="handleNext">Next</button>
      </div>
      <div class="calendar-card__title">{{ title }}</div>
      <div class="calendar-card__views">
        <button
          v-for="btn in viewButtons"
          :key="btn.id"
          class="ghost-btn"
          :class="{ 'is-active': activeView === btn.id }"
          type="button"
          @click="handleViewChange(btn.id)"
        >
          {{ btn.label }}
        </button>
      </div>
    </div>

    <div class="calendar-card__body">
      <FullCalendar ref="calendarRef" :options="options" />
    </div>
  </section>

  <div v-if="isModalOpen" class="modal-overlay" @click.self="closeModal">
    <div class="modal-card">
      <div class="modal-card__header">
        <h3>{{ isEditing ? 'Edit event' : 'Add event' }}</h3>
        <button class="modal-card__close" type="button" aria-label="Close" @click="closeModal">+</button>
      </div>
      <div class="modal-card__body">
        <label class="modal-field">
          <span>Event name</span>
          <input v-model="formState.title" maxlength="30" type="text" placeholder="Event name" />
        </label>
        <label class="modal-field">
          <span>Date</span>
          <input v-model="formState.date" type="date" />
        </label>
        <div class="modal-field modal-field--row">
          <label>
            <span>Start time</span>
            <input v-model="formState.startTime" type="time" />
          </label>
          <label>
            <span>End time</span>
            <input v-model="formState.endTime" type="time" />
          </label>
        </div>
        <div class="modal-field">
          <span>Color</span>
          <div class="color-row">
            <button
              v-for="color in colorOptions"
              :key="color"
              class="color-swatch"
              :style="{ backgroundColor: color }"
              :class="{ 'is-active': formState.color === color }"
              type="button"
              @click="formState.color = color"
              aria-label="Pick color"
            ></button>
          </div>
        </div>
        <p v-if="formError" class="modal-error">{{ formError }}</p>
      </div>
      <div class="modal-card__footer">
        <button class="modal-link" type="button" @click="isEditing ? handleDelete() : closeModal()">
          {{ isEditing ? 'Delete' : 'Cancel' }}
        </button>
        <button class="ghost-btn is-primary" type="button" @click="handleSave">
          {{ isEditing ? 'Update' : 'Save' }}
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.calendar-card {
  background: var(--card-bg);
  border-radius: 14px;
  padding: 18px 18px 8px;
  box-shadow: var(--shadow);
  max-width: 1240px;
  width: 100%;
}

.calendar-card__label {
  font-size: 13px;
  font-weight: 600;
  color: var(--muted);
  margin-bottom: 10px;
}

.calendar-card__toolbar {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  align-items: center;
  gap: 12px;
  padding-bottom: 12px;
}

.calendar-card__nav,
.calendar-card__views {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.calendar-card__title {
  text-align: center;
  font-weight: 600;
  color: var(--muted);
}

.ghost-btn {
  border: 1px solid var(--border);
  background: #f8f9fd;
  color: var(--muted);
  padding: 6px 14px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
}

.ghost-btn.is-primary {
  color: var(--brand);
  border-color: rgba(61, 123, 253, 0.3);
}

.ghost-btn.is-active {
  color: #ffffff;
  background: var(--brand);
  border-color: var(--brand);
}

.calendar-card__body {
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 8px;
  background: #fbfcff;
}

:deep(.fc) {
  --fc-border-color: #edf0f7;
  --fc-today-bg-color: rgba(61, 123, 253, 0.08);
  font-size: 12px;
}

:deep(.fc .fc-col-header-cell) {
  background: #f7f8fc;
  font-size: 11px;
  text-transform: uppercase;
  color: var(--muted);
}

:deep(.fc .fc-daygrid-day-number) {
  color: var(--muted);
  font-weight: 600;
}

:deep(.fc .fc-event) {
  border: none;
  border-radius: 6px;
  padding: 2px 6px;
  font-size: 11px;
}

:deep(.fc .fc-daygrid-event .fc-event-time) {
  display: none;
}

:deep(.fc .fc-timegrid-slot-label) {
  color: var(--muted);
}

.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(33, 35, 54, 0.2);
  display: grid;
  place-items: center;
  z-index: 20;
}

.modal-card {
  width: 280px;
  background: #ffffff;
  border-radius: 14px;
  box-shadow: var(--shadow);
  border: 1px solid var(--border);
  padding: 16px;
}

.modal-card__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.modal-card__header h3 {
  margin: 0;
  font-size: 14px;
}

.modal-card__close {
  border: none;
  background: #f3f4fa;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 18px;
  line-height: 1;
  transform: rotate(45deg);
}

.modal-card__body {
  display: grid;
  gap: 10px;
}

.modal-field {
  display: grid;
  gap: 6px;
  font-size: 11px;
  color: var(--muted);
}

.modal-field input {
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 6px 10px;
  font-size: 12px;
  color: var(--text);
}

.modal-field--row {
  grid-template-columns: repeat(2, 1fr);
  gap: 8px;
}

.modal-field--row label {
  display: grid;
  gap: 6px;
}

.color-row {
  display: flex;
  gap: 8px;
}

.color-swatch {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 2px solid transparent;
  cursor: pointer;
}

.color-swatch.is-active {
  border-color: #2d3142;
}

.modal-error {
  margin: 0;
  font-size: 11px;
  color: #d95c5c;
}

.modal-card__footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 14px;
}

.modal-link {
  border: none;
  background: none;
  color: #d95c5c;
  font-size: 12px;
  cursor: pointer;
}

@media (max-width: 980px) {
  .calendar-card__toolbar {
    grid-template-columns: 1fr;
    text-align: left;
  }

  .calendar-card__title {
    text-align: left;
  }
}
</style>
