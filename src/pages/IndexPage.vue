<!-- src/pages/IndexPage.vue -->
<template>
  <div class="container">
    <h1>Filmes Mais Recentes</h1>
    <div v-if="movies.length" class="card-container">
      <q-card v-for="movie in movies" :key="movie.id" class="my-card">
        <q-img
          :src="'https://image.tmdb.org/t/p/w500' + movie.poster_path"
          :alt="movie.title"
          class="q-mb-md"
        />
        <q-card-section>
          <div class="text-h6">{{ movie.title }}</div>
          <div><strong>Lançamento:</strong> {{ movie.release_date }}</div>
          <div><strong>Gêneros:</strong> {{ getGenres(movie.genre_ids) }}</div>
          <div><strong>Nota:</strong> {{ movie.vote_average }}</div>
          <q-separator />
          <div class="text-body2">{{ movie.overview }}</div>
        </q-card-section>
      </q-card>
    </div>
    <div v-else>
      <q-spinner color="primary" />
    </div>
    <div class="pagination q-mt-md">
      <q-btn
        @click="changePage(currentPage - 1)"
        :disabled="currentPage <= 1"
        label="Anterior"
        color="primary"
      />
      <span>Página {{ currentPage }} de {{ totalPages }}</span>
      <q-btn
        @click="changePage(currentPage + 1)"
        :disabled="currentPage >= totalPages"
        label="Próximo"
        color="primary"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useQuasar } from "quasar";

// Lista de filmes, gêneros e paginação
const movies = ref([]);
const genres = ref([]);
const currentPage = ref(1);
const totalPages = ref(1);

// Função para buscar filmes com base na página atual
const fetchMovies = async (page = 1) => {
  try {
    const response = await fetch(
      `https://api.themoviedb.org/3/movie/now_playing?language=en-US&page=${page}`,
      options
    );
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    const data = await response.json();
    movies.value = data.results;
    totalPages.value = data.total_pages;
    currentPage.value = page;
  } catch (error) {
    console.error("Erro ao buscar filmes:", error);
  }
};

// Função para buscar gêneros
const fetchGenres = async () => {
  try {
    const response = await fetch(
      "https://api.themoviedb.org/3/genre/movie/list?language=en-US",
      options
    );
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    const data = await response.json();
    genres.value = data.genres;
  } catch (error) {
    console.error("Erro ao buscar gêneros:", error);
  }
};

// Função para alterar a página
const changePage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    fetchMovies(page);
  }
};

// Configurações do fetch com cabeçalhos personalizados
const options = {
  method: "GET",
  headers: {
    accept: "application/json",
    Authorization:
      "Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJjM2NjNGQzNTk0NDhkMmNiZjI4NDFjYTRkNTNjMGY2OCIsIm5iZiI6MTcyMTU3MzI5Mi45NzY2MTYsInN1YiI6IjYzMGJjZWUxMjc5MGJmMDA3YzEyMjI1MCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.0xJB6210SJ1wSsOWO3gdtPGcDXxN6r6jvfLPgffwpao",
  },
};

// Buscar filmes e gêneros ao montar o componente
onMounted(() => {
  fetchMovies();
  fetchGenres();
});

// Função para buscar gêneros dos filmes
const getGenres = (genreIds) => {
  return genreIds
    .map((id) => {
      const genre = genres.value.find((g) => g.id === id);
      return genre ? genre.name : "Desconhecido";
    })
    .join(", ");
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.card-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.my-card {
  max-width: 300px;
  margin: 10px;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
