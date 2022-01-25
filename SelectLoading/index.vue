<template>
  <div>
    <el-select
      :remote-method="
        (query) => {
          remoteMethod(query, labelName);
        }
      "
      remote
      style="width: 100%"
      reserve-keyword
      v-el-select-loadmore="loadmore"
      @change="change(scope.row, scope.$index)"
      clearable
      filterable
      size="mini"
      @visible-change="showLoading"
      v-model="scope.row[model]"
      :disabled="disabled"
      :placeholder="placeholder"
    >
      <el-option
        v-for="(item, index) in allTankList1"
        :key="index"
        :label="item[labelName]"
        :value="item[keyName]"
      ></el-option>
      <div v-if="allTankList1.length > 19" class="search-keyword">
        <span>默认显示模糊搜索的前20条</span>
      </div>
    </el-select>
  </div>
</template>

<script>
export default {
  name: "SelectLoading",
  directives: {
    "el-select-loadmore": {
      bind(el, binding) {
        const SELECTWRAP_DOM = el.querySelector(
          ".el-select-dropdown .el-select-dropdown__wrap"
        );
        SELECTWRAP_DOM.addEventListener("scroll", function () {
          const condition =
            this.scrollHeight - this.scrollTop - 10 <= this.clientHeight;
          // 容器高度          滚动距离          当前可见屏幕高度
          if (condition) {
            binding.value();
          }
        });
      },
    },
  },
  data() {
    return {
      numbers: 0,
      numberX: 0,
      allTankList1: [],
      indexItem: 0,
      searchType: false,
      query: "",
    };
  },
  computed: {
    allTankList: {
      get() {
        if (this.indexItem >= this.data.length) {
          this.indexItem = 0;
          this.numbers = 0;
        }
        // this.data.forEach((item, index) => {
        //   if (item[this.keyName] == this.scope[this.model]) {
        //     this.indexItem = index;
        //     this.numbers = index;
        //   }
        // });
        this.allTankList1 = this.data.slice(
          this.indexItem,
          (this.numbers += 10)
        );
        return this.data.slice(this.indexItem, (this.numbers += 10));
      },
      set(val) {
        this.allTankList1 = val;
        // console.log(111,val)
      },
    },
  },
  watch: {
    // "scope.row": {
    //   handler(val, oldval) {
    //     this.data.forEach((item, index) => {
    //       if (item[this.keyName] == val[this.model]) {
    //         this.indexItem = index;
    //         this.numbers = index;
    //       }
    //     });
    //   },
    //   deep: true,
    // },
  },
  props: {
    data: {
      //所有的数据
      required: true,
      type: Array,
    },
    scope: {
      // scope
      required: true,
      type: Object,
    },
    model: {
      // 字段
      required: true,
      type: String,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    placeholder: {
      required: false,
      type: String,
    },
    keyName: {
      required: true,
      type: String,
    },
    labelName: {
      required: true,
      type: String,
    },
    change: {
      //change事件函数
      required: false,
      type: Function,
      default: function () {},
    },
  },
  methods: {
    showLoading(e) {
      this.searchType = false;
      this.query = "";
      if (this.scope.row[this.model]) {
        this.data.forEach((item, index) => {
          if (item[this.keyName] == this.scope.row[this.model]) {
            this.indexItem = index;
            this.numbers = index;
          }
        });
      } else {
        this.indexItem = 0;
        this.numbers = 0;
      }
      this.allTankList = this.data.slice(this.indexItem, (this.numbers += 10)); //要展示的数据
    },
    remoteMethod(query, key) {
      this.allTankList = [];
      if (query !== "") {
        this.searchType = true;
        this.query = query;
        this.allTankList = this.data
          .filter((item) => {
            return item[key].indexOf(query) > -1;
          })
          .slice(0, 20); //根据完整的列表来搜, 搜到的结果同样截取前20条, 不足20条忽略即可
      } else {
        this.searchType = false;
        this.numberX = 20;
        this.allTankList = this.data.slice(0, 20); // 关键字为空的时候又将完整的列表数据截取前20条渲染回去
      }
    },
    // 方法1
    loadmore(e) {
      if (this.searchType) {
        this.allTankList = this.data
          .filter((item) => {
            return item[this.labelName].indexOf(this.query) > -1;
          })
          .slice(0, (this.numberX += 20)); //要展示的数据
      } else {
        this.allTankList = this.data.slice(
          this.indexItem,
          (this.numbers += 20)
        ); //要展示的数据
        let list = this.allTankList;
      }
    },
    async initAllTankList() {
      this.data.forEach((item, index) => {
        if (item[this.keyName] == this.scope.row[this.model]) {
          this.indexItem = index;
          this.numbers = index;
        }
      });
      this.allTankList = this.data.slice(this.indexItem, (this.numbers += 10)); //要展示的数据
    },
  },
  created() {
    // console.log(this.scope.row[this.model], this.allTankList);
    this.initAllTankList();
    // console.log(this.scope.row[this.model], this.allTankList);
  },
};
</script>

<style scoped>
.search-keyword {
  text-align: center;
  padding: 3px 0;
  color: #ccc;
  font-size: 13px;
}
</style>
