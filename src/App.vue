<script setup>
import HelloWorld from './components/HelloWorld.vue'
import Calendar from "./components/Calendar.vue";
import {ref} from "vue";


let dateList =  ref([{date: 20241128, message: {type: 'info', info: `我要起飞啦`}, task: [`今天要打螺丝`, `下午要干电焊工`]}]);
</script>

<template>
  <div>
    <a href="https://vite.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <HelloWorld msg="Vite + Vue" />
  <Calendar :date-list="dateList"></Calendar>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
