<template>
  <div>
    <h1>Filmes Recentes</h1>
    <div class="card-container">
      <div v-for="movie in movies" :key="movie.id" class="card">
        <img
          :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
          alt="Poster"
          class="poster"
        />
        <div class="card-content">
          <p class="release-date">Lançamento: {{ movie.release_date }}</p>
          <h2 class="title">{{ movie.title }}</h2>
          <p class="overview">
            {{ isExpanded(movie.id) ? movie.overview : truncatedOverview(movie.overview) }}
          </p>
          <button
            v-if="needsShowMoreButton(movie.overview)"
            @click="toggleOverview(movie.id)"
            class="toggle-button"
          >
            {{ isExpanded(movie.id) ? 'Ver menos' : 'Ver mais' }}
          </button>
          <p class="rating">Nota: {{ movie.vote_average }}</p>
          <p class="genres">Gêneros: {{ getGenres(movie.genre_ids).join(', ') }}</p>
        </div>
      </div>
    </div>
    <div class="pagination">
      <button @click="previousPage" :disabled="page === 1">Anterior</button>
      <button @click="nextPage" :disabled="page === totalPages">Próxima</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'

export default {
  setup() {
    const movies = ref([])
    const genres = ref([])
    const page = ref(1)
    const totalPages = ref(1)
    const expandedOverviews = ref({})

    const fetchMovies = async () => {
      const options = {
        method: 'GET',
        headers: {
          accept: 'application/json',
          Authorization:
            'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJjM2NjNGQzNTk0NDhkMmNiZjI4NDFjYTRkNTNjMGY2OCIsIm5iZiI6MTcyMTM4ODI3Mi4zMzA4MjIsInN1YiI6IjYzMGJjZWUxMjc5MGJmMDA3YzEyMjI1MCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.5jRCoQGwo5_I5Mr1s8DA-ZFRddAT6GKkTeU0PothI-c'
        }
      }

      try {
        const response = await fetch(
          `https://api.themoviedb.org/3/movie/now_playing?language=en-US&page=${page.value}`,
          options
        )
        const data = await response.json()
        movies.value = data.results
        totalPages.value = data.total_pages
      } catch (err) {
        console.error(err)
      }
    }

    const fetchGenres = async () => {
      const options = {
        method: 'GET',
        headers: {
          accept: 'application/json',
          Authorization:
            'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJjM2NjNGQzNTk0NDhkMmNiZjI4NDFjYTRkNTNjMGY2OCIsIm5iZiI6MTcyMTM4ODI3Mi4zMzA4MjIsInN1YiI6IjYzMGJjZWUxMjc5MGJmMDA3YzEyMjI1MCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.5jRCoQGwo5_I5Mr1s8DA-ZFRddAT6GKkTeU0PothI-c'
        }
      }

      try {
        const response = await fetch(
          'https://api.themoviedb.org/3/genre/movie/list?language=en-US',
          options
        )
        const data = await response.json()
        genres.value = data.genres
      } catch (err) {
        console.error(err)
      }
    }

    const getGenres = (genreIds) => {
      return genreIds.map(
        (id) => genres.value.find((genre) => genre.id === id)?.name || 'Desconhecido'
      )
    }

    const nextPage = () => {
      if (page.value < totalPages.value) {
        page.value++
        fetchMovies()
      }
    }

    const previousPage = () => {
      if (page.value > 1) {
        page.value--
        fetchMovies()
      }
    }

    const needsShowMoreButton = (overview) => {
      return overview.length > 150 // Aproximadamente 4 linhas de texto
    }

    const truncatedOverview = (overview) => {
      return overview.length > 150 ? `${overview.slice(0, 150)}...` : overview
    }

    const toggleOverview = (movieId) => {
      expandedOverviews.value[movieId] = !expandedOverviews.value[movieId]
    }

    const isExpanded = (movieId) => {
      return expandedOverviews.value[movieId]
    }

    onMounted(() => {
      fetchGenres()
      fetchMovies()
    })

    return {
      movies,
      page,
      totalPages,
      nextPage,
      previousPage,
      getGenres,
      expandedOverviews,
      needsShowMoreButton,
      truncatedOverview,
      toggleOverview,
      isExpanded
    }
  }
}
</script>

<style scoped>
body {
  background-color: #f3f4f6;
  font-family: Arial, sans-serif;
  color: #333;
  margin: 0;
  padding: 0;
}

h1 {
  text-align: center;
  margin: 20px 0;
  font-size: 2em;
  color: #333;
}

.card-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
}

.card {
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
  max-width: 320px;
  width: 100%;
  overflow: hidden;
  text-align: left;
  display: flex;
  flex-direction: column;
}

.card:hover {
  transform: translateY(-10px);
}

.poster {
  width: 100%;
  height: auto;
  object-fit: cover;
}

.card-content {
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.release-date {
  font-size: 0.85em;
  color: #777;
}

.title {
  font-size: 1.3em;
  margin: 0;
  color: #333;
}

.overview {
  font-size: 1em;
  line-height: 1.5;
  color: #555;
}

.toggle-button {
  background: none;
  border: none;
  color: #007bff;
  cursor: pointer;
  padding: 0;
  margin: 0;
  font-size: 1em;
  text-align: left;
}

.toggle-button:hover {
  text-decoration: underline;
}

.rating,
.genres {
  font-size: 0.9em;
  color: #555;
  margin: 0;
}

.pagination {
  display: flex;
  justify-content: center;
  margin: 20px 0;
}

.pagination button {
  background-color: #007bff;
  border: none;
  color: #fff;
  cursor: pointer;
  margin: 0 5px;
  padding: 10px 20px;
  transition: background-color 0.2s;
  font-size: 1em;
  border-radius: 4px;
}

.pagination button:disabled {
  background-color: #c0c0c0;
  cursor: not-allowed;
}

.pagination button:not(:disabled):hover {
  background-color: #0056b3;
}
</style>
