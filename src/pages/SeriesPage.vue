<template>
  <div class="container">
    <div class="title">Séries Mais Recentes</div>
    <div v-if="loading" class="spinner-container">
      <q-spinner color="primary" />
    </div>
    <div v-else-if="tvShows.length" class="card-container">
      <q-card v-for="show in tvShows" :key="show.id" class="my-card">
        <q-img
          :src="'https://image.tmdb.org/t/p/w500' + show.poster_path"
          :alt="show.name"
          class="q-mb-md show-poster"
        />
        <q-card-section class="card-content">
          <div class="text-h6 show-title">{{ show.name }}</div>
          <div class="show-info">
            <strong>Primeiro Episódio:</strong> {{ show.first_air_date }}
          </div>
          <div class="show-info">
            <strong>Gêneros:</strong> {{ getGenres(show.genre_ids) }}
          </div>
          <div class="show-info">
            <strong>Nota:</strong> {{ show.vote_average.toFixed(1) }}
          </div>
          <q-separator />
          <div class="text-body2 show-description">
            {{ truncatedDescriptions[show.id] }}
          </div>
          <q-btn
            v-if="show.overview.length > 150"
            @click="toggleDescription(show.id)"
            color="primary"
            flat
            class="toggle-btn"
          >
            {{ expandedDescriptions[show.id] ? "Ver menos" : "Ver mais" }}
          </q-btn>
        </q-card-section>
      </q-card>
    </div>
    <div v-else class="no-shows">
      <div>Nenhuma série encontrada.</div>
    </div>
    <div class="pagination q-mt-md">
      <q-btn
        @click="changePage(currentPage - 1)"
        :disabled="currentPage <= 1"
        label="Anterior"
        color="primary"
        class="pagination-btn"
      />
      <span class="pagination-info"
        >Página {{ currentPage }} de {{ totalPages }}</span
      >
      <q-btn
        @click="changePage(currentPage + 1)"
        :disabled="currentPage >= totalPages"
        label="Próximo"
        color="primary"
        class="pagination-btn"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";

// Lista de séries, gêneros e paginação
const tvShows = ref([]);
const genres = ref([]);
const currentPage = ref(1);
const totalPages = ref(1);
const loading = ref(false);

// Estado para descrições expandidas
const expandedDescriptions = ref({});

// Função para buscar séries com base na página atual
const fetchTvShows = async (page = 1) => {
  loading.value = true;
  try {
    const response = await fetch(
      `https://api.themoviedb.org/3/tv/on_the_air?language=pt-BR&page=${page}`,
      options
    );
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    const data = await response.json();
    tvShows.value = data.results;
    totalPages.value = data.total_pages;
    currentPage.value = page;
  } catch (error) {
    console.error("Erro ao buscar séries:", error);
  } finally {
    loading.value = false;
  }
};

// Função para buscar gêneros
const fetchGenres = async () => {
  try {
    const response = await fetch(
      "https://api.themoviedb.org/3/genre/tv/list?language=pt-BR",
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
    fetchTvShows(page);
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

// Buscar séries e gêneros ao montar o componente
onMounted(() => {
  fetchTvShows();
  fetchGenres();
});

// Função para buscar gêneros das séries
const getGenres = (genreIds) => {
  return genreIds
    .map((id) => {
      const genre = genres.value.find((g) => g.id === id);
      return genre ? genre.name : "Desconhecido";
    })
    .join(", ");
};

// Computed property para descrições truncadas
const truncatedDescriptions = computed(() => {
  const descriptions = {};
  tvShows.value.forEach((show) => {
    descriptions[show.id] = expandedDescriptions.value[show.id]
      ? show.overview
      : show.overview.length > 150
      ? show.overview.slice(0, 150) + "..."
      : show.overview;
  });
  return descriptions;
});

// Função para alternar a descrição
const toggleDescription = (showId) => {
  expandedDescriptions.value[showId] = !expandedDescriptions.value[showId];
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  max-width: 1200px;
  margin: auto;
}

.title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
}

.spinner-container,
.no-shows {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 200px;
}

.card-container {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  gap: 20px;
}

@media (min-width: 768px) {
  .card-container {
    grid-template-columns: repeat(2, 1fr);
  }
}

.my-card {
  max-width: 300px;
  margin: 10px;
  display: flex;
  flex-direction: column;
}

.show-poster {
  height: 450px;
  object-fit: cover;
}

.card-content {
  display: flex;
  flex-direction: column;
}

.show-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 10px;
}

.show-info {
  margin-bottom: 5px;
}

.show-description {
  margin-top: 10px;
  margin-bottom: 10px;
}

.toggle-btn {
  align-self: flex-start;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.pagination-btn {
  margin: 0 10px;
}

.pagination-info {
  font-size: 16px;
}
</style>
