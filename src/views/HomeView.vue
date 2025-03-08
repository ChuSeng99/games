<script setup>
import { ref, onMounted, computed } from 'vue';

const cards = ref([]);

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

const flash = ref('');

const pause = (milliseconds = 500) => {
  return new Promise(resolve => setTimeout(resolve, milliseconds));
}

const showFlashMessage = async (message, duration = 500) => {
  flash.value = message;
  await pause(duration);
  flash.value = '';
}

const attempts = ref(0);

const incrementAttempts = () => {
  attempts.value++;
}

const currentAttempt = computed(() => {
  return attempts.value;
});

// update scoreboard after each round of game 
// prompt player for next round
// update menu bar round display 

const rounds = ref([]);

onMounted(() => {
  const storedRounds = localStorage.getItem('scoreboard');
  if (storedRounds) {
    rounds.value = JSON.parse(storedRounds);
  }
});

const updateScoreboard = (currentRound, points) => {
  const round = { round: currentRound, points };
  rounds.value.push(round);

  localStorage.setItem('scoreboard', JSON.stringify(rounds.value));
}

const currentRound = computed(() => {
  return rounds.value.length + 1;
});

const flipCard = async (card) => {
  card.flipped = !card.flipped;

  if (hasMatch.value) {
    await showFlashMessage('You found a match.');

    flippedCards.value.forEach(card => card.cleared = true);

    if (! remainingCards.value.length) {
      updateScoreboard(currentRound, points);
      await showFlashMessage('You won!');
    }
  }

  incrementAttempts();

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
  return clearedCards.value.length - currentAttempt.value;
});

const restartGame = () => {
  window.location.reload();
}
</script>

<template>
  <main class="min-h-screen px-10">

    <!-- Display Bar -->
    <div class="p-10 mb-10 font-bold flex justify-center items-baseline gap-8">
      <div>
        <span
          class="text-3xl text-blue-700"
          v-text="points"
        ></span>
        <span class="text-xs">pts</span>
      </div>

      <div>
        <span>Attempt: </span>
        <span
          class="text-3xl text-blue-700"
          v-text="currentAttempt"
        ></span>
      </div>

      <div>
        <span>Round: </span>
        <span
          class="text-3xl text-blue-700"
          v-text="currentRound"
        ></span>
      </div>

      <button
        @click="restartGame"
        class="bg-blue-700 py-2 px-4 rounded-md text-white"
      >Restart</button>
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

    <!-- Scoreboard -->
    <div class="mt-10">
      <h2 class="mb-2">Scoreboard</h2>
      <ul
        v-if="rounds.length > 0"
        class="flex flex-col w-1/4"
      >
        <li
          v-for="(round, index) in rounds"
          :key="index"
          class="border border-gray-300 px-2 py-3"
        >
          Round {{ parseInt(round.round) - 1 }}: {{ round.points }} points
        </li>
      </ul>
    </div>

    <!-- flash message -->
    <div
      v-if="flash"
      class="fixed bottom-0 right-0 bg-green-500 text-white p-2 mb-4 mr-4 rounded"
    >{{ flash }}</div>
  </main>
</template>