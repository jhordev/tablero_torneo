<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// Initialize bracket data structure
const defaultBracket = {
  roundOf16: [
    { team1: 'Hector Tarrillo', score1: 0, team2: 'Frank Rojas', score2: 0 },
    { team1: 'Julio Saavedra', score1: 0, team2: 'Alex Chuqui', score2: 0 },
    { team1: 'Jorge Cortez', score1: 0, team2: 'Pool Siesquen', score2: 0 },
    { team1: 'Erick Vasquez', score1: 0, team2: 'Luis Rios', score2: 0 },
    { team1: 'Jhoseph Troncos', score1: 0, team2: 'Wesley Menor', score2: 0 },
    { team1: 'Eli Campos', score1: 0, team2: 'Julian Rodríguez', score2: 0 },
    { team1: 'Luis Perez', score1: 0, team2: 'Joan Diaz', score2: 0 },
    { team1: 'Sam Vasquez', score1: 0, team2: 'Fernando Rentería', score2: 0 }
  ],
  quarterFinals: [
    { team1: '', score1: 0, team2: '', score2: 0 },
    { team1: '', score1: 0, team2: '', score2: 0 },
    { team1: '', score1: 0, team2: '', score2: 0 },
    { team1: '', score1: 0, team2: '', score2: 0 }
  ],
  semiFinals: [
    { team1: '', score1: 0, team2: '', score2: 0 },
    { team1: '', score1: 0, team2: '', score2: 0 }
  ],
  final: { team1: '', score1: 0, team2: '', score2: 0 }
}

// Load from localStorage or use default
const loadBracket = () => {
  const saved = localStorage.getItem('tournamentBracket')
  return saved ? JSON.parse(saved) : defaultBracket
}

const bracket = ref(loadBracket())

// Save to localStorage when bracket changes
watch(bracket, (newBracket) => {
  localStorage.setItem('tournamentBracket', JSON.stringify(newBracket))
}, { deep: true })

// Determine winner of a match
const getWinner = (match) => {
  if (!match.team1 || !match.team2) return ''
  if (match.score1 > match.score2) return match.team1
  if (match.score2 > match.score1) return match.team2
  return ''
}

// Watch for changes in Round of 16 and update Quarter Finals
watch(() => bracket.value.roundOf16, (matches) => {
  for (let i = 0; i < 4; i++) {
    const match1 = matches[i * 2]
    const match2 = matches[i * 2 + 1]
    bracket.value.quarterFinals[i].team1 = getWinner(match1)
    bracket.value.quarterFinals[i].team2 = getWinner(match2)
  }
}, { deep: true })

// Watch for changes in Quarter Finals and update Semi Finals
watch(() => bracket.value.quarterFinals, (matches) => {
  for (let i = 0; i < 2; i++) {
    const match1 = matches[i * 2]
    const match2 = matches[i * 2 + 1]
    bracket.value.semiFinals[i].team1 = getWinner(match1)
    bracket.value.semiFinals[i].team2 = getWinner(match2)
  }
}, { deep: true })

// Watch for changes in Semi Finals and update Final
watch(() => bracket.value.semiFinals, (matches) => {
  bracket.value.final.team1 = getWinner(matches[0])
  bracket.value.final.team2 = getWinner(matches[1])
}, { deep: true })

