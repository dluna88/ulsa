<script setup>
import { ref, onMounted } from 'vue'

const currentStep = ref('video') // 'video' | 'slider'
const videoEnded = ref(false)
const videoPlayer = ref(null)
const isMuted = ref(false)
const videoProgress = ref(0)

const photos = ref([
  { id: 1, src: '/fotos/001.jpeg', desc: 'La actual biblioteca del campus fue construída sobre lo que originalmente era un área de boliche.' },
  { id: 2, src: '/fotos/002.jpeg', desc: 'Mientras que la plaza cívica albergaba una alberca.' },
  { id: 3, src: '/fotos/003.jpeg', desc: 'El terreno donde actualmente se levanta el edificio del Möet albergaba anteriormente las canchas de basquetbol y futbol al aire libre de la universidad.' },
  { id: 4, src: '/fotos/004.jpeg', desc: 'En los primeros años de la universidad, el área que hoy ocupa el Parque de Innovación era famosa por sus intensas corrientes de aire, motivo por el cual la comunidad universitaria la apodó "El estrecho de Bering".' },
  { id: 5, src: '/fotos/005.jpeg', desc: 'En sus primeros años, la universidad contaba unicamente con el Edificio 1; en esa etapa incial, el campus carecía en gran medida de jardines, árboles y áreas verdes.' },
  { id: 6, src: '/fotos/006.jpeg', desc: 'Antes de la apertura de la entrada principal actual, el acceso al campus se realizaba por la carretera a Arteaga.' },
  { id: 7, src: '/fotos/007.jpeg', desc: 'Los cambios de clase no se anunciaban con un timbre tradicional; sonaba el himno de la universidad y posteriormente se sustituyó por música clásica.' },
  { id: 8, src: '/fotos/008.jpeg', desc: 'Antes de construírse los talleres actuales, el laboratorio de fotografía, conocido como "El cuarto oscuro", se encontraba ubicado a un lado de la capilla.' },
  { id: 9, src: '/fotos/009.jpeg', desc: 'El campus ha registrado nevadas históricas que llegaron a cubrir las instalaciones por completo; en una ocasión las condiciones climáticas obligaron a reprogramar la aplicación de examenes semestrales para un sábado.' },
  { id: 10, src: '/fotos/010.jpeg', desc: 'Originalmente las ceremonias de graduación se celebraban en el auditorio del campus debido a que las generaciones eran pequeñas (de 35 a 40 alumnos). Al incrementar la matrícula a más de 90 graduados por generación, los eventos tuvieron que trasladarse al auditorio del CIZ.' },
  { id: 11, src: '/fotos/011.jpeg', desc: 'La campana del campus se encontraba detrás de la capilla y posteriormente fue reubicada junto con la escultura de San Juan Bautista.' }
])

const currentSlide = ref(0)

const handleVideoEnded = () => {
  videoEnded.value = true
}

const handleTimeUpdate = () => {
  if (videoPlayer.value) {
    const progress = (videoPlayer.value.currentTime / videoPlayer.value.duration) * 100
    videoProgress.value = isNaN(progress) ? 0 : progress
  }
}

const toggleMute = () => {
  if (videoPlayer.value) {
    videoPlayer.value.muted = !videoPlayer.value.muted
    isMuted.value = videoPlayer.value.muted
  }
}

const goToSlider = () => {
  currentStep.value = 'slider'
}

const nextSlide = () => {
  currentSlide.value = (currentSlide.value + 1) % photos.value.length
}

const prevSlide = () => {
  currentSlide.value = (currentSlide.value - 1 + photos.value.length) % photos.value.length
}

const setSlide = (index) => {
  currentSlide.value = index
}

const restartFlow = () => {
  currentStep.value = 'video'
  videoEnded.value = false
  currentSlide.value = 0
  videoProgress.value = 0
  isMuted.value = false
  
  setTimeout(() => {
    if (videoPlayer.value) {
      videoPlayer.value.currentTime = 0
      videoPlayer.value.muted = false
      videoPlayer.value.play().catch(err => console.log('Autoplay request:', err))
    }
  }, 100)
}

onMounted(() => {
  if (videoPlayer.value) {
    // Intentar reproducir con sonido
    videoPlayer.value.muted = false
    isMuted.value = false
    
    videoPlayer.value.play().catch(err => {
      console.log('Autoplay con sonido bloqueado por política del navegador. Intentando reproducción silenciada...', err)
      // Fallback a silenciado para que al menos se reproduzca automáticamente
      isMuted.value = true
      if (videoPlayer.value) {
        videoPlayer.value.muted = true
        videoPlayer.value.play().catch(playErr => {
          console.error('La reproducción automática silenciada también falló:', playErr)
        })
      }
    })
  }
})
</script>

