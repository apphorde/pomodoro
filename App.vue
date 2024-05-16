<template>
  <main class="flex flex-col items-center justify-center h-screen bg-primary overflow-auto p-8">
    <div class="max-w-md w-full space-y-8">
      <div class="text-center">
        <h1 class="text-4xl font-bold mb-2">Pomodoro Timer</h1>
        <p class="">Focus on your tasks with this simple timer.</p>
      </div>
      <div class="bg-focus rounded-xl p-8 space-y-8 mx-8 text-whitse">
        <div class="flex items-center justify-center">
          <div class="relative w-64 h-64">
            <div class="absolute inset-0 flex items-center justify-center space-x-1 z-10">
              <span v-if="running" class="text-5xl font-bold"
                >{{ padLeft(minutesLeft) }}:{{ padLeft(secondsLeft) }}</span
              >
              <template v-else>
                <input
                  class="text-5xl w-16 rounded-lg font-bold bg-transparent text-right text-white"
                  type=""
                  min="1"
                  max="99"
                  v-model="requestedTime"
                />
                <span class="text-white text-3xl">min</span>
              </template>
            </div>
            <svg class="w-full h-full transform -rotate-90" viewBox="0 0 100 100">
              <circle
                cx="50"
                cy="50"
                r="40"
                fill="transparent"
                :stroke="running ? '#d13834' : '#ffffff99'"
                stroke-width="8"
              ></circle>
            </svg>
          </div>
        </div>
        <div class="flex items-center justify-stretch space-x-4 w-full">
          <button
            @click="onStart()"
            :class="running ? 'bg-red' : 'bg-green-700'"
            class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 h-10 px-4 py-2 text-white"
          >
            {{ running ? 'Stop' : 'Start' }}
          </button>
          <button
            v-if="running"
            @click="onPause()"
            class="inline-flex w-full items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 h-10 px-4 py-2 text-white"
          >
          {{ paused ? 'Continue' : 'Pause' }}
          </button>
          <button
            @click="onReset()"
            class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 h-10 px-4 py-2 text-white"
          >
            Reset
          </button>
        </div>
      </div>
    </div>
  </main>
</template>
<style>
.bg-primary {
  background-color: #629438;
}
.bg-red {
  background-color: #d13934;
}
.bg-focus {
  background-color: rgba(255, 255, 255, 0.1);
}
</style>
<script setup lang="ts">
import { computed, ref } from 'vue';

let timer = 0;
const running = ref(false);
const paused = ref(false);
const initialTime = 25;
const requestedTime = ref(String(initialTime));
const timeLeft = ref(initialTime * 60);
const minutesLeft = computed(() => Math.floor(timeLeft.value / 60));
const secondsLeft = computed(() => Math.floor(timeLeft.value % 60));

const padLeft = (n: number) => String(n).padStart(2, '0');

function onStop() {
  clearInterval(timer);
  running.value = false;
}

function onPause() {
  paused.value = !paused.value;
}

function onStart() {
  if (running.value) {
    onStop();
    return;
  }
  timeLeft.value = (parseInt(requestedTime.value) || initialTime) * 60;
  timer = setInterval(onTick, 1000);
  running.value = true;

  if (Notification.permission !== 'granted') {
    Notification.requestPermission();
  }
}

function onReset() {
  timeLeft.value = initialTime * 60;
}

function onTick() {
  if (paused.value) return;

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
