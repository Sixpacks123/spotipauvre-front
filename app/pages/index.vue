<template>
  <div class="min-h-screen bg-black text-white font-bold overflow-x-hidden">
    <!-- Header décalé -->
    <div class="bg-gradient-to-r from-pink-500 to-pink-600 shadow-2xl border-b-4 border-lime-400">
      <UContainer class="py-8">
        <div class="text-center">
          <h1 class="text-5xl md:text-6xl font-black text-white drop-shadow-lg animate-pulse">
            🎵 Spotipauvre seulement 25 par jour 🎯
          </h1>
          <p class="text-2xl md:text-3xl text-white mt-4 font-bold tracking-wide">
            PRÊT POUR LE CHAOS SONORE ? 🚀
          </p>
        </div>
      </UContainer>
    </div>

    <UContainer class="py-8">
      <!-- Bouton Roulette Géant -->
      <div class="text-center mb-12">
        <button
          @click="playRandomSong"
          class="bg-gradient-to-r from-orange-500 to-red-500 hover:from-orange-400 hover:to-red-400 
                 text-white font-black text-2xl md:text-3xl px-12 py-8 rounded-full 
                 shadow-2xl transform hover:scale-110 transition-all duration-300 
                 border-4 border-lime-400 hover:border-pink-400 animate-bounce"
        >
          🎰 ROULETTE RUSSE MUSICALE 🎰
        </button>
      </div>

      <!-- Lecteur Audio Principal -->
      <div class="max-w-4xl mx-auto mb-12">
        <div 
          v-if="currentTrack"
          class="bg-gradient-to-r from-lime-500 to-green-500 rounded-3xl p-8 shadow-2xl border-4 border-pink-500"
        >
          <div class="text-center mb-6">
            <div class="text-6xl mb-4">{{ currentTrack.emoji || '🎵' }}</div>
            <h2 class="text-3xl font-black text-black mb-2">{{ currentTrack.title }}</h2>
            <p class="text-xl font-bold text-gray-800">{{ currentTrack.artist }}</p>
          </div>
          
          <div class="flex items-center justify-center gap-6 mb-6">
            <button
              @click="toggleCurrentTrack"
              class="bg-black hover:bg-gray-800 text-white font-black text-xl px-8 py-4 
                     rounded-full shadow-lg transform hover:scale-110 transition-all duration-200
                     border-3 border-white"
            >
              {{ isCurrentPlaying ? '⏸️ STOP LE BEAT !' : '▶️ BALANCE LE SON !' }}
            </button>
          </div>

          <!-- Contrôle Volume -->
          <div class="text-center">
            <label class="block text-black font-black text-lg mb-2">
              🔊 PUISSANCE DU BEAT 🔊
            </label>
            <input
              type="range"
              min="0"
              max="100"
              v-model="volume"
              class="w-64 h-3 bg-black rounded-lg appearance-none cursor-pointer"
            />
            <div class="text-black font-bold mt-2">{{ volume }}%</div>
          </div>
        </div>
        
        <div 
          v-else
          class="bg-gradient-to-r from-gray-700 to-gray-800 rounded-3xl p-8 text-center border-4 border-orange-500"
        >
          <div class="text-6xl mb-4">😴</div>
          <h2 class="text-2xl font-black text-orange-400">SÉLECTIONNE TON CHAOS SONORE !</h2>
        </div>
      </div>

      <!-- Liste Arsenal Sonore -->
      <div class="max-w-6xl mx-auto">
        <h2 class="text-4xl font-black text-center mb-8 text-transparent bg-clip-text bg-gradient-to-r from-pink-400 to-lime-400">
          🎯 ARSENAL SONORE 🎯
        </h2>

        <!-- Loading -->
        <div
          v-if="pending"
          class="space-y-4"
        >
          <div
            v-for="i in 5"
            :key="i"
            class="h-20 bg-gradient-to-r from-gray-700 to-gray-800 rounded-2xl animate-pulse"
          />
        </div>

        <!-- Error -->
        <div
          v-else-if="error"
          class="bg-gradient-to-r from-red-600 to-pink-600 rounded-3xl p-8 text-center border-4 border-orange-500"
        >
          <div class="text-6xl mb-4">
            💥
          </div>
          <h3 class="text-2xl font-black text-white mb-2">
            ERREUR CRITIQUE !
          </h3>
          <p class="text-white font-bold">
            {{ error.message }}
          </p>
        </div>

        <!-- Songs List -->
        <div
          v-else
          class="grid grid-cols-1 md:grid-cols-2 gap-6"
        >
          <div
            v-for="(track, index) in tracks"
            :key="track.id"
            :class="[
              'rounded-3xl p-6 shadow-2xl transform hover:scale-105 transition-all duration-300 cursor-pointer border-4',
              index % 3 === 0 ? 'bg-gradient-to-br from-pink-500 to-pink-600 border-lime-400' : 
              index % 3 === 1 ? 'bg-gradient-to-br from-lime-500 to-green-500 border-orange-500' : 
              'bg-gradient-to-br from-orange-500 to-red-500 border-pink-400',
              currentTrack?.id === track.id ? 'ring-4 ring-white animate-pulse' : ''
            ]"
            @click="selectTrack(track)"
          >
            <div class="flex items-center gap-4">
              <div class="text-5xl">
                {{ track.emoji || '🎵' }}
              </div>
              
              <div class="flex-1 min-w-0">
                <h3 class="font-black text-xl text-white truncate mb-1">
                  {{ track.title }}
                </h3>
                <p class="font-bold text-lg text-gray-200 truncate">
                  {{ track.artist }}
                </p>
              </div>

              <div class="flex flex-col gap-2">
                <button
                  :class="[
                    'px-4 py-2 rounded-full font-black text-sm transition-all duration-200 transform hover:scale-110',
                    currentTrack?.id === track.id && isCurrentPlaying 
                      ? 'bg-black text-white border-2 border-white' 
                      : 'bg-white text-black border-2 border-black hover:bg-gray-200'
                  ]"
                  @click.stop="toggleTrack(track)"
                >
                  {{ currentTrack?.id === track.id && isCurrentPlaying ? '⏸️ STOP' : '▶️ GO!' }}
                </button>
              </div>
            </div>
          </div>

          <!-- Empty State -->
          <div
            v-if="!tracks?.length"
            class="col-span-full bg-gradient-to-r from-gray-700 to-gray-800 rounded-3xl p-12 text-center border-4 border-orange-500"
          >
            <div class="text-8xl mb-6">
              😵
            </div>
            <h3 class="text-3xl font-black text-orange-400 mb-4">
              ARSENAL VIDE !
            </h3>
            <p class="text-xl text-gray-300 font-bold">
              Ajoute des bombes sonores dans ton CMS Strapi ! 💣
            </p>
          </div>
        </div>
      </div>
    </UContainer>

    <!-- Footer fun -->
    <footer class="bg-gradient-to-r from-purple-600 to-pink-600 border-t-4 border-lime-400 mt-16">
      <UContainer class="py-8">
        <div class="text-center">
          <h2 class="text-3xl font-black text-white mb-4">
            🚀 Découvre des sons bizarres comme jamais ! 🚀
          </h2>
          <div class="flex justify-center gap-8 text-2xl">
            <a href="#" class="hover:scale-125 transition-transform duration-200">🎮</a>
            <a href="#" class="hover:scale-125 transition-transform duration-200">🎪</a>
            <a href="#" class="hover:scale-125 transition-transform duration-200">🎨</a>
            <a href="#" class="hover:scale-125 transition-transform duration-200">🎭</a>
            <a href="#" class="hover:scale-125 transition-transform duration-200">🎯</a>
          </div>
        </div>
      </UContainer>
    </footer>

    <!-- Lecteur fixe en bas -->
    <ClientOnly>
      <div class="fixed bottom-0 left-0 right-0 bg-gradient-to-r from-black to-gray-900 border-t-4 border-lime-400 shadow-2xl">
        <MusicFlow
          :options="{
            autoplay: false
          }"
        />
      </div>
      <template #fallback>
        <div class="fixed bottom-0 left-0 right-0 bg-gradient-to-r from-black to-gray-900 border-t-4 border-lime-400 shadow-2xl">
          <div class="flex items-center justify-center h-16 text-lime-400 font-black text-lg">
            🎵 CHARGEMENT DU CHAOS... 🎵
          </div>
        </div>
      </template>
    </ClientOnly>

    <!-- Padding pour le lecteur fixe -->
    <div class="h-24" />
  </div>
