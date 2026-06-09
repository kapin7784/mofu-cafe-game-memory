<template>
  <div class="game">
    <div v-if="!hasStarted" class="start-screen">
      <h1 class="title">♪ もふカフェ絵あわせ ♪</h1>

      <input
        v-model="password"
        class="password-input"
        type="text"
        placeholder="合言葉を入力"
        @keydown.enter="startGame"
      />

      <button class="start-button" @click="startGame">ゲーム開始</button>

      <p v-if="passwordError" class="password-error">
        {{ passwordError }}
      </p>
    </div>

    <template v-else>
      <h1 class="title">♪ もふカフェ絵あわせ ♪</h1>

      <div class="status">
        <div>
          <div class="difficulty">
            難易度: {{ difficulty }}
            <button class="difficulty-toggle" @click="toggleDifficultyMenu">
              🔼
            </button>

            <div v-if="showDifficultyMenu" class="difficulty-menu">
              <button
                v-for="option in difficultyOptions"
                :key="option.value"
                :class="{ active: difficulty === option.value }"
                @click="changeDifficulty(option.value)"
              >
                {{ option.label }}
              </button>
            </div>
          </div>

          <div>経験値: {{ experience }}</div>
        </div>

        <button class="reset-button" @click="showResetConfirm = true">
          経験値リセット
        </button>
      </div>

      <div
        class="board"
        :style="{ gridTemplateColumns: `repeat(${boardColumns}, 80px)` }"
      >
        <div
          v-for="card in cards"
          :key="card.id"
          class="card"
          @click="flipCard(card)"
        >
          <img
            v-if="card.flipped || card.matched"
            :src="card.image"
            :alt="card.name"
            class="card-image"
            :class="{
              'matched-effect': card.animating,
              'clear-effect': isCleared
            }"
          />
          <span v-else>?</span>
        </div>

        <div v-if="isCleared" class="clear-message">
          <p>⭐️ ゲームクリア！⭐️</p>
          <p>経験値 +{{ experienceReward }}</p>
          <button @click="initializeGame">もう一度遊ぶ</button>
        </div>
      </div>
    </template>

    <div v-if="showResetConfirm" class="confirm-overlay">
      <div class="confirm-box">
        <p>経験値が0になります！本当によろしいですか？</p>

        <div class="confirm-buttons">
          <button @click="resetExperience">はい</button>
          <button @click="showResetConfirm = false">いいえ</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from "vue"

const cards = ref([])
const selectedCards = ref([])
const isChecking = ref(false)
const hasReceivedReward = ref(false)
const showResetConfirm = ref(false)
const showDifficultyMenu = ref(false)

const hasStarted = ref(false)
const password = ref("")
const passwordError = ref("")
const correctPassword = "mochi"

const difficulty = ref("easy")
const experience = ref(Number(localStorage.getItem("experience")) || 0)

const difficultyOptions = [
  { value: "easy", label: "easy" },
  { value: "normal", label: "normal" },
  { value: "hard", label: "hard" }
]

const allCharacters = [
  { name: "kurumi", image: `${import.meta.env.BASE_URL}kurumi.png` },
  { name: "mitarashi", image: `${import.meta.env.BASE_URL}mitarashi.png` },
  { name: "syrup", image: `${import.meta.env.BASE_URL}syrup.png` },
  { name: "petit", image: `${import.meta.env.BASE_URL}petit.png` },

  { name: "yomogi", image: `${import.meta.env.BASE_URL}yomogi.png` },
  { name: "azuki", image: `${import.meta.env.BASE_URL}azuki.png` },
  { name: "tiramisu", image: `${import.meta.env.BASE_URL}tiramisu.png` },
  { name: "souffle", image: `${import.meta.env.BASE_URL}souffle.png` },

  { name: "leche", image: `${import.meta.env.BASE_URL}leche.png` },
  { name: "eclair", image: `${import.meta.env.BASE_URL}eclair.png` },
  { name: "earlgrey", image: `${import.meta.env.BASE_URL}earlgrey.png` },
  { name: "momiji", image: `${import.meta.env.BASE_URL}momiji.png` },
  { name: "daifuku", image: `${import.meta.env.BASE_URL}daifuku.png` },
  { name: "botamochi", image: `${import.meta.env.BASE_URL}botamochi.png` },
  { name: "hotaru", image: `${import.meta.env.BASE_URL}hotaru.png` },
  { name: "kikyo", image: `${import.meta.env.BASE_URL}kikyo.png` }
]

const characterCount = computed(() => {
  if (difficulty.value === "easy") return 4
  if (difficulty.value === "normal") return 8
  if (difficulty.value === "hard") return 16
  return 4
})

const boardColumns = computed(() => {
  if (difficulty.value === "hard") return 8
  return 4
})

const experienceReward = computed(() => {
  if (difficulty.value === "easy") return 50
  if (difficulty.value === "normal") return 100
  if (difficulty.value === "hard") return 200
  return 0
})

const isCleared = computed(() => {
  return cards.value.length > 0 && cards.value.every((card) => card.matched)
})

