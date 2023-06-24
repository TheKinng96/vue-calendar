<script setup lang="ts">
import Calendar from "./components/Calendar.vue";
import { ref } from "vue";

const dates = ref<Date[]>([]);
const onDateChanged = (range: Array<Date>) => {
  console.log(range);
  dates.value = range;
};
const notBefore = ref<Date>();
</script>

<template>
  <div class="body">
    <input type="text" disabled v-model="dates" />
    <input type="date" v-model="notBefore" />
    {{ dates }}
    <Calendar
      @on-changed="onDateChanged"
      :previous-month="true"
      :selected-dates="dates"
      :not-before="notBefore"
    />
  </div>
</template>

<style scoped>
.body {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
</style>
