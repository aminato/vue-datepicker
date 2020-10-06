<template>
  <div :style="cssVars" class="tempestive-datepicker--container">
    <div class="tempestive-datepicker--header">
      <button class="tempestive-datepicker--button prev" @click="move(-1)">
        <slot name="prev">
          <svg
            viewBox="0 0 24 24"
            width="20"
            height="20"
            stroke="currentColor"
            stroke-width="2"
            fill="none"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="css-i6dzq1"
          >
            <polyline points="15 18 9 12 15 6" />
          </svg>
        </slot>
      </button>
      <button @click="viewUp()" class="tempestive-datepicker--button display">{{currentView}}</button>
      <button class="tempestive-datepicker--button next" @click="move(+1)">
        <slot name="next">
          <svg
            viewBox="0 0 24 24"
            width="20"
            height="20"
            stroke="currentColor"
            stroke-width="2"
            fill="none"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="css-i6dzq1"
          >
            <polyline points="9 18 15 12 9 6" />
          </svg>
        </slot>
      </button>
    </div>

    <div
      style="display:grid; grid-template-columns: repeat(var(--columns-number), 1fr); gap:2px;"
      class="tempestive-datepicker--calendar"
    >
      <div
        v-for="day in daysTranslations"
        v-show="view==='day'"
        class="tempestive-datepicker--cellheader"
        :key="day"
      >{{$t(day)}}</div>
      <button
        v-for="{value,ts,cellClass} in cells"
        :key="ts"
        class="tempestive-datepicker--cell"
        :class="{'highlighted': highlight(ts),'selected': isSelected(ts),[cellClass]:true}"
        :disabled="disabled(ts,view)"
        @click="selectDate(ts)"
      >{{value}}</button>
    </div>
    <div class="tempestive-datepicker--footer" v-show="(confirm || time) && minView === view">
      <div v-show="time" class="tempestive-datepicker--time">
        <input
          class="tempestive-datepicker--input"
          type="number"
          aria-label="Hour"
          step="1"
          min="1"
          max="23"
          v-model.lazy.number="hours"
        />

        <p class="tempestive-datepicker--time-separator">:</p>

        <input
          class="tempestive-datepicker--input"
          type="number"
          aria-label="Minute"
          step="5"
          min="0"
          max="59"
          v-model.lazy.number="minutes"
        />

        <!-- <button class="tempestive-datepicker--button am-pm"
        title="Click to toggle">PM</button>-->
      </div>
      <button class="tempestive-datepicker--button apply" @click="emit(selected)">
        <slot name="apply">{{$t('apply')}}</slot>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "NubojDatePicker",
  props: {
    language: {
      type: String,
      default: "en"
    },
    min: {
      type: Number,
      default: () => new Date(1492, 9, 12).getTime()
    },
    max: {
      type: Number,
      default: () => new Date(3500, 0, 1).getTime()
    },
    value: {
      type: Number,
      default: null
    },
    firstDayOfWeek: {
      type: Number,
      default: 1,
      validator: value => {
        return value >= 0 && value <= 6;
      }
    },
    time: {
      type: Boolean,
      default: false
    },
    confirm: {
      type: Boolean,
      default: false
    },
    minView: {
      default: "day",
      type: String
    },
    translations: {
      type: Object,
      default: () => {
        return {
          days: ["sun", "mon", "tue", "wed", "thu", "fry", "sat"],
          months: [
            "gen",
            "feb",
            "mar",
            "apr",
            "mag",
            "giu",
            "lug",
            "ago",
            "sep",
            "ott",
            "nov",
            "dic"
          ]
        };
      }
    },
    disabled: {
      default: () => {
        return () => false;
      }
    },
    highlight: {
      default: () => {
        return () => false;
      }
    }
  },
  data() {
    return {
      view: "day",
      display: new Date(),
      selected: new Date()
    };
  },
  computed: {
    currentView() {
      switch (this.view) {
        case "day":
          return `${this.$t(
            this.translations.months[this.display.getMonth()]
          )} ${this.display.getFullYear()}`;

        case "month":
          return this.display.getFullYear();

        case "year":
          return `${this.display.getFullYear()} - ${this.display.getFullYear() +
            11}`;
        default:
          return this.display;
      }
    },
    daysTranslations() {
      const days = this.translations.days.slice();
      return days.concat(days.splice(0, this.firstDayOfWeek));
    },
    cssVars() {
      return {
        "--columns-number": this.view === "day" ? 7 : 3
      };
    },
    minutes: {
      set(value) {
        //console.log(value);
        if (value > 59 || value < 0) return;
        let date = new Date(this.selected);
        date.setMinutes(value);
        this.selected = date.getTime();
      },
      get() {
        return new Date(this.selected).getMinutes();
      }
    },
    hours: {
      set(value) {
        if (value > 23 || value < 0) return;
        let date = new Date(this.selected);
        date.setHours(value);
        this.selected = date.getTime();
      },
      get() {
        return new Date(this.selected).getHours();
      }
    },
    days() {
      const year = this.display.getFullYear();
      const month = this.display.getMonth();

      let days = [];

      const firstOfMonth = new Date(year, month, 1);
      const lastOfMonth = new Date(year, month + 1, 0);

      let start = this.firstDayOfWeek - firstOfMonth.getDay() + 1;

      if (start > 1) {
        start -= 7;
      }

      for (let i = start; i < 42 + start; i++) {
        const d = new Date(year, month, i);
        let cellClass = "current";

        if (d < firstOfMonth) {
          cellClass = "prev";
        } else if (d > lastOfMonth) {
          cellClass = "next";
        }
        days.push({
          value: d.getDate(),
          ts: d.getTime(),
          cellClass: cellClass
        });
      }
      return days;
    },
    months() {
      return [
        {
          value: this.$t(this.translations.months[0]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 0, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[1]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 1, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[2]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 2, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[3]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 3, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[4]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 4, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[5]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 5, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[6]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 6, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[7]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 7, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[8]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 8, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[9]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 9, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[10]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 10, 1).getTime()
        },
        {
          value: this.$t(this.translations.months[11]),
          cellClass: "current",
          ts: new Date(this.display.getFullYear(), 11, 1).getTime()
        }
      ];
    },
    years() {
      return Array.from({ length: 12 }, (value, index) => {
        return {
          value: this.display.getFullYear() + index,
          cellClass: "current",
          ts: new Date(this.display.getFullYear() + index, 0, 1).getTime()
        };
      });
    },
    cells() {
      if (this.view === "year") return this.years;
      else if (this.view === "month") return this.months;
      else return this.days;
    }
  },
  methods: {
    move(quantity) {
      if (this.view === "day") {
        this.display = new Date(
          this.display.getFullYear(),
          this.display.getMonth() + quantity,
          1
        );
      } else if (this.view === "month") {
        this.display = new Date(
          this.display.getFullYear() + quantity,
          this.display.getMonth(),
          1
        );
      } else if (this.view === "year") {
        this.display = new Date(
          this.display.getFullYear() + quantity * 12,
          this.display.getMonth(),
          1
        );
      }
    },
    viewDown() {
      switch (this.view) {
        case "day":
          break;
        case "month":
          this.view = "day";
          break;
        case "year":
          this.view = "month";
          break;
        default:
          this.view = "day";
      }
    },
    viewUp() {
      switch (this.view) {
        case "day":
          this.view = "month";
          break;
        case "month":
          this.view = "year";
          break;
        case "year":
          break;
        default:
          this.view = "day";
      }
    },
    selectDate(ts) {
      if (this.view == this.minView) {
        this.selected = ts;
        if (!this.time && !this.confirm) {
          this.emit(ts);
        }
      } else {
        this.display = new Date(ts);
        this.viewDown();
      }
    },
    emit(ts) {
      this.$emit("input", ts);
      this.$emit("selected", ts);
    },
    isSelected(ts) {
      const date = new Date(ts);
      const selected = new Date(this.selected);
      switch (this.view) {
        case "day":
          return (
            date.getFullYear() === selected.getFullYear() &&
            date.getMonth() === selected.getMonth() &&
            date.getDate() === selected.getDate()
          );

        case "month":
          return (
            date.getFullYear() === selected.getFullYear() &&
            date.getMonth() === selected.getMonth()
          );

        case "year":
          return date.getFullYear() === selected.getFullYear();
        default:
          return ts === this.value;
      }
    }
  },
  watch: {
    selected() {
      if (this.selected) {
        this.display = new Date(this.selected);
      }
    },
    minView: {
      immediate: true,
      handler() {
        this.view = this.minView;
      }
    },
    value: {
      immediate: true,
      handler() {
        this.selected = this.value;
      }
    }
  }
};
</script>

