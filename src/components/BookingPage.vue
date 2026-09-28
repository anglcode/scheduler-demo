<script>
import { computed, defineComponent, ref } from 'vue'

const coaches = [
  { id: 'maya', name: 'Maya Chen', title: 'Senior swim coach', initials: 'MC', tone: 'coral', days: [1, 2, 3, 4, 5, 6] },
  { id: 'james', name: 'James Rivera', title: 'Kids & beginner lessons', initials: 'JR', tone: 'blue', days: [1, 3, 4, 5, 6] },
  { id: 'amelia', name: 'Amelia Brooks', title: 'Stroke technique coach', initials: 'AB', tone: 'gold', days: [2, 3, 4, 5, 6] },
]

const weekdayNames = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
const slotSets = [
  ['9:00 AM', '10:30 AM', '1:00 PM', '3:30 PM'],
  ['8:30 AM', '11:00 AM', '2:00 PM', '4:30 PM'],
  ['9:30 AM', '12:00 PM', '2:30 PM', '5:00 PM'],
  ['8:00 AM', '10:00 AM', '1:30 PM', '4:00 PM'],
  ['9:00 AM', '11:30 AM', '2:00 PM', '4:30 PM'],
  ['8:30 AM', '10:30 AM', '1:00 PM', '3:00 PM'],
  ['9:00 AM', '11:00 AM', '1:30 PM', '3:30 PM'],
]
const coachTimeIndexes = {
  maya: [0, 1, 2],
  james: [1, 3],
  amelia: [0, 2, 3],
}

