<template>
  <el-input :value="value" @input="change($event)">
    <template slot="append">
      <el-upload
        action
        :show-file-list="false"
        :limit="1"
        :before-upload="imgAdd"
        :file-list="fileList"
      >
        <el-button size="small" type="primary">
          <i class="el-icon-upload el-icon--right"></i>
        </el-button>
      </el-upload>
    </template>
  </el-input>
</template>

<script>
import qiniuUploader from "@/service/QiniuUploader";
import smmsUploader from "@/service/SmmsUploader";
import githubUploader from "@/service/GithubUploader";
import aliyunOssUploader from "@/service/AliyunOssUploader";
import tencentOssUploader from "@/service/TencentOssUploader";

export default {
  name: "UploadInput",
  props: {
    value: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      fileList: [],
      uploading: false,
      uploadingText: "loading...", // TODO gaoyoubo @ 2019-02-28 没显示上传文案
    };
  },
  mounted() {},
  beforeDestroy() {},
  methods: {
    /**
     * 添加图片
     */
    async imgAdd(file) {
      let me = this;
      me.uploading = true;
      me.uploadingText = "正在上传 " + file.name;
      let sysConfig = me.$store.state.Config.config;

      if (sysConfig.uploadType === "qiniu") {
        qiniuUploader.upload(file, sysConfig).then(
          (url) => {
            me.change(url);
            me.uploading = false;
          },
          (err) => {
            me.$notify.error({ message: "图片上传失败：" + err });
            me.uploading = false;
          }
        );
      } else if (sysConfig.uploadType === "sm.ms") {
        smmsUploader.upload(file).then(
          (url) => {
            // me.$refs.editor.$img2Url(pos, url);
            me.change(url);
            me.uploading = false;
          },
          (err) => {
            me.$notify.error({ message: "图片上传失败：" + err });
            me.uploading = false;
          }
        );
      } else if (sysConfig.uploadType === "aliyunOss") {
        try {
          let url = await aliyunOssUploader.upload(file, sysConfig);
          //   me.$refs.editor.$img2Url(pos, url);
          me.change(url);
          me.uploading = false;
        } catch (e) {
          console.error(e);
          me.$notify.error({ message: "图片上传失败：" + e });
          me.uploading = false;
        }
      } else if (sysConfig.uploadType === "tencentOss") {
        try {
          let url = await tencentOssUploader.upload(file, sysConfig);
          //   me.$refs.editor.$img2Url(pos, url);
          me.change(url);
          me.uploading = false;
        } catch (e) {
          console.error(e);
          me.$notify.error({ message: "图片上传失败：" + e });
          me.uploading = false;
        }
      } else {
        githubUploader.upload(file, sysConfig).then(
          (url) => {
            // me.$refs.editor.$img2Url(pos, url);
            me.change(url);
            me.uploading = false;
          },
          (err) => {
            me.$notify.error({ message: "图片上传失败：" + err });
            me.uploading = false;
          }
        );
      }

      throw new Error("no to upload");
    },
    /**
     * 内容变更
     * @param value
     * @param render
     */
    change(value) {
      this.$emit("input", value);
    },
  },
  computed: {},
};
</script>

<style scoped></style>
