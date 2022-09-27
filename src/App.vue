<script setup>
import { ref, computed, onMounted } from 'vue';

const search_query = ref('')
const my_anime = ref([])
const search_results = ref([])

const my_anime_asc = computed(() => {
  return my_anime.value.sort((a, b) => {
    return a.title.localeCompare(b.title)
  })
})

const searchAnime = () => {
  const url = `https://api.jikan.moe/v4/anime?q=${search_query.value}`
  fetch(url)
  .then(res => res.json())
  .then(res => {
    search_results.value= res.data
  })
}

const handleInput = e => {
  if (!e.target.value) {
    search_results.value = []
  }
}

const addAnime = anime => {
  search_results.value = []
  search_query.value = ''

  my_anime.value.push({
    id: anime.mal_id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    total_episodes: anime.episodes,
    watched_episodes: 0,
  })

  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const increaseWatch = anime => {
  anime.watched_episodes++
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
} 

const decreaseWatch = anime => {
  anime.watched_episodes--
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
} 

onMounted(() => {
  my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || []
})

</script>

<template>
  <main>
    <header>
      <img src="./assets/MAS.png" alt="">
      <h1>My Anime Space</h1>
      <form @submit.prevent="searchAnime">
        <input 
        type="text"
        placeholder="Digite o nome do anime..."
        v-model="search_query" 
        @input="handleInput"
        />
        <button type="submit">Pesquisar</button>
      </form>
    </header>

    <div class="results" v-if="search_results.length > 0">
      <div class="result" v-for="anime in search_results">
        <img :src="anime.images.jpg.image_url" alt="">
        <div class="details">
          <h4>{{ anime.title}}</h4>
          <p :title="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0,120)}}...
          </p>
          <span></span>
          <button @click="addAnime(anime)">Adicionar</button>
        </div>
      </div>
    </div>

    <div class="myanime" v-if="my_anime.length > 0">
      <div class="my-animes">
        <div v-for="anime in my_anime_asc" class="my-anime">
          <img :src="anime.image" alt="" />
          <h3>{{anime.title}}</h3>
          <span class="episodes">
            {{ anime.watched_episodes }} / {{ anime.total_episodes }}
          </span>
          <button class="epHandle"
          @click="increaseWatch(anime)"
          v-if="anime.total_episodes !== anime.watched_episodes">+
          </button>

          <button class="epHandle"
          @click="decreaseWatch(anime)" v-if="anime.watched_episodes > 0">-</button>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>

</style>
