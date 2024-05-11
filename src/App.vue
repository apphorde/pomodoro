<template>
  <main class="flex flex-col items-center justify-center h-screen bg-gray-900 text-white">
    <div class="max-w-md w-full space-y-8">
      <div class="text-center">
        <h1 class="text-4xl font-bold">Pomodoro Timer</h1>
        <p class="text-gray-400">Focus on your tasks with this simple timer.</p>
      </div>
      <div class="bg-gray-800 rounded-xl p-8 space-y-8">
        <div class="flex items-center justify-center">
          <div class="relative w-64 h-64">
            <div class="absolute inset-0 flex items-center justify-center">
              <span class="text-5xl font-bold">{{ padLeft(minutesLeft) }}:{{ padLeft(secondsLeft) }}</span>
            </div>
            <svg class="w-full h-full transform -rotate-90" viewBox="0 0 100 100">
              <circle cx="50" cy="50" r="40" fill="transparent" stroke="#4B5563" stroke-width="8"></circle>
              <circle
                cx="50"
                cy="50"
                r="40"
                fill="transparent"
                stroke="#6B7280"
                stroke-width="8"
                :stroke-dasharray="angle"
                stroke-dashoffset="1"
              ></circle>
            </svg>
          </div>
        </div>
        <div class="flex items-center justify-between">
          <div class="flex items-center justify-center space-x-4 w-full">
            <button
              @click="onStart()"
              class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 px-4 py-2"
            >
              Start
            </button>
            <button
              @click="onStop()"
              class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 px-4 py-2"
            >
              Pause
            </button>
            <button
              @click="onReset()"
              class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 px-4 py-2"
            >
              Reset
            </button>
          </div>
          <div class="flex items-center space-x-4 hidden">
            <label
              class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
              for="break-interval"
            >
              Break Interval:
            </label>
            <input
              class="flex h-10 rounded-md border px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50 w-20 bg-gray-700 border-gray-600 text-white"
              id="break-interval"
              min="1"
              max="60"
              type="number"
              value="5"
            />
            <span>minutes</span>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
<script setup lang="ts">
import { computed, ref } from 'vue';

let timer = 0;
const initialTime = 25 * 60;
const timeLeft = ref(initialTime);
const minutesLeft = computed(() => Math.floor(timeLeft.value / 60));
const secondsLeft = computed(() => Math.floor(timeLeft.value % 60));
const angle = computed(() => Math.floor(timeLeft.value % 60));

const padLeft = (n: number) => String(n).padStart(2, '0');

function onStart() {
  timer = setInterval(onTick, 1000);
}

function onStop() {
  clearInterval(timer);
}

function onReset() {
  timeLeft.value = initialTime;
}

function onTick() {
  timeLeft.value -= 1;

  if (timeLeft.value <= 0) {
    onStop();
  }
}
</script>
