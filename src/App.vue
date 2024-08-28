<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import GameError from './components/GameError.vue'
import GameFigure from './components/GameFigure.vue'
import GameHeader from './components/GameHeader.vue'
import GameNotification from './components/GameNotification.vue'
import GamePopup from './components/GamePopup.vue'
import GameWord from './components/GameWord.vue'

const word = ref('марк'); // тут ми завжди передаємо імя Марк і це не дуже коректно щоб імя завжди було одинакове тому колись зробимо так щоб при перезагрузці гри імя було різне 
const letters = ref<string[]>([])
const correctLetters = computed(() => letters.value.filter(x => word.value.includes(x)))
const wrongLetters = computed(() => letters.value.filter(x => !word.value.includes(x)))
const isLose = computed(() => wrongLetters.value.length >= 6)
const isWin = computed(() => [...word.value].every(x => correctLetters.value.includes(x)))
const notification = ref<InstanceType<typeof GameNotification> | null>(null)

const popup = ref<InstanceType<typeof GamePopup> | null>(null)

watch (wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})

watch (correctLetters, () => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({key}) => {
  if (isLose.value || isWin.value) {
    return
  }
  if(letters.value.includes(key)){
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return
  }

   if (/[а-яА-Я]/.test(key)) {
    letters.value.push(key.toLowerCase())
   }
})

const restart = () => {
  letters.value = []
  popup.value?.close()
}
</script>

<template>

  <div id="app">
    <GameHeader />
    <div class="game-container">
      <GameFigure :wrong-letters-count='wrongLetters.length' />
      <GameError :wrong-letters="wrongLetters" />
      <GameWord :word="word" :correct-letters="correctLetters" />
    </div>
    <GamePopup ref='popup' :word="word" @restart="restart" />
    <GameNotification ref='notification' />
  </div>
</template>