<style lang="scss">
.tempestive-datepicker--container {
  @apply border p-2 rounded;
  min-width: 300px;
}

.tempestive-datepicker--cellheader {
  @apply p-2 text-2xs uppercase tracking-wider text-center;
}
.tempestive-datepicker--cell {
  @apply p-2;
  &:hover {
    @apply bg-grey-75;
  }
  &.next {
    @apply opacity-50;
  }
  &.prev {
    @apply opacity-50;
  }
  &.highlighted {
    @apply bg-primary-200;
  }
  &.selected {
    @apply font-bold bg-primary-300;
  }

  &:disabled {
    @apply cursor-default opacity-30;
  }
}

.highlighted:last-child {
  @apply bg-yellow-300;
}

.tempestive-datepicker--time-separator {
  @apply self-center px-1;
}

.tempestive-datepicker--header {
  @apply flex pb-3;
}
.tempestive-datepicker--footer {
  @apply flex pt-3;
}

.tempestive-datepicker--time {
  @apply flex-1 flex;

  .tempestive-datepicker--input {
    @apply w-full border leading-loose text-center min-w-0;

    &:focus,
    &:hover {
      @apply bg-grey-75 outline-none;
    }
    &:invalid {
      @apply shadow-none;
    }
  }
}
.tempestive-datepicker--button {
  @apply px-2;

  &:hover {
    @apply bg-grey-75;
  }

  &.display {
    @apply flex-1 py-2;
  }
  &.apply {
    @apply ml-auto;
  }
}
</style>
