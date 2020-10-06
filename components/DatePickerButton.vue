<template>
  <div class="relative outline-none" tabindex="-1" v-click-outside="hide" @click="show=!show">
    <slot name="trigger">
      <button class="btn btn-outline btn-sm">
        <slot>
          <n-svg icon="calendar" class="h-6 w-6" />
        </slot>
      </button>
    </slot>

    <div
      v-if="breakpoints.mobile"
      v-show="show"
      @click.stop="cancel"
      class="fixed inset-0 bg-black-32 flex items-center justify-center z-20"
    >
      <date-picker
        :value="value"
        @click.native.stop
        class="bg-grey-0 shadow w-full m-10"
        :min-view="view"
        :disabled="disabled"
        :highlight="highlight"
        :time="time"
        :first-day-of-week="Number($store.state.settings.firstDayOfWeek)"
        @selected="selected"
      />
    </div>

    <div v-else v-show="show" @click.stop class="absolute z-20" :class="classes">
      <date-picker
        :value="value"
        class="bg-grey-0 shadow"
        :min-view="view"
        :disabled="disabled"
        :highlight="highlight"
        :time="time"
        :first-day-of-week="Number($store.state.settings.firstDayOfWeek)"
        @selected="selected"
      />
    </div>
  </div>
</template>

<script>
import DatePicker from "./DatePicker.vue";
import { breakpoints } from "@/vue-options/Observables";

export default {
  name: "NubojDatePickerButton",
  components: {
    DatePicker
  },
  props: {
    value: { required: true },
    format: { default: "YYYY-MM-DD" },
    view: {
      default: "day"
    },
    time: {
      type: Boolean,
      default: false
    },
    position: {
      type: String,
      default: "bottom"
    },
    disabled: {
      default: () => {
        return value => value > Date.now();
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
      show: false,
      breakpoints
    };
  },
  computed: {
    classes() {
      switch (this.position) {
        case "left": return "left-0";
        case "right": return "top-0 left-full";
        default:
          return "left-0";
      }
    }
  },
  methods: {
    selected(date) {
      if (!date) return;

      this.$emit("input", date);
      this.$emit("changed");
      this.show = false;
    },
    hide() {
      this.show = false;
    },
    cancel() {
      this.start = this.from;
      this.end = this.to;
      this.show = false;
    }
  }
};
</script>