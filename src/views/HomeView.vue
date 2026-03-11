<script setup lang="ts">
import { ref, nextTick, watch, onUnmounted } from 'vue'

const luisImg = new URL('@/assets/avatars/luis/image.png', import.meta.url).href
const mauroImg = new URL('@/assets/avatars/mauro/image.png', import.meta.url).href

const CORRECT_AVATAR = 'mauro'

const avatars = [
  { id: 'carlos', name: 'Carlos', image: null, initials: 'CA' },
  { id: 'mauro', name: 'Mauro', image: mauroImg, initials: 'MA' },
  { id: 'luis', name: 'Luis', image: luisImg, initials: 'LU' },
]

const showDemo = ref(false)
const step = ref(1)
const selectedAvatar = ref<string | null>(null)
const avatarError = ref(false)
const shakeError = ref(false)
const topic = ref('')
const topicError = ref(false)

const isGenerating = ref(false)
const isVideoUnlocked = ref(false)
const generationProgress = ref(0)
const timeLeft = ref(35)
let generationInterval: ReturnType<typeof setInterval> | null = null
let generationTimeout: ReturnType<typeof setTimeout> | null = null

function openDemo() {
  showDemo.value = true
  step.value = 1
  selectedAvatar.value = null
  avatarError.value = false
  shakeError.value = false
  topic.value = ''
  topicError.value = false
  isGenerating.value = false
  isVideoUnlocked.value = false
  if (generationInterval) clearInterval(generationInterval)
  if (generationTimeout) clearTimeout(generationTimeout)
}

function closeDemo() {
  showDemo.value = false
  if (generationInterval) clearInterval(generationInterval)
  if (generationTimeout) clearTimeout(generationTimeout)
}

onUnmounted(() => {
  if (generationInterval) clearInterval(generationInterval)
  if (generationTimeout) clearTimeout(generationTimeout)
})

function selectAvatar(id: string) {
  selectedAvatar.value = id
  avatarError.value = false
  shakeError.value = false
}

async function confirmAvatar() {
  if (!selectedAvatar.value) return
  if (selectedAvatar.value === CORRECT_AVATAR) {
    step.value = 2
  } else {
    avatarError.value = true
    shakeError.value = true
    await nextTick()
    setTimeout(() => { shakeError.value = false }, 600)
  }
}

function confirmTopic() {
  if (!topic.value.trim()) {
    topicError.value = true
    return
  }
  topicError.value = false
  step.value = 3

  isGenerating.value = true
  isVideoUnlocked.value = false
  timeLeft.value = 35
  generationProgress.value = 0

  if (generationInterval) clearInterval(generationInterval)
  if (generationTimeout) clearTimeout(generationTimeout)

  generationInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--
      generationProgress.value = Math.min(100, Math.round(((35 - timeLeft.value) / 35) * 100))
    }
  }, 1000)

  generationTimeout = setTimeout(() => {
    if (generationInterval) clearInterval(generationInterval)
    isGenerating.value = false
  }, 35000)
}

watch(step, (val) => {
  if (val === 3 && !document.querySelector('script[src*="uxjz9v8jyh"]')) {
    const s = document.createElement('script')
    s.src = 'https://fast.wistia.com/embed/uxjz9v8jyh.js'
    s.async = true
    s.type = 'module'
    document.head.appendChild(s)
  }
})
</script>

