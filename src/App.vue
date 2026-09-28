<script>
import { defineComponent, ref } from 'vue'
import BookingPage from './components/BookingPage.vue'

export default defineComponent({
  name: 'App',
  components: { BookingPage },
  setup() {
    const selectedRole = ref('')

    function selectRole(role) {
      selectedRole.value = role
    }

    return { selectedRole, selectRole }
  },
})
</script>

<template>
  <BookingPage v-if="selectedRole === 'client'" @back="selectedRole = ''" />
  <main v-else class="welcome-page">
    <section class="welcome-card" aria-labelledby="page-title">
      <div class="welcome-art" aria-hidden="true">
        <div class="sun-glow"></div>
        <div class="pool-ring pool-ring-one"></div>
        <div class="pool-ring pool-ring-two"></div>
        <svg class="water-lines" viewBox="0 0 720 190" fill="none">
          <path d="M-15 54c44 0 44-24 88-24s44 24 88 24 44-24 88-24 44 24 88 24 44-24 88-24 44 24 88 24 44-24 88-24 44 24 88 24" />
          <path d="M-15 112c44 0 44-24 88-24s44 24 88 24 44-24 88-24 44 24 88 24 44-24 88-24 44 24 88 24 44-24 88-24 44 24 88 24" />
          <path d="M-15 170c44 0 44-24 88-24s44 24 88 24 44-24 88-24 44 24 88 24 44-24 88-24 44 24 88 24 44-24 88-24 44 24 88 24" />
        </svg>
        <div class="art-caption">
          <span class="caption-mark">✳</span>
          <span>Make a splash<br />with every lesson</span>
        </div>
      </div>

      <div class="welcome-content">
        <div class="brand-mark" aria-hidden="true">
          <svg viewBox="0 0 40 40" fill="none">
            <circle cx="20" cy="20" r="19" fill="currentColor" />
            <path d="M9 22c3.2 0 3.2-3 6.4-3s3.2 3 6.4 3 3.2-3 6.4-3 3.2 3 6.4 3M9 28c3.2 0 3.2-3 6.4-3s3.2 3 6.4 3 3.2-3 6.4-3 3.2 3 6.4 3" stroke="white" stroke-width="2.2" stroke-linecap="round" />
            <circle cx="20" cy="12" r="3" fill="white" />
          </svg>
        </div>
        <p class="eyebrow">WELCOME TO YOUR NEXT SWIM</p>
        <h1 id="page-title">Swim lesson<br /><span>booking</span></h1>
        <p class="intro">Your journey to confident swimming starts here. Choose how you’d like to sign in.</p>

        <div class="login-options" aria-label="Choose a login type">
          <button
            class="login-button employee-button"
            type="button"
            :aria-pressed="selectedRole === 'employee'"
            @click="selectRole('employee')"
          >
            <span class="button-icon employee-icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none">
                <path d="M16 20v-1.7a3.3 3.3 0 0 0-3.3-3.3H6.3A3.3 3.3 0 0 0 3 18.3V20M9.5 11a4 4 0 1 0 0-8 4 4 0 0 0 0 8ZM16 3.2a4 4 0 0 1 0 7.6m5 9.2v-1.7a3.3 3.3 0 0 0-2.5-3.2" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
              </svg>
            </span>
            <span class="button-copy"><strong>Employee login</strong><small>For instructors &amp; staff</small></span>
            <svg class="button-arrow" viewBox="0 0 20 20" fill="none" aria-hidden="true"><path d="M4 10h12m-5-5 5 5-5 5" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round" /></svg>
          </button>

          <button
            class="login-button client-button"
            type="button"
            :aria-pressed="selectedRole === 'client'"
            @click="selectRole('client')"
          >
            <span class="button-icon client-icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none">
                <path d="M3 20.2c0-3.7 3-6.7 6.7-6.7h4.6c3.7 0 6.7 3 6.7 6.7M12 11a4.5 4.5 0 1 0 0-9 4.5 4.5 0 0 0 0 9Z" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
              </svg>
            </span>
            <span class="button-copy"><strong>Client login</strong><small>Book and manage lessons</small></span>
            <svg class="button-arrow" viewBox="0 0 20 20" fill="none" aria-hidden="true"><path d="M4 10h12m-5-5 5 5-5 5" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round" /></svg>
          </button>
        </div>

        <p v-if="selectedRole" class="selection-note" role="status">
          {{ selectedRole === 'employee' ? 'Employee login' : 'Client login' }} selected
        </p>
        <p v-else class="secure-note">
          <svg viewBox="0 0 16 16" fill="none" aria-hidden="true"><path d="M4.5 7V5.2a3.5 3.5 0 1 1 7 0V7m-8 .2h9v6.3h-9V7.2Z" stroke="currentColor" stroke-width="1.3" stroke-linecap="round" stroke-linejoin="round" /></svg>
          A safe, easy place to plan your swim
        </p>
      </div>
    </section>
    <footer class="page-footer">GOOD THINGS HAPPEN IN THE WATER <span>·</span> SWIM, LEARN, GROW</footer>
  </main>
</template>
