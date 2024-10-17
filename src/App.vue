<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import { emojis } from '~/static/emojis';
import CountDown from '~/components/CountDown.vue';

// Mostra a opção de reiniciar a partida (false / true)
const showRestart: boolean = false
// Adiciona uma quantidade de segundos ao acertar o emoji repetido
const addSecs: number = 3
// Remove uma quantidade de segundos ao acertar o emoji repetido
const removeSecs: number = 1
// Quantidade de emojis iniciais no nivel 1 (52 no maximo)
const amountInitialEmojis: number = 16
// Quantidade de segundos ao iniciar uma partida
const initialTimer: number = 10
// Nome do jogo
const gameName: string = "Emoji Match"
// Mensagem que aparece quando o tempo acaba
const endGameMessage: string = "Game Over"
// Quantidade de emojis adicionados a cada X niveis
const addEmojisEachLevel: number = 2
// Quantidade de emojis adicionados a cada X niveis
const addAmountEmojis: number = 4
// Tamanho maximo que ocupam na tela
const gameWidth: number = 400

// Daqui em diante é aonde a magia acontece
const gameOver = localStorage.getItem('emoji-game-over') === 'true' && !showRestart
const totalEmoji = ref<number>(amountInitialEmojis)
const timer = ref<number>(initialTimer)
const level = ref<number>(0)
const isPlaying = ref<boolean>(false)
const isGameOver = ref<boolean>(gameOver)

const allEmojis = ref<Array<string>>([])
const correctEmoji = ref<string>()
const selected = ref<string>()
const paused = ref<boolean>(true)
const plusAnimation = ref<boolean>(false)
const minusAnimation = ref<boolean>(false)
const score = ref<number>(0)

const width = computed(() => {
  return { width: `${gameWidth}px` }
})

const interval = ref()

function startTimer() {
  interval.value = setInterval(() => {
    if (paused.value) {
      return
    }

    if (timer.value > 0) {
      timer.value--
    } else {
      clearInterval(interval.value)

      paused.value = true
      isGameOver.value = true
    }
  }, 1000)
}

function addSeconds() {
  timer.value += addSecs

  plusAnimation.value = true

  setTimeout(() => {
    plusAnimation.value = false
  }, 1000)
}

function removeSeconds() {
  timer.value -= removeSecs

  minusAnimation.value = true

  setTimeout(() => {
    minusAnimation.value = false
  }, 1500)
}

function restartGame() {
  isGameOver.value = false
  isPlaying.value = false
}

function startGame() {
  score.value = 0
  timer.value = initialTimer
  isPlaying.value = true
  level.value = 1
  paused.value = false
  selected.value = ''
  totalEmoji.value = amountInitialEmojis
  allEmojis.value.splice(0)

  startTimer()
  renderEmojis()
}

function renderEmojis() {
  const emojisInGame = shuffle(emojis).slice(0, totalEmoji.value - 1)
  const duplicated = getRandomEmoji(emojisInGame)

  emojisInGame.push(duplicated)

  allEmojis.value.push(...shuffle(emojisInGame))
}

function nextLevel() {
  if (!(level.value % addEmojisEachLevel)) {
    totalEmoji.value += addAmountEmojis
  }

  level.value++
  selected.value = ''
  paused.value = false
  allEmojis.value.splice(0)

  setTimeout(() => {
    renderEmojis()

    paused.value = false
  }, 500)
}

function selectEmoji(emoji: string) {
  if (paused.value) {
    return
  }

  if (selected.value === emoji) {
    return
  }

  selected.value = emoji

  if (selected.value === correctEmoji.value) {
    paused.value = true
    score.value += totalEmoji.value
    addSeconds()

    setTimeout(() => {
      nextLevel()
    }, 1000)
  } else {
    removeSeconds()
  }
}

function getRandomEmoji(emojis: Array<string>) {
  const randomIndex = Math.floor(Math.random() * emojis.length)
  const emojiChosen = emojis[randomIndex]

  correctEmoji.value = emojiChosen

  return emojiChosen
}

function shuffle(array: Array<string>) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));

    [array[i], array[j]] = [array[j], array[i]];
  }

  return array;
}

watch(isGameOver, (val) => {
  localStorage.setItem('emoji-game-over', val.toString())
})
</script>

