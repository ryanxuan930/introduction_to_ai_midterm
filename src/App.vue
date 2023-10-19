<script setup lang="ts">
import { ref } from 'vue';
import type { Ref } from 'vue';

const rows: Ref<number> = ref(10);
const columns: Ref<number> = ref(10);
const maxGen: Ref<number> = ref(10);
const elapse: Ref<number> = ref(1);
const currentElapse: Ref<number> = ref(0);
const cellSize: Ref<number> = ref(50);
const cellCss: Ref<string> = ref(`width: 50px; height: 50px;`);
const isRunning: Ref<boolean> = ref(false);

const position: Ref<boolean[][][]> = ref([]);

function detect(row: number, column: number, time: number): number {
  let count: number = 0;
  for (let i = Math.max(0, row - 1); i <= Math.min(rows.value - 1, row + 1); i++) {
    for (let j = Math.max(0, column - 1); j <= Math.min(columns.value - 1, column + 1); j++) {
      if (i === row && j === column) {
          continue;
      }
      if (position.value[time][i][j]) {
          count++;
      }
    }
  }
  return count;
}
function customizeMapSize(): void {
  position.value = Array(maxGen.value).fill(0).map(() => Array(rows.value).fill(0).map(() => Array(columns.value).fill(false)));
}
function next(): void {
  for (let t = 0; t < maxGen.value - 1; t++) {
    for (let i = 0; i < rows.value; i++) {
      for (let j = 0; j < columns.value; j++) {
        let count = detect(i, j, t);
        if (position.value[t][i][j]) {
            position.value[t + 1][i][j] = count === 2 || count === 3;
        } else {
            position.value[t + 1][i][j] = count === 3;
        }
      }
    }
  }
}

function randomPosition() {
  for (let i = 0; i < rows.value; i++) {
    for (let j = 0; j < columns.value; j++) {
      position.value[0][i][j] = Math.random() >= 0.5;
    }
  }
}

async function start(): Promise<void> {
  if (rows.value === 0 || columns.value === 0 || maxGen.value === 0 || elapse.value === 0) {
    alert('Please fill in all the fields');
    return;
  }
  if (position.value.length === 0) {
    alert('Please create the map first');
    return;
  }
  isRunning.value = true;
  for (let t = 0; t < maxGen.value; t++) {
    next();
    currentElapse.value = t;
    await new Promise(resolve => setTimeout(resolve, elapse.value*1000));
  }
  isRunning.value = false;
}
function reset(): void {
  currentElapse.value = 0;
  position.value = [];
  isRunning.value = false;
}
</script>

<template>
  <div class="bg-gray-50 p-10 min-h-screen">
    <div class="function-bar">
      <div>
        <label for="rows">Rows</label>
        <input id="rows" type="number" :disabled="isRunning" v-model.number="rows" />
      </div>
      <div>
        <label for="columns">Columns</label>
        <input id="columns" type="number" :disabled="isRunning" v-model.number="columns" />
      </div>
      <div>
        <label for="maxGen">Max Generation</label>
        <input id="maxGen" type="number" :disabled="isRunning" v-model.number="maxGen" />
      </div>
      <div>
        <label for="elapse">Elapse (Sec)</label>
        <input id="elapse" type="number" :disabled="isRunning" v-model.number="elapse" />
      </div>
      <div>
        <label for="cellSize">Cell Size</label>
        <input id="cellSize" type="number" :disabled="isRunning" v-model.number="cellSize" @blur="cellCss = `width: ${cellSize}px; height: ${cellSize}px;`" />
      </div>
      <div v-if="isRunning">Current Elapse : {{ currentElapse + 1 }}</div>
      <button v-if="!isRunning" @click="customizeMapSize">Create Map</button>
      <button v-if="!isRunning" @click="randomPosition">Random</button>
      <button v-if="!isRunning" @click="start">Start</button>
      <button @click="reset">Reset</button>
    </div>
    <div class="py-5">
      <table>
        <tbody v-if="rows > 0 && columns > 0 && position.length > 0">
          <tr v-for="i in rows" :key="i">
            <td v-for="j in columns" :key="j">
              <div :style="cellCss" :class="(position[currentElapse][i - 1][j - 1] ? 'bg-black' : 'bg-white')" @click="position[currentElapse][i - 1][j - 1] = !position[currentElapse][i - 1][j - 1]"></div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped lang="scss">
.function-bar {
  @apply flex items-center gap-3 rounded bg-white text-gray-900 p-5 shadow;
  & > div {
    @apply flex items-center gap-2;
  }
  input, select {
    @apply w-24 py-0.5 px-1 rounded border border-gray-300;
  }
  button {
    @apply text-white bg-black hover:bg-gray-800 rounded duration-150 px-3 py-1;
  }
}
table {
  @apply border-collapse text-center m-auto;
  td {
    @apply border;
  }
}
</style>