<template>
  <div class="min-h-screen bg-brand-bg text-slate-100 flex flex-col font-sans selection:bg-brand-primary selection:text-white overflow-hidden relative">
    
    <!-- Background Image with Glassmorphism (Visible only in Slider step) -->
    <div class="absolute inset-0 pointer-events-none z-0 transition-opacity duration-700" :class="currentStep === 'slider' ? 'opacity-100' : 'opacity-0'">
      <Transition name="fade">
        <img 
          :key="currentSlide"
          :src="photos[currentSlide].src" 
          class="w-full h-full object-cover scale-110 blur-[60px] opacity-25"
          alt=""
        />
      </Transition>
      <!-- Dark backdrop to blend it with our background and make content readable -->
      <div class="absolute inset-0 bg-brand-bg/60 backdrop-blur-md"></div>
    </div>

    <!-- Background Glow Effects (Visible only in Video step) -->
    <div class="absolute inset-0 pointer-events-none z-0 opacity-20 transition-opacity duration-700" :class="currentStep === 'video' ? 'opacity-100' : 'opacity-0'">
      <div class="absolute top-[-20%] left-[-10%] w-[600px] h-[600px] rounded-full bg-brand-primary/20 blur-[150px]"></div>
      <div class="absolute bottom-[-10%] right-[-10%] w-[600px] h-[600px] rounded-full bg-brand-primary/10 blur-[150px]"></div>
    </div>

    <!-- MAIN TRANSITIONS -->
    <Transition name="fade" mode="out-in">
      
      <!-- STEP 1: VIDEO INTRO -->
      <div v-if="currentStep === 'video'" class="flex-grow flex flex-col items-center justify-center p-6 relative z-10 w-full max-w-5xl mx-auto">
        <div class="w-full flex flex-col items-center space-y-6">
          
          <div class="text-center space-y-2">
            <span class="inline-flex items-center gap-2 px-3 py-1 rounded-full border border-brand-primary/25 bg-brand-primary/5 text-brand-primary/95 text-xs font-semibold tracking-wide uppercase">
              <span class="w-1.5 h-1.5 rounded-full bg-brand-primary animate-pulse"></span>
              Presentación Inicial
            </span>
            <h1 class="text-3xl md:text-4xl font-extrabold tracking-tight bg-clip-text text-transparent bg-gradient-to-r from-white via-slate-200 to-slate-400">
              Descubre Nuestro Campus
            </h1>
          </div>

          <!-- Video Wrapper with premium glassmorphic border and soft shadow -->
          <div class="relative w-full aspect-video rounded-3xl overflow-hidden border border-slate-800 bg-slate-950/80 shadow-2xl shadow-indigo-950/20 group">
            <video 
              ref="videoPlayer"
              src="/video.mp4" 
              class="w-full h-full object-cover"
              autoplay 
              :muted="isMuted"
              playsinline
              @ended="handleVideoEnded"
              @timeupdate="handleTimeUpdate"
            ></video>

            <!-- Video Overlay Gradient -->
            <div class="absolute inset-0 bg-gradient-to-t from-slate-950/40 via-transparent to-transparent pointer-events-none"></div>

            <!-- Mute/Unmute Control button (top right) -->
            <button 
              @click="toggleMute"
              class="absolute top-4 right-4 z-20 p-3 rounded-full bg-slate-900/80 hover:bg-brand-primary/95 text-white backdrop-blur border border-slate-700/50 transition-all duration-200 shadow-lg active:scale-95"
            >
              <svg v-if="isMuted" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="w-5 h-5">
                <path stroke-linecap="round" stroke-linejoin="round" d="M17.25 9.75L19.5 12m0 0l2.25 2.25M19.5 12l2.25-2.25M19.5 12l-2.25 2.25m-10.5-6L4.5 9H1.5v6h3l4.5 3.75V5.25z" />
              </svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="w-5 h-5">
                <path stroke-linecap="round" stroke-linejoin="round" d="M19.114 5.636a9 9 0 010 12.728M16.463 8.288a5.25 5.25 0 010 7.424M6.75 8.25l4.72-4.72a.75.75 0 011.28.53v15.88a.75.75 0 01-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.01 9.01 0 012.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75z" />
              </svg>
            </button>

            <!-- Custom Progress Bar (bottom) -->
            <div class="absolute bottom-0 left-0 w-full h-1.5 bg-slate-900">
              <div 
                class="h-full bg-brand-primary transition-all duration-100 ease-out" 
                :style="{ width: `${videoProgress}%` }"
              ></div>
            </div>
          </div>

          <!-- Action Container -->
          <div class="h-20 flex items-center justify-center w-full">
            <Transition name="scale-fade">
              <button 
                v-if="videoEnded"
                @click="goToSlider"
                class="px-8 py-4 bg-brand-primary hover:bg-brand-primary/90 text-white font-bold text-lg rounded-2xl shadow-xl shadow-brand-primary/30 hover:shadow-brand-primary/50 transition-all duration-300 transform hover:-translate-y-1 active:scale-95 animate-bounce flex items-center gap-2 group"
              >
                Siguiente
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="w-5 h-5 group-hover:translate-x-1 transition-transform">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M13.5 4.5L21 12m0 0l-7.5 7.5M21 12H3" />
                </svg>
              </button>
            </Transition>
          </div>

        </div>
      </div>

      <!-- STEP 2: PHOTO SLIDER -->
      <div v-else class="flex-grow flex flex-col items-center justify-center p-4 md:p-8 relative z-10 w-full max-w-6xl mx-auto h-full">
        
        <!-- Back and info header -->
        <div class="w-full flex items-center justify-between mb-4 md:mb-6">
          <button 
            @click="restartFlow"
            class="flex items-center gap-2 px-4 py-2 text-sm font-medium text-slate-400 hover:text-white bg-slate-900/60 hover:bg-slate-800/80 rounded-xl border border-slate-800 transition-all active:scale-95"
          >
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="w-4 h-4">
              <path stroke-linecap="round" stroke-linejoin="round" d="M9 15L3 9m0 0l6-6M3 9h12a6 6 0 010 12h-3" />
            </svg>
            Ver Video de Nuevo
          </button>
          <span class="text-xs md:text-sm font-semibold text-slate-500 bg-slate-900/60 px-3 py-1 rounded-full border border-slate-800">
            FOTO {{ currentSlide + 1 }} DE {{ photos.length }}
          </span>
        </div>

        <!-- Slider Main Section -->
        <div class="w-full grid grid-cols-1 lg:grid-cols-12 gap-6 items-stretch">
          
          <!-- Image Display (Col-span 8) -->
          <div class="lg:col-span-8 relative aspect-video md:aspect-[16/10] lg:aspect-auto lg:h-[550px] rounded-3xl overflow-hidden border border-slate-800 bg-slate-950 shadow-2xl flex items-center justify-center group">
            
            <!-- Slides Transition -->
            <Transition name="slide" mode="out-in">
              <img 
                :key="currentSlide"
                :src="photos[currentSlide].src" 
                :alt="photos[currentSlide].desc"
                class="w-full h-full object-cover select-none absolute inset-0"
              />
            </Transition>

            <!-- Image overlay for gradients and better visibility -->
            <div class="absolute inset-0 bg-gradient-to-t from-slate-950/60 via-transparent to-slate-950/20 pointer-events-none"></div>

          </div>

          <!-- Description and Side Panel (Col-span 4) -->
          <div class="lg:col-span-4 flex flex-col justify-between p-6 md:p-8 rounded-3xl border border-slate-800 bg-slate-950/70 backdrop-blur-md shadow-2xl relative overflow-hidden">
            
            <!-- Accent glow in container -->
            <div class="absolute top-[-50%] right-[-50%] w-full h-full rounded-full bg-brand-primary/5 blur-[80px] pointer-events-none"></div>

            <div class="space-y-6 relative z-10">
              <Transition name="slide-up" mode="out-in">
                <div :key="currentSlide" class="space-y-4">
                  <p class="text-slate-300 text-base md:text-lg font-light leading-relaxed">
                    {{ photos[currentSlide].desc }}
                  </p>
                </div>
              </Transition>
            </div>

            <!-- Navigation Controls -->
            <div class="mt-8 pt-6 border-t border-slate-900 relative z-10">
              
              <!-- Button actions -->
              <div class="flex items-center justify-between gap-4">
                <button 
                  @click="prevSlide"
                  class="flex-1 py-3 px-4 rounded-xl border border-slate-800 hover:border-slate-700 bg-slate-900/50 hover:bg-slate-900 text-slate-200 font-semibold transition-all duration-200 text-center active:scale-95 text-sm"
                >
                  Anterior
                </button>
                <button 
                  @click="nextSlide"
                  class="flex-1 py-3 px-4 rounded-xl bg-brand-primary hover:bg-brand-primary/90 text-white font-semibold transition-all duration-200 text-center active:scale-95 text-sm shadow-lg shadow-brand-primary/10"
                >
                  Siguiente
                </button>
              </div>

            </div>

          </div>

        </div>

      </div>

    </Transition>

  </div>
</template>

<style>
/* Smooth Fades for step transitions */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Slide animations for images */
.slide-enter-active,
.slide-leave-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}
.slide-enter-from {
  opacity: 0;
  transform: scale(1.02);
}
.slide-leave-to {
  opacity: 0;
  transform: scale(0.98);
}

/* Slide up animation for descriptions */
.slide-up-enter-active,
.slide-up-leave-active {
  transition: all 0.35s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}
.slide-up-enter-from {
  opacity: 0;
  transform: translateY(15px);
}
.slide-up-leave-to {
  opacity: 0;
  transform: translateY(-15px);
}

/* Scale & Fade for next button */
.scale-fade-enter-active,
.scale-fade-leave-active {
  transition: all 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.scale-fade-enter-from {
  opacity: 0;
  transform: scale(0.8) translateY(10px);
}
.scale-fade-leave-to {
  opacity: 0;
  transform: scale(0.8) translateY(10px);
}
</style>
