<template>
  <div class="slider-container">
    <div class="view-switcher-wrapper">
      <button
        :class="{ active: viewMode === 'years' }"
        class="view-switcher-wrapper__button"
        @click="switchView('years')"
      >
        <span class="view-switcher-wrapper__button--text">Все года</span>
      </button>
      <button
        :class="{ active: viewMode === 'months' }"
        class="view-switcher-wrapper__button"
        @click="switchView('months')"
      >
        <span class="view-switcher-wrapper__button--text">Месяца</span>
      </button>
    </div>
    <vue-slider
      v-if="viewMode === 'years'"
      v-model="monthRange"
      :width="1117"
      :height="10"
      :dotSize="[20, 20]"
      :interval="1"
      :min="0"
      :max="monthMax"
      :marks="yearMarks"
      :tooltip="'always'"
      :process="true"
      :clickable="false"
      :stepStyle="stepStyle"
      :processStyle="processStyle"
      :railStyle="railStyle"
    >
      <template v-slot:tooltip="{ value, index }">
        <div
          class="slider-tooltip"
          :class="{
            'slider-tooltip--top': index === 0,
            'slider-tooltip--bottom': index === 1,
          }"
        >
          <div v-html="formatTooltip(value).month"></div>
          <div v-html="formatTooltip(value).year"></div>
        </div>
      </template>
    </vue-slider>

    <vue-slider
      v-if="viewMode === 'months'"
      v-model="monthRange"
      :width="1117"
      :height="10"
      :dotSize="[20, 20]"
      :interval="1"
      :min="jan2015Index"
      :max="monthMax"
      :marks="monthMarks"
      :tooltip="'always'"
      :process="true"
      :clickable="false"
      :stepStyle="stepStyle"
      :processStyle="processStyle"
      :railStyle="railStyle"
    >
      <template v-slot:tooltip="{ value, index }">
        <div
          class="slider-tooltip"
          :class="{
            'slider-tooltip--top': index === 0,
            'slider-tooltip--bottom': index === 1,
          }"
        >
          <div v-html="formatTooltip(value).month"></div>
          <div v-html="formatTooltip(value).year"></div>
        </div>
      </template>
    </vue-slider>
  </div>
</template>

<script>
import { ref, computed, watch } from "vue";
import VueSlider from "vue-3-slider-component";

