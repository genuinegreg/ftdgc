<template>
  <div class="config container-fluid">
    <h1>Configuration du match</h1>

    <!-- Sélectionnez le trou de départ via des boutons -->
    <div class="hole-selection">
      <p>Sélectionnez le trou de départ :</p>
      <div class="holes">
        <button v-for="n in 18" :key="n" @click="startHole = n" :class="{ 'selected': startHole === n }">
          {{ n }}
        </button>
      </div>
    </div>

    <div>
      <h2>Sélectionnez l'équipe de départ</h2>
      <label>
        <input type="radio" :value="StartingTeam.TeamA" v-model="startingTeam" /> Équipe A
      </label>
      <label>
        <input type="radio" :value="StartingTeam.TeamB" v-model="startingTeam" /> Équipe B
      </label>
      <label>
        <input type="radio" :value="StartingTeam.Random" v-model="startingTeam" /> Aléatoire
      </label>
    </div>

    <!-- Nouveau sélecteur pour le type de match -->
    <div class="match-type">
      <h2>Type de match</h2>
      <label>
        <input type="radio" :value="MatchType.Simple" v-model="matchType" /> Simple
      </label>
      <label>
        <input type="radio" :value="MatchType.Double" v-model="matchType" /> Double
      </label>
    </div>

    <br><br>

    <!-- Section de sélection des joueurs pour le double -->
    <div v-if="matchType === MatchType.Double" class="starting-players">
      <h3>
        Joueur au départ des trous {{ isStartHoleEven ? 'pair' : 'impair' }} <small>({{ fiveHoles.join(', ') }})</small>
      </h3>
      <div class="team-players">
        <div>
          <h3>Équipe A</h3>
          <select v-model="team1StartingPlayer">
            <option value="1">Joueur 1</option>
            <option value="2">Joueur 2</option>
          </select>
        </div>
        <div>
          <h3>Équipe B</h3>
          <select v-model="team2StartingPlayer">
            <option value="1">Joueur 1</option>
            <option value="2">Joueur 2</option>
          </select>
        </div>
      </div>
    </div>

    <div v-if="matchType === MatchType.Double" class="starting-players">
      <h3>
        Joueur au départ des trous 
        {{ isStartHoleEven ? 'impairs' : 'pairs' }} <small>({{ fourHoles.join(', ') }})</small>
      </h3>
      <div class="team-players">
        <div>
          <h3>Équipe A</h3>
          <select v-model="team1StartingPlayer">
            <option value="2">Joueur 1</option>
            <option value="1">Joueur 2</option>
          </select>
        </div>
        <div>
          <h3>Équipe B</h3>
          <select v-model="team2StartingPlayer">
            <option value="2">Joueur 1</option>
            <option value="1">Joueur 2</option>
          </select>
        </div>
      </div>
    </div>

    <br><br>

    <button @click="handleStart" :disabled="!startingTeam">
      {{ startingTeam === StartingTeam.Random ? 'Sélectionner une équipe puis démarrer le match' : 'Démarrer le match' }}
    </button>
  </div>

  <!-- Overlay -->
  <div v-if="overlayVisible" class="overlay">
    <div class="overlay-content">
      Résultat : {{ startingTeam === StartingTeam.TeamA ? 'Équipe A' : 'Équipe B' }}
    </div>
  </div>
</template>

<script setup lang="ts">

import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'

enum StartingTeam {
  TeamA = 'teamA',
  TeamB = 'teamB',
  Random = 'random'
}

enum MatchType {
  Simple = 'simple',
  Double = 'double'
}

const router = useRouter()
const startHole = ref(1)
const startingTeam = ref<StartingTeam>(StartingTeam.Random)
const overlayVisible = ref(false)
const matchType = ref<MatchType>(MatchType.Double)
const team1StartingPlayer = ref('1')
const team2StartingPlayer = ref('1')

const isStartHoleEven = computed(() => startHole.value % 2 === 0)
const fiveHoles = computed(() => {
  const result: number[] = []
  for (let i = 0; i < 5; i++) {
    result.push(((startHole.value - 1 + i * 2) % 18) + 1)
  }
  return result
})
const fourHoles = computed(() => {
  
  const result: number[] = []
  for (let i = 0; i < 4; i++) {
    result.push(((startHole.value + i * 2) % 18) + 1)
  }
  return result
})

function handleStart() {
  if (!startingTeam.value) {
    alert("Veuillez sélectionner une équipe de départ.")
    return
  }
  if (startingTeam.value === StartingTeam.Random) {
    startingTeam.value = Math.random() < 0.5 ? StartingTeam.TeamA : StartingTeam.TeamB
    overlayVisible.value = true
    setTimeout(() => {
      overlayVisible.value = false
      startMatch()
    }, 2000)
  } else {
    startMatch()
  }
}

function startMatch() {
  router.push({
    path: '/match',
    query: {
      startHole: startHole.value,
      startingTeam: startingTeam.value,
      matchType: matchType.value, // Ajout du type de match
      // Ajouter les joueurs de départ seulement en double
      ...(matchType.value === 'double' && {
        team1StartingPlayer: team1StartingPlayer.value,
        team2StartingPlayer: team2StartingPlayer.value,
      }),
    },
  })
}
</script>

<style scoped>
.config {
  max-width: 500px;
  margin: auto;
  text-align: center;
}

.hole-selection p {
  margin-bottom: 0.5rem;
}

.holes {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.5rem;
}

.holes button {
  width: 40px;
  height: 40px;
  border: 1px solid #ccc;
  background-color: #f9f9f9;
  cursor: pointer;
}

.holes button.selected {
  background-color: #0d6efd;
  color: white;
  font-weight: bold;
}

/* Nouveau style pour l'overlay */
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.overlay-content {
  background: white;
  padding: 20px;
  border-radius: 5px;
}

.match-type {
  margin-top: 1rem;
  text-align: center;
}

.match-type label {
  margin: 0 1rem;
}

.starting-players {
  margin-top: 2rem;
}

.team-players {
  display: flex;
  justify-content: space-around;
  margin-top: 1rem;
}

.team-players h3 {
  margin-bottom: 0.5rem;
}

.team-players select {
  padding: 0.5rem;
  margin: 0.5rem 0;
  width: 120px;
  border-radius: 4px;
  border: 1px solid #ccc;
}
</style>