// Computed winner
const champion = computed(() => getWinner(bracket.value.final))
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br  from-slate-900 via-blue-900 to-slate-900 p-4 md:p-8">
    <div class="max-w-[1800px] mx-auto">
      <h1 class="text-4xl md:text-5xl font-bold text-center text-white mb-8 md:mb-12">
        I TORNEO DE VIDEOJUEGOS PES 2019
      </h1>

      <div class="bracket-container overflow-x-auto pb-8">
        <div class="bracket-grid min-w-[1400px]">

          <!-- LEFT SIDE - Round of 16 (Top 4 matches) -->
          <div class="round-column">
            <h2 class="round-title">Grupo 1</h2>
            <div class="matches-container space-y-8">
              <div v-for="(match, index) in bracket.roundOf16.slice(0, 4)" :key="`r16-${index}`" class="match-card">
                <div class="team-row">
                  <input v-model="match.team1" type="text" placeholder="Team Name" class="team-input" />
                  <input v-model.number="match.score1" type="number" min="0" class="score-input" />
                </div>
                <div class="team-row">
                  <input v-model="match.team2" type="text" placeholder="Team Name" class="team-input" />
                  <input v-model.number="match.score2" type="number" min="0" class="score-input" />
                </div>
              </div>
            </div>
          </div>

          <!-- LEFT SIDE - Quarter Finals (Top 2 matches) -->
          <div class="round-column">
            <h2 class="round-title">Cuartos de final</h2>
            <div class="matches-container space-y-16">
              <div v-for="(match, index) in bracket.quarterFinals.slice(0, 2)" :key="`qf-${index}`" class="match-card">
                <div class="team-row">
                  <div class="team-display">{{ match.team1 || 'Team' }}</div>
                  <input v-model.number="match.score1" type="number" min="0" class="score-input"
                    :disabled="!match.team1" />
                </div>
                <div class="team-row">
                  <div class="team-display">{{ match.team2 || 'Team' }}</div>
                  <input v-model.number="match.score2" type="number" min="0" class="score-input"
                    :disabled="!match.team2" />
                </div>
              </div>
            </div>
          </div>

          <!-- LEFT SIDE - Semi Final (Top match) -->
          <div class="round-column">
            <h2 class="round-title">Semi-final</h2>
            <div class="matches-container">
              <div class="match-card mt-32">
                <div class="team-row">
                  <div class="team-display">{{ bracket.semiFinals[0].team1 || 'Team' }}</div>
                  <input v-model.number="bracket.semiFinals[0].score1" type="number" min="0" class="score-input"
                    :disabled="!bracket.semiFinals[0].team1" />
                </div>
                <div class="team-row">
                  <div class="team-display">{{ bracket.semiFinals[0].team2 || 'Team' }}</div>
                  <input v-model.number="bracket.semiFinals[0].score2" type="number" min="0" class="score-input"
                    :disabled="!bracket.semiFinals[0].team2" />
                </div>
              </div>
            </div>
          </div>

          <!-- CENTER - FINAL -->
          <div class="final-column">
            <div class="trophy-container">
              <svg class="trophy-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                <path d="M6 9H4.5a2.5 2.5 0 0 1 0-5H6" stroke-width="2" stroke-linecap="round"
                  stroke-linejoin="round" />
                <path d="M18 9h1.5a2.5 2.5 0 0 0 0-5H18" stroke-width="2" stroke-linecap="round"
                  stroke-linejoin="round" />
                <path d="M4 22h16" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                <path d="M10 14.66V17c0 .55-.47.98-.97 1.21C7.85 18.75 7 20.24 7 22" stroke-width="2"
                  stroke-linecap="round" stroke-linejoin="round" />
                <path d="M14 14.66V17c0 .55.47.98.97 1.21C16.15 18.75 17 20.24 17 22" stroke-width="2"
                  stroke-linecap="round" stroke-linejoin="round" />
                <path d="M18 2H6v7a6 6 0 0 0 12 0V2Z" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
              </svg>
            </div>
            <h2 class="final-title">FINAL</h2>
            <div class="final-match-card">
              <div class="team-row">
                <div class="team-display-large">{{ bracket.final.team1 || 'Team' }}</div>
                <input v-model.number="bracket.final.score1" type="number" min="0" class="score-input-large"
                  :disabled="!bracket.final.team1" />
              </div>
              <div class="team-row">
                <div class="team-display-large">{{ bracket.final.team2 || 'Team' }}</div>
                <input v-model.number="bracket.final.score2" type="number" min="0" class="score-input-large"
                  :disabled="!bracket.final.team2" />
              </div>
            </div>
            <div v-if="champion" class="champion-display">
              🏆 {{ champion }}
            </div>
          </div>

          <!-- RIGHT SIDE - Semi Final (Bottom match) -->
          <div class="round-column">
            <h2 class="round-title">Semi-final</h2>
            <div class="matches-container">
              <div class="match-card mt-32">
                <div class="team-row">
                  <div class="team-display">{{ bracket.semiFinals[1].team1 || 'Team' }}</div>
                  <input v-model.number="bracket.semiFinals[1].score1" type="number" min="0" class="score-input"
                    :disabled="!bracket.semiFinals[1].team1" />
                </div>
                <div class="team-row">
                  <div class="team-display">{{ bracket.semiFinals[1].team2 || 'Team' }}</div>
                  <input v-model.number="bracket.semiFinals[1].score2" type="number" min="0" class="score-input"
                    :disabled="!bracket.semiFinals[1].team2" />
                </div>
              </div>
            </div>
          </div>

          <!-- RIGHT SIDE - Quarter Finals (Bottom 2 matches) -->
          <div class="round-column">
            <h2 class="round-title">Cuartos de final</h2>
            <div class="matches-container space-y-16">
              <div v-for="(match, index) in bracket.quarterFinals.slice(2, 4)" :key="`qf-${index + 2}`"
                class="match-card">
                <div class="team-row">
                  <div class="team-display">{{ match.team1 || 'Team' }}</div>
                  <input v-model.number="match.score1" type="number" min="0" class="score-input"
                    :disabled="!match.team1" />
                </div>
                <div class="team-row">
                  <div class="team-display">{{ match.team2 || 'Team' }}</div>
                  <input v-model.number="match.score2" type="number" min="0" class="score-input"
                    :disabled="!match.team2" />
                </div>
              </div>
            </div>
          </div>

          <!-- RIGHT SIDE - Round of 16 (Bottom 4 matches) -->
          <div class="round-column">
            <h2 class="round-title">Grupo 2</h2>
            <div class="matches-container space-y-8">
              <div v-for="(match, index) in bracket.roundOf16.slice(4, 8)" :key="`r16-${index + 4}`" class="match-card">
                <div class="team-row">
                  <input v-model="match.team1" type="text" placeholder="Team Name" class="team-input" />
                  <input v-model.number="match.score1" type="number" min="0" class="score-input" />
                </div>
                <div class="team-row">
                  <input v-model="match.team2" type="text" placeholder="Team Name" class="team-input" />
                  <input v-model.number="match.score2" type="number" min="0" class="score-input" />
                </div>
              </div>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bracket-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 3rem;
  align-items: center;
}