<template>
  <!-- LANDING -->
  <div class="landing">
    <div class="landing__noise"></div>
    <div class="landing__glow landing__glow--top"></div>
    <div class="landing__glow landing__glow--bottom"></div>

    <div class="landing__content">
      <span class="landing__badge">FACELESS CONTENT · IA</span>
      <h1 class="landing__title">
        Andersson<br /><span class="landing__title--accent">Boscán</span>
      </h1>
      <p class="landing__subtitle">
        Descubre cómo crear videos virales sin mostrar tu cara,<br />
        con inteligencia artificial que trabaja por ti.
      </p>
      <button class="btn-cta" @click="openDemo">
        <span>Ver Demo en Vivo</span>
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M5 12h14M12 5l7 7-7 7"/></svg>
      </button>
    </div>
  </div>

  <!-- DEMO OVERLAY -->
  <Transition name="overlay">
    <div v-if="showDemo" class="overlay" @click.self="closeDemo">
      <div class="demo-card">
        <!-- Close -->
        <button class="demo-card__close" @click="closeDemo" aria-label="Cerrar">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6 6 18M6 6l12 12"/></svg>
        </button>

        <!-- Step indicator -->
        <div class="steps">
          <div class="step" :class="{ active: step >= 1, done: step > 1 }">
            <span class="step__dot">{{ step > 1 ? '✓' : '1' }}</span>
            <span class="step__label">Avatar</span>
          </div>
          <div class="step__line" :class="{ active: step > 1 }"></div>
          <div class="step" :class="{ active: step >= 2, done: step > 2 }">
            <span class="step__dot">{{ step > 2 ? '✓' : '2' }}</span>
            <span class="step__label">Tema</span>
          </div>
          <div class="step__line" :class="{ active: step > 2 }"></div>
          <div class="step" :class="{ active: step >= 3 }">
            <span class="step__dot">3</span>
            <span class="step__label">Video</span>
          </div>
        </div>

        <!-- STEP 1: Avatar -->
        <Transition name="fade" mode="out-in">
          <div v-if="step === 1" key="step1" class="demo-step">
            <h2 class="demo-step__title">Elige tu Avatar</h2>
            <p class="demo-step__subtitle">Selecciona el avatar con el que quieres crear contenido</p>

            <div class="avatar-grid">
              <button
                v-for="av in avatars"
                :key="av.id"
                class="avatar-card"
                :class="{
                  'avatar-card--selected': selectedAvatar === av.id,
                  'avatar-card--error': avatarError && selectedAvatar === av.id,
                  shake: shakeError && selectedAvatar === av.id,
                }"
                @click="selectAvatar(av.id)"
              >
                <div class="avatar-card__img-wrap">
                  <img v-if="av.image" :src="av.image" :alt="av.name" />
                  <div v-else class="avatar-card__initials">{{ av.initials }}</div>
                </div>
                <span class="avatar-card__name">{{ av.name }}</span>
                <div v-if="selectedAvatar === av.id" class="avatar-card__check">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>
                </div>
              </button>
            </div>

            <Transition name="fade">
              <p v-if="avatarError" class="error-msg">
                <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="12" cy="12" r="10"/><path d="M12 8v4M12 16h.01"/></svg>
                Avatar incorrecto. Intenta con otro.
              </p>
            </Transition>

            <button class="btn-primary" :disabled="!selectedAvatar" @click="confirmAvatar">
              Continuar
            </button>
          </div>
        </Transition>

        <!-- STEP 2: Tema -->
        <Transition name="fade" mode="out-in">
          <div v-if="step === 2" key="step2" class="demo-step">
            <h2 class="demo-step__title">¿Sobre qué tema?</h2>
            <p class="demo-step__subtitle">Escribe el tema de tu video y la IA lo creará por ti</p>

            <div class="topic-wrap">
              <label class="topic-label">Tema del contenido</label>
              <input
                v-model="topic"
                class="topic-input"
                :class="{ 'topic-input--error': topicError }"
                type="text"
                placeholder="Ej: Fitness, Finanzas, Viajes, Tecnología..."
                maxlength="80"
                @keydown.enter="confirmTopic"
                @input="topicError = false"
              />
              <Transition name="fade">
                <p v-if="topicError" class="error-msg">
                  <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="12" cy="12" r="10"/><path d="M12 8v4M12 16h.01"/></svg>
                  Escribe un tema para continuar.
                </p>
              </Transition>
            </div>

            <button class="btn-primary" @click="confirmTopic">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polygon points="5 3 19 12 5 21 5 3"/></svg>
              Generar Video
            </button>
          </div>
        </Transition>

        <!-- STEP 3: Video -->
        <Transition name="fade" mode="out-in">
          <div v-if="step === 3" key="step3" class="demo-step demo-step--video">
            
            <!-- GENERATING STATE -->
            <template v-if="isGenerating">
              <div class="generating-state">
                <div class="spinner"></div>
                <h2 class="demo-step__title">Generando video...</h2>
                <p class="demo-step__subtitle" style="max-width: 440px; margin: 0 auto; line-height: 1.6;">
                  Nos demoraremos aprox de <strong>30 a 45 segundos</strong>, es una prueba.<br />
                  <span style="font-size: 0.8rem; opacity: 0.6; display: block; margin-top: 0.5rem;">Un video más largo podría demorar hasta 4 minutos.</span>
                </p>
                <div class="progress-wrap">
                  <div class="progress-bar">
                    <div class="progress-bar__fill" :style="{ width: generationProgress + '%' }"></div>
                  </div>
                  <span class="time-left">{{ timeLeft }}s restantes</span>
                </div>
              </div>
            </template>

            <!-- RESULT STATE -->
            <template v-else>
              <div class="video-header">
                <span class="video-badge">VIDEO GENERADO</span>
                <h2 class="demo-step__title">Tu video está listo</h2>
                <p class="demo-step__subtitle">
                  Tema: <strong>"{{ topic }}"</strong>
                </p>
              </div>

              <div class="video-wrap" :class="{ 'video-wrap--blurred': !isVideoUnlocked }">
                <Transition name="fade">
                  <div v-if="!isVideoUnlocked" class="video-overlay">
                    <button class="btn-cta-watch" @click="isVideoUnlocked = true">
                      <svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><polygon points="5 3 19 12 5 21 5 3"/></svg>
                      Ver Video
                    </button>
                  </div>
                </Transition>
                <wistia-player media-id="uxjz9v8jyh" aspect="1.7777777777777777"></wistia-player>
              </div>

              <!-- Automation CTA -->
            <div class="automation">
              <div class="automation__divider">
                <span>POTENCIA TU ALCANCE</span>
              </div>
              <p class="automation__title">
                Este video podría publicarse automáticamente en tus redes sociales
              </p>
              <p class="automation__desc">
                Con un solo clic, tu contenido llega a miles de personas — sin grabar tu cara, sin cámaras, sin complicaciones.
              </p>
              <div class="social-cards">
                <div class="social-card social-card--instagram">
                  <div class="social-card__icon">
                    <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24"><path d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zM12 0C8.741 0 8.333.014 7.053.072 2.695.272.273 2.69.073 7.052.014 8.333 0 8.741 0 12c0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98C8.333 23.986 8.741 24 12 24c3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98C15.668.014 15.259 0 12 0zm0 5.838a6.162 6.162 0 1 0 0 12.324 6.162 6.162 0 0 0 0-12.324zM12 16a4 4 0 1 1 0-8 4 4 0 0 1 0 8zm6.406-11.845a1.44 1.44 0 1 0 0 2.881 1.44 1.44 0 0 0 0-2.881z"/></svg>
                  </div>
                  <span class="social-card__name">Instagram</span>
                  <span class="social-card__detail">Reels · Stories · Feed</span>
                </div>
                <div class="social-card social-card--twitter">
                  <div class="social-card__icon">
                    <svg viewBox="0 0 24 24" fill="currentColor" width="22" height="22"><path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/></svg>
                  </div>
                  <span class="social-card__name">Twitter / X</span>
                  <span class="social-card__detail">Posts · Threads · Clips</span>
                </div>
                <div class="social-card social-card--facebook">
                  <div class="social-card__icon">
                    <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24"><path d="M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z"/></svg>
                  </div>
                  <span class="social-card__name">Facebook</span>
                  <span class="social-card__detail">Videos · Reels · Posts</span>
                </div>
              </div>
              <p class="automation__tagline">
                Automatización total. Contenido constante. Resultados reales.
              </p>
            </div>
            </template>
          </div>
        </Transition>
      </div>
    </div>
  </Transition>
