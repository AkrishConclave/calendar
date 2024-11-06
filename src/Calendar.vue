<template>
  <div id="calendar-container">
    <input
      type="hidden"
      id="time"
      v-model="formattedDateTime"
      placeholder="Выберите дату и время"
      readonly
      @click="toggleCalendar"
    />
    <div v-show="isOpen" class="calendar-container">
      <!-- Шапка календаря с названием месяца и годом -->
      <div class="calendar-header">
        <div>
          <p><span class="title-date-hour">{{ textMonth[currentMonth] }}</span> {{ currentYear }}</p>
        </div>
        <div>
          <span id="prev-month" @click="changeMonth(-1)"><</span>
          <span @click="reset" class="today-day-c">сегодня</span>
          <span id="next-month" @click="changeMonth(1)">></span>
        </div>
      </div>

      <!-- Дни недели -->
      <div class="calendar-weekdays">
        <div
          v-for="day in weekdays"
          :key="day"
          :class="['calendar-day','calendar-weekday', { 'weekend-color': day === 'Сб' || day === 'Вс' }]"
        >
          {{ day }}
        </div>
      </div>

      <!-- Дни месяца -->
      <div class="calendar-days">
        <div v-for="empty in emptyDays" :key="'empty-' + empty" class="calendar-day empty"></div>
        <div
          v-for="day in daysInMonth"
          :key="'day-' + day"
          :class="[
            'calendar-day',
            { weekend: isWeekend(day), active: currentDay === day, holiday: isHoliday(day), busy: isBusy(day) }
          ]"
          :style="isToday(day) ? { textDecoration: 'underline', fontWeight: '900' } : {}"
          @click="selectDay(day)"
        >
          {{ day }}
        </div>
      </div>

      <!-- Выбор часов и минут, если день выбран -->
      <div v-if="currentDay" class="time-selector">
        <div class="hours">
          <p class="title-date-hour">Выберите часы:</p>
          <button :class="['time-button', {active: currentHour === hour,}]" v-for="hour in availableHours" :key="'hour-' + hour" @click="selectHour(hour)">
            {{ hour }}
          </button>
        </div>
        <div v-if="currentHour" class="minutes">
          <p class="title-date-hour">Выберите минуты:</p>
          <button class="time-button" v-for="minute in availableMinutes" :key="'minute-' + minute" @click="selectMinute(minute)">
            {{ minute }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Calendar",
  data() {
    return {
      isOpen: false,
      currentYear: new Date().getFullYear(),
      currentMonth: new Date().getMonth(),
      currentDay: null,
      currentHour: null,
      currentMinute: null,
      dayNow: new Date().getDate(),
      monthNow: new Date().getMonth(),
      textMonth: [
        'Январь',
        'Февраль',
        'Март',
        'Апрель',
        'Май',
        'Июнь',
        'Июль',
        'Август',
        'Сентябрь',
        'Октябрь',
        'Ноябрь',
        'Декабрь',
      ],
      weekdays: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'],
      holidays: [
        { day: 1, month: 0 },
        { day: 7, month: 0 },
        { day: 23, month: 1 },
        { day: 8, month: 2 },
        { day: 1, month: 4 },
        { day: 9, month: 4 },
        { day: 12, month: 5 },
        { day: 4, month: 10 },
      ],
      busyDays: [3],
      workingDays: [8],
      testMode: true
    };
  },
  mounted() {
    if(this.testMode){ console.log("Calendar.vue смонтирован"); }
  },
  computed: {
    daysInMonth() {
      return new Date(this.currentYear, this.currentMonth + 1, 0).getDate();
    },
    emptyDays() {
      return (new Date(this.currentYear, this.currentMonth, 1).getDay() || 7) - 1;
    },
    availableHours() {
      return Array.from({ length: 11 }, (_, i) => i + 9); // Часы с 9 до 19
    },
    availableMinutes() {
      return ['00', 15, 30, 45];
    },
    formattedDateTime() {
      if (!this.currentDay || !this.currentHour || !this.currentMinute) return '';
      const formattedMonth = (this.currentMonth + 1).toString().padStart(2, '0');
      const formattedDay = this.currentDay.toString().padStart(2, '0');
      const formattedHour = this.currentHour.toString().padStart(2, '0');
      const formattedMinute = this.currentMinute.toString().padStart(2, '0');
      return `${this.currentYear}-${formattedMonth}-${formattedDay}T${formattedHour}:${formattedMinute}`;
    },
  },
  methods: {
    toggleCalendar() {
      this.isOpen = !this.isOpen;
    },
    reset() {
      this.currentDay = null;
      this.currentHour = null;
      this.currentMinute = null;
      this.currentMonth = this.monthNow;
      this.currentYear = new Date().getFullYear();
    },
    changeMonth(direction) {
      this.currentMonth += direction;
      if (this.currentMonth < 0) {
        this.currentMonth = 11;
        this.currentYear--;
      } else if (this.currentMonth > 11) {
        this.currentMonth = 0;
        this.currentYear++;
      }
    },
    isWeekend(day) {
      const weekday = new Date(this.currentYear, this.currentMonth, day).getDay();
      return weekday === 0 || weekday === 6;
    },
    isToday(day) {
      return this.dayNow === day && this.monthNow === this.currentMonth;
    },
    isHoliday(day) {
      return this.holidays.some(h => h.day === day && h.month === this.currentMonth);
    },
    isBusy(day) {
      return this.busyDays.includes(day);
    },
    selectDay(day) {
      if (this.isWeekend(day) || this.isBusy(day)) {
        return;
      }
      this.currentDay = day;
      this.currentHour = null;
      this.currentMinute = null;
    },
    selectHour(hour) {
      this.currentHour = hour;
    },
    selectMinute(minute) {
      this.currentMinute = minute;
      this.$emit("dateSelected", this.formattedDateTime);
      this.toggleCalendar();
    }
  }
};
</script>

