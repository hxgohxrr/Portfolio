<script setup>
import { onMounted, onUnmounted, reactive, ref } from 'vue'
import IntroOverlay from './components/IntroOverlay.vue'
import ProfileCard from './components/ProfileCard.vue'
import ProjectsGallery from './components/ProjectsGallery.vue'
import ActionIcons from './components/ActionIcons.vue'
import Toast from './components/Toast.vue'

const profile = reactive({
  name: 'hxgohxrr',
  subtitle: 'Spanish Developer and Producer',
  location: 'Spain',
  handle: 'hxgohxrr',
  status: 'Developing a better world, one line of code at a time.',
  skills: [
    'Vue',
    'TypeScript',
    'UI/UX',
    'Node.js',
    'Audio Production',
    'Web Development',
    'Open Source',
    'Design',
    'Bun',
    'Backend',
    'Frontend',
    'Fullstack',
    'Godot Engine',
  ],
  email: 'me@nullonrise.dev',
  githubUrl: 'https://github.com/hxgohxrr',
  discord: 'hxgohxrr',
  discordUrl: '',
  avatarUrl: '/img/profile.png',
  bgAudioUrl: '/audio/bg.mp3',
})

const projects = ref([
  {
    title: 'Rise Logger',
    description: 'Lightweight logging library for Node.js.',
    imageUrl: '/projects/proj1.png',
    url: 'https://github.com/hxgohxrr/Rise-Logger',
    tags: ['Node.js', 'Logging', 'Library', 'Open Source', 'TypeScript'],
  },
  {
    title: 'Simple Youtube Downloader',
    description: 'Application to download YouTube videos as MP3 or MP4.',
    imageUrl: '/projects/proj2.png',
    url: 'https://github.com/hxgohxrr/Simple-Youtube-Downloader',
    tags: ['Node.js', 'YouTube', 'Downloader', 'Open Source', 'TypeScript'],
  },
  {
    title: 'Eternal Christmas Mod',
    description: 'A mod that brings Christmas vibes all year round in Minecraft.',
    imageUrl: '/projects/proj3.png',
    url: 'https://github.com/hxgohxrr/EternalChristmas',
    tags: ['Minecraft', 'Modding', 'Java', 'Open Source'],
  },
  {
    title: 'Yugen Interactive',
    description: 'Indie Game Studio based in Spain.',
    imageUrl: '/projects/proj4.png',
    url: 'https://store.steampowered.com/developer/yugeninteractive',
    tags: ['Game Development', 'Indie Games', 'Godot Engine'],
  },
])

const showOverlay = ref(true)
const audio = ref(null)
const audioStarted = ref(false)
const isMuted = ref(false)
const toast = reactive({ visible: false, message: '' })
const showScrollTop = ref(false)
let audioCtx = null
let osc = null
let oscGain = null

const showToast = (message) => {
  toast.message = message
  toast.visible = true
  setTimeout(() => (toast.visible = false), 2000)
}

const startAudio = async () => {
  if (!audio.value) return
  try {
    if (!isMuted.value) {
      await audio.value.play()
    }
    audioStarted.value = true
  } catch (err) {
    console.warn('Audio playback blocked, using fallback tone:', err)
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)()
      osc = audioCtx.createOscillator()
      oscGain = audioCtx.createGain()
      osc.type = 'sine'
      osc.frequency.value = 420
      oscGain.gain.value = isMuted.value ? 0 : 0.12
      osc.connect(oscGain).connect(audioCtx.destination)
      osc.start()
    } else if (osc && audioCtx.state === 'suspended' && !isMuted.value) {
      audioCtx.resume()
    }
  }
}

const handleProceed = () => {
  showOverlay.value = false
  startAudio()
}

const toggleMute = () => {
  isMuted.value = !isMuted.value
  localStorage.setItem('nr-mute', String(isMuted.value))
  if (audio.value) {
    audio.value.muted = isMuted.value
    if (!isMuted.value && !audioStarted.value) {
      startAudio()
    }
  }
  if (osc) {
    const targetGain = isMuted.value ? 0 : 0.12
    if (oscGain) {
      oscGain.gain.value = targetGain
    }
    if (!isMuted.value && audioCtx && audioCtx.state === 'suspended') {
      audioCtx.resume()
    }
  }
}

const openGithub = () => {
  if (profile.githubUrl) window.open(profile.githubUrl, '_blank', 'noopener')
}

const copyEmail = async () => {
  await navigator.clipboard.writeText(profile.email)
  showToast('Email copied ✅')
}

const handleDiscord = async () => {
  if (profile.discordUrl) {
    window.open(profile.discordUrl, '_blank', 'noopener')
  } else {
    await navigator.clipboard.writeText(profile.discord)
    showToast('Discord copied ✅')
  }
}

const handleScroll = () => {
  showScrollTop.value = window.scrollY > 200
}

const scrollToTop = () => {
  const start = window.scrollY
  const duration = 900
  const startTime = performance.now()
  const easeOutCubic = (t) => 1 - Math.pow(1 - t, 3)

  const step = (now) => {
    const elapsed = now - startTime
    const progress = Math.min(1, elapsed / duration)
    const eased = easeOutCubic(progress)
    window.scrollTo(0, start * (1 - eased))
    if (progress < 1) requestAnimationFrame(step)
  }

  requestAnimationFrame(step)
}

onMounted(() => {
  audio.value = new Audio(profile.bgAudioUrl)
  audio.value.loop = true
  audio.value.volume = 0.25
  audio.value.preload = 'auto'
  const storedMute = localStorage.getItem('nr-mute')
  if (storedMute === 'true') {
    isMuted.value = true
    audio.value.muted = true
  } else {
    audio.value.muted = false
  }
  window.addEventListener('scroll', handleScroll, { passive: true })
  handleScroll()
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})
</script>

<template>
  <div class="page">
    <div class="bg-layer"></div>
    <div class="orb orb-a"></div>
    <div class="orb orb-b"></div>
    <div class="orb orb-c"></div>
    <transition name="intro-fade">
      <IntroOverlay v-if="showOverlay" @proceed="handleProceed" />
    </transition>

    <main class="content">
      <section class="hero">
        <ProfileCard :profile="profile" />
        <ActionIcons
          :is-muted="isMuted"
          @open-github="openGithub"
          @copy-email="copyEmail"
          @handle-discord="handleDiscord"
          @toggle-mute="toggleMute"
        />
      </section>

      <ProjectsGallery :projects="projects" />
      <footer class="footer-note">
        <p>
          This is also open source in
          <a href="https://github.com/NullOnRise/Portfolio/tree/main" target="_blank" rel="noopener">GitHub</a>
        </p>
      </footer>
    </main>

    <Toast :toast="toast" />

    <button
      v-show="showScrollTop"
      class="scroll-top-btn"
      aria-label="Back to top"
      @click="scrollToTop"
    >
      <i class="ri-arrow-up-line" aria-hidden="true"></i>
    </button>
  </div>
</template>

