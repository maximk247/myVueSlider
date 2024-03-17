<template>
  <MySlider
    :min-date="minDate"
    :max-date="maxDate"
    :selected-start-date="selectedStartDate"
    :selected-end-date="selectedEndDate"
  />
</template>

<script setup>
import { ref } from "vue";
import MySlider from "./components/MySlider.vue";

const minDate = ref(new Date(2014, 0, 1));
const maxDate = ref(new Date(2021, 0, 1));
const selectedStartDate = ref(new Date(2014, 0, 1));
const selectedEndDate = ref(new Date(2016, 0, 1));
</script>
