<template>
  <div>
    <mavon-editor
      ref="editor"
      :value="value"
      :toolbars="toolbars"
      :ishljs="true"
      @imgAdd="imgAdd"
      @fullScreen="fullScreen"
      @change="change"
      @save="save"
      :language="editorLanguage"
      :boxShadow="false"
      :subfield="true"
      defaultOpen="preview"
      :placeholder="$t('articleContentPlaceholder')"
      codeStyle="atom-one-dark"
      :style="{ height: contentHeight, width: '100%' }"
    />
    <el-upload action :show-file-list="false" :limit="1" :before-upload="fileAdd">
      <el-button size="small" type="primary">
        <i class="el-icon-upload el-icon--right"></i> 上传文件
      </el-button>
    </el-upload>
  </div>
</template>

<script>
import qiniuUploader from "@/service/QiniuUploader";
import smmsUploader from "@/service/SmmsUploader";
import githubUploader from "@/service/GithubUploader";
import aliyunOssUploader from "@/service/AliyunOssUploader";
import tencentOssUploader from "@/service/TencentOssUploader";

export default {
  name: "Editor",
  props: {
    value: {
      type: String,
      default: ""
    },
    initValue: {
      type: String,
      default: ""
    }
  },
  data() {
    return {
      uploading: false,
      uploadingText: "loading...", // TODO gaoyoubo @ 2019-02-28 没显示上传文案
      contentHeight: this.height,
      fullScreenStatus: false,
      toolbars: {
        bold: true, // 粗体
        italic: true, // 斜体
        header: true, // 标题
        underline: true, // 下划线
        strikethrough: true, // 中划线
        mark: true, // 标记
        superscript: true, // 上角标
        subscript: true, // 下角标
        quote: true, // 引用
        ol: true, // 有序列表
        ul: true, // 无序列表
        link: true, // 链接
        imagelink: true, // 图片链接
        code: true, // code
        table: true, // 表格
        fullscreen: true, // 全屏编辑
        // readmodel: true, // 沉浸式阅读
        // htmlcode: true, // 展示html源码
        help: true, // 帮助
        // /* 1.3.5 */
        // undo: true, // 上一步
        // redo: true, // 下一步
        // trash: true, // 清空
        save: true, // 保存（触发events中的save事件）
        /* 1.4.2 */
        navigation: true, // 导航目录
        /* 2.1.8 */
        alignleft: true, // 左对齐
        aligncenter: true, // 居中
        alignright: true, // 右对齐
        /* 2.2.1 */
        subfield: true, // 单双栏模式
        preview: true // 预览
      }
    };
  },
  mounted() {
    let sysConfig = this.$store.state.Config.config;
    this.$refs.editor.markdownIt.use(githubUploader.markdownItPlugin, {
      match: "/images",
      prefix: sysConfig.path + "/source"
    });

    this.resize();
    window.addEventListener("resize", this.resize);
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.resize);
  },
  methods: {
    async fileUpload(file) {
      let me = this;
      me.uploading = true;
      me.uploadingText = "正在上传 " + file.name;
      let sysConfig = me.$store.state.Config.config;
      try {
        let url;
        if (sysConfig.uploadType === "qiniu") {
          url = await qiniuUploader.upload(file, sysConfig);
        } else if (sysConfig.uploadType === "sm.ms") {
          url = await smmsUploader.upload(file);
        } else if (sysConfig.uploadType === "aliyunOss") {
          url = await aliyunOssUploader.upload(file, sysConfig);
        } else if (sysConfig.uploadType === "tencentOss") {
          url = await tencentOssUploader.upload(file, sysConfig);
        } else {
          url = await githubUploader.upload(file, sysConfig);
        }
        me.uploading = false;
        return url;
      } catch (error) {
        me.$notify.error({ message: "图片上传失败：" + error });
        me.uploading = false;
      }
    },
    async fileAdd(file) {
      const link = await this.fileUpload(file);
      const insert_text = {
        prefix: `[input file name here](`,
        subfix: `){target="_blank"}`,
        str: link
      };
      this.$refs.editor.insertText(
        this.$refs.editor.getTextareaDom(),
        insert_text
      );
    },
    /**
     * 添加图片
     */
    async imgAdd(pos, file) {
      const url = await this.fileUpload(file);
      this.$refs.editor.$img2Url(pos, url);
    },
    /**
     * 内容变更
     * @param value
     * @param render
     */
    change(value) {
      this.$emit("input", value);
      if (value !== this.initValue) {
        this.$emit("change", value);
      }
    },
    /**
     * 保存
     */
    save(value) {
      this.$emit("save", value);
    },
    /**
     * 切换全屏
     * @param status
     */
    fullScreen(status) {
      this.fullScreenStatus = status;
      this.resize();
    },

    resize() {
      if (this.fullScreenStatus) {
        this.contentHeight = document.documentElement.clientHeight + "px";
      } else {
        this.contentHeight = document.documentElement.clientHeight - 110 + "px";
      }
    }
  },
  computed: {
    editorLanguage() {
      let config = this.$store.state.Config.config;
      if (config.language === "zh") {
        return "zh-CN";
      } else {
        return "en";
      }
    }
  }
};
</script>

<style scoped></style>
