<template>
  <div class="game">
    <div v-if="!hasStarted" class="start-screen">
      <div class="title-row">
        <h1 class="title">♡もふカフェ絵あわせ♡</h1>
        <button
          class="info-button"
          type="button"
          aria-label="ゲームの説明を表示"
          @click="showInfo = true"
        >
          ?
        </button>
      </div>

      <input
        v-model="password"
        class="password-input"
        type="text"
        placeholder="あいことば を にゅうりょく"
        @keydown.enter="startGame"
      />

      <button class="start-button" @click="startGame">はじめる♪</button>

      <p v-if="passwordError" class="password-error">
        {{ passwordError }}
      </p>
    </div>

    <template v-else>
      <div class="title-row">
        <h1 class="title">♡もふカフェ絵あわせ♡</h1>
        <button
          class="info-button"
          type="button"
          aria-label="ゲームの説明を表示"
          @click="showInfo = true"
        >
          ?
        </button>
      </div>

      <div class="status">
        <div>
          <div class="difficulty">
            むずかしさ: {{ difficultyLabel }}
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

          <div>けいけんち: {{ experience }}</div>
        </div>

        <button class="reset-button" @click="showResetConfirm = true">
          リセット
        </button>
      </div>

      <div
        class="board"
        :class="{ 'board-hard': difficulty === 'hard' }"
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
          <p>けいけんち +{{ experienceReward }}</p>
          <button @click="initializeGame">もう一回あそぶ</button>
        </div>
      </div>
    </template>

    <div v-if="showInfo" class="info-overlay" @click.self="showInfo = false">
      <div class="info-box">
        <button
          class="info-close"
          type="button"
          aria-label="とじる"
          @click="showInfo = false"
        >
          ×
        </button>

        <h2>あそびかた</h2>
        <p>
          おなじ絵(え)のカードを、2まいずつみつけるゲームです。
          ぜんぶそろえると、むずかしさ ごとのけいけんちがふえます。
          ブラウザやタブをとじても、けいけんちはのこります。          
        </p>

        <h2>ちゅうい</h2>
        <p>
          ブラウザの「webサイトデータ」をさくじょすると、けいけんちが0にリセットされます。かいはつしゃがコードをなおしたときもおなじです。<br />
        </p>

        <h2>がぞうについて</h2>
        <p>
          もふもふペットCafe さまのイラストをおかりしています。<br />:
          <a :href="imageCreditUrl" target="_blank" rel="noopener noreferrer">
            {{ imageCreditUrl }}
          </a>
        </p>
      </div>
    </div>

    <div v-if="showResetConfirm" class="confirm-overlay">
      <div class="confirm-box">
        <p>けいけんち が0になります！本当によろしいですか？</p>

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
const showInfo = ref(false)
const imageCreditUrl = "https://mofucafe.xii.jp/#purohu"

const hasStarted = ref(false)
const password = ref("")
const passwordError = ref("")
const correctPassword = "mochi"

const difficulty = ref("easy")
const difficultyLabel = computed(() => {
  const option = difficultyOptions.find((option) => option.value === difficulty.value)
  return option?.label || difficulty.value
})
const experience = ref(Number(localStorage.getItem("experience")) || 0)

const difficultyOptions = [
  { value: "easy", label: "かんたん" },
  { value: "normal", label: "ふつう" },
  { value: "hard", label: "むずい" }
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

@font-face {
  font-family: 'MisakiGothic';
  src: url('/misaki_gothic.ttf') format('truetype');
  font-weight: normal;
  font-style: normal;
}

.game {
  min-height: 100vh;
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #ead0d6;
  color: #4a4a4a;
  font-family: 'MisakiGothic', sans-serif;
}

.start-screen {
  width: 350px;
  display: flex;
  flex-direction: column;
  align-items: stretch;
}

.title-row {
  position: relative;
  width: 350px;
  margin: 0 0 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.title {
  margin: 0;
  font: inherit;
  font-size: 26px;
  font-weight: bold;
  color: #7a6d72;
  text-align: center;
}

.info-button {
  position: absolute;
  right: 0;
  width: 28px;
  height: 28px;
  padding: 0;
  border-radius: 50%;
  color: #7a6d72;
  background: #ead0d6;
  cursor: pointer;
}

.password-input {
  padding: 10px;
  font-size: 16px;
  font: inherit;
  background-color: white;
  border: 1px solid #4a4a4a;
  color: #4a4a4a;
}

button {
  font: inherit;
  font-size: 16px;
  color: white;
  background-color: #c887a0;
  border: 1px solid #4a4a4a;
  border-radius: 4px;
  -webkit-appearance: none;
  appearance: none;
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
  grid-template-columns: repeat(4, 80px);
  gap: 10px;
}

.board-hard {
  grid-template-columns: repeat(8, 80px);
}

@media (max-width: 760px) {
  .board-hard {
    grid-template-columns: repeat(4, 80px);
  }
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

.info-overlay {
  position: fixed;
  inset: 0;
  z-index: 20;
  background: rgb(0 0 0 / 40%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  box-sizing: border-box;
}

.info-box {
  position: relative;
  width: min(340px, 100%);
  padding: 24px 20px 20px;
  background: white;
  border: 1px solid #4a4a4a;
  text-align: left;
  line-height: 1.7;
}

.info-box h2 {
  margin: 0 0 8px;
  font: inherit;
  font-size: 18px;
  font-weight: bold;
  color: #7a6d72;
}

.info-box p {
  margin: 0 0 16px;
}

.info-box p:last-child {
  margin-bottom: 0;
}

.info-box a {
  color: #9c5270;
  word-break: break-all;
}

.info-close {
  position: absolute;
  top: 8px;
  right: 8px;
  width: 28px;
  height: 28px;
  padding: 0;
  color: #7a6d72;
  background: transparent;
  border: none;
  cursor: pointer;
}

@media (max-width: 760px) {
  .game {
    justify-content: flex-start;
    padding: 32px 0 40px;
    box-sizing: border-box;
  }
  .title-row,
  .start-screen,
  .status {
    width: min(350px, calc(100vw - 32px));
  }
}
</style>