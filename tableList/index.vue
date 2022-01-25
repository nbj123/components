<template>
  <div>
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <slot name="header"></slot>
      </div>
      <div class="box-card-body">
        <div class="el-tabs-table el-tabs-table-padding">
          <el-table max-height="406" key="tableList" ref="multipleTable" highlight-current-row tooltip-effect="dark" @row-click="handleRowClick" stripe :show-summary="showSummary" :summary-method="getSummaries" v-loading="Loading" :cell-style="cellStyle" :header-cell-style="rowClass" element-loading-text="拼命加载中" element-loading-spinner="el-icon-loading" element-loading-background="rgba(0, 0, 0, 0.2)" :data="list" border :default-sort="{ prop: 'date', order: 'descending' }" @sort-change="tableSortChange" @selection-change="selectRow" style="width: 100%" @row-dblclick="rowDbClick" @row-contextmenu="rightClick">
            <template v-for="(item, index) in tableList" style="display: inline-block">
              <el-table-column v-if="item.type == 'index'" :fixed="item.fixed" :key="index" type="index">
              </el-table-column>
              <el-table-column v-if="item.type == 'selection'" :key="index" width="55" :selectable="checkboxInit" :fixed="item.fixed" type="selection">
              </el-table-column>
              <!-- 无插槽，有排序 -->
              <el-table-column v-if="
                  !item.solt &&
                  item.sortable &&
                  item.type !== 'selection' &&
                  item.type !== 'index'
                " :key="item.prop" sortable="custom" show-overflow-tooltip :sort-orders="['ascending', 'descending']" :prop="item.prop" :fixed="item.fixed" :label="item.name ? $t(`main.${item.name}`) : item.label"></el-table-column>
              <!-- 无插槽，无排序 -->
              <el-table-column v-else-if="
                  !item.solt &&
                  !item.sortable &&
                  item.type !== 'selection' &&
                  item.type !== 'index'
                " :key="item.prop" :prop="item.prop" :fixed="item.fixed" show-overflow-tooltip :label="item.name ? $t(`main.${item.name}`) : item.label"></el-table-column>
              <!-- 有插槽 -->
              <slot v-else :name="item.solt"></slot>
            </template>
            <slot name="footer"></slot>
          </el-table>
        </div>
      </div>
      <div class="txtbtn_abs">
        <div class="dataTotal">
          <div slot="datasTotal">
            <slot name="datasTotal"></slot>
          </div>
        </div>
        <pagination v-if="pageShow" :total="total" :pageSizes="pageSizes" :page.sync="currentPage" :limit.sync="pageSize" @pagination="pagination" />
      </div>
    </el-card>
    <div id="menu" v-if="menusList.length > 0">
      <div class="menu" v-for="(item, index) in menusList" :key="index" @click.stop="item.eventName(rightClickData)">
        <div>{{ item.name }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import Pagination from "@/components/Pagination";
export default {
  name: "tableList",
  components: { Pagination },
  data() {
    return {
      rightClickData: {},
    };
  },
  props: {
    tableList: {
      required: true,
      type: Array,
    },
    dateList: {
      required: true,
      type: Array,
    },
    // titleList: {
    //   default: () => [],
    //   type: Array,
    // },
    selectionChange: {
      type: Array,
    },
    pageSizes: {
      type: Array,
      default() {
        return [10, 20, 30, 50, 100];
      },
    },
    loading: {
      required: true,
      type: Boolean,
    },
    total: {
      type: Number,
    },
    page: {
      type: Number,
      default: 1,
    },
    limit: {
      type: Number,
      default: 10,
    },
    pageShow: {
      type: Boolean,
      default: true,
    },
    showSummary: {
      type: Boolean,
      default: false,
    },
    getSummaries: {
      type: Function,
      default: () => { },
    },
    menus: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    list: {
      get() {
        return this.dateList;
      },
      set(val) {
        this.$emit("update:dateList", val);
      },
    },
    selectlistRow: {
      get() {
        return this.selectionChange;
      },
      set(val) {
        this.$emit("update:selectionChange", val);
      },
    },
    Loading: {
      get() {
        return this.loading;
      },
      set(val) {
        this.$emit("update:loading", val);
      },
    },
    menusList: {
      get() {
        let list = this.menus.filter((item) => {
          return item.id;
        });
        return list;
      },
    },
    currentPage: {
      get() {
        return this.page;
      },
      set(val) {
        this.$emit("update:page", val);
      },
    },
    pageSize: {
      get() {
        return this.limit;
      },
      set(val) {
        this.$emit("update:limit", val);
      },
    },
  },
  created() {
    // this.$nextTick(() => {
    //   this.$refs['multipleTable'].doLayout();
    // });
  },
  methods: {
    //点击行触发，选中或不选中复选框
    handleRowClick(row, column, event) {
      // this.$refs.multipleTable.toggleRowSelection(row);
    },
    // 首列是否可选中
    checkboxInit(row, index) {
      if (row.model && (row.model.judge != undefined || row.model.judge != null)) {
        if (row.model.judge == 0) {
          return 0; //不可勾选
        } else {
          return 1; //可勾选
        }
      } else {
        if (row.judge != null) {
          if (row.judge == 0) {
            return 0; //不可勾选
          } else {
            return 1; //可勾选
          }
        } else {
          return 1;
        }
      }
    },
    // 分页
    pagination(e) {
      this.$emit("pagination", e);
    },
    cellStyle({ row, column, rowIndex, columnIndex }) {
      return "text-align:center;padding:2px";
    },
    rowClass({ row, column, rowIndex, columnIndex }) {
      return "background:#f5f5f5;color:#606266;text-align:center;padding:3px;font-weight:700;";
    },
    // 选中项
    selectRow(val) {
      this.$emit("selectRow", val);
      this.selectlistRow = val;
    },
    // 排序
    tableSortChange(column) {
      // this.search(column);
      this.$emit("sortChange", column);
    },
    // 行双击事件
    rowDbClick(val) {
      this.$emit("rowDbClick", val);
    },
    // table的右键点击当前行事件
    rightClick(row, column, event) {
      if (!document.querySelector("#menu")) return;
      var menu = document.querySelector("#menu");
      event.preventDefault();
      //获取我们自定义的右键菜单
      this.rightClickData = row;
      // 根据事件对象中鼠标点击的位置，进行定位
      menu.style.left = event.pageX - 205 + "px";
      menu.style.top = event.pageY - 85 + "px";
      // 改变自定义菜单的隐藏与显示
      menu.style.display = "block";
      document.addEventListener(
        "click",
        function (e) {
          menu.style.display = "none";
        },
        true
      );
    },
  },
};
</script>

<style >
.el-tooltip__popper {
  max-width: 500px;
}
</style>
