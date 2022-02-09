<template>
  <div class="card_container">
    <img
      class="poster"
      :src="
        data.poster_path != null
          ? `https://image.tmdb.org/t/p/w300/${data.poster_path}`
          : '/_nuxt/assets/notFound.png'
      "
      alt=""
    />
    <div class="card_info">
      <div class="txt-info">
        <h2 class="title_data">{{ data.name ? data.name : data.title }}</h2>
        <span>{{ dateFr() }}</span>
        <div></div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "Card",
  props: ["data"],
  data() {
    return {
      date: "",
    };
  },
  methods: {
    dateFr() {
      if (this.data.release_date) {
        const maDate = new Date(this.data.release_date);
        return maDate.toLocaleDateString("fr");
      }
    },
  },
};
</script>

<style scoped>
a {
  color: white;
}

.title_data {
  padding: 0;
  margin: 0;
  font-size: 1em;
  font-weight: 500;
}
.poster {
  width: 300px;
  height: 450px;
  object-fit: cover;
}
.card_container {
  width: 300px;
  position: relative;
  height: 450px;
  margin-bottom: 1em;
  box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 15px;
  transition: 300ms ease-in-out;
}

.card_container:hover {
  transform: scale(1.05);
}
.card_info {
  display: block;
  position: absolute;
  bottom: 0;
  background: rgba(0, 0, 0, 0.733);
  width: 100%;
}
.txt-info {
  padding: 1em;
}
</style>