function localDateKey(date) {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

function dateFromKey(key) {
  const [year, month, day] = key.split('-').map(Number)
  return new Date(year, month - 1, day)
}

export default defineComponent({
  name: 'BookingPage',
  emits: ['back'],
  setup() {
    const bookingMode = ref('date')
    const selectedCoachId = ref('')
    const selectedDate = ref('')
    const selectedTime = ref('')
    const monthOffset = ref(0)
    const confirmed = ref(false)

    const today = new Date()
    today.setHours(0, 0, 0, 0)
    const lastBookableDay = new Date(today)
    lastBookableDay.setDate(today.getDate() + 60)

    const visibleMonth = computed(() => new Date(today.getFullYear(), today.getMonth() + monthOffset.value, 1))
    const monthLabel = computed(() => new Intl.DateTimeFormat('en', { month: 'long', year: 'numeric' }).format(visibleMonth.value))
    const calendarCells = computed(() => {
      const firstDay = visibleMonth.value.getDay()
      const daysInMonth = new Date(visibleMonth.value.getFullYear(), visibleMonth.value.getMonth() + 1, 0).getDate()
      return [...Array(firstDay).fill(null), ...Array.from({ length: daysInMonth }, (_, index) => index + 1)]
    })
    const selectedCoach = computed(() => coaches.find((coach) => coach.id === selectedCoachId.value) || null)
    const matchingCoaches = computed(() => {
      if (!selectedDate.value) return coaches
      const day = dateFromKey(selectedDate.value).getDay()
      const selectedTimeIndex = slotSets[day].indexOf(selectedTime.value)
      return coaches.filter((coach) => coach.days.includes(day)
        && (!selectedTime.value || coachTimeIndexes[coach.id].includes(selectedTimeIndex)))
    })
    const availableTimes = computed(() => {
      if (!selectedDate.value) return []
      const times = slotSets[dateFromKey(selectedDate.value).getDay()]
      if (bookingMode.value !== 'coach' || !selectedCoachId.value) return times
      return coachTimeIndexes[selectedCoachId.value].map((index) => times[index])
    })
    const chosenDateLabel = computed(() => selectedDate.value
      ? new Intl.DateTimeFormat('en', { weekday: 'long', month: 'long', day: 'numeric' }).format(dateFromKey(selectedDate.value))
      : '')
    const canGoBack = computed(() => monthOffset.value > 0)
    const canGoForward = computed(() => {
      const nextMonth = new Date(today.getFullYear(), today.getMonth() + monthOffset.value + 1, 1)
      return nextMonth <= lastBookableDay
    })
    const showCoachOptions = computed(() => bookingMode.value === 'date' && selectedDate.value && selectedTime.value)

    function isDateAvailable(day) {
      if (!day) return false
      const date = new Date(visibleMonth.value.getFullYear(), visibleMonth.value.getMonth(), day)
      const coach = coaches.find((item) => item.id === selectedCoachId.value)
      return date >= today && date <= lastBookableDay && date.getDay() !== 0 && (!coach || coach.days.includes(date.getDay()))
    }

    function dateKeyFor(day) {
      return localDateKey(new Date(visibleMonth.value.getFullYear(), visibleMonth.value.getMonth(), day))
    }

    function setMode(mode) {
      bookingMode.value = mode
      selectedCoachId.value = ''
      selectedDate.value = ''
      selectedTime.value = ''
      confirmed.value = false
    }

    function chooseDate(day) {
      if (!isDateAvailable(day)) return
      selectedDate.value = dateKeyFor(day)
      selectedTime.value = ''
      if (bookingMode.value === 'date') selectedCoachId.value = ''
      confirmed.value = false
    }

    function chooseCoach(id) {
      selectedCoachId.value = id
      if (bookingMode.value === 'coach') {
        selectedDate.value = ''
        selectedTime.value = ''
      }
      confirmed.value = false
    }

    function chooseTime(time) {
      selectedTime.value = time
      if (bookingMode.value === 'date') selectedCoachId.value = ''
      confirmed.value = false
    }

    function changeMonth(amount) {
      monthOffset.value += amount
      selectedDate.value = ''
      selectedTime.value = ''
      confirmed.value = false
    }

    function formatDateKey(key) {
      return new Intl.DateTimeFormat('en', { month: 'long', day: 'numeric', year: 'numeric' }).format(dateFromKey(key))
    }

    return {
      bookingMode,
      calendarCells,
      canGoBack,
      canGoForward,
      changeMonth,
      chooseCoach,
      chooseDate,
      chooseTime,
      coaches,
      confirmed,
      dateKeyFor,
      formatDateKey,
      isDateAvailable,
      matchingCoaches,
      monthLabel,
      selectedCoach,
      selectedCoachId,
      selectedDate,
      selectedTime,
      setMode,
      showCoachOptions,
      weekdayNames,
      availableTimes,
      chosenDateLabel,
    }
  },
})
</script>

<template>
  <main class="booking-page">
    <header class="booking-topbar">
      <button class="back-link" type="button" @click="$emit('back')">
        <svg viewBox="0 0 20 20" fill="none" aria-hidden="true"><path d="M16 10H4m5 5-5-5 5-5" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round" /></svg>
        Back
      </button>
      <div class="booking-brand"><span class="mini-wave" aria-hidden="true">〰</span> SWIM LESSON BOOKING</div>
      <span class="topbar-spacer" aria-hidden="true"></span>
    </header>

    <section class="booking-shell" aria-labelledby="booking-title">
      <div class="booking-heading">
        <p class="eyebrow">YOUR NEXT ADVENTURE STARTS HERE</p>
        <h1 id="booking-title">Book a swim lesson</h1>
        <p>Find a time that works for you, then choose the coach you’d love to learn with.</p>
      </div>

      <div class="booking-mode" role="group" aria-label="Choose how to find a lesson">
        <button type="button" :class="{ active: bookingMode === 'date' }" :aria-pressed="bookingMode === 'date'" @click="setMode('date')">
          <svg viewBox="0 0 20 20" fill="none" aria-hidden="true"><rect x="3" y="4.5" width="14" height="12" rx="2" stroke="currentColor" stroke-width="1.4"/><path d="M6.5 3v3M13.5 3v3M3 8h14" stroke="currentColor" stroke-width="1.4" stroke-linecap="round"/></svg>
          Date &amp; time first
        </button>
        <button type="button" :class="{ active: bookingMode === 'coach' }" :aria-pressed="bookingMode === 'coach'" @click="setMode('coach')">
          <svg viewBox="0 0 20 20" fill="none" aria-hidden="true"><circle cx="8" cy="6" r="3" stroke="currentColor" stroke-width="1.4"/><path d="M2.5 16.5a5.5 5.5 0 0 1 11 0M14 5.5a2.5 2.5 0 0 1 0 5m1 2a4 4 0 0 1 2.5 3.7" stroke="currentColor" stroke-width="1.4" stroke-linecap="round"/></svg>
          Coach first
        </button>
      </div>

      <div class="booking-layout">
        <section class="calendar-panel" aria-label="Choose a date">
          <div class="calendar-header">
            <div>
              <span class="panel-kicker">PICK YOUR DAY</span>
              <h2>{{ monthLabel }}</h2>
            </div>
            <div class="month-controls">
              <button type="button" aria-label="Previous month" :disabled="!canGoBack" @click="changeMonth(-1)">‹</button>
              <button type="button" aria-label="Next month" :disabled="!canGoForward" @click="changeMonth(1)">›</button>
            </div>
          </div>
          <div class="calendar-grid calendar-weekdays" aria-hidden="true">
            <span v-for="day in weekdayNames" :key="day">{{ day }}</span>
          </div>
          <div class="calendar-grid calendar-days">
            <span v-for="(day, index) in calendarCells" :key="`day-${index}`" class="calendar-cell">
              <button
                v-if="day"
                type="button"
                :disabled="!isDateAvailable(day)"
                :class="{ chosen: selectedDate === dateKeyFor(day) }"
                :aria-label="`${monthLabel.split(' ')[0]} ${day}${isDateAvailable(day) ? ', available' : ', unavailable'}`"
                :aria-pressed="selectedDate === dateKeyFor(day)"
                @click="chooseDate(day)"
              >{{ day }}</button>
              <span v-else aria-hidden="true"></span>
            </span>
          </div>
          <div class="calendar-legend"><span class="legend-dot"></span> Lessons available Monday–Saturday</div>
        </section>

        <section class="booking-options" aria-live="polite">
          <template v-if="bookingMode === 'date'">
            <div class="panel-title-row">
              <div><span class="panel-kicker">STEP 2</span><h2>{{ selectedDate ? chosenDateLabel : 'Choose a date' }}</h2></div>
              <span v-if="selectedDate" class="picked-check" aria-label="Date selected">✓</span>
            </div>
            <template v-if="selectedDate">
              <p class="options-hint">Available lesson times for your chosen day.</p>
              <div class="time-grid">
                <button v-for="time in availableTimes" :key="time" type="button" :class="{ selected: selectedTime === time }" :aria-pressed="selectedTime === time" @click="chooseTime(time)">{{ time }}</button>
              </div>
            </template>
            <div v-else class="empty-prompt">
              <span class="prompt-icon" aria-hidden="true">⌑</span>
              <p>Choose any highlighted day<br />to see available lesson times.</p>
            </div>
          </template>

          <template v-else>
            <div class="panel-title-row">
              <div><span class="panel-kicker">STEP 1</span><h2>{{ selectedCoach ? selectedCoach.name : 'Choose your coach' }}</h2></div>
              <span v-if="selectedCoach" class="picked-check" aria-label="Coach selected">✓</span>
            </div>
            <div v-if="!selectedCoach" class="coach-list">
              <button v-for="coach in coaches" :key="coach.id" class="coach-option" type="button" @click="chooseCoach(coach.id)">
                <span class="coach-avatar" :class="coach.tone">{{ coach.initials }}</span>
                <span class="coach-copy"><strong>{{ coach.name }}</strong><small>{{ coach.title }}</small></span>
                <svg viewBox="0 0 20 20" fill="none" aria-hidden="true"><path d="M4 10h12m-5-5 5 5-5 5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
              </button>
            </div>
            <template v-else>
              <p class="options-hint">Choose a day on the calendar to see {{ selectedCoach.name.split(' ')[0] }}’s available times.</p>
              <div v-if="selectedDate" class="time-grid">
                <button v-for="time in availableTimes" :key="time" type="button" :class="{ selected: selectedTime === time }" :aria-pressed="selectedTime === time" @click="chooseTime(time)">{{ time }}</button>
              </div>
              <button class="change-coach" type="button" @click="chooseCoach('')">← Choose a different coach</button>
            </template>
          </template>

          <div v-if="showCoachOptions" class="coach-after-time">
            <div class="coach-divider"><span>THEN PICK YOUR COACH</span></div>
            <div class="coach-list compact-coach-list">
              <button v-for="coach in matchingCoaches" :key="coach.id" class="coach-option" :class="{ 'coach-selected': selectedCoachId === coach.id }" type="button" :aria-pressed="selectedCoachId === coach.id" @click="chooseCoach(coach.id)">
                <span class="coach-avatar" :class="coach.tone">{{ coach.initials }}</span>
                <span class="coach-copy"><strong>{{ coach.name }}</strong><small>{{ coach.title }}</small></span>
                <svg v-if="selectedCoachId === coach.id" class="coach-check" viewBox="0 0 20 20" fill="none" aria-hidden="true"><path d="m5 10 3.2 3.2L15.5 6" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/></svg>
                <svg v-else viewBox="0 0 20 20" fill="none" aria-hidden="true"><path d="M4 10h12m-5-5 5 5-5 5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
              </button>
            </div>
          </div>

          <div v-if="bookingMode === 'coach' && selectedCoach && selectedDate && selectedTime" class="booking-summary">
            <span class="panel-kicker">YOUR LESSON</span>
            <p>{{ selectedCoach.name }} <span>·</span> {{ formatDateKey(selectedDate) }} <span>·</span> {{ selectedTime }}</p>
            <button type="button" class="confirm-booking" @click="confirmed = true">Confirm lesson <span aria-hidden="true">→</span></button>
          </div>
          <div v-if="bookingMode === 'date' && selectedDate && selectedTime && selectedCoach" class="booking-summary">
            <span class="panel-kicker">YOUR LESSON</span>
            <p>{{ selectedCoach.name }} <span>·</span> {{ formatDateKey(selectedDate) }} <span>·</span> {{ selectedTime }}</p>
            <button type="button" class="confirm-booking" @click="confirmed = true">Confirm lesson <span aria-hidden="true">→</span></button>
          </div>
          <p v-if="confirmed" class="booking-confirmation" role="status">Lesson reserved in this demo. See you at the pool!</p>
        </section>
      </div>
      <p class="booking-footnote">All lessons are 45 minutes <span>·</span> You can change your booking later</p>
    </section>
  </main>
</template>
