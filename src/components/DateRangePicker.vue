<template>
  <div style="position: relative; display: inline-block;">
    <div class="reportrange-text" @click="togglePicker">
      <slot name="input">
        <i class="glyphicon glyphicon-calendar fa fa-calendar"></i>&nbsp;
        <span>{{label}}</span>
        <b class="caret"></b>
      </slot>
    </div>
    <transition name="slide-fade" mode="out-in">
      <div class="daterangepicker dropdown-menu ltr show-ranges" :class="pickerStyles()" v-if="open" v-on-clickaway="clickAway">
        <div class="calendars">
          <calendar-ranges @clickRange="clickRange" :ranges="ranges"></calendar-ranges>

          <div class="drp-calendar left">
            <div class="daterangepicker_input hidden-xs" v-if="false">
              <input class="input-mini form-control" type="text" name="daterangepicker_start"
                                   :value="startText"/>
              <i class="fa fa-calendar glyphicon glyphicon-calendar"></i>
            </div>
            <div class="calendar-table">
              <calendar :monthDate="monthDate" :locale="locale" :start="start" :end="end" @nextMonth="nextMonth" @prevMonth="prevMonth" @dateClick="dateClick" @hoverDate="hoverDate" :minDate="min" :maxDate="max"></calendar>
            </div>
          </div>

          <div class="drp-calendar right hidden-xs">
            <div class="daterangepicker_input" v-if="false">
              <input class="input-mini form-control" type="text" name="daterangepicker_end"
                                   :value="endText"/>
              <i class="fa fa-calendar glyphicon glyphicon-calendar"></i>
            </div>
            <div class="calendar-table">
              <calendar :monthDate="nextMonthDate" :locale="locale" :start="start" :end="end" @nextMonth="nextMonth" @prevMonth="prevMonth" @dateClick="dateClick" @hoverDate="hoverDate" :minDate="min" :maxDate="max"></calendar>
            </div>
          </div>
        </div>

        <div class="drp-buttons">
          <button class="applyBtn btn btn-sm btn-success" :disabled="inSelection" type="button" @click="clickedApply">{{locale.applyLabel}}
          </button>
          <button class="cancelBtn btn btn-sm btn-default" type="button" @click="open=false">
            {{locale.cancelLabel}}
          </button>
        </div>

      </div>
    </transition>
  </div>
</template>

<script>
import moment from "moment";
import { mixin as clickaway } from "vue-clickaway";
import { findKey } from "lodash";

import Calendar from "./Calendar.vue";
import CalendarRanges from "./CalendarRanges";
import { nextMonth, prevMonth } from "./util";

