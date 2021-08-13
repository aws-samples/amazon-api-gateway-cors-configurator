<template>
  <div class="">
    <label>Host:</label>
    <input type="text" v-model="host" placeholder="Server address to test" />
    <select v-model="testMethod" placeholder="Select available method to test">
      <option v-for="item in data.AllowMethods" :value="item" :key="item">{{item}}</option>
    </select>
    <button @click="testCors">Test</button>
  </div>
  <ul>
    <li v-for="(value, key) in results.headers" :key="key">{{key}}: {{value}}</li>
  </ul>
</template>

<script setup>
import { defineProps, ref, computed } from 'vue'
import axios from 'axios';

const props = defineProps({
  data: {}
})

const testMethod = ref("");
const host = ref("")
const results = ref({})

const testCors = () => {
  let axiosConfig = {method: testMethod.value, url: host.value}
  let headerConfig = {
    headers:{
      "access-control-allow-method": props.data.AllowMethods.toString(),
      "access-control-allow-headers": props.data.AllowHeaders.toString(),
    }
  }

  if(testMethod == "OPTIONS") axiosConfig.config = headerConfig

  axios(axiosConfig).then(res => {results.value = res})
}

</script>