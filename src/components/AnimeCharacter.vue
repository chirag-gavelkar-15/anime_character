<template>
  <div class="min-h-screen bg-gray-900 text-white">
    <div class="container mx-auto py-5">
      <h1 class="text-4xl font-bold text-center mb-6">Anime Character Search</h1>

      <div class="mb-6">
        <input
          v-model="searchQuery"
          @input="handleSearchInput"
          class="w-full p-3 rounded-lg bg-gray-800 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
          type="text"
          placeholder="Search for a character..."
        />
      </div>

      <p v-if="!loading && totalCount > 0" class="mb-6 text-lg text-center">
        Total {{ totalCount }} matching anime characters found
      </p>

      <hr class="mb-4">

      <div v-if="loading" class="text-center text-lg text-blue-500">
        Loading...
      </div>

      <div v-if="noResults && !loading" class="text-center text-lg text-red-500">
        No results found for "{{ searchQuery }}".
      </div>

      <div v-if="!loading && characters.length > 0">
        <div class="flex flex-col gap-6">
          <div
            v-for="character in characters"
            :key="character.mal_id"
            class="flex flex-col bg-gray-800 rounded-lg overflow-hidden shadow-lg"
          >
            <div class="flex items-center p-3">
              <img
                :src="character.images.jpg.image_url"
                alt="character image"
                class="h-24 object-cover rounded"
              />
              <div class="ml-3">
                <h2 class="text-base mb-1 mx-1">{{ character.name }}</h2>
                <p class="inline-block mx-1 bg-gray-700 text-white px-3 py-1 rounded-full text-xs"
                   v-for="nickname in character.nicknames" :key="nickname">
                  {{ nickname }}
                </p>
              </div>
            </div>
            <div class="flex items-center justify-between px-3 py-2 bg-gray-700">
              <div class="flex items-center">
                <i class="fa fa-heart text-red-400 text-lg" aria-hidden="true"></i>
                <p class="text-white-400 mx-1 text-sm">{{ character.favorites }}</p>
              </div>
              <a
                :href="character.url"
                target="_blank"
                rel="noopener noreferrer"
                class="text-blue-500 underline text-sm"
              >
                <i class="fa fa-arrow-right text-xl font-bold" aria-hidden="true"></i>
              </a>
            </div>
          </div>
        </div>

        <div class="flex justify-between items-center mt-6 space-y-2">
          <button
            @click="fetchCharacters(currentPage - 1)"
            :disabled="currentPage === 1"
            class="px-4 py-2 bg-blue-500 rounded-lg hover:bg-blue-600 disabled:bg-gray-500 text-sm"
          >
            Back
          </button>
          <span class="text-lg">Page {{ currentPage }}</span>
          <button
            @click="fetchCharacters(currentPage + 1)"
            :disabled="!hasNextPage"
            class="px-4 py-2 bg-blue-500 rounded-lg hover:bg-blue-600 disabled:bg-gray-500 text-sm"
          >
            Next
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      searchQuery: '',
      characters: [],
      currentPage: 1,
      hasNextPage: true,
      noResults: false,
      totalCount: 0,
      loading: false, 
    };
  },
  methods: {
    async fetchCharacters(page = 1) {
      this.loading = true;
      this.noResults = false;
      try {
        const response = await axios.get(
          `https://api.jikan.moe/v4/characters?page=${page}&limit=15&q=${this.searchQuery}&order_by=favorites&sort=desc`
        );
        
        this.characters = response.data.data.map((character) => ({
          mal_id: character.mal_id,
          name: character.name,
          anime: character.title,
          images: character.images,
          favorites: character.favorites,
          url: character.url,
          nicknames: character.nicknames,
        }));
        
        this.totalCount = response.data.pagination.items.total;
        this.currentPage = page;
        this.hasNextPage = response.data.pagination.has_next_page;
        this.noResults = this.characters.length === 0;
      } catch (error) {
        console.error('Error fetching characters:', error);
      } finally {
        this.loading = false; 
      }
    },
    handleSearchInput() {
      this.currentPage = 1; 
      this.fetchCharacters();
    }
  },
  mounted() {
    this.fetchCharacters();
  },
};
</script>
