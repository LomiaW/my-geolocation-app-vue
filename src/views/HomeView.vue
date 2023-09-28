<script setup>
import getCurrentLocation from '../components/getCurrentLocation.vue';
import getLocationByName from '../components/getLocationByName.vue';
</script>

<template>
   <div class="ui container">
      <h1 class="ui centered header">My Location Search App</h1>
      <getCurrentLocation />
      <getLocationByName />
   </div>
</template>
