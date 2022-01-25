<template>
  <div>
    <el-date-picker
      format="yyyy-MM-dd"
      value-format="yyyy-MM-dd HH:mm:ss"
      type="date"
      :disabled="disabled"
      :style="styleWidth"
      :placeholder="placeholder"
      v-model="scope.row[nowTime]"
      :picker-options="pickerOptions"
    ></el-date-picker>
  </div>
</template>

<script>
export default {
  name: "DatePicker",
  data() {
    return {
      pickerOptions: {
        disabledDate: time => {
          if (this.start && !this.end) {
            return time.getTime() > new Date(this.start).getTime();
          }
          if (!this.start && this.end) {
            return time.getTime() < new Date(this.end).getTime();
          }
          if (Boolean(this.start) && Boolean(this.end)) {
            return (
              time.getTime() < new Date(this.end).getTime() ||
              time.getTime() > new Date(this.start).getTime()
            );
          }
        }
      }
    };
  },
  props: {
    styleWidth: {
      required: false,
      type: String
    },
    nowTime: {
      required: true,
      type: String
    },
    disabled: {
      required: false,
      type: Boolean
    },
    placeholder: {
      required: true,
      type: String
    },
    // 开始
    start: {
      required: false,
      type: String
    },
    // 结束
    end: {
      required: false,
      type: String
    },
    // scope
    scope: {
      required: true,
      type: Object
    }
  },
  methods: {
    // 时间开始选择
    change: function(value) {
      //   this.$emit("change", value);
    }
  }
};
</script>

<style scoped lang=scss>
</style>