<script setup>
import { ref, onMounted, computed } from 'vue';

const cards = ref([]);
const flash = ref('');

onMounted(() => {
  fetch('http://localhost:3000/cards')
    .then(response => response.json())
    .then(data => {
      cards.value = shuffleArray(data);
    })
    .catch(error => {
      console.error('Error fetching data:', error);
    });
});

const shuffleArray = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }

  return array;
}

const pause = (milliseconds = 500) => {
  return new Promise(resolve => setTimeout(resolve, milliseconds));
}

const showFlashMessage = async (message, duration = 500) => {
  flash.value = message;
  await pause(duration);
  flash.value = '';
}

const flipCard = async (card) => {
  card.flipped = !card.flipped;

  if (hasMatch.value) {
    await showFlashMessage('You found a match.');

    flippedCards.value.forEach(card => card.cleared = true);

    if (! remainingCards.value.length) {
      await showFlashMessage('You won!', 2000);
    }
  }

  await pause();

  flippedCards.value.forEach(card => card.flipped = false);
}

const flippedCards = computed(() => {
  return cards.value.filter(card => card.flipped);
});

const clearedCards = computed(() => {
  return cards.value.filter(card => card.cleared);
});

const remainingCards = computed(() => {
  return cards.value.filter(card => ! card.cleared);
});

const hasMatch = computed(() => {
  return flippedCards.value[0].color === flippedCards.value[1].color;
});

const points = computed(() => {
  return clearedCards.value.length;
});

const restartGame = () => {
  window.location.reload();
}
</script>

<template>
  <main class="min-h-screen px-10">

    <!-- Points & Restart -->
    <div class="p-10 mb-10 font-bold flex justify-center gap-8">
      <div>
        <span
          class="text-3xl text-blue-700"
          v-text="points"
        ></span>
        <span class="text-xs">pts</span>
      </div>

      <span>
        <button 
          @click="restartGame"
          class="bg-blue-700 py-2 px-4 rounded-md text-white"
        >Restart</button>
      </span>
    </div>

    <!-- Cards -->
    <div class="flex-1 grid grid-cols-4 gap-10">
      <div
        v-for="card in cards"
        :key="card.id"
      >
        <button
          :style="'background: ' + (card.cleared ? 'white' : (card.flipped ? card.color : '#999'))"
          :disabled="flippedCards.length >= 2"
          class="h-32 w-full"
          @click="flipCard(card)"
        ></button>
      </div>
    </div>

    <!-- flash message -->
    <div
      v-if="flash"
      class="fixed bottom-0 right-0 bg-green-500 text-white p-2 mb-4 mr-4 rounded"
    >{{ flash }}</div>
  </main>
</template>