<template>
  <header class="flex items-center justify-evenly m-4 md:h-auto h-10">
    <template v-if="isPlaying">
      <div class="text-2xl h-8 text-slate-300 font-bold mb-2">
        <template v-if="level > 0">
          <span>Level </span>
          <span>{{ level }}</span>
        </template>
      </div>
      <div class="relative transition-all duration-300 scale-50 md:scale-100" :class="{ 'opacity-0': isGameOver }">
        <CountDown :timer="timer" :max="initialTimer" />
        <div v-auto-animate class="font-bold absolute top-5 left-12 text-xl">
          <div v-if="plusAnimation" class="text-emerald-600 ml-4 w-4">
            {{ '+' + addSecs }}
          </div>
          <div v-if="minusAnimation" class="text-red-600 w-4 -ml-4">
            {{ '-' + removeSecs }}
          </div>
        </div>
      </div>
      <div class="text-slate-300 font-bold w-10">
        <div>Score</div>
        <div class="text-right">{{ score }}</div>
      </div>
    </template>
  </header>

  <main v-auto-animate class="overflow-auto select-none flex items-center justify-center flex-1">
    <template v-if="!isGameOver">
      <div v-if="!isPlaying" class="absolute">
        <img class="m-auto h-40" src="./assets/logo.png" alt="logo">
        <h1 class="text-4xl h-8 font-bold text-green-300 game-title my-2 text-center">
          {{ gameName }}
        </h1>
        <button class="ui-btn mx-auto mt-24" @click="startGame()">
          <span>
            Start Game
          </span>
        </button>
      </div>
      <div v-else class="h-full content-center" :style="width">
        <div class="flex flex-wrap justify-center">
          <div v-for="(emoji, index) in allEmojis" :key="index"
            class="h-24 w-24 flex items-center justify-center transition-opacity duration-500 ease-in-out"
            :class="{ 'opacity-0': paused }">
            <div
              class="pb-2 h-24 w-24 text-6xl flex items-center justify-center cursor-pointer border-4 border-transparent transition-all duration-300 rounded-full"
              :class="[{ '!border-emerald-500': selected === emoji }, { '!border-red-500': (selected === emoji) && selected !== correctEmoji }]"
              @click="selectEmoji(emoji)">
              {{ emoji }}
            </div>
          </div>
        </div>
      </div>
    </template>
    <div v-else class="-mt-8 select-none flex items-center justify-center flex-1 flex-col">
      <h2 class="game-title text-white text-2xl mb-20 mx-auto">
        {{ endGameMessage }}
      </h2>
      <button v-if="showRestart" class="ui-btn m-auto" @click="restartGame()">
        <span>
          Restart Game
        </span>
      </button>
    </div>
  </main>

  <footer>
    <div class="mx-2 p-4 text-center border-t border-slate-400 footer-copyright border-primary">
      <p class="text-white content">develop with 💡 by
        <a href="https://labi9.com/" target="_blank" class="text-emerald-400 font-bold">
          Labi9
        </a>
      </p>
    </div>
  </footer>
</template>

<style>
html,
body,
#app {
  overflow-y: hidden
}

#app {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #222222;
}

.game-title {
  font-family: "Press Start 2P";
  animation: neon4 1.5s ease-in-out infinite alternate;
}

/* From Uiverse.io by Galahhad */
.ui-btn {
  --btn-default-bg: rgb(41, 41, 41);
  --btn-padding: 15px 20px;
  --btn-hover-bg: rgb(51, 51, 51);
  --btn-transition: .3s;
  --btn-letter-spacing: .1rem;
  --btn-animation-duration: 1.2s;
  --btn-shadow-color: rgba(0, 0, 0, 0.137);
  --btn-shadow: 0 2px 10px 0 var(--btn-shadow-color);
  --hover-btn-color: #34d399;
  --default-btn-color: #fff;
  --font-size: 16px;
  --font-weight: 600;
  --font-family: Menlo, Roboto Mono, monospace;
}

.ui-btn {
  box-sizing: border-box;
  padding: var(--btn-padding);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--default-btn-color);
  font: var(--font-weight) var(--font-size) var(--font-family);
  background: var(--btn-default-bg);
  border: none;
  cursor: pointer;
  transition: var(--btn-transition);
  overflow: hidden;
  box-shadow: var(--btn-shadow);
}

.ui-btn span {
  letter-spacing: var(--btn-letter-spacing);
  transition: var(--btn-transition);
  box-sizing: border-box;
  position: relative;
  background: inherit;
}

.ui-btn span::before {
  box-sizing: border-box;
  position: absolute;
  content: "";
  background: inherit;
}

.ui-btn:hover,
.ui-btn:focus {
  background: var(--btn-hover-bg);
}

.ui-btn:hover span,
.ui-btn:focus span {
  color: var(--hover-btn-color);
}

.ui-btn:hover span::before,
.ui-btn:focus span::before {
  animation: chitchat linear both var(--btn-animation-duration);
}

@keyframes chitchat {
  0% {
    content: "#";
  }

  5% {
    content: ".";
  }

  10% {
    content: "^{";
  }

  15% {
    content: "-!";
  }

  20% {
    content: "#$_";
  }

  25% {
    content: "№:0";
  }

  30% {
    content: "#{+.";
  }

  35% {
    content: "@}-?";
  }

  40% {
    content: "?{4@%";
  }

  45% {
    content: "=.,^!";
  }

  50% {
    content: "?2@%";
  }

  55% {
    content: "\;1}]";
  }

  60% {
    content: "?{%:%";
    right: 0;
  }

  65% {
    content: "|{f[4";
    right: 0;
  }

  70% {
    content: "{4%0%";
    right: 0;
  }

  75% {
    content: "'1_0<";
    right: 0;
  }

  80% {
    content: "{0%";
    right: 0;
  }

  85% {
    content: "]>'";
    right: 0;
  }

  90% {
    content: "4";
    right: 0;
  }

  95% {
    content: "2";
    right: 0;
  }

  100% {
    content: "";
    right: 0;
  }
}
</style>
