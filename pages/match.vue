<template>
  <div class="match container">
    <hgroup>
      <h1>Match de Disc Golf</h1>
      <p>Trou {{ currentHole }}</p>
    </hgroup>

    <section v-if="!matchOver && currentHoleIndex < 9">
      <!-- Affichage du match en cours -->
      <article>

        <header>Trou {{ currentHole }}, ordre de jeux :</header>

        <ol>
          <li v-if="!currentHoleIsEven && matchType === 'double'">Le match simple</li>
          <li v-if="currentHoleIsEven && matchType === 'simple'">Le match double (paire sur les pairs)</li>
          <li><strong>{{ currentTeamAndPlayer }}</strong></li>
          <li><strong>{{ otherTeamAndPlayer }}</strong></li>
          <li v-if="currentHoleIsEven && matchType === 'double'">Le match simple</li>
          <li v-if="!currentHoleIsEven && matchType === 'simple'">Le match double (paire sur les pairs)</li>
        </ol>
        <footer>
          <h6>Qui a gagné le trou ?</h6>
        <div class="actions" role="group">
          <button @click="recordResult(StartingTeam.TeamA)">Équipe A</button>
          <button @click="recordResult(StartingTeam.TeamB)">Équipe B</button>
          <button @click="recordResult('draw')">Nul</button>
        </div>
        </footer>
      </article>
    </section>

    <!-- Match terminé -->
    <article v-else>
      <header>Match terminé</header>
      <p>
        <strong>Score final : </strong>
        <br>{{ team1Name }} : {{ teamAScore }}
        <br>{{ team2Name }} : {{ teamBScore }}
      </p>
      <h2 class="Vainqueur" v-if="matchOver || currentHoleIndex === 9">
        Vainqueur : {{ winnerDisplay }}
      </h2>
    </article>

    <!-- Tableau de progression -->
    <article class="progression">
      <header>Progression du match</header>
      <table>
        <thead>
          <tr>
            <th>Trou</th>
            <th v-for="hole in holes" :key="'hole-' + hole">{{ hole }}</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Équipe A</td>
            <td v-for="(hole, index) in holes" :key="'teamA-' + hole" :class="{
              'win-cell': progression[index]?.holeResult === StartingTeam.TeamA,
              'draw-cell': progression[index]?.holeResult === 'draw',
              'skipped': matchOver && index + 1 > currentHoleIndex
            }">
              {{ matchOver && index + 1 > currentHoleIndex ? '❌' : progression[index]?.teamAScore }}
            </td>
          </tr>
          <tr>
            <td>Équipe B</td>
            <td v-for="(hole, index) in holes" :key="'teamB-' + hole" :class="{
              'win-cell': progression[index]?.holeResult === StartingTeam.TeamB,
              'draw-cell': progression[index]?.holeResult === 'draw',
              'skipped': matchOver && index + 1 > currentHoleIndex
            }">
              {{ matchOver && index + 1 > currentHoleIndex ? '❌' : progression[index]?.teamBScore }}
            </td>
          </tr>
        </tbody>
      </table>
    </article>

    <!-- Bouton pour retourner à la configuration -->
    <div class="return-config">
      <button @click="goBackConfig">Retour configuration</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'

enum StartingTeam {
  TeamA = 'teamA',
  TeamB = 'teamB',
  Random = 'random'
}

const route = useRoute()
const router = useRouter()
const startHoleParam = Number(route.query.startHole) || 1
// Utiliser les noms d'équipe passés via la route, avec une valeur par défaut
const team1Name = ref(route.query.team1 || 'Team A')
const team2Name = ref(route.query.team2 || 'Team B')
const initialStartingTeam = route.query.startingTeam as StartingTeam || StartingTeam.TeamA

const matchType = route.query.matchType as string || 'simple'
const team1StartingPlayer: number = parseInt(route.query.team1StartingPlayer) || 1
const team2StartingPlayer: number = parseInt(route.query.team2StartingPlayer) || 1

const holes = Array.from({ length: 9 }, (_, i) => ((startHoleParam - 1 + i) % 18) + 1)

const currentHoleIndex = ref(0)
const teamAScore = ref(0)
const teamBScore = ref(0)
const progression = ref<{ teamAScore: number; teamBScore: number; holeResult?: string }[]>([])

// Nouvelle variable pour suivre la fin du match
const matchOver = ref(false)

// Remplacer currentStartingTeam computed par une variable réactive
const currentStartingTeam = ref<StartingTeam>(initialStartingTeam)
const currentHole = computed(() => holes[currentHoleIndex.value])

const remainingHoles = computed(() => 9 - currentHoleIndex.value)
const winnerDisplay = computed(() => {
  if (teamAScore.value === teamBScore.value) return 'match nul'
  const winningTeam = teamAScore.value > teamBScore.value ? 'Équipe A' : 'Équipe B'
  const winningScore = teamAScore.value > teamBScore.value ? teamAScore.value : teamBScore.value
  return `« ${winningTeam} » avec ${winningScore}&${remainingHoles.value}`
})