export default {
  components: { Calendar, CalendarRanges },
  mixins: [clickaway],
  props: {
    localeData: {
      type: Object,
      default() {
        return {};
      }
    },
   showRangeLabel: {
        type: Boolean,
        default: false
    },
    minDate: {},
    maxDate: {},
    startDate: {
      default() {
        return new Date();
      }
    },
    endDate: {
      default() {
        return new Date();
      }
    },
    ranges: {
      type: Object,
      default() {
        return {
          Today: [moment(), moment()],
          Yesterday: [
            moment().subtract(1, "days"),
            moment().subtract(1, "days")
          ],
          "This month": [moment().startOf("month"), moment().endOf("month")],
          "This year": [moment().startOf("year"), moment().endOf("year")],
          "Last week": [
            moment()
              .subtract(1, "week")
              .startOf("week"),
            moment()
              .subtract(1, "week")
              .endOf("week")
          ],
          "Last month": [
            moment()
              .subtract(1, "month")
              .startOf("month"),
            moment()
              .subtract(1, "month")
              .endOf("month")
          ]
        };
      }
    },
    opens: {
      type: String,
      default: "center"
    }
  },
  data() {
    return {
      monthDate: new Date(this.startDate),
      min: this.minDate ? new Date(this.minDate) : null,
      max: this.maxDate ? new Date(this.maxDate) : null,
      start: new Date(this.startDate),
      end: new Date(this.endDate),
      inSelection: false,
      open: false
    };
  },
  methods: {
    nextMonth() {
      this.monthDate = nextMonth(this.monthDate);
    },
    prevMonth() {
      this.monthDate = prevMonth(this.monthDate);
    },
    dateClick(value) {
      if (this.inSelection) {
        this.inSelection = false;
        this.end = new Date(value);
        if (this.end < this.start) {
          this.inSelection = true;
          this.start = new Date(value);
        }
      } else {
        this.inSelection = true;
        this.start = new Date(value);
        this.end = new Date(value);
      }
    },
    hoverDate(value) {
      let dt = new Date(value);
      if (this.inSelection && dt > this.start) this.end = dt;
    },
    togglePicker() {
      this.open = !this.open;
    },
    pickerStyles() {
      return {
        "show-calendar": this.open,
        opensright: this.opens === "right",
        opensleft: this.opens === "left",
        openscenter: this.opens === "center"
      };
    },
    clickedApply() {
      this.open = false;
      this.$emit("update", { startDate: this.start, endDate: this.end });
    },
    clickAway() {
      if (this.open) {
        this.open = false;
      }
    },
    clickRange(value) {
      this.start = new Date(value[0]);
      this.end = new Date(value[1]);
      this.monthDate = new Date(value[0]);
      this.clickedApply();
    }
  },
  computed: {
    locale() {
      const DEFAULT_LOCALE_DATA = {
        direction: "ltr",
        format: moment.localeData().longDateFormat("L"),
        separator: " - ",
        applyLabel: "Apply",
        cancelLabel: "Cancel",
        weekLabel: "W",
        customRangeLabel: "Custom Range",
        daysOfWeek: moment.weekdaysMin(),
        monthNames: moment.monthsShort(),
        firstDay: moment.localeData().firstDayOfWeek()
      };

      // update day names order to firstDay
      let locale = { ...DEFAULT_LOCALE_DATA, ...this.localeData };

      if (locale.firstDay !== 0) {
        let iterator = locale.firstDay;
        while (iterator > 0) {
          locale.daysOfWeek.push(locale.daysOfWeek.shift());
          iterator--;
        }
      }
      return locale;
    },
    isRange() {
      return findKey(this.ranges, range => {
        return (
          moment.utc(this.start).isSame(range[0], "day") &&
          moment.utc(this.end).isSame(range[1], "day")
        );
      });
    },
    label() {
        return this.showRangeLabel && this.isRange || this.startText + this.locale.separator + this.endText;
      },
    nextMonthDate() {
      return nextMonth(this.monthDate);
    },
    startText() {
      return moment(this.start).format(this.localeData.format);
    },
    endText() {
      return moment(new Date(this.end)).format(this.localeData.format);
    }
  },
  watch: {
    minDate(value) {
        this.min = new Date(value);
      },
      maxDate(value) {
        this.max = new Date(value);
      },
    startDate(value) {
      this.start = new Date(value);
    },
    endDate(value) {
      this.end = new Date(value);
    }
  }
};
</script>

<style>
.reportrange-text {
  background: #fff;
  cursor: pointer;
  padding: 5px 10px;
  border: 1px solid #ccc;
  width: 100%;
}

.daterangepicker {
  flex-direction: column;
  display: flex;
  width: auto;
}

.calendars {
  display: flex;
}

div.daterangepicker.opensleft {
  top: 35px;
  right: 10px;
  left: auto;
}

div.daterangepicker.openscenter {
  top: 35px;
  right: auto;
  left: 50%;
  transform: translate(-50%, 0);
}

div.daterangepicker.opensright {
  top: 35px;
  left: 10px;
  right: auto;
}

/* Enter and leave animations can use different */
/* durations and timing functions.              */
.slide-fade-enter-active {
  transition: all 0.2s ease;
}

.slide-fade-leave-active {
  transition: all 0.1s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter, .slide-fade-leave-to
        /* .slide-fade-leave-active for <2.1.8 */
 {
  transform: translateX(10px);
  opacity: 0;
}
</style>
