<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { SearchResult } from '~/types/types'
import LoadingSpinner from "~/components/LoadingSpinner.vue";

const props = defineProps<{
  movie: Partial<SearchResult>
  readOnly?: boolean
}>()

const isFavorite = ref(false)
const loading = ref(true)
const isOfflineMode = ref(false)

const checkLocalFavorite = () => {
  const favorites = JSON.parse(localStorage.getItem('local_favorites') || '[]')
  if (props.movie.id) {
    isFavorite.value = favorites.some((f: any) => f.id === props.movie.id)
  }
}

onMounted(async () => {
  try {
    // 1. Try to fetch from API (Supabase)
    const { data, error } = await useFetch<any[]>('/api/favorites', { timeout: 2000 })
    
    if (error.value || !data.value) {
      throw new Error('Supabase unreachable')
    }

    if (props.movie.id) {
      isFavorite.value = data.value.some(f => f.movie_id == props.movie.id)
    }
  } catch (err) {
    // 2. Fallback to LocalStorage if Supabase fails
    isOfflineMode.value = true
    checkLocalFavorite()
  } finally {
  loading.value = false
  }
})

const toggleFavorite = async () => {
  if (!props.movie.id) return

  if (isOfflineMode.value) {
    const favorites = JSON.parse(localStorage.getItem('local_favorites') || '[]')
    if (isFavorite.value) {
      const newFavorites = favorites.filter((f: any) => f.id !== props.movie.id)
      localStorage.setItem('local_favorites', JSON.stringify(newFavorites))
      isFavorite.value = false
    } else {
      favorites.push(props.movie)
      localStorage.setItem('local_favorites', JSON.stringify(favorites))
      isFavorite.value = true
    }
    return
  }

  // Try API first, fallback to local on error
  const method = isFavorite.value ? 'DELETE' : 'POST'
  const { error } = await useFetch('/api/favorites', { method, body: { movie_id: props.movie.id, movie_data: props.movie } })
  
  if (error.value) {
    isOfflineMode.value = true
    toggleFavorite() // retry in offline mode
  } else {
    isFavorite.value = !isFavorite.value
  }
}
</script>
<template>
  <!-- Show loading spinner on first mount -->
  <div v-if="loading">
    <LoadingSpinner></LoadingSpinner>
  </div>

  <!-- Show favorite toggle button -->
  <button
    v-else
    :class="[
      'rounded px-3 py-1 border text-sm transition',
      isFavorite ? 'bg-red-500 text-white' : 'bg-gray-200 text-black'
    ]"
    @click="toggleFavorite"
    :disabled="readOnly"
  >
    {{ isFavorite ? 'Remove from Favorites' : 'Add to Favorites' }}
  </button>
</template>