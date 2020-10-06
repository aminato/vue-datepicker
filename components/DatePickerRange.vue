<template>
  <div>
    <div class>
      <slot name="start-label">
        <p class="text-grey-800 text-md pb-3">{{$t('startDate')}}</p>
      </slot>
      <date-picker
        :first-day-of-week="Number($store.state.settings.firstDayOfWeek)"
        :highlight="highlight"
        :value="start"
        @selected="setStartDate"
        :min-view="view"
        :time="time"
      />
    </div>

    <div class>
      <slot name="end-label">
        <p class="text-grey-800 text-md pb-3">{{$t('endDate')}}</p>
      </slot>
      <date-picker
        :first-day-of-week="Number($store.state.settings.firstDayOfWeek)"
        :highlight="highlight"
        :value="end"
        @selected="setEndDate"
        :min-view="view"
        :time="time"
      />
    </div>
  </div>
</template>

<script>
import DatePicker from "./DatePicker.vue";
export default {
  name: "NubojDatePickerRange",
  components: {
    DatePicker
  },
  props: {
    from: {
      required: true
    },
    to: {
      required: true
    },
    view: {
      default: "day"
    },
    disabled: {
      type: Object,
      default: () => ({
        customPredictor: date => {
          date.setHours(0, 0, 0, 0);
          if (date.getTime() > Date.now()) {
            return true;
          }
        }
      })
    },
    time: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      start: this.from,
      end: this.to
    };
  },

  methods: {
    highlight(value) {
      return value <= this.end && value >= this.start;
    },
    apply() {
      if (!(this.start && this.end)) return;

      this.$emit("from", Math.min(this.start, this.end));
      this.$emit("to", Math.max(this.start, this.end));

      this.$emit("changed");
    },
    setStartDate(date) {
      if (!date) return;

      if (date > this.end) this.end = date + 24 * 60 * 60 * 1000;
      this.start = date;
      this.apply();
    },
    setEndDate(date) {
      if (!date) return;
      const newDate = new Date(date);
      newDate.setHours(23, 59, 59, 999);
      date = newDate.getTime();

      if (date < this.start) this.start = date - 24 * 60 * 60 * 1000;
      this.end = date;
      this.apply();
    }
  },
  watch: {
    from(from) {
      this.start = from;
    },
    to(to) {
      this.end = to;
    }
  }
};
</script>
