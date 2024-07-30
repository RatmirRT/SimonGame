<template>
  <div class="game_result">
    <h3>Результаты</h3>
    <p>Ваш счет: <b>{{ score }}</b></p>
    <p>Лучший счет: <b>{{ bestScore }}</b></p>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'GameResult',
  props: {
    score: {
      type: Number,
      default: 0
    },
    bestScore: {
      type: Number,
      default: 0
    }
  }
})
</script>

<style scoped>
.game_result h3 {
  text-align: center;
  margin-bottom: 15px;
  color: #c40909;
  font-size: 20px;
  text-transform: uppercase;
}

.game_result p {
  margin-bottom: 10px;
  text-transform: uppercase;
  font-size: 15px;
}

.game_result p:last-child {
  margin-bottom: 15px;
}
</style>