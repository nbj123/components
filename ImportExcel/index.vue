<template>
  <div>
    <el-button
      style="margin-left: 10px"
      icon="el-icon-upload2"
      @click="dialogVisible = true"
      type="primary"
      size="mini"
      round
    >{{ titleName }}</el-button>
    <!-- 导入 -->
    <el-dialog :title="$t('main.Import')" :visible.sync="dialogVisible" width="30%">
      <div style="display: flex; justify-content: space-around">
        <el-upload
          id="importExcel"
          class="upload-demo"
          action="/api/file/upload"
          :before-upload="inspectFilebeforeUpload"
        >
          <!-- 导出 -->
          <el-button size="small" type="primary">{{ "Excel " }}{{$t('main.Import')}}</el-button>
        </el-upload>
        <!-- 导出excel模板 -->
        <el-button
          v-if="!modelHidden"
          size="small"
          @click="getTemplate()"
          type="primary"
        >{{ $t('main.exportExcelTemplate') }}</el-button>
      </div>
      <span slot="footer" class="dialog-footer">
        <!-- 注：请使用导出后的模板填写数据进行导入 -->
        <p v-if="!modelHidden" style="color: red">{{$t('main.exportedTemplate')}}</p>
        <!-- 取消 -->
        <el-button @click="dialogVisible = false">{{$t('table.cancel')}}</el-button>
        <!-- 确定 -->
        <el-button type="primary" @click="dialogVisible = false">{{$t('table.confirm')}}</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { importExcel, exportTemplate } from "@/api/tsmRequestApi";
import { importExcel1, exportTemplate1 } from "@/api/mtmsRequestApi";
export default {
  name: "",
  props: {
    importStr: {
      required: true,
      type: String,
      default: "",
    },
    exportStr: {
      required: true,
      type: String,
    },
    afterImport: {
      required: false,
      type: Function,
    },
    titleName: {
      required: false,
      type: String,
      default: window.vm.$t("main.Import"),
    },
    modelHidden: {
      required: false,
      type: Boolean,
      default: false,
    },
    systemType: {
      required: false,
      type: String,
      default: "tsm",
    },
  },
  data() {
    return {
      dialogVisible: false,
      excelName: {
        Vehicle: "车辆基础信息表(模板)",
        Trailer: "车挂基础信息表(模板)",
        Employee: "人员基础信息表(模板)",
        InnerRefueling: "内加油信息表(模板)",
        HighwayRoute: "路线基础信息表(模板)",
        priceInfo: "定价规则基础信息表(模板)",
      },
    };
  },
  created() {
    // console.log(this.exportStr, this.importStr);
  },
  beforeMount() {},
  methods: {
    // 导入excel
    inspectFilebeforeUpload(file) {
      console.log(file, "aaa", file.name.split(".")[1]);
      const isJPG =
        file.name.split(".")[file.name.split(".").length - 1] === "xlsx" ||
        file.name.split(".")[file.name.split(".").length - 1] === "xls";
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isJPG) {
        this.$message.error(this.$t("main.canOnly") + " Excel");
        return;
      }
      if (!isLt2M) {
        this.$message.error(this.$t("main.fileSizeCannotExceed") + "2M");
        return;
      }
      let fd = new FormData();
      fd.append("file", file);
      fd.append("type", this.importStr);
      if (this.systemType == "tsm") {
        importExcel(fd).then((res) => {
          if (res.status == 0) {
            this.dialogVisible = false;
            this.$message.success(this.$t("main.ImportSucceeded"));
            this.afterImport();
          } else {
            this.$message.error(res.message);
          }
        });
      } else {
        importExcel1(fd).then((res) => {
          if (res.status == 0) {
            this.dialogVisible = false;
            this.$message.success(this.$t("main.ImportSucceeded"));
            this.afterImport();
          } else {
            this.$message.error(res.message);
          }
        });
      }
    },
    // 导出模板
    getTemplate() {
      if (this.systemType == "tsm") {
        exportTemplate({ type: this.exportStr }).then((res) => {
          if (res) {
            // console.log(res);
            let link = document.createElement("a");
            link.href = window.URL.createObjectURL(res);
            link.download = this.excelName[this.exportStr] + ".xls";
            link.click();
          } else {
            this.$message.success(this.$t("main.ExportFailed"));
          }
        });
      } else {
        exportTemplate1({ type: this.exportStr }).then((res) => {
          if (res) {
            // console.log(res);
            let link = document.createElement("a");
            link.href = window.URL.createObjectURL(res);
            link.download = this.excelName[this.exportStr] + ".xls";
            link.click();
          } else {
            this.$message.success(this.$t("main.ExportFailed"));
          }
        });
      }
    },
  },
};
</script>

<style scoped>
</style>