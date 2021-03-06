<template>
    <table class="table-condensed">
        <thead>
        <tr>
            <th class="prev available" @click="$emit('prevMonth')"><i :class="[arrowLeftClass]"></i></th>
            <th colspan="5" class="month">{{monthName}} {{year}}</th>
            <th class="next available" @click="$emit('nextMonth')"><i :class="[arrowRightClass]"></i></th>
        </tr>
        </thead>
        <tbody>
        <tr>
            <th v-for="weekDay in locale.daysOfWeek" :key="weekDay">{{weekDay}}</th>
        </tr>
        <tr v-for="(dateRow, index) in calendar" :key="'date-row-' + index">
            <slot name="date-slot" v-for="(date, index) in dateRow">
                <td :key="'date-' + index" :class="dayClass(date)" @click="$emit('dateClick', date)" @mouseover="$emit('hoverDate', date)">
                    {{date | dateNum}}
                </td>
            </slot>
        </tr>
        </tbody>
    </table>
</template>

<script>
import moment from "moment";

export default {
  name: "calendar",
  props: ["monthDate", "locale", "start", "end", "minDate", "maxDate"],
  methods: {
    dayClass(date) {
      let dt = new Date(date);
      dt.setHours(0, 0, 0, 0);
      let start = new Date(this.start);
      start.setHours(0, 0, 0, 0);
      let end = new Date(this.end);
      end.setHours(0, 0, 0, 0);

      return {
        off: date.month() !== this.month,
        weekend: date.isoWeekday() > 5,
        today: dt.setHours(0, 0, 0, 0) == new Date().setHours(0, 0, 0, 0),
        active:
          dt.setHours(0, 0, 0, 0) ==
            new Date(this.start).setHours(0, 0, 0, 0) ||
          dt.setHours(0, 0, 0, 0) == new Date(this.end).setHours(0, 0, 0, 0),
        disabled:
          (this.minDate &&
            moment(dt)
              .startOf("day")
              .isBefore(moment(this.minDate).startOf("day"))) ||
          (this.maxDate &&
            moment(dt)
              .startOf("day")
              .isAfter(moment(this.maxDate).startOf("day"))),
        "in-range": dt >= start && dt <= end,
        "start-date": dt.getTime() === start.getTime(),
        "end-date": dt.getTime() === end.getTime()
      };
    }
  },
  computed: {
    arrowLeftClass() {
      return "fa fa-chevron-left glyphicon glyphicon-chevron-left";
    },
    arrowRightClass() {
      return "fa fa-chevron-right glyphicon glyphicon-chevron-right";
    },
    monthName() {
      return this.locale.monthNames[this.monthDate.getMonth()];
    },
    year() {
      return this.monthDate.getFullYear();
    },
    month() {
      return this.monthDate.getMonth();
    },
    calendar() {
      let month = this.month;
      let year = this.monthDate.getFullYear();
      let daysInMonth = new Date(year, month, 0).getDate();
      let firstDay = new Date(year, month, 1);
      let lastDay = new Date(year, month, daysInMonth);
      let lastMonth = moment(firstDay)
        .subtract(1, "month")
        .month();
      let lastYear = moment(firstDay)
        .subtract(1, "month")
        .year();
      let daysInLastMonth = moment([lastYear, lastMonth]).daysInMonth();

      let dayOfWeek = firstDay.getDay();

      let calendar = [];

      for (let i = 0; i < 6; i++) {
        calendar[i] = [];
      }

      let startDay = daysInLastMonth - dayOfWeek + this.locale.firstDay + 1;
      if (startDay > daysInLastMonth) startDay -= 7;

      if (dayOfWeek === this.locale.firstDay) startDay = daysInLastMonth - 6;

      let curDate = moment([lastYear, lastMonth, startDay, 12, 0, 0]);
      for (
        let i = 0, col = 0, row = 0;
        i < 6 * 7;
        i++, col++, curDate = moment(curDate).add(1, "day")
      ) {
        if (i > 0 && col % 7 === 0) {
          col = 0;
          row++;
        }
        calendar[row][col] = curDate.clone();
        curDate.hour(12);
      }

      return calendar;
    }
  },
  filters: {
    dateNum(value) {
      return value.date();
    }
  }
};
</script>

<style>
td.disabled {
  pointer-events: none;
}
td.today {
  font-weight: bold;
}
</style>
