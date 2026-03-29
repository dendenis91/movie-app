<script lang="ts" setup>
import type { SearchResults, SearchResult } from "~/types/types";
import { ref, watch } from 'vue';
import { useRouter } from 'vue-router';
import Pagination from "~/components/Pagination.vue";

const movies = ref<SearchResult[]>([]);
const showCustomAlert = ref(false);
const alertMessage = ref('');
const isSearching = ref(false);
const router = useRouter();

const currentPage = ref<number>(1);
const totalPages = ref<number>(1);
const totalResults = ref<number>(0);

const sortOrder = ref<'none' | 'desc' | 'asc'>('none');

const { data, status, error } = await useFetch<SearchResults>(() => `/api/movies?page=${currentPage.value}&sort=${sortOrder.value}`, {
  watch: [currentPage, sortOrder]
});

watchEffect(() => {
  if (data.value) {
    movies.value = data.value.results;
    totalPages.value = data.value.total_pages;
    totalResults.value = data.value.total_results;
  }
});
</script>

<template>
  <div>
    <transition name="fade">
      <div v-if="showCustomAlert" class="fixed left-1/2 top-8 transform -translate-x-1/2 z-50">
        <div
          class="bg-[#FF1E1E] text-white px-6 py-3 rounded-full shadow-lg text-lg font-semibold flex items-center gap-2 animate-bounce">
          <svg class="h-6 w-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path d="M12 8v4m0 4h.01M21 12c0 4.97-4.03 9-9 9s-9-4.03-9-9 4.03-9 9-9 9 4.03 9 9z" stroke-linecap="round"
              stroke-linejoin="round" stroke-width="2" />
          </svg>
          {{ alertMessage }}
        </div>
      </div>
    </transition>

    <div class="grid grid-cols-2 gap-4 p-4 lg:grid-cols-5 md:grid-cols-4 sm:grid-cols-3">
      <div v-for="movie in movies" :key="movie.id">
        <MovieCard :movie="movie" />
      </div>
    </div>

    <div class="p-4 flex justify-center">
      <Pagination :currentPage="currentPage" :totalPages="totalPages" @update:currentPage="currentPage = $event" />
    </div>
    <div v-if="status === 'pending'" class="text-center mt-4">Loading...</div>
    <div v-if="error" class="text-center mt-4 text-red-600">Unable to load movies. Please check your connection.</div>
  </div>
</template>

<style scoped>
/* your existing styles */
</style>
