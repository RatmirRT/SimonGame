<template>
  <div class="difficulty_level">
    <h4>Уровень сложности</h4>
    <select v-model="levelNumber" @change="getDifficultyLevel" :disabled="disableSelect">
      <option :value="1">Легко</option>
      <option :value="2">Средний</option>
      <option :value="3">Сложный</option>
    </select>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'

export default defineComponent({
  name:'DifficultyLevel',
  props: {
    difficultyLevel: {
      type: Number,
      default: 1
    },
    disableSelect: {
      type: Boolean,
      default: true
    }
  },
  setup (props, { emit } ) {
    const levelNumber = ref(props.difficultyLevel)
    const getDifficultyLevel = () => {
      emit('update:difficultyLevel', levelNumber.value)
    }
    return {
      levelNumber,
      getDifficultyLevel
    }
  }
})
</script>

<style scoped>
.difficulty_level h4 {
  margin-bottom: 15px;
  font-size: 16px;
  color: #3004a8;
}

.difficulty_level select {
  background-color: transparent;
  padding: 7px;
  border: 1px solid red;
  border-radius: 5px;
  color: #3004a8;
  font-size: 16px;
}
</style>