</template>

<style lang="scss" scoped>
/* ─────────── LANDING ─────────── */
.landing {
  min-height: 100vh;
  background: $primary-dark;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.landing__noise {
  position: absolute;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.03'/%3E%3C/svg%3E");
  background-size: 200px;
  pointer-events: none;
  opacity: 0.4;
}

.landing__glow {
  position: absolute;
  border-radius: 50%;
  filter: blur(120px);
  pointer-events: none;

  &--top {
    width: 600px;
    height: 600px;
    background: rgba($primary, 0.18);
    top: -200px;
    left: -100px;
  }

  &--bottom {
    width: 500px;
    height: 500px;
    background: rgba($secondary, 0.12);
    bottom: -150px;
    right: -100px;
  }
}

.landing__content {
  position: relative;
  z-index: 1;
  text-align: center;
  padding: 2rem;
  max-width: 700px;
  animation: fadeUp 0.8s ease both;
}

.landing__badge {
  display: inline-block;
  font-family: $font-principal;
  font-size: 0.7rem;
  font-weight: 700;
  letter-spacing: 0.2em;
  color: $secondary;
  border: 1px solid rgba($secondary, 0.4);
  border-radius: 100px;
  padding: 0.35rem 1rem;
  margin-bottom: 1.5rem;
}

.landing__title {
  font-family: $font-principal;
  font-size: clamp(3rem, 9vw, 6.5rem);
  font-weight: 900;
  line-height: 1;
  color: $white;
  margin-bottom: 1.5rem;
  letter-spacing: -0.02em;

  &--accent {
    background: linear-gradient(135deg, $primary, $secondary);
    -webkit-background-clip: text;
    background-clip: text;
    -webkit-text-fill-color: transparent;
  }
}

.landing__subtitle {
  font-family: $font-secondary;
  font-size: clamp(0.95rem, 2vw, 1.15rem);
  color: rgba($white, 0.55);
  line-height: 1.7;
  margin-bottom: 2.5rem;
}

.btn-cta {
  display: inline-flex;
  align-items: center;
  gap: 0.6rem;
  background: linear-gradient(135deg, $primary, darken($primary, 10%));
  color: $white;
  border: none;
  border-radius: 100px;
  font-family: $font-principal;
  font-size: 1rem;
  font-weight: 700;
  padding: 0.9rem 2.2rem;
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  box-shadow: 0 0 40px rgba($primary, 0.4);

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 0 60px rgba($primary, 0.6);
  }

  &:active {
    transform: translateY(0);
  }
}

