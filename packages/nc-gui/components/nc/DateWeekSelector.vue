<script lang="ts" setup>
import dayjs from 'dayjs'

interface Props {
  selectedDate?: dayjs.Dayjs | null
  isDisabled?: boolean
  pageDate?: dayjs.Dayjs
  activeDates?: Array<dayjs.Dayjs>
  isMondayFirst?: boolean
  isWeekPicker?: boolean
  disablePagination?: boolean
  selectedWeek?: {
    start: dayjs.Dayjs
    end: dayjs.Dayjs
  } | null
}

const props = withDefaults(defineProps<Props>(), {
  selectedDate: null,
  isDisabled: false,
  isMondayFirst: true,
  pageDate: dayjs(),
  isWeekPicker: false,
  disablePagination: false,
  activeDates: [] as Array<dayjs.Dayjs>,
  selectedWeek: null,
})
const emit = defineEmits(['change', 'update:selectedDate', 'update:pageDate', 'update:selectedWeek'])
// Page date is the date we use to manage which month/date that is currently being displayed
const pageDate = useVModel(props, 'pageDate', emit)

const selectedDate = useVModel(props, 'selectedDate', emit)

const activeDates = useVModel(props, 'activeDates', emit)

const selectedWeek = useVModel(props, 'selectedWeek', emit)

const days = computed(() => {
  if (props.isMondayFirst) {
    return ['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su']
  } else {
    return ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']
  }
})

// Used to display the current month and year
const currentMonthYear = computed(() => {
  return dayjs(pageDate.value).format('MMMM YYYY')
})

const selectWeek = (date: dayjs.Dayjs) => {
  const dayOffset = +props.isMondayFirst
  const dayOfWeek = (date.day() - dayOffset + 7) % 7
  const startDate = date.subtract(dayOfWeek, 'day')
  selectedWeek.value = {
    start: startDate,
    end: startDate.endOf('week'),
  }
}

// Generates all dates should be displayed in the calendar
// Includes all blank days at the start and end of the month
const dates = computed(() => {
  const startOfMonth = dayjs(pageDate.value).startOf('month')
  const dayOffset = +props.isMondayFirst
  const firstDayOfWeek = startOfMonth.day()
  const startDay = startOfMonth.subtract((firstDayOfWeek - dayOffset + 7) % 7, 'day')

  const datesArray = []
  for (let i = 0; i < 42; i++) {
    datesArray.push(startDay.add(i, 'day'))
  }

  return datesArray
})

// Check if the date is in the selected week
const isDateInSelectedWeek = (date: dayjs.Dayjs) => {
  if (!selectedWeek.value) return false
  return date.isBetween(selectedWeek.value.start, selectedWeek.value.end, 'day', '[]')
}

// Used to check if two dates are the same
const isSameDate = (date1: dayjs.Dayjs, date2: dayjs.Dayjs) => {
  if (!date1 || !date2) return false
  return date1.isSame(date2, 'day')
}

// Used in DatePicker for checking if the date is currently selected
const isSelectedDate = (dObj: dayjs.Dayjs) => {
  if (!selectedDate.value) return false
  const propDate = dayjs(selectedDate.value)
  return props.selectedDate ? isSameDate(propDate, dObj) : false
}

const isDayInPagedMonth = (date: dayjs.Dayjs) => {
  return date.month() === dayjs(pageDate.value).month()
}

// Since we are using the same component for week picker and date picker we need to handle the date selection differently
const handleSelectDate = (date: dayjs.Dayjs) => {
  if (props.isWeekPicker) {
    selectWeek(date)
  } else {
    if (!isDayInPagedMonth(date)) {
      pageDate.value = date
      emit('update:pageDate', date)
    }
    selectedDate.value = date
    emit('update:selectedDate', date)
  }
}

// Used to check if a date is in the current month
const isDateInCurrentMonth = (date: dayjs.Dayjs) => {
  return date.month() === dayjs(pageDate.value).month()
}

// Used to Check if an event is in the date
const isActiveDate = (date: dayjs.Dayjs) => {
  return activeDates.value.some((d) => isSameDate(d, date))
}

// Paginate the calendar
const paginate = (action: 'next' | 'prev') => {
  let newDate = dayjs(pageDate.value)
  if (action === 'next') {
    newDate = newDate.add(1, 'month')
  } else {
    newDate = newDate.subtract(1, 'month')
  }
  pageDate.value = newDate
  emit('update:pageDate', newDate)
}
</script>

<template>
  <div class="px-4 pt-3 pb-4 flex flex-col gap-4">
    <div
      :class="{
        'justify-center': disablePagination,
        'justify-between': !disablePagination,
      }"
      class="flex items-center"
    >
      <NcTooltip>
        <NcButton v-if="!disablePagination" size="small" type="secondary" @click="paginate('prev')">
          <component :is="iconMap.doubleLeftArrow" class="h-4 w-4" />
        </NcButton>
        <template #title>
          <span>{{ $t('labels.previousMonth') }}</span>
        </template>
      </NcTooltip>

      <span class="font-bold text-gray-700">{{ currentMonthYear }}</span>
      <NcTooltip>
        <NcButton v-if="!disablePagination" size="small" type="secondary" @click="paginate('next')">
          <component :is="iconMap.doubleRightArrow" class="h-4 w-4" />
        </NcButton>
        <template #title>
          <span>{{ $t('labels.nextMonth') }}</span>
        </template>
      </NcTooltip>
    </div>
    <div class="border-1 border-gray-200 rounded-y-xl max-w-[320px]">
      <div class="flex flex-row bg-gray-100 gap-2 rounded-t-xl justify-between px-2">
        <span v-for="(day, index) in days" :key="index" class="h-9 flex items-center justify-center w-9 text-gray-500">{{
          day
        }}</span>
      </div>
      <div class="grid grid-cols-7 gap-2 p-2">
        <span
          v-for="(date, index) in dates"
          :key="index"
          :class="{
            'rounded-lg': !isWeekPicker,
            'bg-brand-50 border-1 !border-brand-500': isSelectedDate(date) && !isWeekPicker && isDayInPagedMonth(date),
            'hover:(border-1 border-gray-200 bg-gray-100)': !isSelectedDate(date) && !isWeekPicker,
            'nc-selected-week z-1': isDateInSelectedWeek(date) && isWeekPicker,
            'text-gray-400': !isDateInCurrentMonth(date),
            'nc-selected-week-start': isSameDate(date, selectedWeek?.start),
            'nc-selected-week-end': isSameDate(date, selectedWeek?.end),
            'rounded-md bg-brand-50 text-brand-500': isSameDate(date, dayjs()) && isDateInCurrentMonth(date),
          }"
          class="h-9 w-9 px-1 py-2 relative font-medium flex items-center cursor-pointer justify-center"
          data-testid="nc-calendar-date"
          @click="handleSelectDate(date)"
        >
          <span v-if="isActiveDate(date)" class="absolute z-2 h-1.5 w-1.5 rounded-full bg-brand-500 top-1 right-1"></span>
          <span class="z-2">
            {{ date.get('date') }}
          </span>
        </span>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.nc-selected-week {
  @apply relative;
}

.nc-selected-week:before {
  @apply absolute top-0 left-0 w-full h-full border-y-1 bg-brand-50 border-brand-500;
  content: '';
  width: 124%;
  height: 100%;
}

.nc-selected-week-start:before {
  @apply !border-l-1 !rounded-l-lg;
}

.nc-selected-week-end:before {
  width: 100%;
  @apply !border-r-1 !rounded-r-lg;
}
</style>
