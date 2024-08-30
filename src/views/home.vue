<script setup>
import { defineAsyncComponent } from 'vue';



// import weathercon from '../components/weathercard.vue'
 

const weathercon=defineAsyncComponent(()=>
{
  import('../components/weathercard.vue')
})

</script>

<template>




  <weathercon/>
</template>