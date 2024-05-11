<template>
  <main class="flex flex-col items-center justify-center h-screen bg-gray-900 text-white">
    <div class="max-w-md w-full space-y-8">
      <div class="text-center">
        <h1 class="text-4xl font-bold mb-2">Pomodoro Timer</h1>
        <p class="text-gray-400">Focus on your tasks with this simple timer.</p>
      </div>
      <div class="bg-gray-800 rounded-xl p-8 space-y-8 mx-8">
        <div class="flex items-center justify-center">
          <div class="relative w-64 h-64">
            <div class="absolute inset-0 flex items-center justify-center">
              <span class="text-5xl font-bold">{{ padLeft(minutesLeft) }}:{{ padLeft(secondsLeft) }}</span>
            </div>
            <svg class="w-full h-full transform -rotate-90" viewBox="0 0 100 100">
              <circle cx="50" cy="50" r="40" fill="transparent" stroke="#4B5563" stroke-width="8"></circle>
            </svg>
          </div>
        </div>
        <div class="flex items-center justify-center space-x-4 w-full">
          <button
            @click="onStart()"
            :class="running ? 'bg-red-600' : 'bg-green-700'"
            class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 px-4 py-2"
          >
            {{ running ? 'Stop' : 'Start' }}
          </button>
          <button
            @click="onReset()"
            class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 px-4 py-2"
          >
            Reset
          </button>
        </div>
      </div>
    </div>
  </main>
</template>
<script setup lang="ts">
import { computed, ref } from 'vue';

let timer = 0;
const running = ref(false);
const initialTime = 25 * 60;
const timeLeft = ref(initialTime);
const minutesLeft = computed(() => Math.floor(timeLeft.value / 60));
const secondsLeft = computed(() => Math.floor(timeLeft.value % 60));

const padLeft = (n: number) => String(n).padStart(2, '0');

function onStart() {
  if (running.value) {
    clearInterval(timer);
    running.value = false;
    return;
  }

  timer = setInterval(onTick, 1000);
  running.value = true;

  if (Notification.permission !== 'granted') {
    Notification.requestPermission();
  }
}

function onReset() {
  timeLeft.value = initialTime;
}

function onTick() {
  timeLeft.value -= 1;

  if (timeLeft.value <= 0) {
    onStop();
    notify();
  }
}

function timeIsUp() {
  new Notification('Pomodoro', { body: 'Time is up!', icon: '/assets/icon.svg' });
}

function notify() {
  if (Notification.permission === 'denied') {
    return;
  }

  Notification.requestPermission().then((permission) => {
    if (permission === 'granted') {
      timeIsUp();
    }
  });
}
</script>
