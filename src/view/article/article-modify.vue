<template>
  <div class="container">
    <div class="title">
      <span>修改新闻</span> <span class="back" @click="back"> <i class="iconfont icon-fanhui"></i> 返回 </span>
    </div>
    <el-divider></el-divider>
    <div class="wrap">
      <el-row>
        <el-col :lg="16" :md="20" :sm="24" :xs="24">
          <el-form :model="form" status-icon ref="form" label-width="100px" v-loading="loading" @submit.native.prevent>
            <el-form-item label="新闻标题" prop="title">
              <el-input size="medium" v-model="form.title" placeholder="请填写新闻标题"></el-input>
            </el-form-item>
            <el-form-item label="作者" prop="author" v-show="false">
              <el-input size="medium" v-model="form.author" placeholder="请填写作者"></el-input>
            </el-form-item>
            <el-form-item label="封面" prop="thumbnail">
              <el-input size="medium" v-model="form.thumbnail" placeholder="请填写封面地址"></el-input>
            </el-form-item>
            <el-form-item label="简介" prop="summary">
              <el-input size="medium" type="textarea" :rows="4" placeholder="请输入简介" v-model="form.summary">
              </el-input>
            </el-form-item>
            <el-form-item label="新闻内容" prop="content">
              <tinymce
                :defaultContent="form.content"
                @change="tinymceChange"
                upload_url="http://dev.lin.colorful3.com/cms/file/"
              />
            </el-form-item>
            <el-form-item class="submit">
              <el-button type="primary" @click="submitForm('form')">保 存</el-button>
              <el-button @click="resetForm('form')">重 置</el-button>
            </el-form-item>
          </el-form>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
import article from '@/model/article'
import Tinymce from '@/component/base/tinymce'

export default {
  components: {
    Tinymce,
  },
  props: {
    editArticleID: {
      type: Number,
    },
  },
  data() {
    return {
      loading: false,
      form: {
        title: '',
        author: '',
        summary: '',
        thumbnail: '',
        content: '',
      },
    }
  },
  async mounted() {
    this.loading = true
    this.form = await article.getArticle(this.editArticleID)
    this.loading = false
  },
  methods: {
    async submitForm() {
      const res = await article.editArticle(this.editArticleID, this.form)
      if (res.code < window.MAX_SUCCESS_CODE) {
        this.$message.success(`${res.message}`)
        this.$emit('editClose')
      }
    },
    // 重置表单
    resetForm(formName) {
      this.$refs[formName].resetFields()
    },
    back() {
      this.$emit('editClose')
    },
    tinymceChange(val) {
      this.form.content = val
    },
  },
}
</script>

<style lang="scss" scoped>
.el-divider--horizontal {
  margin: 0;
}

.container {
  .title {
    height: 59px;
    line-height: 59px;
    color: $parent-title-color;
    font-size: 16px;
    font-weight: 500;
    text-indent: 40px;

    .back {
      float: right;
      margin-right: 40px;
      cursor: pointer;
    }
  }

  .wrap {
    padding: 20px;
  }

  .submit {
    float: left;
  }
}
</style>
