<template>
  <div class="card mb-4 movie-fixed-card">
    <div class="d-flex flex-column align-items-center bg-light" style="min-height: 220px;">
      <img :src="getPosterUrl" :alt="getTitle" class="img-fluid mt-2" style="width: 70%; border-radius: 8px; max-height: 180px; object-fit: cover;" />
      <span v-if="getRating" class="badge bg-warning text-dark mt-2" style="font-size: 1.1em;">⭐ {{ getRating }}</span>
    </div>
    <div class="card-body d-flex flex-column">
      <h2 class="card-title h5">{{ getTitle }}</h2>
      <div class="mb-2">
        <span class="badge bg-secondary">{{ getYear }}</span>
      </div>
      <p class="card-text flex-grow-1" v-if="getPlot">{{ getPlot.length > 120 ? getPlot.slice(0, 120) + '...' : getPlot }}</p>
      <a :href="getLink" target="_blank" class="btn btn-primary w-100 mt-auto">Detay</a>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MovieCard',
  props: {
    movie: {
      type: Object,
      required: true
    }
  },
  computed: {
    getTitle() {
      return this.movie.title || this.movie.Title || 'Başlık Yok';
    },
    getPosterUrl() {
      if (this.movie.poster_path) {
        return 'https://image.tmdb.org/t/p/w500/' + this.movie.poster_path;
      } else if (this.movie.Poster && this.movie.Poster !== 'N/A') {
        return this.movie.Poster;
      } else {
        return 'https://via.placeholder.com/180x260?text=No+Image';
      }
    },
    getRating() {
      const val = this.movie.vote_average || this.movie.imdbRating;
      if (!val) return '';
      return Number(val).toFixed(1);
    },
    getYear() {
      if (this.movie.release_date) return this.movie.release_date.split('-')[0];
      if (this.movie.Year) return this.movie.Year;
      return 'Yıl Yok';
    },
    getPlot() {
      return this.movie.overview || this.movie.Plot || '';
    },
    getLink() {
      if (this.movie.id) {
        return 'https://www.themoviedb.org/movie/' + this.movie.id;
      } else if (this.movie.imdbID) {
        return 'https://www.imdb.com/title/' + this.movie.imdbID;
      } else {
        return '#';
      }
    }
  }
}
</script>

<style scoped>
.movie-fixed-card {
  min-width: 260px;
  max-width: 320px;
  width: 100%;
  min-height: 420px;
  max-height: 480px;
  display: flex;
  flex-direction: column;
}
@media (max-width: 576px) {
  .movie-fixed-card {
    min-width: 100%;
    max-width: 100%;
  }
}
</style> 