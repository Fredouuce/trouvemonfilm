<template>
  <div class="container">
    <nav class="navbar">
      <nuxt-link to="/" class="lien lien_home" @click.native="reloadPage"
        >Accueil</nuxt-link
      >
      |
      <nuxt-link to="/about" class="lien">A propos</nuxt-link>
    </nav>

    <div>
      <!-- Sondage pour demander a l'utilisateurs des informations sur les films qu'il veut voir -->
      <form @onChange.prevent="showType" v-if="formShow">
        <!--ETAPE 1 : Sélection film ou série tv -->
        <div v-if="step_1">
          <div class="title_container">
            <h1 class="title">T'es plutôt film ou série ?</h1>
            <img
              src="~assets/thinkemoji.png"
              alt="emoji qui réfléchis"
              class="emoji"
            />
          </div>
          <!-- Boutton RADIO pour la séléctions des films -->
          <div class="btn_container">
            <div>
              <input
                type="radio"
                id="film"
                name="type"
                value="film"
                v-model="type"
                class="choose_btn"
              />
              <label for="film" name="film" class="choose_btn_label"
                >Film</label
              >
            </div>
            <div>
              <input
                type="radio"
                id="tv"
                name="type"
                value="tv"
                v-model="type"
                class="choose_btn"
              />
              <label for="tv" name="tv" class="choose_btn_label"
                >Série TV</label
              >
            </div>
          </div>

          <div class="btn_next_container">
            <button
              @click.prevent="next(step_1)"
              v-if="type != null"
              class="btn_next"
            >
              Suivant
            </button>
          </div>

          <div class="txt_container">
            <p class="hello_txt">
              Bonjour bienvenue sur mon site. <br />
              Avez vous déjà passer des minutes ou des heures à chercher le film
              qui vous convient le plus ? Ce site est la solution, il a été
              conçu pour réduire les heures de recherche en des heures de
              visionnage , vous pourrez naviguez et rechercher un film que vous
              connaissez déjà , ou bien vous laisser guider par nos service qui
              vous permetterons de voir quel films sont fais pour vous. <br />
              Je travaille sur des fonctionnalités qui sera disponible dans le
              futur, l'histoire de ce site ne fais que commencer
            </p>
          </div>
        </div>
        <!-- ETAPE 2 :Sélection par genres -->
        <div v-else-if="step_2">
          <!-- Le format a été choisis -->
          <h1 class="title">
            Vous avez choisis un
            <span class="title_span">{{
              type == "film" ? "film" : "série tv"
            }}</span>
          </h1>
          <h2 class="title">
            Veuillez choisir les genres qui vous intéresse
            <span v-if="selectedGenres.length == 3">(3 maximum)</span>
          </h2>

          <!-- Button retour -->
          <!-- <p @click="type = null" class="back_btn">Retour</p> -->
          <div>
            <!-- Boucles des genres -->
            <div class="genre_btn_container">
              <div v-for="genre in genres" :key="genre.id">
                <input
                  type="checkbox"
                  name="genre"
                  :value="genre.id"
                  :id="genre.name"
                  v-model="selectedGenres"
                  :disabled="
                    selectedGenres.length >= 3 &&
                    selectedGenres.indexOf(genre.id) === -1
                  "
                  class="choose_btn"
                />
                <label
                  class="choose_btn_label label_checkbox"
                  :for="genre.name"
                  >{{ genre.name }}</label
                >
              </div>
            </div>
            <div class="btn_next_container">
              <button
                v-if="selectedGenres.length > 0"
                @click.prevent="show"
                class="btn_next"
              >
                Terminé
              </button>
            </div>
            <!-- Button pour finir la selections et passer a la suite -->
          </div>
        </div>
      </form>
      <!-- Les genres on été choisis -->
      <!-- Etape 3: Affichage des films ou séries trouver -->
      <!-- <div class="loader" v-else-if="step_2 == false && isLoading == true">
        <div class="lds-roller">
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
        </div>
      </div> -->
      <div v-else-if="step_2 == false" class="datas">
        <h2>
          Vous avez séléctionner :
          <span
            v-for="selectedGenreName in selectedGenresName"
            :key="selectedGenreName.id"
            >{{ selectedGenreName + "  " }}</span
          >
        </h2>
        <!--Selection de filtrage "Sort by"  -->
        <div class="select_container">
          <select
            v-model="trie"
            name="trie"
            id="plan"
            @change="fetchBySort"
            class="select_form"
          >
            <option value="popularity.desc" selected>
              Par popularité descendante
            </option>
            <option value="popularity.asc">Par popularité ascendante</option>
            <option value="primary_release_date.desc">
              Plus récent au plus vieux
            </option>
            <option value="primary_release_date.asc">
              Plus vieux au plus rescente
            </option>
            <option value="vote_count.desc">Par notre descendante</option>
            <option value="vote_count.asc">Par notre ascendante</option>
            <option value="original_title.desc">
              Par nom A-Z (Titre original)
            </option>
            <option value="original_title.asc">
              Par nom Z-A (Titre original)
            </option>
          </select>
        </div>

        <!-- Button retour -->
        <!-- <p @click.prevent="back" class="back_btn">Retour</p> -->
        <div class="data_container">
          <div v-for="data in datas" :key="data.id">
            <nuxt-link
              :to="type == 'film' ? `film/${data.id}` : `tv/${data.id}`"
            >
              <div>
                <Card :data="data" />
              </div>
            </nuxt-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "IndexPage",
  data() {
    return {
      type: null,
      genres: [],
      selectedGenres: [],
      selectedGenresName: [],
      formShow: true,
      datas: [],
      trie: "popularity.asc",
      todayDate: "",
      step_1: true,
      step_2: false,
      step_3: false,
      isLoading: true,
    };
  },
  methods: {
    /**********************************Fetching de tous les films******************************************* */
    show() {
      this.formShow = false;
      this.step_2 = false;
      if (this.type == "film") {
        //Selections pour les films
        this.$axios
          .get(
            `https://api.themoviedb.org/3/discover/movie?api_key=${
              process.env.apiKey
            }&primary_release_date.lte=${
              this.todayDate
            }&include_adult=false&language=fr-FR&sort_by=popularity.desc&page=1&with_genres=${this.selectedGenres.join(
              ","
            )}`
          )

          .then((response) => {
            this.datas = response.data.results;
            this.genreById();
            this.isLoading = false;
          })
          .catch((error) => {
            console.log(error);
          });
      } else if (this.type == "tv") {
        //Selections pour les séries TV
        this.$axios
          .get(
            `https://api.themoviedb.org/3/discover/tv?api_key=${
              process.env.apiKey
            }&primary_release_date.lte=${
              this.todayDate
            }&include_adult=false&language=fr-FR&sort_by=popularity.desc&include_adult=false&include_video=false&page=1&with_watch_monetization_types=flatrate&with_genres=${this.selectedGenres.join(
              ","
            )}`
          )
          .then((response) => {
            this.datas = response.data.results;
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },

    /**********************************Fetching des films mais avec un filtrage******************************************* */
    fetchBySort() {
      this.$axios
        .get(
          `https://api.themoviedb.org/3/discover/${
            this.type == "film" ? "movie" : "tv"
          }?api_key=${process.env.apiKey}&language=fr-FR&sort_by=${
            this.trie
          }&include_adult=false&primary_release_date.lte=${
            this.todayDate
          }&page=1&with_genres=${this.selectedGenres.join(",")}`
        )
        .then((response) => {
          this.datas = response.data.results;
        });
    },
    /**********************************Bouttons de validation qui passe a la suite ******************************************* */
    next() {
      if (this.step_1) {
        this.step_1 = false;
        this.step_2 = true;
      }
    } /*Reload on click */,
    reloadPage() {
      window.location.reload();
    },
    /***************Permet d'affiché sur la page qu'elle est les genres séléctionner*********************/
    genreById() {
      this.selectedGenres.forEach((element) => {
        const genresId = element;
        this.genres.forEach((element) => {
          if (genresId == element.id) {
            this.selectedGenresName.push(element.name);
          }
        });
      });
    },
  },
  mounted() {
    /*******************Séléction des genres a vérifier si ca ne créer pas des bug au niveau de certains genres(L'api vise les films ,a vérifié la compatibilité avec le lien série)********************/
    this.$axios
      .get(
        `https://api.themoviedb.org/3/genre/movie/list?api_key=${process.env.apiKey}&language=fr-FR`
      )
      .then((response) => {
        this.genres = response.data.genres;
      })
      .catch((error) => {
        console.log(error);
      });

    this.todayDate = new Date().toISOString().split("T")[0];
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;700&display=swap");
body {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  background: #262626;
  color: white;
  font-family: "Prompt", sans-serif;
}

.container {
  max-width: 1440px;
  position: relative;
  left: 50%;
  transform: translateX(-50%);
  background: #303030;
  padding: 0em 2em 2em 2em;
}
.title {
  color: #ffffff;
  font-weight: 500;
  text-align: center;
  letter-spacing: 1px;
}
.navbar {
  padding-top: 1em;
  text-align: center;
}
.lien {
  text-decoration: none;
  color: white;
  transition: 300ms ease-in-out;
}
.lien:hover {
  color: #f21f0c;
}
.title_container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}
.emoji {
  width: 50px;
  height: 50px;
  padding-left: 1em;
}
a {
  color: white;
  text-decoration: none;
}
a.nuxt-link-exact-active {
  color: #f21f0c;
}
.back_btn {
  cursor: pointer;
  transition: 300ms ease-in-out;
}
.back_btn:hover {
  color: #f21f0c;
}
.choose_btn_label {
  color: #f21f0c;
  border: 1px solid #f21f0c;
  display: inline-block;
  padding: 0.5em 1em;
  border-radius: 50px;
  cursor: pointer;
  font-size: 1.2em;
  transition: 300ms ease-in-out;
}
.choose_btn {
  display: none;
}
.choose_btn_label:hover {
  background: #f21f0c;
  color: white;
}
.btn_container {
  width: 230px;
  position: relative;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  left: 50%;
  transform: translateX(-50%);
}
.hello_txt {
  text-align: center;
}
.txt_container {
  padding: 0 2em;
}

.data_container {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: space-around;
}
.select_container {
  margin: 1.5em;

  position: relative;
  height: 30px;
}
.select_form {
  position: absolute;
  right: 0%;
  height: 30px;
  background: white;
  border: none;
  color: #131313;
  font-family: "Prompt", sans-serif;
  border: 1px solid #fc2e2e94;
  cursor: pointer;
  border-radius: 5px;
}
input[type="radio"]:checked + label {
  background: #f21f0c8f;
  color: white;
}
input[type="checkbox"]:checked + label {
  background: #f21f0c8f;
  color: white;
}
.btn_next_container {
  display: flex;
  justify-content: center;
  padding: 1em;
  margin-right: 2rem;
  margin-top: 2em;
}
.btn_next {
  background: none;
  background: #432ff0;
  border: none;
  color: white;
  font-size: 1.2em;
  padding: 0.5em 1em;
  border-radius: 5px;
  cursor: pointer;
}
.genre_btn_container {
  position: relative;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: space-around;
  max-width: 600px;
}

.label_checkbox {
  margin-right: 1em;
  margin-bottom: 1em;
}

/******************************** LOADER */
.loader {
  height: 50vh;
}
.lds-roller {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.lds-roller div {
  animation: lds-roller 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  transform-origin: 40px 40px;
}
.lds-roller div:after {
  content: " ";
  display: block;
  position: absolute;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #fff;
  margin: -4px 0 0 -4px;
}
.lds-roller div:nth-child(1) {
  animation-delay: -0.036s;
}
.lds-roller div:nth-child(1):after {
  top: 63px;
  left: 63px;
}
.lds-roller div:nth-child(2) {
  animation-delay: -0.072s;
}
.lds-roller div:nth-child(2):after {
  top: 68px;
  left: 56px;
}
.lds-roller div:nth-child(3) {
  animation-delay: -0.108s;
}
.lds-roller div:nth-child(3):after {
  top: 71px;
  left: 48px;
}
.lds-roller div:nth-child(4) {
  animation-delay: -0.144s;
}
.lds-roller div:nth-child(4):after {
  top: 72px;
  left: 40px;
}
.lds-roller div:nth-child(5) {
  animation-delay: -0.18s;
}
.lds-roller div:nth-child(5):after {
  top: 71px;
  left: 32px;
}
.lds-roller div:nth-child(6) {
  animation-delay: -0.216s;
}
.lds-roller div:nth-child(6):after {
  top: 68px;
  left: 24px;
}
.lds-roller div:nth-child(7) {
  animation-delay: -0.252s;
}
.lds-roller div:nth-child(7):after {
  top: 63px;
  left: 17px;
}
.lds-roller div:nth-child(8) {
  animation-delay: -0.288s;
}
.lds-roller div:nth-child(8):after {
  top: 56px;
  left: 12px;
}
@keyframes lds-roller {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
