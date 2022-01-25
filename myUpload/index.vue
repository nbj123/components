<template>
  <div style="height: 100%">
    <el-upload
      :class="{ hide: hideUpload }"
      action
      ref="newupload"
      :limit="limitCount"
      list-type="picture-card"
      :file-list="uploadEditChange"
      :before-upload="beforeUpload"
      :before-remove="beforeRemove"
      :on-preview="handlePictureCardPreview"
      :on-remove="handleRemove"
      :on-change="isChange"
    >
      <i class="el-icon-plus"></i>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible">
      <img width="100%" :src="dialogImageUrl" alt />
    </el-dialog>
  </div>
</template>
<script>
import { fileUpload, deleteFilePath } from "@/api/tsmRequestApi";
export default {
  name: "myUpload",
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      fileList: [], // 图片展示
      fileList1: [], // 图片上传
      hideUpload: false,
      time: 0,
    };
  },
  props: {
    limitCount: {
      type: Number,
      default: 1,
    },
    pageNum: {
      type: Number,
      default: 0,
    },
    uploadEdit: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    uploadEditChange: {
      get() {
        if (this.uploadEdit != null) {
          this.hideUpload = this.uploadEdit.length >= this.limitCount;
          this.fileList1 = this.uploadEdit;
          for (let index = this.time; index < this.uploadEdit.length; index++) {
            this.fileList.push({
              url:
                window.location.protocol +
                "//" +
                window.location.host +
                this.uploadEdit[index], // 线上
            });
          }
          this.time = this.uploadEdit.length;
          // console.log(this.fileList, "get val");
          return this.fileList;
        } else {
          return [];
        }
      },
      set(val) {
        let newVal = [];
        val.forEach((item) => {
          if (item.name) {
            newVal.push(item.url);
          } else {
            newVal.push(item);
          }
        });
        // console.log(newVal, "set val");
        this.$emit("update:uploadEdit", newVal);
      },
    },
  },
  created() {},
  methods: {
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    beforeRemove(file, fileList) {
      let isDel = true;
      if (file && file.status === "success") {
        isDel = this.$confirm(this.$t("main.confirmedRemoval") + "此文件");
      }
      return isDel;
    },
    handleRemove(file, fileList) {
      let removeFile = file.url.split("/");
      let removeUrl =
        "/" +
        removeFile[removeFile.length - 2] +
        "/" +
        removeFile[removeFile.length - 1];
      let index = this.uploadEdit.indexOf(removeUrl);
      let data = {
        filePath: removeUrl,
        page: this.pageNum,
      };
      // console.log(index);
      deleteFilePath(data)
        .then((result) => {
          if (result.status == 0) {
            this.uploadEdit.splice(index, 1);
            this.uploadEditChange.splice(index, 1);
            // this.fileList1.splice(index, 1);
            this.hideUpload = this.uploadEdit.length >= this.limitCount;
            this.$message({
              type: "success",
              message: "操作成功!",
            });
          }
        })
        .catch((err) => {
          // this.$message({
          //   type: "warning",
          //   message: "操作失败!",
          // });
          fileList.push(file);
          this.hideUpload = fileList.length >= this.limitCount;
          return false;
        });
    },
    beforeUpload(file) {
      const isJPG =
        file.type === "image/jpeg" ||
        file.type === "image/png" ||
        file.type === "image/jpg" ||
        file.type === "image/gif";
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isJPG) {
        this.$message.error("上传图片只能是 JPG / PNG / JPEG / gif 格式 !");
      } else if (!isLt2M) {
        this.$message.error("上传图片大小不能超过 2MB!");
      } else {
        let fd = new FormData();
        fd.append("file", file);
        fd.append("page", this.pageNum);
        fileUpload(fd).then((res) => {
          if (res.status == 0) {
            this.fileList1.push({ url: res.result, name: file.name });
            this.uploadEditChange = this.fileList1;
          }
        });
      }
    },
    // 自定义文件上传
    // uploadFile(file) {
    //   let fd = new FormData();
    //   fd.append("file", file.file);
    //   fd.append("page", this.pageNum);
    //   fileUpload(fd).then((res) => {
    //     if (res.status == 0) {
    //       this.fileList1.push({ url: res.result, name: file.file.name });
    //       // this.uploadEditChange = this.fileList1;
    //     }
    //   });
    // },
    isChange(file, fileList) {
      this.hideUpload = fileList.length >= this.limitCount;
    },
  },
};
</script>

<style lang=scss>
.hide .el-upload--picture-card {
  display: none;
}
</style>
