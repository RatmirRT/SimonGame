<template>
  <div class="container">
    <modal-dialog v-if="showModal">
      <game-result :score="currentScore" :bestScore="bestScore"></game-result>
      <difficulty-level v-model:difficultyLevel="currentDifficult" :disableSelect="isStarted">
      </difficulty-level>
      <button class="playButton" @click="startGame">Играть</button>
    </modal-dialog>
    <div
      class="play_zone"
      :class="{
        large: currentDifficult === 3
      }"
    >
      <button
        class="game_btn"
        v-for="(button, key) in buttons"
        :class="{
          active: button.isActive
        }"
        :key="key"
        @click="inputPattern(key)"
        :disabled="!playerTurn && isStarted"
      ></button>
    </div>
    <div class="score">
      <h2>Счет: {{ currentScore }}</h2>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, watch, computed } from 'vue'
import DifficultyLevel from '@/components/DifficultyLevelSelect.vue'
import ModalDialog from '@/components/ModalDialog.vue'
import GameResult from '@/components/GameResult.vue'
import importedSounds from '@/assets/index'

export default defineComponent({
  components: { DifficultyLevel, ModalDialog, GameResult },
  setup() {
    const showModal = ref(true)
    const isStarted = ref(false) // обозначение старта игры
    const playerTurn = ref(false) // обозначение очереди игрока
    let playerPattern: number[] = [] // история паттернов нажатий игрока
    const currentDifficult = ref(1)
    const currentScore = ref(0)
    let currentPattern: number[] = [] // необходимый паттерн нажатия
    const bestScore = ref(0)
    const round = ref(1) // текущий раунд
    let buttons = ref<{ isActive: boolean }[]>([]) // кнопки на игровом поле

    // подгрузка фудио файлов
    const sounds = Array.from({ length: 9 }, (value, i) => {
      return new Audio(importedSounds[i])
    })

    watch(currentDifficult, () => {
      generateButtons()
    })

    // отслеживаем сосотояние игры, если false то обнуляем значения
    watch(isStarted, (newVal) => {
      if (!newVal) {
        checkScore()
        playerPattern = []

        round.value = 1

        playerTurn.value = false
        showModal.value = true
      }
    })

    // скорость воспроизведения игры относительно сложности
    const speed = computed(() => {
      const currentSpeed =
        currentDifficult.value === 1 ? 1200 : currentDifficult.value === 2 ? 800 : 400
      return currentSpeed
    })

    // если локально хранятся данные о Счете то подгружаем их
    const checkLocalScore = () => {
      const localScore = Number(localStorage.getItem('bestScore'))
      bestScore.value = localScore ? localScore : 0
    }

    // если локальные значения счета меньше того что храниться в текущей сессии то сохраняем
    const checkScore = () => {
      if (currentScore.value > bestScore.value) {
        bestScore.value = currentScore.value
        localStorage.setItem('bestScore', bestScore.value.toString())
      }
    }

    // кол-во кнопок относительно сложности
    const generateButtons = () => {
      const buttonsCount = currentDifficult.value === 1 ? 4 : currentDifficult.value === 2 ? 6 : 9
      const newButtons = Array.from({ length: buttonsCount }, () => ({
        isActive: false
      }))
      buttons.value = newButtons
    }

    // паттерн на текущий раунд
    const generatePattern = () => {
      let patternArray = []
      for (let i = 0; i < round.value; i++) {
        patternArray.push(Math.floor(Math.random() * buttons.value.length))
      }
      return patternArray
    }

    // пауза между переключениями кнопок
    const wait = async (ms: number) => {
      return await new Promise<void>((resolve) => {
        setTimeout(() => {
          resolve()
        }, ms)
      })
    }

    // запуск паттерна игры
    const startRound = async () => {
      currentPattern = generatePattern()
      await wait(1000)
      for (const element of currentPattern) {
        buttons.value[element].isActive = true // подсветка кнопки
        sounds[element].play()
        await wait(speed.value)
        buttons.value[element].isActive = false
        await wait(200)
      }
      playerTurn.value = true
    }

    // запоминаем и сравниеваем все то что вводит пользователь
    const inputPattern = (value: number) => {
      if (!playerTurn.value || !isStarted.value) return // выключаем кнопку на время проигрыша паттерна
      sounds[value].play()
      playerPattern.push(value)
      playerPattern.forEach((item, key) => {
        if (item !== currentPattern[key]) {
          // если ошибка в паттерне то заканчиваем игру
          isStarted.value = false
          return
        }
        if (key === round.value - 1) {
          // если раунд проиден то идет следующий
          nextRound()
        }
      })
    }

    const nextRound = () => {
      round.value++
      currentScore.value++
      playerTurn.value = false
      playerPattern = []
      startRound()
    }

    const startGame = () => {
      round.value = 1
      currentScore.value = 0

      showModal.value = false
      playerTurn.value = false
      isStarted.value = true
      playerPattern = []
      startRound()
    }

    onMounted(() => {
      checkLocalScore()
      generateButtons()
    })

    return {
      showModal,
      playerTurn,
      isStarted,
      bestScore,
      currentScore,
      buttons,
      currentDifficult,
      startGame,
      inputPattern
    }
  }
})
</script>

<style>
* {
  margin: 0;
  padding: 0;
  font-family: Arial, Helvetica, sans-serif;
}

.container {
  max-width: 800px;
  min-height: 100vh;
  margin: 0 auto;
  padding: 20px 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.playButton {
  margin-left: auto;
  display: block;
  background-color: rgb(190, 5, 5);
  color: #fff;
  border: none;
  padding: 10px;
  font-size: 18px;
  font-weight: 700;
  border-radius: 10px;
  cursor: pointer;
}

.play_zone {
  max-width: 400px;
  margin: 0 auto;
  display: flex;
  flex-wrap: wrap;
  margin-bottom: 30px;
  width: 100%;
}

.play_zone .game_btn {
  width: 50%;
  padding: 25% 0;
  background-color: gray;
  opacity: 0.4;
  cursor: pointer;
  border: none;
}

.play_zone.large .game_btn {
  width: calc(100% / 3);
  padding: calc(100% / 6) 0;
}

.play_zone .game_btn:active:not([disabled]),
.play_zone .game_btn.active {
  opacity: 1;
}

.play_zone .game_btn:nth-child(1) {
  background-color: rgb(255, 0, 0);
}

.play_zone .game_btn:nth-child(2) {
  background-color: rgb(255, 217, 0);
}

.play_zone .game_btn:nth-child(3) {
  background-color: rgb(51, 255, 0);
}

.play_zone .game_btn:nth-child(4) {
  background-color: rgb(0, 255, 234);
}

.play_zone .game_btn:nth-child(5) {
  background-color: rgb(0, 26, 255);
}

.play_zone .game_btn:nth-child(6) {
  background-color: rgb(255, 0, 255);
}

.play_zone .game_btn:nth-child(7) {
  background-color: rgb(179, 255, 0);
}

.play_zone .game_btn:nth-child(8) {
  background-color: rgb(255, 153, 0);
}

.play_zone .game_btn:nth-child(8) {
  background-color: rgb(89, 0, 255);
}

.score {
  position: relative;
  text-transform: uppercase;
}

.score h2 {
  text-align: center;
  font-size: 22px;
  color: #3004a8;
  margin-bottom: 15px;
}

@media screen and (max-width: 900px) {
  .container {
    width: calc(100% - 40px);
    padding: 20px;
  }
}
</style>