// Nouvelle computed pour suivre l'équipe en retard et ses marges
const trailingInfo = computed(() => {
  if (teamAScore.value === teamBScore.value) return null
  // L'équipe en retard a le score le plus bas
  const teamGeneric = teamAScore.value < teamBScore.value ? 'Team A' : 'Team B'
  const d = Math.abs(teamAScore.value - teamBScore.value)
  const R = remainingHoles.value
  // Pour revenir, l'équipe en retard doit gagner suffisamment de trous
  // Chaque victoire réduit l'écart de 2 points.
  const winsNeeded = Math.ceil(d / 2)
  const drawsAllowed = Math.max(0, R - winsNeeded)
  // Si l'équipe perd un trou, l'écart augmente de 2.
  // Pour maximiser le comeback, si elle perd L trous et gagne le reste (R - L),
  // il faut que 2*(R - L) >= d + 2*L  => L <= (2R - d) / 4
  const lossesAllowed = Math.floor((2 * R - d) / 4)
  return { team: teamGeneric, lossesAllowed, drawsAllowed }
})

// Nouvelle computed pour afficher l'équipe et éventuellement le joueur
const currentTeamAndPlayer = computed(() => {
  if (matchType === 'double') {
    if (currentStartingTeam.value === StartingTeam.TeamA) {
      return `Équipe A (Joueur ${(team1StartingPlayer + currentHole.value) % 2 + 1})`
    } else {
      return `Équipe B (Joueur ${(team2StartingPlayer + currentHole.value) % 2 + 1})`
    }
  }
  return currentStartingTeam.value === StartingTeam.TeamA ? 'Équipe A' : 'Équipe B'
})

const currentHoleIsEven = computed(() => currentHole.value % 2 === 0)

const otherTeamAndPlayer = computed(() => {
  if (matchType === 'double') {
    if (currentStartingTeam.value === StartingTeam.TeamA) {
      return `Équipe B (Joueur ${(team2StartingPlayer + currentHole.value) % 2 + 1})`
    } else {
      return `Équipe A (Joueur ${(team1StartingPlayer + currentHole.value) % 2 + 1})`
    }
  }
  return currentStartingTeam.value === StartingTeam.TeamA ? 'Équipe B' : 'Équipe A'
})

function saveState() {
  const state = {
    currentHoleIndex: currentHoleIndex.value,
    teamAScore: teamAScore.value,
    teamBScore: teamBScore.value,
    progression: progression.value,
    matchOver: matchOver.value,
    currentStartingTeam: currentStartingTeam.value
  }
  localStorage.setItem('matchState', JSON.stringify(state))
}

onMounted(() => {
  const saved = localStorage.getItem('matchState')
  if (saved) {
    const state = JSON.parse(saved)
    currentHoleIndex.value = state.currentHoleIndex || 0
    teamAScore.value = state.teamAScore || 0
    teamBScore.value = state.teamBScore || 0
    progression.value = state.progression || []
    matchOver.value = state.matchOver || false
    currentStartingTeam.value = state.currentStartingTeam || initialStartingTeam
  }
})

function recordResult(result: string) {
  // Mise à jour des scores et mise à jour de l'équipe de départ si une équipe gagne
  if (result === StartingTeam.TeamA) {
    teamAScore.value += 1
    teamBScore.value -= 1
    currentStartingTeam.value = StartingTeam.TeamA
  } else if (result === StartingTeam.TeamB) {
    teamBScore.value += 1
    teamAScore.value -= 1
    currentStartingTeam.value = StartingTeam.TeamB
  }
  // Pour un match nul, l'équipe de départ reste inchangée

  // Sauvegarde des scores cumulés pour ce trou
  progression.value.push({
    holeResult: result,
    teamAScore: teamAScore.value,
    teamBScore: teamBScore.value,
  })

  // Passage au trou suivant
  currentHoleIndex.value++

  // Vérifier si un comeback est impossible
  const remaining = 9 - currentHoleIndex.value
  if (Math.abs(teamAScore.value - teamBScore.value) > (remaining * 2)) {
    matchOver.value = true
  }
  saveState()
}

function goBackConfig() {
  if (confirm("Voulez-vous vraiment retourner à la page de configuration ?")) {
    localStorage.removeItem('matchState')
    router.push('/')
  }
}
</script>

<style scoped>

.win-cell {
  background-color: hwb(120 85% 0%);
}

.draw-cell {
  background-color: hwb(64 85% 0%);
}

.skipped {
  background-color: hwb(0 85% 0%);
}

td, th {
  text-align: center;
}
</style>