<style scoped>
  #calendar-container {
  /*    display: flex;
      flex-direction: column;*/
      width: 22rem;
  }

  .calendar-table {
      width: 100%;
      border-collapse: collapse;
  }

  .calendar-table td {
      width: 40px;
      height: 40px;
      text-align: center;
  }

  .current-day {
      background-color: black;
      color: white;
  }

  .selected-day {
      background-color: grey;
      color: white;
  }

  .holiday {
      background-color: lightcoral;
      color: darkred;
  }

  .working-day {
      background-color: lightgreen;
      color: darkgreen;
  }

  .legend {
      display: flex;
      justify-content: space-between;
  }

  .calendar-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 10px;
      gap: 1rem;
  }

  .calendar-weekdays, .calendar-days {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 5px;
  }

  .calendar-weekday, .calendar-day {
      text-align: center;
      padding: 3px;
      background-color: #ffffff;
      border-radius: 8px;
  }

  .calendar-weekday {
      margin-bottom: 10px;
      text-transform: uppercase;
      font-weight: 900;
      color: #949393;
      border: 1px solid #c2c2c2;
  }

  .calendar-day {
      cursor: pointer;
  }

  .active {
      background-color: #ccc;
  } 

  .calendar-day:hover {
      background-color: #ccc;
  }

  .calendar-day.empty {
      background-color: transparent;
      pointer-events: none;
  }

  .holiday {
      background-color: lightcoral;
      color: white;
  }

  .busy {
      background-color: red;
      color: white;
  }

  .working {
      background-color: lightgreen;
      color: black;
  }

  .time-button {
      margin-right: 5px;
      margin-bottom: 5px;
      padding: 5px 10px;
      border: 1px solid #ddd;
      cursor: pointer;
      border-radius: 8px;
  }

  .time-button:hover {
      background-color: #ccc;
  }

  #prev-month, #next-month {
      margin: 0;
      cursor: pointer;
      color: #949393;
      font-weight: 600;
  }

  .weekend-color {
      color: #dc9090!important;
      border: 1px solid #dc9090;
      border-radius: 8px;
      text-align: center;
  }

  .today-day-c {
      padding: 2px 8px;
      background: #eaeaea;
      border-radius: 8px;
      cursor: pointer;
      margin: 0 10px;
  }

  .calendar-day.weekend {
      color: gray;
      opacity: 0.5;
  }

  .reset-button {
      background: #dad7f4!important;
  }

  .title-date-hour {
      margin: 1rem 0;
      font-weight: 600;
  }
</style>
