<template>
  <div class="space-y-4">
    <div class="flex items-center justify-between mb-6">
      <h2 class="text-xl font-semibold">
        Playlist ({{ songs.length }} {{ songs.length > 1 ? 'morceaux' : 'morceau' }})
      </h2>
      <div class="flex items-center gap-2">
        <UButton
          icon="i-lucide-shuffle"
          variant="outline"
          size="sm"
          @click="$emit('shuffle')"
        >
          Mélanger
        </UButton>
        <UButton
          icon="i-lucide-play-circle"
          color="green"
          variant="solid"
          size="sm"
          @click="$emit('play-all')"
        >
          Tout lire
        </UButton>
      </div>
    </div>

    <div class="grid gap-3">
      <UCard
        v-for="(song, index) in songs"
        :key="song.id"
        :class="{
          'ring-2 ring-primary-500 bg-primary-50 dark:bg-primary-950': currentSong?.id === song.id,
          'hover:shadow-md': currentSong?.id !== song.id
        }"
        class="transition-all duration-200 cursor-pointer group"
        @click="$emit('play-song', song)"
      >
        <div class="flex items-center gap-4">
          <!-- Track Number / Play Button -->
          <div class="w-8 h-8 flex items-center justify-center relative">
            <span
              v-if="currentSong?.id !== song.id"
              class="text-sm text-gray-500 group-hover:opacity-0 transition-opacity"
            >
              {{ index + 1 }}
            </span>
            <UButton
              v-if="currentSong?.id !== song.id"
              :icon="'i-lucide-play'"
              size="xs"
              color="green"
              variant="solid"
              class="opacity-0 group-hover:opacity-100 transition-opacity absolute"
              @click.stop="$emit('play-song', song)"
            />
            <UIcon
              v-else
              :name="isPlaying ? 'i-lucide-volume-2' : 'i-lucide-pause'"
              :class="isPlaying ? 'text-green-500 animate-pulse' : 'text-gray-500'"
              class="w-4 h-4"
            />
          </div>

          <!-- Song Info -->
          <div class="flex-1 min-w-0">
            <div class="flex items-center gap-2">
              <h3
                :class="{
                  'text-primary-600 dark:text-primary-400 font-semibold': currentSong?.id === song.id,
                  'font-medium': currentSong?.id !== song.id
                }"
                class="truncate"
              >
                {{ song.Titre }}
              </h3>
              <UBadge
                v-if="!song.url"
                label="Indisponible"
                color="red"
                variant="soft"
                size="xs"
              />
            </div>
            <p class="text-sm text-gray-600 dark:text-gray-400 truncate">
              {{ song.artist }}
            </p>
          </div>

          <!-- Genre -->
          <div class="hidden sm:block">
            <UBadge
              v-if="song.genre"
              :label="song.genre"
              variant="soft"
              size="xs"
            />
          </div>

          <!-- Duration placeholder -->
          <div class="hidden md:block w-12 text-right">
            <span class="text-sm text-gray-500">--:--</span>
          </div>

          <!-- Actions -->
{{ song }}
          <div class="flex items-center gap-1 opacity-0 group-hover:opacity-100 transition-opacity">
            <UButton
              v-if="song.url"
              :icon="currentSong?.id === song.id && isPlaying ? 'i-lucide-pause' : 'i-lucide-play'"
              size="xs"
              :color="currentSong?.id === song.id && isPlaying ? 'red' : 'green'"
              variant="outline"
              @click.stop="$emit('play-song', song)"
            />
            <UButton
              icon="i-lucide-heart"
              size="xs"
              variant="ghost"
              color="gray"
              @click.stop="$emit('toggle-favorite', song)"
            />
            <UButton
              icon="i-lucide-more-horizontal"
              size="xs"
              variant="ghost"
              color="gray"
              @click.stop="$emit('show-menu', song)"
            />
          </div>
        </div>
      </UCard>
    </div>

    <!-- Empty state -->
    <div
      v-if="!songs.length"
      class="text-center py-12"
    >
      <UIcon
        name="i-lucide-music-off"
        class="w-16 h-16 mx-auto text-gray-400 mb-4"
      />
      <h3 class="text-lg font-medium text-gray-900 dark:text-gray-100 mb-2">
        Aucune musique disponible
      </h3>
      <p class="text-gray-600 dark:text-gray-400">
        Ajoutez des musiques avec des URLs valides dans Strapi
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
interface Song {
  id: number
  Titre: string
  artist: string
  genre?: string
  url?: string
}

interface Props {
  songs: Song[]
  currentSong?: Song | null
  isPlaying?: boolean
}

defineProps<Props>()

defineEmits<{
  'play-song': [song: Song]
  'toggle-favorite': [song: Song]
  'show-menu': [song: Song]
  'shuffle': []
  'play-all': []
}>()
</script>