</template>

<script setup lang="ts">
import type { TMusicFlow } from 'vue-music-flow'

// Composable Vue Music Flow
const { onPlayAsPlaylist } = useMusicFlow()

// Fetch songs from Strapi
const { find } = useStrapi()

// State
const tracks = ref<TMusicFlow[]>([])
const currentTrack = ref<TMusicFlow | null>(null)
const isCurrentPlaying = ref(false)
const volume = ref(75)

// Initialize reactive data with proper error handling
const songs = ref(null)
const pending = ref(true)
const error = ref(null)

// Transformer les données Strapi avec vérifications
const transformSongsToTracks = (strapiSongs: unknown): TMusicFlow[] => {
  console.log('Données reçues:', strapiSongs)

  if (!strapiSongs) {
    console.log('Pas de données reçues')
    return []
  }

  // Vérifier si c'est un tableau ou un objet avec data
  let songsArray = []

  if (Array.isArray(strapiSongs)) {
    songsArray = strapiSongs
  } else if (strapiSongs && typeof strapiSongs === 'object' && 'data' in strapiSongs) {
    const data = (strapiSongs as { data: unknown }).data
    if (Array.isArray(data)) {
      songsArray = data
    } else if (data && typeof data === 'object' && 'data' in data) {
      const nestedData = (data as { data: unknown }).data
      if (Array.isArray(nestedData)) {
        songsArray = nestedData
      }
    }
  }

  console.log('Tableau de chansons:', songsArray)

  if (!songsArray.length) {
    console.log('Aucune chanson trouvée')
    return []
  }

  return songsArray.map((song: unknown, index: number) => {
    console.log('Processing song:', song)

    // Gestion des attributs Strapi v4 (song.attributes)
    const songObj = song && typeof song === 'object' ? song as Record<string, unknown> : {}
    const attributes = songObj.attributes && typeof songObj.attributes === 'object'
      ? songObj.attributes as Record<string, unknown>
      : {}
    const songData = Object.keys(attributes).length > 0 ? attributes : songObj

    // Gestion des médias populés
    let audioUrl = ''
    
    const songMedia = songData?.song || songData?.audio || songData?.media
    
    // Traitement de l'audio
    if (songMedia && typeof songMedia === 'object') {
      const mediaObj = songMedia as Record<string, unknown>
      if (mediaObj.url && typeof mediaObj.url === 'string') {
        audioUrl = mediaObj.url.startsWith('http')
          ? mediaObj.url
          : `http://localhost:1337${mediaObj.url}`
      }
    } else if (typeof songMedia === 'string') {
      audioUrl = songMedia
    }

    // Fallback vers les anciens champs
    if (!audioUrl) {
      audioUrl = (songData?.audioUrl || songData?.url || songData?.URL_Audio || '') as string
    }

    return {
      id: songObj?.id || songData?.id || index + 1,
      audio: audioUrl,
      title: (songData?.title || songData?.Title || songData?.titre || songData?.Titre || `Titre ${index + 1}`) as string,
      artist: (songData?.artist || songData?.Artist || songData?.artiste || 'Artiste inconnu') as string,
      artwork: 'https://via.placeholder.com/300x300/6b7280/ffffff?text=♪',
      album: (songData?.album || 'Album') as string,
      emoji: getRandomEmoji(index),
      genre: (songData?.genre || 'Inconnu') as string,
      original: {
        source: 'Strapi'
      }
    }
  })
}

