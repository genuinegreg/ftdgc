<template>
  <div class="config container">
    <hgroup>
      <h2>Match de discgolf</h2>
      <p>Configuration du match</p>
    </hgroup>

    <!-- Sélectionnez le trou de départ via des boutons -->
    <article class="hole-selection">
      <header>Sélectionnez le trou de départ :</header>
      <div class="holes">
        <button v-for="n in 18" :key="n" @click="startHole = n" :class="{ 'outline': startHole !== n }">
          {{ n }}
        </button>
      </div>
    </article>

    <article>
      <header>Sélectionnez l'équipe de départ</header>
      <form action="">
        <label>
          <input type="radio" :value="StartingTeam.TeamA" v-model="startingTeam" /> Équipe A
        </label>
        <label>
          <input type="radio" :value="StartingTeam.TeamB" v-model="startingTeam" /> Équipe B
        </label>
        <label>
          <input type="radio" :value="StartingTeam.Random" v-model="startingTeam" /> Aléatoire
        </label>

        <blockquote v-if="matchType === MatchType.Double && startingTeam === StartingTeam.Random">
          ⚠ le tirage au sort ne fonctionne pas pour le Double.
          Il faut :
          <ol>
            <li>Faire un tirage</li>
            <li>L'équipe qui gagne le tirage a le choix entre : </li>
            <ul>
              <li>Choisir l'équipe qui débute le match</li>
              <li>Choisir connaitre quel joueur de l'autre pair va débuter</li>
            </ul>
            <li>L'autre équipe prend le choix restant</li>
          </ol>

        </blockquote>

      </form>
    </article>

    <!-- Nouveau sélecteur pour le type de match -->
    <article class="match-type">
      <header>Type de match</header>
      <label>
        <input type="radio" :value="MatchType.Simple" v-model="matchType" /> Simple
      </label>
      <label>
        <input type="radio" :value="MatchType.Double" v-model="matchType" /> Double
      </label>
    </article>


    <!-- Section de sélection des joueurs pour le double -->
    <article v-if="matchType === MatchType.Double" class="starting-players">
      <header>
        Joueur au départ des trous {{ isStartHoleEven ? 'pair' : 'impair' }} <small>({{ fiveHoles.join(', ')
        }})</small>
      </header>
      <div class="team-players grid">
        <fieldset>
          <legend>Équipe A</legend>
          <select v-model="team1StartingPlayer">
            <option value="1">Joueur 1</option>
            <option value="2">Joueur 2</option>
          </select>
        </fieldset>
        <fieldset>
          <legend>Équipe B</legend>
          <select v-model="team2StartingPlayer">
            <option value="1">Joueur 1</option>
            <option value="2">Joueur 2</option>
          </select>
        </fieldset>
      </div>
    </article>

    <article v-if="matchType === MatchType.Double" class="starting-players">
      <header>
        Joueur au départ des trous
        {{ isStartHoleEven ? 'impairs' : 'pairs' }} <small>({{ fourHoles.join(', ') }})</small>
      </header>
      <div class="team-players grid">
        <fieldset>
          <legend>Équipe A</legend>
          <select v-model="team1StartingPlayer">
            <option value="2">Joueur 1</option>
            <option value="1">Joueur 2</option>
          </select>
        </fieldset>
        <fieldset>
          <legend>Équipe B</legend>
          <select v-model="team2StartingPlayer">
            <option value="2">Joueur 1</option>
            <option value="1">Joueur 2</option>
          </select>
        </fieldset>
      </div>
    </article>


    <div class="grid">
      <button @click="handleStart" :disabled="!startingTeam">
        {{ startingTeam === StartingTeam.Random ? 'Sélectionner une équipe puis démarrer le match' : 'Démarrer le match'
        }}
      </button>
    </div>


    <br />
    <br />

  </div>

  <!-- Overlay -->
  <div v-if="overlayVisible" class="overlay">
    <div class="overlay-content">
      <h2>
        {{ startingTeam === StartingTeam.TeamA ? 'L\'équipe A' : 'L\'equipe B' }} débute le match
      </h2>
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

<style scoped type="scss">
/* .hole-selection {
  display: grid;
  gap: 1rem;
  grid-auto-flow: column;
} */

.hole-selection>.holes {

  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(6ch, 1fr));

  gap: var(--pico-spacing);



  >button {
    width: 6ch;
    aspect-ratio: 1;
  }


}

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
</style>
