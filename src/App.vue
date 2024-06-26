<template>
  <div class="">
    <!-- Preloader -->
    <div id="preloader" class="fixed inset-0 items-center justify-center bg-white z-50 flex flex-col">
      <div><img src="../public/pokemon-logo.png" alt=""></div>
      <span class="loader"></span>
    </div>

    <!-- Conteúdo principal -->
    <div id="app-content" class="flex flex-col backdrop-blur-md bg-black/30 rounded-2xl opacity-0 transition-opacity duration-500">
      <div class="header border select-none rounded-3xl font-semibold items-center bg-white h-24 flex flex-row justify-around text-center mx-auto m-7">
        <div class="logo drop-shadow-xl">
          <img class="drop-shadow-xl" src="../public/pokemon-logo.png" alt="">
        </div>
        <div>
          <Nav />
        </div>
      </div>

      <!-- Cards section -->
      <div class="flex flex-wrap items-center justify-center select-none rounded-xl">
        <div class="content flex items-center justify-center" v-for="poke in pokes" :key="poke.id">
          <div class="card hover:scale-105 hover:bg-slate-400 border rounded-tl-xl inset-14">
            <img :src="poke.sprite" :alt="`Imagem do ${poke.name}`" class="poke-image capitalize">
            <div class="contentName">
              <p class="text-black text-inherit font-semibold rounded-lg border" :style="{ backgroundColor: poke.color }">#{{ poke.id }}</p>
              <p><strong class="capitalize">{{ poke.name }}</strong></p>
            </div>
            <button @click.prevent="openModal(poke.id)" class="details-button">
              <strong>Detalhes</strong>
            </button>
          </div>
        </div>

        <!-- Overlay -->
        <div v-if="isModalOpen" class="overlay"></div>

        <!-- Modal -->
        <dialog ref="dialog" class="modal">
          <div class="dialog-content">
            <strong><h1 class="text-center font-semibold capitalize">{{ selectedPoke?.name }}</h1></strong>
            <div v-if="selectedPoke">
              <img :src="selectedPoke.sprite" :alt="`Imagem do ${selectedPoke.name}`" class="modal-image">
              <ul class="details-list">
                <li><strong>ID:</strong> {{ selectedPoke.id }}</li>
                <li><strong>Nome:</strong> <span class="capitalize">{{ selectedPoke.name }}</span></li>
                <li><strong>Altura:</strong> {{ selectedPoke.details.height }}</li>
                <li><strong>Peso:</strong> {{ selectedPoke.details.weight }}</li>
                <li class="abilities">
                  <strong>Habilidades</strong>
                  <ul>
                    <li v-for="ability in selectedPoke.details.abilities" :key="ability.ability.name" class="capitalize">
                      {{ ability.ability.name }}
                    </li>
                  </ul>
                </li>
              </ul>
            </div>
            <button @click="closeModal" class="close-button">Close</button>
          </div>
        </dialog>
      </div>

      <!-- Waves para o background -->
      <div>
        <div class="wave"></div>
        <div class="wave"></div>
        <div class="wave"></div>
      </div>
    </div>
  </div>
</template>
<script setup>
import axios from 'axios';
import { ref, onMounted } from 'vue';
import Nav from './components/Nav.vue';

const pokes = ref([]);
const selectedPoke = ref(null);
const dialog = ref(null);
const isModalOpen = ref(false);

// Função para obter detalhes do Pokémon e sua cor
const getPokeData = async (pokemon, index) => {
  const pokeDetails = await axios.get(pokemon.url);
  const colorResponse = await axios.get(`https://pokeapi.co/api/v2/pokemon-species/${index + 1}`);
  const colorName = colorResponse.data.color.name;
  return {
    id: index + 1,
    name: pokemon.name,
    sprite: pokeDetails.data.sprites.front_default,
    details: pokeDetails.data,
    color: colorName,
  };
};

// Get na api principal para exibir detalhes
axios.get('https://pokeapi.co/api/v2/pokemon?limit=100')
  .then(async (response) => {
    const pokeList = response.data.results;
    const pokesData = await Promise.all(pokeList.map(getPokeData));
    pokes.value = pokesData;
  })
  .catch(error => {
    console.error('Ocorreu um erro: ', error);
  });

function openModal(id) {
  selectedPoke.value = pokes.value.find(poke => poke.id === id);
  if (dialog.value) {
    dialog.value.showModal();
  }
  isModalOpen.value = true;
}

function closeModal() {
  if (dialog.value) {
    dialog.value.close();
  }
  isModalOpen.value = false;
}

onMounted(() => {
  window.addEventListener('load', () => {
    const preloader = document.getElementById('preloader');
    const appContent = document.getElementById('app-content');
    if (preloader) {
      preloader.style.display = 'none';
    }
    if (appContent) {
      appContent.style.opacity = '1';
    }
  });
});

</script>


<style>
@import url('https://fonts.cdnfonts.com/css/logotype');
body {
    font-family: 'Public Pixel', sans-serif;                                                                                         
    margin: auto;
    font-family: -apple-system, BlinkMacSystemFont, sans-serif;
    overflow: auto;
    background: url(../public/pokemon-3840x2160.jpg) 90% repeat;
    background-attachment: fixed;
}

*{
 font-family: 'LOGOTYPE', sans-serif;
font-family: 'LOGOTYPE-Hollow', sans-serif;
font-family: 'LOGOTYPE-Inverse', sans-serif;
font-family: 'LOGOTYPE-Light', sans-serif;                                              
}

/* Preloader */
#preloader {
  background: #fff;
}

.loader {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left-color: #3498db;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Animations and styles */
.wave {
    background: rgb(255 255 255 / 25%);
    border-radius: 1000% 1000% 0 0;
    position: fixed;
    width: 200%;
    height: 12em;
    animation: wave 10s -3s linear infinite;
    transform: translate3d(0, 0, 0);
    opacity: 0.8;
    bottom: 0;
    left: 0;
    z-index: -1;
}

.wave:nth-of-type(2) {
    bottom: -1.25em;
    animation: wave 18s linear reverse infinite;
    opacity: 0.8;
}

.wave:nth-of-type(3) {
    bottom: -2.5em;
    animation: wave 20s -1s reverse infinite;
    opacity: 0.9;
}

@keyframes wave {
    2% {
        transform: translateX(1);
    }

    25% {
        transform: translateX(-25%);
    }

    50% {
        transform: translateX(-50%);
    }

    75% {
        transform: translateX(-25%);
    }

    100% {
        transform: translateX(1);
    }
}

.header {
  width: 80%;
}

.card {
  padding: 2rem;
  margin: 1rem;
  height: 18rem;
  width: 14rem;
  text-align: center;
  background: white;
  border-radius: 1rem;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.1);
}

.poke-image {
  border-bottom: 4px solid red;
}

.details-button {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 600;
  border: 1px solid gray;
  border-radius: 9999px;
  transition: all 0.2s;
}

.details-button:hover {
  background-color: black;
  color: white;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1;
}

.modal {
  border-radius: 1rem;
  z-index: 2;
}

.dialog-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 25rem;
  padding: 1rem;
  background: white;
  border-radius: 0.5rem;
}

.modal-image {
  width: 100%;
}

.details-list {
  text-align: left;
  margin-top: 1rem;
}

.abilities {
  margin-top: 1rem;
  border-top: 2px solid red;
  padding-top: 0.5rem;
}

.close-button {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  border: 1px solid gray;
  border-radius: 9999px;
  font-weight: 600;
  transition: all 0.2s;
}

.close-button:hover {
  background-color: black;
  color: white;
}
</style>
