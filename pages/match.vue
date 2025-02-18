<template>
    <div class="match">
        <h1>Match de Disc Golf</h1>

        <!-- Affichage du match en cours -->
        <div v-if="!matchOver && currentHoleIndex < 9">
            <h2>Trou {{ currentHole }}</h2>
            <p>
                <strong>Next to Tee :</strong> {{ currentStartingTeam }}<br>
            </p>
            <!-- Afficher les infos pour l'équipe en retard si les scores diffèrent -->
            <div class="actions">
                <button @click="recordResult('Team A')">équipe 1</button>
                <button @click="recordResult('Team B')">équipe 2</button>
                <button @click="recordResult('draw')">Nul</button>
            </div>
        </div>

        <!-- Match terminé -->
        <div v-else>
            <h2>Match terminé</h2>
            <p>
                Score final :
                <br>{{ team1Name }} : {{ teamAScore }}
                <br>{{ team2Name }} : {{ teamBScore }}
            </p>
            <p v-if="matchOver">
                Vainqueur : {{ winnerDisplay }}
            </p>
        </div>

        <!-- Tableau de progression -->
        <div class="progression">
            <h3>Progression du match</h3>
            <table>
                <thead>
                    <tr>
                        <th>Trou</th>
                        <th v-for="hole in holes" :key="'hole-' + hole">{{ hole }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>{{ team1Name }}</td>
                        <td v-for="(hole, index) in holes" :key="'teamA-' + hole" :class="{'skipped': matchOver && index+1 > currentHoleIndex}">
                            {{ matchOver && index+1 > currentHoleIndex ? '❌' : (progression[index] ? progression[index].teamAScore : '') }}
                        </td>
                    </tr>
                    <tr>
                        <td>{{ team2Name }}</td>
                        <td v-for="(hole, index) in holes" :key="'teamB-' + hole" :class="{'skipped': matchOver && index+1 > currentHoleIndex}">
                            {{ matchOver && index+1 > currentHoleIndex ? '❌' : (progression[index] ? progression[index].teamBScore : '') }}
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <!-- Bouton pour retourner à la configuration -->
        <div class="return-config">
            <button @click="goBackConfig">Retour configuration</button>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()
const startHoleParam = Number(route.query.startHole) || 1
// Utiliser les noms d'équipe passés via la route, avec une valeur par défaut
const team1Name = ref(route.query.team1 || 'Team A')
const team2Name = ref(route.query.team2 || 'Team B')
const initialStartingTeam = route.query.startingTeam || team1Name.value

const holes = Array.from({ length: 9 }, (_, i) => ((startHoleParam - 1 + i) % 18) + 1)

const currentHoleIndex = ref(0)
const teamAScore = ref(0)
const teamBScore = ref(0)
const progression = ref([])

// Nouvelle variable pour suivre la fin du match
const matchOver = ref(false)

// Remplacer currentStartingTeam computed par une variable réactive
const currentStartingTeam = ref(initialStartingTeam)
const currentHole = computed(() => holes[currentHoleIndex.value])

const remainingHoles = computed(() => 9 - currentHoleIndex.value)
const winnerDisplay = computed(() => {
  if (teamAScore.value === teamBScore.value) return 'Nul'
  // Déterminer l'équipe gagnante et utiliser le nom passé via la route
  const winningTeamGeneric = teamAScore.value > teamBScore.value ? 'Team A' : 'Team B'
  const winningTeam = winningTeamGeneric === 'Team A' ? team1Name.value : team2Name.value
  const winningScore = winningTeamGeneric === 'Team A' ? teamAScore.value : teamBScore.value
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

function recordResult(result) {
    // Mise à jour des scores et mise à jour de l'équipe de départ si une équipe gagne
    if (result === 'Team A') {
        teamAScore.value += 1
        teamBScore.value -= 1
        currentStartingTeam.value = team1Name.value
    } else if (result === 'Team B') {
        teamBScore.value += 1
        teamAScore.value -= 1
        currentStartingTeam.value = team2Name.value
    }
    // Pour un match nul, l'équipe de départ reste inchangée

    // Sauvegarde des scores cumulés pour ce trou
    progression.value.push({
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
.match {
    max-width: 600px;
    margin: auto;
    text-align: center;
    font-family: sans-serif;
}

.actions button {
    margin: 0.5rem;
    padding: 0.5rem 1rem;
    text-transform: capitalize;
}

.progression {
    margin-top: 2rem;
}

table {
    width: 100%;
    border-collapse: collapse;
}

th,
td {
    border: 1px solid #ccc;
    padding: 0.5rem;
    text-align: center;
}

.skipped {
  background-color: red;
  color: white;
  font-weight: bold;
}

.return-config {
  margin-top: 1rem;
}
.return-config button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
}
</style>
