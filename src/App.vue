<template>
  <div id="app">
    <div class="block-preview" style="background-image: url('./img/bg-head.jpg');">
        <div class="mainer">
            <div class="block-play"></div>
            <div class="block-caption">
                <h1>RINGS</h1>
                EVIL IS REBORN
            </div>
            <div class="block-description">
                Paramount Pictures have just released the first trailer for the upcoming Horror movie “Rings“.
            </div>
        </div>
    </div>
    <div class="block-genres">
        <div class="mainer">
            <ul>
                <li v-for="genre in genresMenu"
                :key="genre.id"
                @click="getMovies(genre.id); genreActiveId = genre.id"
                :class="{active: genre.id === genreActiveId}">{{genre.name}}</li>
            </ul>
        </div>
    </div>

    <div class="block-grid">
        <div class="mainer">
            <div class="grid-wrapper">
                <div class="grid-cell" v-for="movie in movies" :key="movie.id">
                    <div class="block-card">
                        <div class="label" v-if="movie.popularity > 60">Popular</div>
                        <div class="poster" :style="{'background-image': 'url(https://image.tmdb.org/t/p/w500' + movie.poster_path + ')'}">
                            <div class="buttons">
                                <button class="btn">info</button>
                                <button class="btn">trailer</button>
                            </div>
                            <button class="favourite" @click="addToFavourite(movie.id)">+</button>
                        </div>
                        <div class="body">
                            <div class="caption">{{movie.title}}</div>
                            <div class="genres">{{getGenresString(movie.genre_ids)}}</div>
                            <div class="raiting">
                                <div class="stars">
                                  <span :style="{width: movie.vote_average *10 + '%'}"></span>
                                </div>
                                <div class="amount">{{movie.vote_average}}</div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="grid-cell empty"></div>
                <div class="grid-cell empty"></div>
            </div>
            <button class="more" @click="getMovies(genreActiveId, ++currentPage)">
              <span class="dots"></span>
            </button>
        </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      apiUrl: "https://api.themoviedb.org/",
      apiKey: 'b65eae8e043ef97a115ed08236151ad7',
      genres: [],
      movies: [],
      genresMenu: [],
      genreActiveId: null,
      currentPage: 1,

    };
  },
  methods: {
    getGenres() {
      fetch(
        this.apiUrl + "3/genre/movie/list?api_key=" + 
        this.apiKey + "&language=ru"
        )
        .then(result => {
        return result.json();
      })
      .then (result => {
        // объявляем и записываем результат
        this.genres = result.genres;
        this.genresMenu = result.genres.slice(0, 5);
        this.genreActiveId = result.genres[0].id;
        this.getMovies(this.genreActiveId, this.currentPage);
        // this.addToFavourite(this.id);
      });
    },
// загрузка параметров с нашей ссылки
    getMovies(genreId, page) {
      fetch(
        this.apiUrl +
        "3/discover/movie?api_key=" +
        this.apiKey +
        "&language=ru&sort_by=popularity.desc&include_adult=true&include_video=true&page="+
        page +
        "&with_genres=" +
        genreId
      ).then(result => {
        return result.json();
      }).then (result => {
        // console.log(page);
        if(page > 1) {
          // console.log(result.results);
          this.movies = this.movies.concat(result.results);
        }
        else {
          this.movies = result.results;
        }
      });
    },
    getGenresString(ids) {
      let result = ``;
      for(let id of ids) {
        for(let genre of this.genres) {
         if(id === genre.id) {
           result += ", " + genre.name;
         }
        }
      }
      return result.slice(2, result.length);
    },
    addToFavourite(id) {
      const favoriteList = localStorage.getItem("favourite");
      const currentList = JSON.parse(favoriteList) || [];

      if(favoriteList) {
        if(currentList.indexOf(id) < 0) {
          currentList.push(id);
        } else {
          currentList.splice(currentList.indexOf(id), 1);
        } 
      } else {
        currentList.push(id);
      }
      localStorage.setItem("favourite", JSON.stringify(currentList));
    }
  },
  created: function() {
    this.getGenres();
  }
};
</script>

<style lang="scss">

  @import './src/style/style.scss';

</style>
