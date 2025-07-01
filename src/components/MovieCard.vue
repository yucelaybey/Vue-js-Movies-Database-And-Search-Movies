<template>
  <div class="movie-card-pro">
    <div class="poster-wrap">
      <img :src="getPosterUrl" :alt="getTitle" />
      <span v-if="getRating" class="imdb-badge">⭐ {{ getRating }}</span>
    </div>
    <div class="movie-info">
      <h2 class="movie-title">{{ getTitle }}</h2>
      <div class="movie-meta">
        <span class="badge">{{ getYear }}</span>
      </div>
      <p class="movie-plot" v-if="getPlot">{{ getPlot.length > 120 ? getPlot.slice(0, 120) + '...' : getPlot }}</p>
      <div class="imdb-link-wrap">
        <a :href="getLink" target="_blank" class="imdb-link">Detay</a>
      </div>
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
.movie-card-pro {
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 4px 24px 0 rgba(60,60,90,0.10);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  transition: transform 0.18s, box-shadow 0.18s;
  min-height: 420px;
  position: relative;
}
.movie-card-pro:hover {
  transform: translateY(-6px) scale(1.025);
  box-shadow: 0 8px 32px 0 rgba(60,60,90,0.16);
}
.poster-wrap {
  position: relative;
  width: 100%;
  height: 260px;
  background: #f3f4f6;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}
.poster-wrap img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-bottom: 1px solid #e5e7eb;
}
.imdb-badge {
  position: absolute;
  bottom: 10px;
  right: 12px;
  background: #facc15;
  color: #222;
  font-weight: 700;
  font-size: 1.05em;
  padding: 4px 10px;
  border-radius: 8px;
  box-shadow: 0 2px 8px 0 rgba(0,0,0,0.10);
}
.movie-info {
  padding: 18px 16px 16px 16px;
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 180px;
}
.movie-title {
  font-size: 1.13rem;
  font-weight: 700;
  margin-bottom: 0.3em;
  color: #22223b;
  line-height: 1.2;
}
.movie-meta {
  display: flex;
  gap: 8px;
  margin-bottom: 0.4em;
}
.badge {
  background: #e0e7ff;
  color: #3730a3;
  font-size: 0.93em;
  padding: 2px 10px;
  border-radius: 8px;
  font-weight: 500;
}
.movie-plot {
  color: #444;
  font-size: 0.97em;
  margin-bottom: 1em;
  min-height: 2.2em;
}
.imdb-link-wrap {
  margin-top: auto;
  display: flex;
  align-items: flex-end;
  height: 40px;
}
.imdb-link {
  background: #6366f1;
  color: #fff;
  text-align: center;
  padding: 8px 0;
  border-radius: 7px;
  font-weight: 600;
  text-decoration: none;
  transition: background 0.18s;
  font-size: 1em;
  width: 100%;
  display: block;
}
.imdb-link:hover {
  background: #4338ca;
}
@media (max-width: 600px) {
  .movie-card-pro {
    min-height: 340px;
  }
  .poster-wrap {
    height: 180px;
  }
  .movie-info {
    padding: 12px 8px 10px 8px;
  }
  .imdb-link-wrap {
    height: 36px;
  }
}
</style> 