// Fonction pour obtenir des emojis aléatoires basés sur l'index
const getRandomEmoji = (index: number): string => {
  const emojis = ['🎵', '🎶', '🎤', '🎧', '🎼', '🎸', '🥁', '🎺', '🎷', '🎻', '🎹', '🔥', '⚡', '💫', '🌟', '💥', '🚀', '🎯', '🎪', '🎨']
  return emojis[index % emojis.length]
}

// Load data with proper error handling
const loadSongs = async () => {
  try {
    pending.value = true
    error.value = null

    // Use the new URL with populate parameter to get media fields
    const response = await find('songs', {
      populate: ['song'] // Populate both audio and artwork media fields
    })
    songs.value = response

    if (songs.value) {
      tracks.value = transformSongsToTracks(songs.value)
      console.log('Tracks transformées:', tracks.value)
    }
  } catch (err) {
    console.error('Erreur lors du chargement:', err)
    error.value = err
    tracks.value = []
  } finally {
    pending.value = false
  }
}

// Watcher avec vérifications supplémentaires
watch(songs, (newSongs) => {
  console.log('Watch triggered avec:', newSongs)

  if (newSongs !== null && newSongs !== undefined) {
    try {
      tracks.value = transformSongsToTracks(newSongs)
      console.log('Tracks transformées:', tracks.value)
    } catch (watchError) {
      console.error('Erreur lors de la transformation dans le watcher:', watchError)
      tracks.value = []
    }
  } else {
    console.log('newSongs est null ou undefined')
    tracks.value = []
  }
}, { immediate: false })

// Functions
function selectTrack(track: TMusicFlow) {
  console.log('Track sélectionné:', track)
  currentTrack.value = track
  isCurrentPlaying.value = false
}

