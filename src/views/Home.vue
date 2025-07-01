<template>
  <div class="container modern-container">
    <h1 style="text-align:center; font-size:2.2rem; margin-bottom:1.5rem; letter-spacing:-1px;">🎬 Movies</h1>
    <div class="filters" style="display:flex; flex-wrap:wrap; gap:16px; justify-content:center; margin-bottom:24px; flex-direction:column; align-items:center;">
      <div class="filter-row">
        <label>Kategori</label>
        <select v-model="selectedGenre" @change="applyFilters">
          <option value="">Tümü</option>
          <option v-for="g in genres" :key="g" :value="g">{{ g }}</option>
        </select>
      </div>
      <div class="filter-row">
        <label>Yıl</label>
        <select v-model="selectedYear" @change="applyFilters">
          <option value="">Tümü</option>
          <option v-for="year in years" :key="year" :value="year">{{ year }}</option>
        </select>
      </div>
    </div>
    <div class="search-bar">
      <input v-model="search" @keyup.enter="fetchMovies" type="text" placeholder="Film adı..." />
      <button @click="fetchMovies">ARA</button>
    </div>
    <div v-if="loading" class="modern-loading" style="display:flex; flex-direction:column; align-items:center; justify-content:center; min-height:120px;">
      <span class="loader"></span>
      <span style="margin-top:18px; color:#ff9800; font-size:1.2rem; font-weight:700; letter-spacing:1px;">Yükleniyor, lütfen bekleyin...</span>
    </div>
    <div v-else-if="error" style="color:#e11d48; text-align:center; font-size:1.1rem; margin:2rem 0;">{{ error }}</div>
    <div v-else class="movie-list" style="display:grid; grid-template-columns:repeat(auto-fit,minmax(300px,340px)); gap:28px; justify-content:center;">
      <MovieCard v-for="movie in paginatedMovies" :key="movie.id || movie.imdbID" :movie="movie" />
    </div>
    <div v-if="totalPages > 1 && filteredMovies.length > 0 && movies.length > 0" style="display:flex; justify-content:center; gap:8px; margin:32px 0; flex-wrap:wrap;">
      <button @click="goToPage(currentPage-1)" :disabled="currentPage===1">&lt;</button>
      <button v-for="page in Math.min(totalPages, 10)" :key="page" @click="goToPage(page)" :style="{fontWeight: currentPage===page ? 'bold' : 'normal', background: currentPage===page ? '#ff9800' : '#fff', color: currentPage===page ? '#fff' : '#222', border: '1px solid #ff9800', borderRadius:'6px', padding:'4px 12px', margin:'2px'}">{{ page }}</button>
      <button @click="goToPage(currentPage+1)" :disabled="currentPage===totalPages">&gt;</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import MovieCard from '../components/MovieCard.vue'

const OMDB_API_KEY = '3cb11f6c';
const OMDB_BASE_URL = 'https://www.omdbapi.com/';
const TMDB_API_KEY = 'd927be5ffc5606834610910ea871e7ae';
const TMDB_BEARER_TOKEN = 'eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJkOTI3YmU1ZmZjNTYwNjgzNDYxMDkxMGVhODcxZTdhZSIsIm5iZiI6MTc1MTM2NDMyMC43MDEsInN1YiI6IjY4NjNiMmUwZThhMzlmZTAzYTQ5NTU1OCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.wSueBeRGK8DPwzfgzOx20C-rRbiu5R27k7eYUw2lcKM';
const TMDB_BASE_URL = 'https://api.themoviedb.org/3';