/* ─────────── OVERLAY ─────────── */
.overlay {
  position: fixed;
  inset: 0;
  background: rgba($primary-dark, 0.85);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 1rem;
  overflow-y: auto;
}

.demo-card {
  background: #0d1b3e;
  border: 1px solid rgba($primary, 0.2);
  border-radius: 24px;
  padding: 2.5rem;
  width: 100%;
  max-width: 640px;
  position: relative;
  box-shadow: 0 30px 80px rgba(0, 0, 0, 0.5), 0 0 0 1px rgba($primary, 0.1);
  my: 2rem;
}

.demo-card__close {
  position: absolute;
  top: 1.2rem;
  right: 1.2rem;
  background: rgba($white, 0.06);
  border: 1px solid rgba($white, 0.1);
  border-radius: 50%;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: rgba($white, 0.6);
  transition: background 0.2s, color 0.2s;

  &:hover {
    background: rgba($white, 0.12);
    color: $white;
  }
}

/* ─────────── STEPS INDICATOR ─────────── */
.steps {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
  margin-bottom: 2.5rem;
}

.step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;

  &.done .step__dot {
    background: $secondary;
    border-color: $secondary;
    font-size: 0.7rem;
  }

  &.active:not(.done) .step__dot {
    background: $primary;
    border-color: $primary;
    color: $white;
  }
}

