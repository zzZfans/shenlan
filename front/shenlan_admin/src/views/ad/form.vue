<template>
  <div class="app-container">
    <!-- 输入表单 -->
    <el-form label-width="120px">
      <el-form-item label="广告推荐名称">
        <el-input v-model="ad.title"/>
      </el-form-item>
      <!-- 推荐位 -->
      <el-form-item label="推荐位">
        <el-select
          v-model="ad.typeId"
          placeholder="请选择">
          <el-option
            v-for="adType in adTypeList"
            :key="adType.id"
            :label="adType.title"
            :value="adType.id"/>
        </el-select>
      </el-form-item>
      <el-form-item label="排序">
        <el-input-number v-model="ad.sort" :min="0"/>
      </el-form-item>
      <el-form-item label="广告图片">
        <el-upload
          :on-success="handleAvatarSuccess"
          :on-error="handleAvatarError"
          :on-exceed="handleUploadExceed"
          :before-upload="beforeAvatarUpload"
          :limit="1"
          :file-list="fileList"
          :action="BASE_API+'/admin/oss/file/upload?module=ad'"
          list-type="picture">
          <el-button size="small" type="primary">点击上传</el-button>
        </el-upload>
      </el-form-item>
      <el-form-item label="背景颜色">
        <el-color-picker v-model="ad.color"/>
      </el-form-item>
      <el-form-item label="链接地址">
        <el-input v-model="ad.linkUrl"/>
      </el-form-item>
      <el-form-item>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate()">保存</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import adApi from "@/api/ad";
import adTypeApi from "@/api/adType";

export default {
  data() {
    return {
      ad: {
        sort: 0
      },
      fileList: [], // 上传文件列表
      adTypeList: [],
      saveBtnDisabled: false, // 保存按钮是否禁用，防止表单重复提交
      BASE_API: process.env.BASE_API
    };
  },

  // 页面渲染成功
  created() {
    if (this.$route.params.id) {
      this.fetchDataById(this.$route.params.id);
    }

    // 获取推荐位列表
    this.initAdTypeList();
  },

  methods: {

    initAdTypeList() {
      adTypeApi.list().then(response => {
        this.adTypeList = response.data.items;
      });
    },

    saveOrUpdate() {
      // 禁用保存按钮
      this.saveBtnDisabled = true;
      if (!this.ad.id) {
        this.saveData();
      } else {
        this.updateData();
      }
    },

    // 新增
    saveData() {
      // debugger
      adApi.save(this.ad).then(response => {
        this.$message.success(response.message);
        this.$router.push({ path: "/ad/list" });
      });
    },

    // 根据id更新记录
    updateData() {
      // teacher数据的获取
      adApi.updateById(this.ad).then(response => {
        this.$message.success(response.message);
        this.$router.push({ path: "/ad/list" });
      });
    },

    // 根据id查询记录
    fetchDataById(id) {
      adApi.getById(id).then(response => {
        this.ad = response.data.item;
        this.fileList = [{ "url": this.ad.imageUrl }];
      });
    },

    // 上传多于一个文件
    handleUploadExceed(files, fileList) {
      this.$message.warning("想要重新上传图片，请先删除已上传的视频");
    },

    // 上传校验
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },

    // 上传成功回调
    handleAvatarSuccess(res, file) {
      console.log(res);
      if (res.success) {
        // console.log(res)
        this.ad.imageUrl = res.data.url;
        // 强制重新渲染
        // this.$forceUpdate()
      } else {
        this.$message.error("上传失败1");
      }
    },

    // 错误处理
    handleAvatarError() {
      console.log("error");
      this.$message.error("上传失败2");
    }
  }
};
</script>