.round-column {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.final-column {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  padding: 1.5rem 0;
}

.round-title {
  font-size: 0.75rem;
  font-weight: 600;
  text-align: center;
  color: #60a5fa;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.final-title {
  font-size: 1rem;
  font-weight: 700;
  text-align: center;
  color: #fbbf24;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  margin-bottom: 0.75rem;
}

.matches-container {
  display: flex;
  flex-direction: column;
}

.match-card {
  background: linear-gradient(135deg, rgba(30, 58, 138, 0.6) 0%, rgba(30, 64, 175, 0.4) 100%);
  border: 1px solid rgba(59, 130, 246, 0.5);
  border-radius: 0.375rem;
  padding: 0.375rem;
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
}

.match-card:hover {
  border-color: rgba(96, 165, 246, 0.8);
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
  transform: translateY(-2px);
}

.final-match-card {
  background: linear-gradient(135deg, rgba(30, 58, 138, 0.8) 0%, rgba(30, 64, 175, 0.6) 100%);
  border: 2px solid rgba(251, 191, 36, 0.6);
  border-radius: 0.5rem;
  padding: 0.75rem;
  backdrop-filter: blur(10px);
  min-width: 240px;
  transition: all 0.3s ease;
}

.final-match-card:hover {
  border-color: rgba(251, 191, 36, 0.9);
  box-shadow: 0 8px 24px rgba(251, 191, 36, 0.4);
}

.team-row {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  padding: 0.2rem 0;
}

.team-input {
  flex: 1;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid rgba(71, 85, 105, 0.5);
  border-radius: 0.25rem;
  padding: 0.375rem 0.5rem;
  color: #e2e8f0;
  font-size: 0.75rem;
  transition: all 0.2s ease;
}

.team-input:focus {
  outline: none;
  border-color: #3b82f6;
  background: rgba(15, 23, 42, 0.8);
}

.team-input::placeholder {
  color: #64748b;
  font-size: 0.7rem;
}

.team-display {
  flex: 1;
  background: rgba(15, 23, 42, 0.4);
  border: 1px solid rgba(71, 85, 105, 0.3);
  border-radius: 0.25rem;
  padding: 0.375rem 0.5rem;
  color: #cbd5e1;
  font-size: 0.75rem;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.team-display-large {
  flex: 1;
  background: rgba(15, 23, 42, 0.5);
  border: 1px solid rgba(71, 85, 105, 0.4);
  border-radius: 0.375rem;
  padding: 0.5rem 0.75rem;
  color: #e2e8f0;
  font-size: 0.875rem;
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.score-input {
  width: 2.5rem;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid rgba(71, 85, 105, 0.5);
  border-radius: 0.25rem;
  padding: 0.375rem;
  color: #fbbf24;
  font-size: 0.75rem;
  font-weight: 700;
  text-align: center;
  transition: all 0.2s ease;
}

.score-input:focus {
  outline: none;
  border-color: #fbbf24;
  background: rgba(15, 23, 42, 0.8);
}

.score-input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.score-input-large {
  width: 3rem;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid rgba(71, 85, 105, 0.5);
  border-radius: 0.375rem;
  padding: 0.5rem;
  color: #fbbf24;
  font-size: 1rem;
  font-weight: 700;
  text-align: center;
  transition: all 0.2s ease;
}

.score-input-large:focus {
  outline: none;
  border-color: #fbbf24;
  background: rgba(15, 23, 42, 0.8);
}

.score-input-large:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.trophy-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 0.75rem;
}

.trophy-icon {
  width: 3rem;
  height: 3rem;
  color: #fbbf24;
  filter: drop-shadow(0 0 10px rgba(251, 191, 36, 0.5));
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {

  0%,
  100% {
    transform: scale(1);
  }

  50% {
    transform: scale(1.1);
  }
}

.champion-display {
  margin-top: 1rem;
  padding: 0.75rem 1.5rem;
  background: linear-gradient(135deg, rgba(251, 191, 36, 0.2) 0%, rgba(245, 158, 11, 0.2) 100%);
  border: 2px solid #fbbf24;
  border-radius: 0.5rem;
  color: #fbbf24;
  font-size: 1.25rem;
  font-weight: 700;
  text-align: center;
  animation: glow 2s ease-in-out infinite;
}

@keyframes glow {

  0%,
  100% {
    box-shadow: 0 0 20px rgba(251, 191, 36, 0.4);
  }

  50% {
    box-shadow: 0 0 30px rgba(251, 191, 36, 0.6);
  }
}

/* Remove number input arrows */
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
}
</style>