.step__dot {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: 2px solid rgba($white, 0.2);
  background: transparent;
  color: rgba($white, 0.4);
  font-family: $font-principal;
  font-size: 0.8rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.step__label {
  font-family: $font-principal;
  font-size: 0.65rem;
  font-weight: 600;
  letter-spacing: 0.1em;
  color: rgba($white, 0.35);
  text-transform: uppercase;

  .active & {
    color: rgba($white, 0.8);
  }
}

.step__line {
  width: 60px;
  height: 2px;
  background: rgba($white, 0.1);
  margin: 0 0.5rem;
  margin-bottom: 1.2rem;
  border-radius: 2px;
  transition: background 0.4s ease;

  &.active {
    background: $primary;
  }
}

/* ─────────── DEMO STEP ─────────── */
.demo-step {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;

  &--video {
    gap: 0;
  }
}

.demo-step__title {
  font-family: $font-principal;
  font-size: 1.6rem;
  font-weight: 800;
  color: $white;
  line-height: 1.2;
}

.demo-step__subtitle {
  font-family: $font-secondary;
  font-size: 0.9rem;
  color: rgba($white, 0.5);
  margin-top: -1rem;

  strong {
    color: $secondary;
    font-weight: 600;
  }
}

/* ─────────── AVATAR GRID ─────────── */
.avatar-grid {
  display: flex;
  gap: 1.2rem;
  justify-content: center;
  flex-wrap: wrap;
}

.avatar-card {
  background: rgba($white, 0.04);
  border: 2px solid rgba($white, 0.08);
  border-radius: 18px;
  padding: 1.2rem 1rem;
  width: 140px;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.8rem;
  transition: all 0.25s ease;
  position: relative;

  &:hover {
    border-color: rgba($primary, 0.5);
    background: rgba($primary, 0.08);
    transform: translateY(-3px);
  }

  &--selected {
    border-color: $primary !important;
    background: rgba($primary, 0.12) !important;
  }

  &--error {
    border-color: $alert-error !important;
    background: $alert-error-bg !important;
  }
}

.avatar-card__img-wrap {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  overflow: hidden;
  background: rgba($white, 0.06);
  flex-shrink: 0;

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}

.avatar-card__initials {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: $font-principal;
  font-size: 1.4rem;
  font-weight: 800;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  color: $white;
}

.avatar-card__name {
  font-family: $font-principal;
  font-size: 0.9rem;
  font-weight: 700;
  color: rgba($white, 0.85);
}

.avatar-card__check {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: $primary;
  display: flex;
  align-items: center;
  justify-content: center;
  color: $white;
}

/* ─────────── TOPIC INPUT ─────────── */
.topic-wrap {
  width: 100%;
  max-width: 420px;
  text-align: left;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.topic-label {
  font-family: $font-principal;
  font-size: 0.78rem;
  font-weight: 600;
  letter-spacing: 0.05em;
  color: rgba($white, 0.5);
  text-transform: uppercase;
}

.topic-input {
  width: 100%;
  background: rgba($white, 0.06);
  border: 2px solid rgba($white, 0.1);
  border-radius: 12px;
  padding: 0.9rem 1.1rem;
  font-family: $font-secondary;
  font-size: 1rem;
  color: $white;
  outline: none;
  transition: border-color 0.2s ease;

  &::placeholder {
    color: rgba($white, 0.25);
  }

  &:focus {
    border-color: $primary;
    background: rgba($primary, 0.06);
  }

  &--error {
    border-color: $alert-error;
  }
}

/* ─────────── BUTTONS ─────────── */
.btn-primary {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  background: linear-gradient(135deg, $primary, darken($primary, 8%));
  color: $white;
  border: none;
  border-radius: 100px;
  font-family: $font-principal;
  font-size: 0.95rem;
  font-weight: 700;
  padding: 0.85rem 2rem;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s, opacity 0.2s;
  box-shadow: 0 0 30px rgba($primary, 0.35);

  &:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 0 50px rgba($primary, 0.5);
  }

  &:disabled {
    opacity: 0.35;
    cursor: not-allowed;
  }
}

/* ─────────── ERROR ─────────── */
.error-msg {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  font-family: $font-secondary;
  font-size: 0.85rem;
  color: $alert-error;
  background: $alert-error-bg;
  border: 1px solid rgba($alert-error, 0.3);
  border-radius: 8px;
  padding: 0.5rem 0.9rem;
}

/* ─────────── GENERATING STATE ─────────── */
.generating-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  padding: 3rem 1rem;
}

.spinner {
  width: 54px;
  height: 54px;
  border: 4px solid rgba($primary, 0.15);
  border-left-color: $primary;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.progress-wrap {
  width: 100%;
  max-width: 320px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.6rem;
  margin-top: 1rem;
}

.progress-bar {
  width: 100%;
  height: 6px;
  background: rgba($white, 0.08);
  border-radius: 100px;
  overflow: hidden;
  position: relative;
}

.progress-bar__fill {
  height: 100%;
  background: $primary;
  border-radius: 100px;
  transition: width 1s linear;
  width: 0%;
}

.time-left {
  font-family: $font-secondary;
  font-size: 0.8rem;
  font-weight: 500;
  color: rgba($white, 0.5);
  letter-spacing: 0.02em;
}

/* ─────────── VIDEO SECTION ─────────── */
.video-header {
  text-align: center;
  margin-bottom: 1.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;

  .demo-step__subtitle {
    margin-top: 0;
  }
}

.video-badge {
  display: inline-block;
  font-family: $font-principal;
  font-size: 0.65rem;
  font-weight: 700;
  letter-spacing: 0.2em;
  color: $secondary;
  border: 1px solid rgba($secondary, 0.4);
  border-radius: 100px;
  padding: 0.3rem 0.8rem;
  margin-bottom: 0.8rem;
}

.video-wrap {
  width: 100%;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.4);
  position: relative;
  transition: filter 0.5s ease;

  &--blurred {
    wistia-player {
      filter: blur(12px) brightness(0.7);
      pointer-events: none;
    }
  }

  wistia-player {
    display: block;
    width: 100%;
    transition: filter 0.6s ease;
  }
}

