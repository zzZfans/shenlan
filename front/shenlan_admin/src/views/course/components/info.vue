<template>
  <div class="app-container">

    <!-- 课程信息表单 -->
    <el-form label-width="120px">

      <el-form-item label="课程标题">
        <el-input v-model="courseInfo.title" placeholder=" 示例：机器学习项目课：从基础到搭建项目视频课程。专业名称注意大小写"/>
      </el-form-item>

      <!-- 课程讲师 -->
      <el-form-item label="课程讲师">
        <el-select
          v-model="courseInfo.teacherId"
          placeholder="请选择">
          <el-option
            v-for="teacher in teacherList"
            :key="teacher.id"
            :label="teacher.name"
            :value="teacher.id"/>
        </el-select>
      </el-form-item>

      <!-- 所属分类：级联下拉列表 -->
      <el-form-item label="课程类别">
        <!-- 一级分类 -->
        <el-select
          v-model="courseInfo.subjectParentId"
          placeholder="请选择"
          @change="subjectChanged">
          <el-option
            v-for="subject in subjectList"
            :key="subject.id"
            :label="subject.title"
            :value="subject.id"/>
        </el-select>

        <!-- 二级分类 -->
        <el-select
          v-model="courseInfo.subjectId"
          placeholder="请选择">
          <el-option
            v-for="subject in subjectLevelTwoList"
            :key="subject.id"
            :label="subject.title"
            :value="subject.id"/>
        </el-select>
      </el-form-item>

      <el-form-item label="总课时">
        <el-input-number :min="0" v-model="courseInfo.lessonNum" controls-position="right" placeholder="请填写课程的总课时数"/>
      </el-form-item>

      <!-- 课程简介-->
      <el-form-item label="课程简介">
        <tinymce :height="300" v-model="courseInfo.description"/>
      </el-form-item>

      <!-- 课程封面 -->
      <el-form-item label="课程封面">
        <el-upload
          :show-file-list="false"
          :on-success="handleCoverSuccess"
          :before-upload="beforeCoverUpload"
          :on-error="handleCoverError"
          :action="BASE_API+'/admin/oss/file/upload?module=cover'"
          class="cover-uploader">
          <img v-if="courseInfo.cover" :src="courseInfo.cover">
          <i v-else class="el-icon-plus avatar-uploader-icon"/>
        </el-upload>
      </el-form-item>

      <el-form-item label="课程价格">
        <el-input-number :min="0" v-model="courseInfo.price" controls-position="right" placeholder="免费课程请设置为0元"/>
        元
      </el-form-item>
    </el-form>

    <div style="text-align:center">
      <el-button :disabled="saveBtnDisabled" type="primary" @click="saveAndNext()">保存并下一步</el-button>
    </div>
  </div>
</template>

<script>
import courseApi from "@/api/course";
import teacherApi from "@/api/teacher";
import subjectApi from "@/api/subject";
// 富文本编辑器
import Tinymce from "@/components/Tinymce";

export default {
  components: { Tinymce },
  data() {
    return {
      saveBtnDisabled: false, // 按钮是否禁用
      courseInfo: {// 表单数据
        price: 0,
        lessonNum: 0,
        // 以下解决表单数据不全时insert语句非空校验
        teacherId: "",
        subjectId: "",
        subjectParentId: "",
        cover: "",
        description: ""
      },
      teacherList: [], // 讲师列表
      subjectList: [], // 一级分类列表
      subjectLevelTwoList: [], // 二级分类列表
      BASE_API: process.env.BASE_API
    };
  },
  created() {
    if (this.$parent.courseId) { // 回显
      this.fetchCourseInfoById(this.$parent.courseId);
    } else { // 新增
      // 初始化分类列表
      this.initSubjectList();
    }
    // 获取讲师列表
    this.initTeacherList();
  },

  methods: {
    // 保存并下一步
    saveAndNext() {
      this.saveBtnDisabled = true;
      if (!this.$parent.courseId) {
        this.saveData();
      } else {
        this.updateData();
      }
    },
    // 保存
    saveData() {
      courseApi
        .saveCourseInfo(this.courseInfo)
        .then(response => {
          this.$message.success(response.message);
          this.$parent.courseId = response.data.courseId; // 获取courseId
          this.$parent.active = 1; // 下一步
        });
    },
    // 修改
    updateData() {
      courseApi
        .updateCourseInfoById(this.courseInfo)
        .then(response => {
          this.$message.success(response.message);
          this.$parent.active = 1; // 下一步
        });
    },
    initTeacherList() {
      teacherApi
        .list()
        .then(response => {
          this.teacherList = response.data.items;
        });
    },
    initSubjectList() {
      subjectApi.getNestedTreeList().then(response => {
        this.subjectList = response.data.items;
      });
    },
    subjectChanged(value) {
      this.subjectList.forEach(subject => {
        if (subject.id === value) {
          this.courseInfo.subjectId = "";
          this.subjectLevelTwoList = subject.children;
        }
      });
    },
    // 上传成功回调
    handleCoverSuccess(res, file) {
      if (res.success) {
        // console.log(res)
        this.courseInfo.cover = res.data.url;
        // 强制重新渲染
        this.$forceUpdate();
      } else {
        this.$message.error("上传失败！");
      }
    },
    // 上传校验
    beforeCoverUpload(file) {
      const isJPG = (file.type === "image/jpeg" || file.type === "image/png");
      const isLt2M = file.size / 1024 / 1024 < 10;

      if (!isJPG) {
        this.$message.error("上传封面图片只能是 JPG 或 PNG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传封面图片大小不能超过 2MB！");
      }
      return isJPG && isLt2M;
    },
    // 错误处理
    handleCoverError() {
      this.$message.error("上传失败(http error)");
    },
    fetchCourseInfoById(id) {
      courseApi
        .getCourseInfoById(id)
        .then(response => {
          this.courseInfo = response.data.item;

          // 初始化分类列表
          subjectApi
            .getNestedTreeList()
            .then(response => {
              this.subjectList = response.data.items;

              // 填充二级菜单：遍历一级菜单列表
              this
                .subjectList
                .forEach(subject => {
                  // 找到和courseInfo.subjectParentId一致的父类别记录
                  if (subject.id === this.courseInfo.subjectParentId) {
                    // 拿到当前类别下的子类别列表，将子类别列表填入二级下拉菜单列表
                    this.subjectLevelTwoList = subject.children;
                  }
                });
            });
        });
    }
  }
};
</script>

<style>
.cover-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.cover-uploader .el-upload:hover {
  border-color: #409EFF;
}

.cover-uploader .avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 640px;
  height: 357px;
  line-height: 357px;
  text-align: center;
}

.cover-uploader img {
  width: 640px;
  height: 357px;
  display: block;
}
</style>

