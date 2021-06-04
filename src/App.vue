<template>
  <ListView 
  
    :data="data"
    :item-size-getter="itemSizeGetter"
    :estimated-item-size="30"  
  />
</template>

<script setup>
import { ref } from 'vue'
import ListView from './components/list-view/index.vue'

const data = ref([])
  for (let i = 0; i < 10000; i++) {
    data.value.push({ value: i });
  }

  function itemSizeGetter(item){
    return 30 + item.value % 10
  }

// This starter template is using Vue 3 experimental <script setup> SFCs
// Check out https://github.com/vuejs/rfcs/blob/script-setup-2/active-rfcs/0000-script-setup.md
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