.video-overlay {
  position: absolute;
  inset: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(#0d1b3e, 0.2);
  backdrop-filter: blur(2px);
}

.btn-cta-watch {
  display: inline-flex;
  align-items: center;
  gap: 0.6rem;
  background: $primary;
  color: $white;
  border: none;
  border-radius: 100px;
  font-family: $font-principal;
  font-size: 1.1rem;
  font-weight: 800;
  padding: 1rem 2.4rem;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 0 40px rgba($primary, 0.5);

  &:hover {
    transform: translateY(-3px) scale(1.02);
    box-shadow: 0 0 60px rgba($primary, 0.7);
  }
}

/* ─────────── AUTOMATION SECTION ─────────── */
.automation {
  width: 100%;
  margin-top: 2rem;
  text-align: center;
}

.automation__divider {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;

  &::before,
  &::after {
    content: '';
    flex: 1;
    height: 1px;
    background: rgba($white, 0.1);
  }

  span {
    font-family: $font-principal;
    font-size: 0.6rem;
    font-weight: 700;
    letter-spacing: 0.2em;
    color: rgba($white, 0.3);
    white-space: nowrap;
  }
}

.automation__title {
  font-family: $font-principal;
  font-size: 1.15rem;
  font-weight: 800;
  color: $white;
  line-height: 1.4;
  margin-bottom: 0.75rem;
}

.automation__desc {
  font-family: $font-secondary;
  font-size: 0.88rem;
  color: rgba($white, 0.5);
  line-height: 1.6;
  max-width: 460px;
  margin: 0 auto 1.5rem;
}

.social-cards {
  display: flex;
  gap: 0.8rem;
  justify-content: center;
  flex-wrap: wrap;
  margin-bottom: 1.5rem;
}

.social-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.4rem;
  padding: 1rem 1.2rem;
  border-radius: 14px;
  border: 1px solid transparent;
  min-width: 110px;
  transition: transform 0.2s ease;

  &:hover {
    transform: translateY(-3px);
  }

  &--instagram {
    background: rgba(225, 48, 108, 0.08);
    border-color: rgba(225, 48, 108, 0.25);
    color: #e1306c;
  }

  &--twitter {
    background: rgba(255, 255, 255, 0.05);
    border-color: rgba(255, 255, 255, 0.15);
    color: rgba($white, 0.85);
  }

  &--facebook {
    background: rgba(66, 103, 178, 0.1);
    border-color: rgba(66, 103, 178, 0.3);
    color: #4267b2;
  }
}

.social-card__icon {
  display: flex;
  align-items: center;
  justify-content: center;
}

.social-card__name {
  font-family: $font-principal;
  font-size: 0.78rem;
  font-weight: 700;
  color: $white;
}

.social-card__detail {
  font-family: $font-secondary;
  font-size: 0.68rem;
  color: rgba($white, 0.4);
}

.automation__tagline {
  font-family: $font-principal;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  color: rgba($white, 0.3);
  text-transform: uppercase;
}

/* ─────────── TRANSITIONS ─────────── */
.overlay-enter-active,
.overlay-leave-active {
  transition: opacity 0.3s ease;

  .demo-card {
    transition: transform 0.3s ease, opacity 0.3s ease;
  }
}

.overlay-enter-from,
.overlay-leave-to {
  opacity: 0;

  .demo-card {
    transform: translateY(20px) scale(0.97);
    opacity: 0;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateX(10px);
}

/* ─────────── ANIMATIONS ─────────── */
@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(24px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes shake {

  0%,
  100% {
    transform: translateX(0);
  }

  20% {
    transform: translateX(-8px);
  }

  40% {
    transform: translateX(8px);
  }

  60% {
    transform: translateX(-5px);
  }

  80% {
    transform: translateX(5px);
  }
}

.shake {
  animation: shake 0.5s ease;
}
</style>
