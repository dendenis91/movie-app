<script lang="ts" setup>
import type { SearchResult } from "~/types/types";
import MovieCard from "~/components/MovieCard.vue";

const movies = ref<SearchResult[] | []>([])

onMounted(async () => {
  // Load local
  const local = JSON.parse(localStorage.getItem('local_favorites') || '[]')
  
  try {
    // Try to load remote
    const { data } = await useFetch<any[]>('/api/favorites')
    if (data.value) {
      const remoteMovies = data.value.map(f => f.movie_data)
      // Merge unique by ID
      const all = [...local, ...remoteMovies]
      movies.value = all.filter((v, i, a) => a.findIndex(t => t.id === v.id) === i)
      return
    }
  } catch (e) {
    console.warn("Offline mode: Using local favorites only.")
  }
  movies.value = local
})

</script>

<template>
  <div>
    <h1>Favorite movies</h1>
    <div
      v-for="movie in movies" :key="movie.id"
      class="grid grid-cols-2 gap-4 p-4 lg:grid-cols-5 md:grid-cols-4 sm:grid-cols-3">
      <MovieCard :movie="movie" />
    </div>
  </div>
</template>

<style scoped>

</style>