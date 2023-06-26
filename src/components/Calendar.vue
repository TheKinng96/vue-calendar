<script lang="ts" setup>
import moment from "moment";
import { ref, onMounted, computed, watch } from "vue";

interface ObjectKey {
  [key: string]: any;
}

interface ICalendar extends ObjectKey {
  year: number;
  month: number;
  weeks: Array<Array<Date>>;
}

const emit = defineEmits(["onChanged"]);

const props = defineProps<{
  previousMonth?: boolean;
  selectedDates?: Array<Date>;
  disablePastDates?: boolean;
  selectOne?: boolean;
  notBefore?: Date;
}>();

const shorthandLabel: ObjectKey = {
  monday: "M",
  tuesday: "T",
  wednesday: "W",
  thursday: "T",
  friday: "F",
  saturday: "S",
  sunday: "S",
};

const selectedDates_ = computed(() => {
  if (!props.selectedDates) return [];

  let range: Array<Date> = [...props.selectedDates];
  range.sort((a, b) => a.getTime() - b.getTime());
  return range;
});

watch(
  selectedDates_,
  (val) => {
    if (val?.length && val.length > 0) {
      date_.value = val;
    } else {
      date_.value = [];
    }
  },
  { deep: true }
);

// Get Calendar data
const date = ref(
  props.previousMonth ? moment().subtract(1, "M").toDate() : moment().toDate()
);
const date_ = ref<Array<Date>>([]);
const calendar = ref<ICalendar>();
const weekLabels = ref<Array<string>>([
  "sunday",
  "monday",
  "tuesday",
  "wednesday",
  "thursday",
  "friday",
  "saturday",
]);

const changeMonth = (direction: "next" | "previous") => {
  switch (direction) {
    case "next":
      date.value = moment(date.value).add(1, "M").toDate();
      break;
    case "previous":
      date.value = moment(date.value).subtract(1, "M").toDate();
      break;
  }
  updateCalendar();
};

// Get weeks number of month
const getWeekNumbers = (year: number, month: number) => {
  let firstWeek = moment(new Date(year, month, 1)).isoWeek();
  let lastWeek = moment(new Date(year, month + 1, 0)).isoWeek();

  let firstWeek_ = firstWeek;
  let lastWeekOfPreviousYear = moment(
    new Date(year - 1, month - 1, 0)
  ).isoWeeksInYear();

  if (firstWeek >= 52 || firstWeek == 0) {
    firstWeek = 1;
  } else {
    firstWeek = firstWeek - 1;
  }

  let out = [];
  // If last week is smaller than first week
  if (lastWeek < firstWeek) {
    for (let i = firstWeek; i <= lastWeekOfPreviousYear; i++) {
      out.push(i);
    }
    for (let i = 1; i <= lastWeek; i++) {
      out.push(i);
    }
    return out;
  } else {
    for (let i = firstWeek; i <= lastWeek; i++) {
      if (i === 0) continue;
      out.push(i);
    }
  }

  // Adjust for first week for january
  if (
    firstWeek_ > 50 &&
    date.value.getMonth() + 1 === 1 &&
    firstWeek_ !== lastWeekOfPreviousYear
  ) {
    let out_ = [];
    if (firstWeek_ > lastWeekOfPreviousYear) {
      out_ = [firstWeek_];
    } else {
      for (let i = firstWeek_; i <= lastWeekOfPreviousYear; i++) {
        out_.push(i);
      }
    }
    out = [...out_, ...out];
  }

  return out;
};

const getWeekDaysByWeekNumber = (
  year: number,
  weekNumber: number,
  month: number
) => {
  let year_ = year;
  if (weekNumber > 50 && month === 1) {
    year_ = year - 1;
  } else if (weekNumber < 5 && month === 12) {
    year_ = year + 1;
  }

  let weekDate = moment()
      .year(year_)
      .isoWeek(weekNumber || 1)
      .startOf("week"),
    weekLength = 7,
    result = [],
    otherMonth = 0;

  while (weekLength--) {
    result.push(weekDate.toDate());
    +weekDate.format("M") !== date.value.getMonth() + 1 ? otherMonth++ : "";
    weekDate.add(1, "day");
  }

  return otherMonth === 7 ? [] : result;
};

const weekNumbers = computed(() =>
  getWeekNumbers(date.value.getFullYear(), date.value.getMonth())
);

onMounted(() => {
  updateCalendar();

  if (props.selectedDates?.length && props.selectedDates?.length > 0) {
    date_.value = selectedDates_.value;
  }
});

const updateCalendar = () => {
  const year = date.value.getFullYear();
  const month = date.value.getMonth() + 1;

  calendar.value = {
    year,
    month,
    weeks: weekNumbers.value.map((week) => {
      return getWeekDaysByWeekNumber(date.value.getFullYear(), week, month);
    }),
  };
};

const isToday = (date: Date): boolean => {
  return date.toDateString() === new Date().toDateString();
};

const selectDates = (date: Date) => {
  if (
    props.disablePastDates &&
    moment(date).isBefore(moment().subtract(1, "day"), "day")
  ) {
    return;
  }

  if (
    props.notBefore &&
    moment(date).isBefore(moment(props.notBefore), "day")
  ) {
    return;
  }
  date_.value = getRange(date, date_.value);

  emit("onChanged", date_.value);
};

