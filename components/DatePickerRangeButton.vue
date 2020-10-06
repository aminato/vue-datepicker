<template>
  <div class="relative range-picker btn btn-outline btn-sm"
       tabindex="0"
       @click="show=!show">
    <slot>

      <n-svg icon="calendar"
             class="h-6 w-6" />

    </slot>
    <div v-show="show"
         @click.stop="cancel"
         class="fixed inset-0 bg-black-32 flex items-center justify-center z-20 normal-case overflow-y-auto py-4 tablet:py-0">
      <div class="mt-4 tablet:mt-2 shadow-md rounded py-2 laptop:py-8 px-4 laptop:px-10 bg-grey-0 border border-grey-light cursor-auto"
           @click.stop>
        <div class="tablet-landscape:flex">

          <div class="text-grey-darker mb-10 tablet-landscape:mr-10 tablet-landscape:mb-0">
            <p class="text-grey-800 text-md pb-3">{{$t('startDate')}}</p>
            <date-picker :first-day-of-week="Number($store.state.settings.firstDayOfWeek)"
                         :highlight="highlight"
                         :value="start"
                         @selected="setStartDate"
                         :min-view="view" />
          </div>
          <div class="text-grey-darker">
            <p class="text-grey-800 text-md pb-3">{{$t('endDate')}}</p>
            <date-picker :first-day-of-week="Number($store.state.settings.firstDayOfWeek)"
                         :highlight="highlight"
                         :value="end"
                         @selected="setEndDate"
                         :min-view="view" />

          </div>
        </div>
        <div class="flex justify-end pt-6">
          <button class="btn btn-flat"
                  type="button"
                  @click="cancel">{{$t('cancel')}}</button>
          <button class="btn btn-primary"
                  type="button"
                  @click="apply">{{$t('apply')}}</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import DatePicker from "./DatePicker.vue";
export default {
  name: "NubojDatePickerRangeButton",
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
    }
  },
  data() {
    return {
      show: false,
      start: this.from,
      end: this.to
    };
  },

  methods: {
    highlight(value) {
      return value <= this.end && value >= this.start;
    },
    apply() {
      this.show = false;
      if (!(this.start && this.end)) return;

      this.$emit("from", Math.min(this.start, this.end));
      this.$emit("to", Math.max(this.start, this.end));

      this.$emit("changed");
    },
    cancel() {
      this.start = this.from;
      this.end = this.to;
      this.show = false;
    },
    setStartDate(date) {
      if (!date) return;

      if (date > this.end) this.end = date + 24 * 60 * 60 * 1000;
      this.start = date;
    },
    setEndDate(date) {
      if (!date) return;
      const newDate = new Date(date);
      newDate.setHours(23, 59, 59, 999);
      date = newDate.getTime();

      if (date < this.start) this.start = date - 24 * 60 * 60 * 1000;
      this.end = date;
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