function toggleCurrentTrack() {
  if (!currentTrack.value) return
  
  if (isCurrentPlaying.value) {
    isCurrentPlaying.value = false
    // Logique pour arrêter la lecture
  } else {
    isCurrentPlaying.value = true
    onPlayAsPlaylist(tracks.value, currentTrack.value)
  }
}

function toggleTrack(track: TMusicFlow) {
  console.log('Toggle track:', track)
  if (currentTrack.value?.id === track.id && isCurrentPlaying.value) {
    isCurrentPlaying.value = false
  } else {
    currentTrack.value = track
    isCurrentPlaying.value = true
    onPlayAsPlaylist(tracks.value, track)
  }
}

function playRandomSong() {
  if (!tracks.value?.length) {
    console.log('Aucune piste disponible')
    return
  }

  const randomIndex = Math.floor(Math.random() * tracks.value.length)
  const randomTrack = tracks.value[randomIndex]
  console.log('Piste aléatoire:', randomTrack)

  currentTrack.value = randomTrack
  isCurrentPlaying.value = true
  onPlayAsPlaylist(tracks.value, randomTrack)
}

// Initialize avec gestion d'erreur
onMounted(async () => {
  console.log('=== DEBUG MOUNT ===')
  await loadSongs()
})

// SEO
useHead({
  title: '🎵 MUZIK ATTACK 🎯 - Chaos Sonore Garanti !',
  meta: [
    { name: 'description', content: 'Le lecteur musical le plus décalé du web ! Prépare-toi au chaos sonore !' }
  ]
})
</script>

<style scoped>
/* Import de polices fun */
@import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@400;600;700&display=swap');

/* Animations custom */
@keyframes neon-glow {
  0%, 100% { 
    text-shadow: 0 0 5px #FF1493, 0 0 10px #FF1493, 0 0 15px #FF1493;
  }
  50% { 
    text-shadow: 0 0 10px #32CD32, 0 0 20px #32CD32, 0 0 30px #32CD32;
  }
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
}

@keyframes rainbow {
  0% { filter: hue-rotate(0deg); }
  100% { filter: hue-rotate(360deg); }
}

/* Styles globaux pour cette page */
.min-h-screen {
  font-family: 'Fredoka', cursive;
  background: linear-gradient(45deg, #000000, #1a1a1a, #000000);
  background-size: 400% 400%;
  animation: gradient-shift 8s ease infinite;
}

@keyframes gradient-shift {
  0%, 100% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
}

/* Boutons avec effets spéciaux */
button:hover {
  animation: float 0.6s ease-in-out infinite;
}

/* Effet neon sur les titres */
h1 {
  animation: neon-glow 2s ease-in-out infinite alternate;
}

/* Cards avec effet rainbow au hover */
.hover\:scale-105:hover {
  animation: rainbow 2s linear infinite;
}

/* Scrollbar custom */
::-webkit-scrollbar {
  width: 12px;
}

::-webkit-scrollbar-track {
  background: #000000;
}

::-webkit-scrollbar-thumb {
  background: linear-gradient(45deg, #FF1493, #32CD32, #FF4500);
  border-radius: 6px;
}

::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(45deg, #32CD32, #FF4500, #FF1493);
}

/* Range input custom */
input[type="range"] {
  appearance: none;
  -webkit-appearance: none;
  background: transparent;
}

input[type="range"]::-webkit-slider-track {
  background: #000000;
  height: 12px;
  border-radius: 6px;
  border: 2px solid #32CD32;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 24px;
  width: 24px;
  border-radius: 50%;
  background: linear-gradient(45deg, #FF1493, #FF4500);
  cursor: pointer;
  border: 3px solid #32CD32;
  box-shadow: 0 0 10px #FF1493;
}

input[type="range"]::-webkit-slider-thumb:hover {
  box-shadow: 0 0 20px #32CD32;
  transform: scale(1.2);
}

/* Effets pour les emojis */
.text-6xl, .text-5xl {
  transition: all 0.3s ease;
  display: inline-block;
}

.text-6xl:hover, .text-5xl:hover {
  transform: rotate(360deg) scale(1.2);
}

/* Ajustements pour le lecteur fixe */
body {
  padding-bottom: 0;
}

/* Effet de pulsation pour les éléments actifs */
.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

/* Effet de bounce custom */
.animate-bounce {
  animation: bounce 1s infinite;
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(-25%);
    animation-timing-function: cubic-bezier(0.8, 0, 1, 1);
  }
  50% {
    transform: translateY(0);
    animation-timing-function: cubic-bezier(0, 0, 0.2, 1);
  }
}
</style>
