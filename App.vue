<template>
  <main class="flex flex-col items-center justify-center h-screen bg-primary overflow-auto p-8">
    <div class="max-w-md w-full space-y-8">
      <div class="text-center text-white">
        <img src="/assets/icon.svg" class="w-40 mx-auto" />
        <h1 class="text-4xl font-bold mb-2">Pomodoro Timer</h1>
        <p class="">Focus on your tasks with this simple timer.</p>
      </div>
      <div class="flex space-y-6 items-center justify-center h-20">
        <div v-if="running" class="text-5xl font-bold text-white">
          {{ padLeft(minutesLeft) }}:{{ padLeft(secondsLeft) }}
        </div>
        <div class="flex items-center justify-center" v-else>
          <input
            class="text-5xl font-bold w-16 rounded-lg bg-transparent text-right text-white"
            v-model="requestedTime"
          />
          <span class="text-white text-5xl font-bold">:00</span>
        </div>
      </div>
      <div class="w-full rounded-full border border-white" :class="!running && 'opacity-0'">
        <span :style="'width: ' + progress + '%'" class="h-2 block bg-white opacity-50 rounded-full"></span>
      </div>
      <div class="flex items-center justify-stretch space-x-4">
        <button
          @click="onStart()"
          :class="running ? 'bg-red' : 'bg-green-700'"
          class="flex items-center justify-center whitespace-nowrap rounded-md font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 h-10 px-4 py-2 text-white"
        >
          {{ running ? 'Stop' : 'Start' }}
        </button>
        <button
          v-if="running"
          @click="onPause()"
          class="flex w-full items-center justify-center whitespace-nowrap rounded-md font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 h-10 px-4 py-2 text-white"
        >
          {{ paused ? 'Continue' : 'Pause' }}
        </button>
        <button
          @click="onReset()"
          class="flex items-center justify-center whitespace-nowrap rounded-md font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 h-10 px-4 py-2 text-white"
        >
          Reset
        </button>
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
let wakeLock: WakeLockSentinel;
const running = ref(false);
const paused = ref(false);
const initialTime = 25;
const requestedTime = ref(String(initialTime));
const timeLeft = ref(initialTime * 60);
const minutesLeft = computed(() => Math.floor(timeLeft.value / 60));
const secondsLeft = computed(() => Math.floor(timeLeft.value % 60));
const getRequestedTime = () => parseInt(requestedTime.value) || initialTime;
const padLeft = (n: number) => String(n).padStart(2, '0');
const progress = computed(() => (running.value ? 100 * (timeLeft.value / (Number(requestedTime.value) * 60)) : 0));

function onStop() {
  clearInterval(timer);
  running.value = false;
  paused.value = false;
  wakeLock?.release();
}

function onPause() {
  paused.value = !paused.value;
}

async function onStart() {
  if (running.value) {
    onStop();
    return;
  }

  timeLeft.value = getRequestedTime() * 60;
  timer = setInterval(onTick, 1000);
  running.value = true;

  if (Notification.permission !== 'granted') {
    Notification.requestPermission();
  }

  wakeLock = await navigator.wakeLock.request('screen');
}

function onReset() {
  timeLeft.value = getRequestedTime() * 60;
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