const getRange = (date: Date, current: Array<Date>) => {
  if (props.selectOne) {
    return [date];
  }

  let range: Array<Date> = [];

  if (current.length < 2) {
    range = [...current];
  }

  range.push(date);
  range.sort((a, b) => a.getTime() - b.getTime());

  return range;
};

const isSelected = (date: Date, range: Array<Date>): boolean => {
  let isSelected = false;
  range.forEach((d) => {
    if (d.toDateString() === date.toDateString()) {
      isSelected = true;
    }
  });

  return isSelected;
};

const isBetween = (date: Date, range: Array<Date>): boolean => {
  if (range.length < 2) {
    return false;
  }

  return moment(date).isBetween(range[0], range[1]);
};

const isDisabled = (day: Date): boolean => {
  if (props.notBefore) {
    return moment(day).isBefore(moment(props.notBefore), "day");
  }
  return props.disablePastDates && day < new Date();
};
</script>

<template>
  <div class="calendar">
    <header>
      <button @click="changeMonth('previous')" class="arrow">
        <i class="fas fa-angle-left"></i>
      </button>
      <span class="calendar-title">
        {{ moment(date).format("YYYY - M") }}
      </span>
      <button @click="changeMonth('next')" class="arrow">
        <i class="fas fa-angle-right"></i>
      </button>
    </header>

    <div class="calendar-body">
      <div class="row first">
        <span v-for="label in weekLabels" :key="label">
          {{ shorthandLabel[label] }}
        </span>
      </div>
      <div class="row" v-for="(week, index) in calendar?.weeks" :key="index">
        <button
          v-for="day in week"
          :key="day.getTime()"
          :class="{
            'not-current-month': calendar?.month !== day.getMonth() + 1,
            today: isToday(day),
            selected: isSelected(day, date_),
            'is-between': isBetween(day, date_),
            'is-range': date_.length > 1,
            'is-first':
              date_.length >= 1 &&
              day.toDateString() === date_[0].toDateString(),
            'is-second':
              date_.length >= 2 &&
              day.toDateString() === date_[1].toDateString(),
            'is-disabled': isDisabled(day),
          }"
          @click="selectDates(day)"
        >
          {{ day.getDate() }}
        </button>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
$color-neutral-900: #0a0b12;
$color-neutral-600: #47496b;
$color-neutral-400: #9496b8;
$color-neutral-50: #f5f5fa;

$color-primary-600: #2c35bf;
$color-primary-500: #4750d5;
$color-primary-300: #acb0ec;
$color-primary-50: #f3f3fc;

$color-success-50: #f3fcf7;

$color-warning-600: #b26b1a;
$color-warning-500: #df8620;
$color-warning-300: #f2cfa6;
$color-warning-50: #fdf8f2;

$color-white: #ffffff;

button,
input[type="submit"],
input[type="reset"] {
  background: none;
  color: inherit;
  border: none;
  padding: 0;
  font: inherit;
  cursor: pointer;
  outline: inherit;
}

button {
  min-height: 2rem;
  transition: 0.15s ease-in-out all;

  &:hover {
    background-color: #bca3e4;
  }
}

.row.first span {
  display: flex;
  justify-content: center;
  align-items: center;

  &:hover {
    background-color: transparent !important;
  }
}

.calendar {
  header {
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin-bottom: 1rem;

    button {
      flex: 1;
      transition: 0.2s all linear;

      &.arrow:hover {
        background-color: #bca3e4;
        color: white;
      }
    }

    .calendar-title {
      padding: 1rem;
      cursor: default;
    }
  }
}

.calendar-body {
  display: flex;
  flex-direction: column;

  > .row {
    display: flex;
    justify-content: space-evenly;

    &.first span {
      font-weight: 500;
      color: $color-neutral-900;
    }

    button,
    span {
      flex: 1;
      padding: 0.5rem;
      text-align: center;
      color: $color-neutral-600;
      width: 2rem;
      height: 2.5rem;

      &.not-current-month {
        color: $color-neutral-400;
      }

      &.today {
        border: dashed 1px $color-primary-500;
      }

      &.is-between {
        background-color: $color-primary-50;
        border-radius: 0 !important;
      }

      &.is-disabled {
        color: $color-neutral-400;
        cursor: not-allowed;
      }

      &.is-between-comparison {
        background-color: $color-warning-50;
        border-radius: 0 !important;

        &.is-between {
          background-color: $color-success-50;
        }
      }

      &.selected-comparison {
        background-color: $color-warning-500;
        color: $color-white;

        &.is-first {
          border-top-left-radius: 50% !important;
          border-bottom-left-radius: 50% !important;
        }

        &.is-second {
          border-top-right-radius: 50% !important;
          border-bottom-right-radius: 50% !important;
        }

        &:hover {
          background-color: $color-warning-600;
        }

        &.is-between {
          background-color: $color-warning-300;
        }
      }

      &.selected {
        background-color: $color-primary-500;
        color: $color-white;

        &.is-first {
          border-top-left-radius: 50% !important;
          border-bottom-left-radius: 50% !important;
        }

        &.is-second {
          border-top-right-radius: 50% !important;
          border-bottom-right-radius: 50% !important;
        }

        &:hover {
          background-color: $color-primary-600;
        }

        &.is-between-comparison {
          background-color: $color-primary-300;
        }
      }

      &:hover {
        background-color: $color-primary-600;
      }
    }

    button {
      &:hover {
        background-color: #bca3e4;
        color: white;
      }
    }
  }
}
</style>