function startGame() {
  if (password.value !== correctPassword) {
    passwordError.value = "合言葉が違うようです"
    return
  }

  passwordError.value = ""
  hasStarted.value = true
}

function toggleDifficultyMenu() {
  showDifficultyMenu.value = !showDifficultyMenu.value
}

function changeDifficulty(nextDifficulty) {
  difficulty.value = nextDifficulty
  showDifficultyMenu.value = false
  initializeGame()
}

function initializeGame() {
  const characters = allCharacters.slice(0, characterCount.value)

  const deck = [...characters, ...characters]
    .sort(() => Math.random() - 0.5)
    .map((character, index) => ({
      id: index,
      name: character.name,
      image: character.image,
      flipped: false,
      matched: false,
      animating: false
    }))

  cards.value = deck
  selectedCards.value = []
  isChecking.value = false
  hasReceivedReward.value = false
}

initializeGame()

function saveExperience() {
  localStorage.setItem("experience", experience.value)
}

function addExperience() {
  if (hasReceivedReward.value) return

  experience.value += experienceReward.value
  saveExperience()
  hasReceivedReward.value = true
}

function resetExperience() {
  experience.value = 0
  saveExperience()
  showResetConfirm.value = false
}

function flipCard(card) {
  if (isChecking.value) return
  if (card.flipped || card.matched) return
  if (selectedCards.value.length >= 2) return

  card.flipped = true
  selectedCards.value.push(card)

  if (selectedCards.value.length !== 2) return

  isChecking.value = true

  const [firstCard, secondCard] = selectedCards.value

  if (firstCard.name === secondCard.name) {
    firstCard.matched = true
    secondCard.matched = true
    firstCard.animating = true
    secondCard.animating = true

    selectedCards.value = []
    isChecking.value = false

    setTimeout(() => {
      firstCard.animating = false
      secondCard.animating = false
    }, 2000)

    if (cards.value.every((card) => card.matched)) {
      addExperience()
    }
  } else {
    setTimeout(() => {
      firstCard.flipped = false
      secondCard.flipped = false
      selectedCards.value = []
      isChecking.value = false
    }, 800)
  }
}
</script>

<style scoped>
.game {
  min-height: 100vh;
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #ead0d6;
  color: #4a4a4a;
}

.start-screen {
  width: 350px;
  display: flex;
  flex-direction: column;
  align-items: stretch;
}

.title {
  margin: 0 0 40px;
  font-size: 26px;
  font-weight: 700;
  color: #7a6d72;
  text-align: center;
}

.password-input {
  padding: 10px;
  font-size: 16px;
  background-color: white;
  border: 1px solid #4a4a4a;
  color: #4a4a4a;
}

.start-button {
  padding: 10px 16px;
  margin-top: 12px;
  cursor: pointer;
}

.password-error {
  margin: 12px 0 0;
  color: #b00020;
  text-align: center;
}

.status {
  width: 350px;
  margin-bottom: 30px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  text-align: left;
}

.difficulty {
  position: relative;
}

.difficulty-toggle {
  padding: 0;
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: 16px;
}

.difficulty-menu {
  position: absolute;
  top: 26px;
  left: 64px;
  z-index: 10;
  display: flex;
  flex-direction: column;
  min-width: 90px;
  background: white;
  border: 1px solid #4a4a4a;
}

.difficulty-menu button {
  padding: 8px 12px;
  border: none;
  background: white;
  color: #4a4a4a;
  text-align: left;
  cursor: pointer;
}

.difficulty-menu button:hover,
.difficulty-menu button.active {
  background: #f3dfe4;
}

.reset-button {
  padding: 8px 12px;
  cursor: pointer;
}

.board {
  display: grid;
  gap: 10px;
}

.card {
  width: 80px;
  height: 80px;
  border: 1px solid #4a4a4a;
  background-color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.card-image {
  width: 70px;
  height: 70px;
  object-fit: contain;
}

.matched-effect {
  animation: matched-pop 1s ease-in-out 2;
}

.clear-effect {
  animation: clear-sway 1s ease-in-out infinite;
}

@keyframes matched-pop {
  0% {
    transform: scale(1);
  }

  50% {
    transform: scale(1.2);
  }

  100% {
    transform: scale(1);
  }
}

@keyframes clear-sway {
  0% {
    transform: rotate(-10deg);
  }

  50% {
    transform: rotate(10deg);
  }

  100% {
    transform: rotate(-10deg);
  }
}

.clear-message {
  grid-column: 1 / -1;
  text-align: center;
}

.clear-message button {
  padding: 8px 16px;
  margin-top: 10px;
  cursor: pointer;
}

.confirm-overlay {
  position: fixed;
  inset: 0;
  background: rgb(0 0 0 / 40%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.confirm-box {
  width: 300px;
  padding: 20px;
  background: white;
  border: 1px solid #ccc;
  text-align: center;
}

.confirm-buttons {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-top: 10px;
}

.confirm-buttons button {
  padding: 8px 16px;
  cursor: pointer;
}
</style>