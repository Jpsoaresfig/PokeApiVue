<template>
  <div>
    <center><h1>Pokémons</h1></center>

    <div class="card-container">
      <div class="card" v-for="pokemon in pokemons" :key="pokemon.name" @click="openModal(pokemon)">
        <img :src="getPokemonImage(pokemon.url)" :alt="pokemon.name" />
        <h2>{{ capitalize(pokemon.name) }}</h2>
      </div>
    </div>

    
    <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content" v-if="detailedPokemon">
        <button class="close-btn" @click="closeModal">×</button>
        <img :src="detailedPokemon.sprites.other['official-artwork'].front_default || detailedPokemon.sprites.front_default" :alt="detailedPokemon.name" />
        <h2>{{ capitalize(detailedPokemon.name) }}</h2>

        <section class="info-section">
          <h3>Tipos</h3>
          <div class="types">
            <span v-for="type in detailedPokemon.types" :key="type.type.name" :class="['type', type.type.name]">
              {{ capitalize(type.type.name) }}
            </span>
          </div>
        </section>

        <section class="info-section">
          <h3>Habilidades</h3>
          <ul>
            <li v-for="ability in detailedPokemon.abilities" :key="ability.ability.name">
              {{ capitalize(ability.ability.name) }} <span v-if="ability.is_hidden">(Oculta)</span>
            </li>
          </ul>
        </section>

        <section class="info-section">
          <h3>Status</h3>
          <ul class="stats">
            <li v-for="stat in detailedPokemon.stats" :key="stat.stat.name">
              <strong>{{ capitalize(stat.stat.name) }}:</strong> {{ stat.base_stat }}
              <div class="stat-bar">
                <div class="stat-fill" :style="{ width: stat.base_stat + '%' }"></div>
              </div>
            </li>
          </ul>
        </section>

        <section class="info-section description">
          <h3>Descrição</h3>
          <p>{{ description }}</p><!--não tem a descrição-->
        </section>
      </div>
    </div>
  </div>
</template>

<script setup>

import { ref, onMounted } from 'vue'


const pokemons = ref([])
const showModal = ref(false)
const detailedPokemon = ref(null)
const description = ref('')

function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1)
}

async function fetchPokemons() {
  const res = await fetch('https://pokeapi.co/api/v2/pokemon?limit=1000')
  const data = await res.json()
  pokemons.value = data.results
}

function getPokemonImage(url) {
  const id = url.split('/').filter(Boolean).pop()
  return `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${id}.png`
}

async function openModal(pokemon) {
  showModal.value = true
  detailedPokemon.value = null
  description.value = ''

  const id = pokemon.url.split('/').filter(Boolean).pop()

  
  const resDetails = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)
  detailedPokemon.value = await resDetails.json()

  
  const resSpecies = await fetch(`https://pokeapi.co/api/v2/pokemon-species/${id}`)
  const speciesData = await resSpecies.json()

  const flavorEntry = speciesData.flavor_text_entries.find(
    entry => entry.language.name === 'pt' || entry.language.name === 'pt-br'
  )
  description.value = flavorEntry ? flavorEntry.flavor_text.replace(/\n|\f/g, ' ') : 'Descrição não disponível.'
}

function closeModal() {
  showModal.value = false
  detailedPokemon.value = null
  description.value = ''
}

onMounted(() => {
  fetchPokemons()
})
</script>

<style>
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #f0f4f8;
  margin: 0;
  padding: 2rem 1rem;
  color: #333;
}

h1 {
  font-weight: 700;
  margin-bottom: 2rem;
  color: #2c3e50;
}

.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  justify-content: center;
}

.card {
  background: white;
  border-radius: 15px;
  box-shadow: 0 6px 15px rgba(0,0,0,0.1);
  width: 140px;
  padding: 1rem 0.5rem;
  text-align: center;
  cursor: pointer;
  transition: transform 0.25s ease, box-shadow 0.25s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 25px rgba(0,0,0,0.15);
}

.card img {
  width: 96px;
  height: 96px;
  margin-bottom: 0.5rem;
}

.card h2 {
  font-size: 1.1rem;
  text-transform: capitalize;
  color: #34495e;
  margin: 0;
}


.modal-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 20px;
  max-width: 400px;
  width: 100%;
  max-height: 90vh;
  overflow-y: auto;
  padding: 2rem 2.5rem 2.5rem;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
  position: relative;
  color: #2c3e50;
}

.modal-content img {
  width: 180px;
  height: 180px;
  display: block;
  margin: 0 auto 1rem;
}

.modal-content h2 {
  text-align: center;
  margin-bottom: 1rem;
  font-weight: 700;
  font-size: 2rem;
  text-transform: capitalize;
  color: #27ae60;
}

.close-btn {
  position: absolute;
  top: 15px;
  right: 20px;
  font-size: 2.2rem;
  border: none;
  background: transparent;
  cursor: pointer;
  color: #888;
  transition: color 0.3s ease;
}

.close-btn:hover {
  color: #e74c3c;
}

.info-section {
  margin-top: 1.5rem;
}

.info-section h3 {
  margin-bottom: 0.75rem;
  font-weight: 600;
  font-size: 1.2rem;
  border-bottom: 2px solid #27ae60;
  padding-bottom: 0.2rem;
  color: #34495e;
}


.types {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.type {
  padding: 0.3rem 0.8rem;
  border-radius: 12px;
  font-weight: 600;
  color: white;
  text-transform: capitalize;
  font-size: 0.9rem;
  user-select: none;
}


.type.grass { background-color: #78C850; }
.type.poison { background-color: #A040A0; }
.type.fire { background-color: #F08030; }
.type.water { background-color: #6890F0; }
.type.bug { background-color: #A8B820; }
.type.normal { background-color: #A8A878; }
.type.electric { background-color: #F8D030; color: #555; }
.type.ground { background-color: #E0C068; color: #555; }
.type.fairy { background-color: #EE99AC; }
.type.fighting { background-color: #C03028; }
.type.psychic { background-color: #F85888; }
.type.rock { background-color: #B8A038; }
.type.ghost { background-color: #705898; }
.type.ice { background-color: #98D8D8; color: #555; }
.type.dragon { background-color: #7038F8; }
.type.dark { background-color: #705848; }
.type.steel { background-color: #B8B8D0; color: #555; }


.stats {
  list-style: none;
  padding: 0;
  margin: 0;
}

.stats li {
  margin-bottom: 0.8rem;
  font-weight: 500;
  font-size: 1rem;
  display: flex;
  flex-direction: column;
}

.stat-bar {
  height: 8px;
  background-color: #eee;
  border-radius: 4px;
  margin-top: 4px;
  overflow: hidden;
}

.stat-fill {
  height: 100%;
  background-color: #27ae60;
  border-radius: 4px 0 0 4px;
}


.description p {
  font-style: italic;
  color: #555;
  line-height: 1.4;
  margin: 0;
}
</style>
