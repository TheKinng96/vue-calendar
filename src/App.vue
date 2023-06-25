<script setup lang="ts">
import Calendar from "./components/Calendar.vue";
import { ref } from "vue";

const dates = ref<Date[]>([]);
const onDateChanged = (range: Array<Date>) => {
  dates.value = range;
};
const notBefore = ref<Date>();
</script>

<template>
  <div class="body">
    <div class="controls">
      <div class="control">
        <label for="selectedDates">Selected Dates:</label>
        <pre>{{ dates }}</pre>
      </div>
      <div class="control">
        <label for="disableDateBefore">Disable Date Before:</label>
        <input type="date" name="disableDateBefore" v-model="notBefore" />
      </div>
    </div>

    <Calendar
      @on-changed="onDateChanged"
      :previous-month="true"
      :selected-dates="dates"
      :not-before="notBefore"
      class="calendar"
    />
  </div>
</template>

<style scoped>
.calendar {
  background: rgba(113, 61, 201, 0.13);
  border-radius: 16px;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(5px);
  -webkit-backdrop-filter: blur(5px);
  border: 1px solid rgba(113, 61, 201, 0.49);
  padding: 1rem;
}

.control {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.controls {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.body {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgb(239 239 239);
}
</style>