export default {
  components: {
    VueSlider,
  },
  props: {
    minDate: {
      type: Date,
      required: true,
    },
    maxDate: {
      type: Date,
      required: true,
    },
    selectedStartDate: {
      type: Date,
      default: () => new Date(),
    },
    selectedEndDate: {
      type: Date,
      default: () => new Date(),
    },
  },
  setup(props) {
    const viewMode = ref("years");

    const processStyle = ref({
      top: "2px",
      height: "6px",
      backgroundColor: "rgba(92, 173, 234, 1)",
    });

    const railStyle = ref({
      backgroundColor: "rgba(237, 241, 248, 1)",
    });

    const yearLabelStyle = {
      display: "flex",
      justifyContent: "center",
      width: "40px",
      height: "18px",

      fontSize: "16px",
    };

    const yearLabelStyleForMonths = {
      color: "rgba(51, 51, 51, 1)",
    };

    const yearLabelStyleForYears = {
      color: "rgba(153, 153, 153, 1)",
    };

    const monthLabelStyle = {
      display: "flex",
      justifyContent: "center",
      width: "25px",
      minHeight: "18px",
      minWidth: "27px",

      color: "rgba(153, 153, 153, 1)",
      fontSize: "14px",
    };

    const stepStyle = ref({
      width: "0px",
    });

    const minYear = computed(() => props.minDate.getFullYear());
    const maxYear = computed(() => props.maxDate.getFullYear());

    const yearRange = ref([
      props.selectedStartDate.getFullYear(),
      props.selectedEndDate.getFullYear(),
    ]);

    const yearMarks = computed(() => {
      const marks = {};
      const startYear = props.minDate.getFullYear();
      const endYear = props.maxDate.getFullYear();
      const startMonth = props.minDate.getMonth();
      const endMonth = props.maxDate.getMonth();
      const totalMonths = (endYear - startYear) * 12 + endMonth - startMonth;

      for (let i = 0; i <= totalMonths; i++) {
        const date = new Date(startYear, startMonth + i);
        if (date.getMonth() === 0) {
          marks[i] = {
            label: date.getFullYear().toString(),
            labelStyle: { ...yearLabelStyle, ...yearLabelStyleForYears },
            type: "year",
          };
        }
      }
      return marks;
    });

    const months = [
      "Январь",
      "Февраль",
      "Март",
      "Апрель",
      "Май",
      "Июнь",
      "Июль",
      "Август",
      "Сентябрь",
      "Октябрь",
      "Ноябрь",
      "Декабрь",
    ];

    const jan2015Index = computed(() => {
      const jan2015 = new Date(2015, 0);
      return (
        (jan2015.getFullYear() - props.minDate.getFullYear()) * 12 +
        jan2015.getMonth() -
        props.minDate.getMonth()
      );
    });

    const jan2017Index = computed(() => {
      const jan2017 = new Date(2017, 0);
      return (
        (jan2017.getFullYear() - props.minDate.getFullYear()) * 12 +
        jan2017.getMonth() -
        props.minDate.getMonth()
      );
    });

    const monthMax = computed(() => {
      return viewMode.value === "months"
        ? jan2017Index.value
        : (props.maxDate.getFullYear() - props.minDate.getFullYear()) * 12 +
            props.maxDate.getMonth() -
            props.minDate.getMonth();
    });

    const monthRange = ref([jan2015Index.value, jan2017Index.value]);

    const monthMarks = computed(() => {
      const marks = {};
      const maxIndex =
        viewMode.value === "months" ? jan2017Index.value : monthMax.value;

      for (let i = jan2015Index.value; i <= maxIndex; i++) {
        const year = Math.floor(i / 12) + props.minDate.getFullYear();
        const month = i % 12;
        if (month === 0) {
          marks[i] = {
            label: `${year}`,
            labelStyle: { ...yearLabelStyle, ...yearLabelStyleForMonths },
            type: "year",
          };
        } else {
          marks[i] = {
            label: `${months[month].slice(0, 3).toLowerCase()}`,
            labelStyle: { ...monthLabelStyle },
            type: "month",
          };
        }
      }
      return marks;
    });

    const formatTooltip = (val) => {
      const year = Math.floor(val / 12) + props.minDate.getFullYear();
      const monthIndex = val % 12;
      const formatted = {
        month: months[monthIndex],
        year: year.toString(),
      };
      return val <= monthMax.value ? formatted : "";
    };

    const switchView = (newMode) => {
      viewMode.value = newMode;
      if (newMode === "months") {
        monthRange.value = [
          jan2015Index.value,
          Math.min(monthRange.value[1], jan2017Index.value),
        ];
      }
    };

    watch(viewMode, (newVal) => {
      if (newVal === "months") {
        monthRange.value = [
          jan2015Index.value,
          Math.min(monthRange.value[1], jan2017Index.value),
        ];
      }
    });

    return {
      viewMode,
      switchView,

      minYear,
      maxYear,
      yearRange,
      yearMarks,

      months,
      monthMax,
      monthRange,
      monthMarks,

      jan2015Index,
      jan2017Index,

      railStyle,
      processStyle,
      stepStyle,

      formatTooltip,
    };
  },
};
</script>

<style scoped>
.slider-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  width: 1347px;
}

.view-switcher-wrapper {
  display: flex;
  justify-content: center;
  flex-direction: column;
  max-width: 60px;
  margin-right: 105px;
  gap: 10px;

  text-align: left;
}

.view-switcher-wrapper__button {
  position: relative;

  box-sizing: border-box;
  display: inline-block;
  min-height: 18px;
  padding: 0;

  background-color: transparent;

  border: none;

  color: rgba(1, 103, 179, 1);
  text-align: left;
  font-size: 14px;
  font-weight: 600;
  line-height: 18px;
  letter-spacing: 0em;
  white-space: nowrap;

  cursor: pointer;
}

.view-switcher-wrapper__button:not(.active)
  .view-switcher-wrapper__button--text {
  border-bottom: 1px solid rgba(211, 220, 241, 1);
  opacity: 50%;
}

.slider-tooltip {
  position: relative;
  bottom: 8px;

  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;
  min-height: 60px;
  min-width: 70px;
  max-width: 112px;
  padding: 6px 14px;

  border-color: #c8dafc;
  border-radius: 12px;
  background-color: #fff;
  box-shadow: 0px 5px 15px 0px rgba(14, 88, 143, 0.2);

  color: rgba(1, 103, 179, 1);
  text-align: center;
  font-size: 18px;
  white-space: nowrap;
}

.slider-tooltip::after {
  position: absolute;
  top: 59px;
  left: 50%;

  border-width: 8px;
  border-color: transparent;
  border-style: solid;
  border-top-color: #fff;

  transform: translateX(-50%);

  content: "";
}

.slider-tooltip--top,
.slider-tooltip--bottom {
  align-self: center;
}

.slider-tooltip--bottom {
  position: relative;
  top: 107px;
}

.slider-tooltip--bottom::after {
  position: absolute;
  bottom: 100%;
  left: 50%;

  border-width: 8px;
  border-color: transparent transparent #fff transparent;
  border-style: solid;

  transform: translateX(-50%) translateY(-75px);

  content: "";
}
</style>