export default {
  name: 'Home',
  components: { MovieCard },
  data() {
    return {
      search: '',
      movies: [],
      loading: false,
      error: '',
      types: [
        { label: 'Film', value: 'movie' },
        { label: 'Dizi', value: 'series' },
        { label: 'Bölüm', value: 'episode' }
      ],
      selectedType: '',
      years: [],
      selectedYear: '',
      genres: [],
      selectedGenre: '',
      currentPage: 1,
      pageSize: 20,
      totalResults: 0,
      totalPages: 1,
      allMovies: [],
      genreMap: {
        28: 'Aksiyon', 12: 'Macera', 16: 'Animasyon', 35: 'Komedi', 80: 'Suç', 99: 'Belgesel', 18: 'Dram', 10751: 'Aile', 14: 'Fantastik', 36: 'Tarih', 27: 'Korku', 10402: 'Müzik', 9648: 'Gizem', 10749: 'Romantik', 878: 'Bilim Kurgu', 10770: 'TV Filmi', 53: 'Gerilim', 10752: 'Savaş', 37: 'Western'
      }
    }
  },
  computed: {
    filteredMovies() {
      let filtered = this.allMovies;
      if (this.selectedGenre) {
        filtered = filtered.filter(m => {
          if (m.genre_ids && Array.isArray(m.genre_ids)) {
            return m.genre_ids.some(id => this.genreMap[id] === this.selectedGenre);
          }
          if (m.Genre) {
            return m.Genre.split(',').map(g => g.trim()).includes(this.selectedGenre);
          }
          return false;
        });
      }
      if (this.selectedYear) {
        filtered = filtered.filter(m => (m.release_date && m.release_date.startsWith(this.selectedYear)) || (m.Year && m.Year === this.selectedYear));
      }
      return filtered;
    },
    paginatedMovies() {
      const start = (this.currentPage - 1) * this.pageSize;
      return this.filteredMovies.slice(start, start + this.pageSize);
    },
    totalPages() {
      return Math.ceil(this.filteredMovies.length / this.pageSize);
    },
    years() {
      const yearSet = new Set(this.allMovies.map(m => m.release_date ? m.release_date.split('-')[0] : m.Year).filter(Boolean));
      return Array.from(yearSet).sort((a, b) => b - a);
    },
    genres() {
      let allGenres = [];
      this.allMovies.forEach(m => {
        if (m.genre_ids && Array.isArray(m.genre_ids)) allGenres.push(...m.genre_ids.map(id => this.genreMap[id]));
        if (m.Genre) allGenres.push(...m.Genre.split(',').map(g => g.trim()));
      });
      return Array.from(new Set(allGenres)).filter(Boolean).sort();
    },
    types() {
      let allTypes = [];
      this.allMovies.forEach(m => {
        if (m.Type) allTypes.push({ label: m.Type === 'movie' ? 'Film' : m.Type === 'series' ? 'Dizi' : 'Bölüm', value: m.Type });
        if (m.genre_ids && Array.isArray(m.genre_ids)) allTypes.push(...m.genre_ids.map(id => ({ label: this.genreMap[id], value: id })));
      });
      const unique = [];
      const seen = new Set();
      for (const t of allTypes) {
        if (t && t.value && !seen.has(t.value)) {
          unique.push(t);
          seen.add(t.value);
        }
      }
      return unique;
    }
  },
  methods: {
    selectType(type) {
      this.selectedType = type;
      this.fetchMovies();
    },
    selectGenre(genre) {
      this.selectedGenre = genre;
    },
    handleResize() {
      if (window.innerWidth < 600) this.pageSize = 4;
      else if (window.innerWidth < 900) this.pageSize = 8;
      else this.pageSize = 12;
    },
    async fetchTopMovies() {
      this.loading = true;
      this.error = '';
      this.movies = [];
      this.currentPage = 1;
      this.allMovies = [];
      try {
        // 5 sayfayı paralel çek
        const pagePromises = [];
        for (let page = 1; page <= 5; page++) {
          pagePromises.push(
            axios.get(`${TMDB_BASE_URL}/movie/popular`, {
              params: {
                language: 'tr-TR', // Sadece başlıklar Türkçe
                page
              },
              headers: {
                Authorization: `Bearer ${TMDB_BEARER_TOKEN}`
              }
            })
          );
        }
        const responses = await Promise.all(pagePromises);
        let allMovies = [];
        responses.forEach(res => {
          if (res.data && res.data.results) {
            allMovies = allMovies.concat(res.data.results);
          }
        });
        // Her film için İngilizce overview'u çek
        await Promise.all(allMovies.map(async (movie) => {
          try {
            const detail = await axios.get(`${TMDB_BASE_URL}/movie/${movie.id}`, {
              params: { language: 'en-US' },
              headers: { Authorization: `Bearer ${TMDB_BEARER_TOKEN}` }
            });
            if (detail.data && detail.data.overview && detail.data.overview !== '') {
              movie.overview = detail.data.overview;
            } else {
              movie.overview = "We don't have an overview translated in English. Help us expand our database by adding one.";
            }
          } catch (e) {
            movie.overview = "We don't have an overview translated in English. Help us expand our database by adding one.";
          }
        }));
        allMovies = allMovies.sort((a, b) => b.vote_average - a.vote_average);
        this.allMovies = allMovies;
        this.totalResults = allMovies.length;
        this.totalPages = Math.ceil(allMovies.length / this.pageSize);
        this.selectedGenre = '';
        this.updateMoviesForPage();
        if (!this.movies.length) {
          this.error = 'Film bulunamadı.';
        }
      } catch (e) {
        this.error = 'Film verileri alınamadı.';
      } finally {
        this.loading = false;
      }
    },
    updateMoviesForPage() {
      const start = (this.currentPage - 1) * this.pageSize;
      this.movies = this.allMovies.slice(start, start + this.pageSize);
    },
    goToPage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
        this.updateMoviesForPage();
        window.scrollTo({ top: 0, behavior: 'smooth' });
      }
    },
    fetchMovies() {
      if (this.search && this.search.trim().length > 0) {
        this.fetchSearchMovies();
      } else {
        this.fetchTopMovies();
      }
    },
    async fetchSearchMovies() {
      this.loading = true;
      this.error = '';
      this.movies = [];
      this.currentPage = 1;
      this.allMovies = [];
      try {
        let allMovies = [];
        let page = 1;
        while (allMovies.length < 100) {
          let url = `${OMDB_BASE_URL}?apikey=${OMDB_API_KEY}&s=${encodeURIComponent(this.search)}&type=movie&page=${page}`;
          const res = await axios.get(url);
          if (res.data.Response === 'True') {
            allMovies = allMovies.concat(res.data.Search);
            if (res.data.Search.length < 10) break;
          } else {
            break;
          }
          page++;
        }
        // Her film için İngilizce Plot'u çek
        const detailPromises = allMovies.slice(0, 100).map(async (item) => {
          let detailRes;
          try {
            detailRes = await axios.get(`${OMDB_BASE_URL}?apikey=${OMDB_API_KEY}&i=${item.imdbID}&language=en`);
          } catch {}
          let movie = { ...item, ...((detailRes && detailRes.data) || {}) };
          if (!movie.Plot || movie.Plot === 'N/A') {
            movie.Plot = "We don't have an overview translated in English. Help us expand our database by adding one.";
          }
          return movie;
        });
        const detailedMovies = await Promise.all(detailPromises);
        const sorted = detailedMovies.filter(m => m.imdbRating && m.imdbRating !== 'N/A').sort((a, b) => parseFloat(b.imdbRating) - parseFloat(a.imdbRating));
        this.allMovies = sorted;
        this.totalResults = sorted.length;
        this.totalPages = Math.ceil(sorted.length / this.pageSize);
        this.selectedGenre = '';
        this.updateMoviesForPage();
        if (!this.movies.length) {
          this.error = 'Film bulunamadı.';
        }
      } catch (e) {
        this.error = 'Film verileri alınamadı.';
      } finally {
        this.loading = false;
      }
    },
    applyFilters() {
      this.currentPage = 1;
      this.updateMoviesForPage();
    }
  },
  mounted() {
    this.handleResize();
    window.addEventListener('resize', this.handleResize);
    this.fetchTopMovies();
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handleResize);
  }
}
</script> 