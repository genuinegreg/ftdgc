<template>
  <div class="config container-fluid">
    <h1>Configuration du match</h1>
    
    <!-- Sélectionnez le trou de départ via des boutons -->
    <div class="hole-selection">
      <p>Sélectionnez le trou de départ :</p>
      <div class="holes">
        <button 
          v-for="n in 18" 
          :key="n" 
          @click="startHole = n"
          :class="{'selected': startHole === n}">
          {{ n }}
        </button>
      </div>
    </div>

    <div>
      <h2>Sélectionnez l'équipe de départ</h2>
      <label>
        <input type="radio" value="team1" v-model="startingTeam" /> Équipe 1
      </label>
      <label>
        <input type="radio" value="team2" v-model="startingTeam" /> Équipe 2
      </label>
      <label>
        <input type="radio" value="random" v-model="startingTeam" /> Aléatoire
      </label>
    </div>

    <!-- Champs pour personnaliser les noms des équipes -->
    <h2>Personalisé les noms d'équipes</h2>
    <div>
      <label for="team1">Équipe 1 :</label>
      <input type="text" v-model="team1" id="team1" placeholder="Équipe 1" />
      <button @click="team1 = 'Nous'; team2 = 'Eux'">C'est moi</button>
    </div>
    <div>
      <label for="team2">Équipe 2 :</label>
      <input type="text" v-model="team2" id="team2" placeholder="Équipe 2" />
      <button @click="team2 = 'Nous'; team1 = 'Eux'">C'est moi</button>
    </div>

    <br><br>

    <button @click="handleStart" :disabled="!startingTeam">
      {{ startingTeam === 'random' ? 'Sélectionner une équipe puis démarrer le match' : 'Démarrer le match' }}
    </button>
  </div>
  
  <!-- Overlay -->
  <div v-if="overlayVisible" class="overlay">
    <div class="overlay-content">
    Prochaine équipe à jouer : {{ startingTeam }}
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const startHole = ref(1)
const startingTeam = ref()
const overlayVisible = ref(false)
// Nouvelles variables pour les noms d’équipes personnalisées
const team1 = ref('Équipe 1')
const team2 = ref('Équipe 2')

function randomizeTeam() {
  startingTeam.value = Math.random() < 0.5 ? 'Team A' : 'Team B'
  overlayVisible.value = true
  setTimeout(() => {
    overlayVisible.value = false
    startMatch()
  }, 2000)
}

function handleStart() {
  // Vérifier que l'équipe de départ est sélectionnée
  if (!startingTeam.value || startingTeam.value === '') {
    alert("Veuillez sélectionner une équipe de départ.")
    return
  }
  if (startingTeam.value === 'random') {
    startingTeam.value = Math.random() < 0.5 ? 'team1' : 'team2'
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
  // Pass configuration as query parameters so the match page can pick them up
  router.push({
    path: '/match',
    query: {
      startHole: startHole.value,
      startingTeam: startingTeam.value,
      team1: team1.value,
      team2: team2.value,
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
  background-color: rgba(0,0,0,0.6);
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
</style>
