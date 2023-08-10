<script setup lang="ts">
import GameNotification from './components/GameNotification.vue'
import GamePopup from './components/GamePopup.vue'
import GameWord from './components/GameWord.vue'
import GameWrongLetters from './components/GameWrongLetters.vue'
import GameFigure from './components/GameFigure.vue'
import GameHeader from './components/GameHeader.vue'
import { computed, ref, watch } from 'vue'
import axios from 'axios'
const word = ref('')

const getRandomWord = async () => {
  try {
    const { data } = await axios<{ FirstName: string }>(
      'https://api.randomdatatools.ru/?unescaped=false&params=FirstName'
    )
    word.value = data.FirstName.toLowerCase()
  } catch (error) {
    console.log(error)
  }
}
getRandomWord()
const letters = ref<string[]>([])
const correctLetters = computed(() => letters.value.filter((i) => word.value.includes(i)))
const wrongLetters = computed(() => letters.value.filter((i) => !word.value.includes(i)))
const isLose = computed(() => wrongLetters.value.length === 6)
const isWin = computed(() => [...word.value].every((x) => correctLetters.value.includes(x)))
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)

watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})
watch(correctLetters, () => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})
window.addEventListener('keydown', ({ key }) => {
  if (isLose.value || isWin.value) {
    return
  }

  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return
  }

  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toLowerCase())
  }
})
const restart = async () => {
  letters.value = []
  popup.value?.close()
  await getRandomWord()
}
</script>

<template>
  <GameHeader />
  <div class="game-container">
    <GameFigure :wrongLettersCount="wrongLetters.length" />
    <GameWrongLetters :wrongLetters="wrongLetters" />
    <GameWord :word="word" :correctLetters="correctLetters" />
  </div>

  <GamePopup ref="popup" :word="word" @restart="restart" />
  <GameNotification ref="notification" />

  <!-- Notification -->
</template>
