<template>
  <div>
    <!-- 列表页面 -->
    <div class="container" v-if="!showEdit">
      <div class="header">
        <div class="title">新闻列表</div>
        <div>
          <el-button @click="exprotExcel">导出</el-button>
        </div>
      </div>
      <!-- 表格 -->
      <lin-table
        :tableColumn="tableColumn"
        :tableData="tableData"
        :operate="operate"
        @handleEdit="handleEdit"
        @handleDelete="handleDelete"
        @row-click="rowClick"
        v-loading="loading"
      ></lin-table>
    </div>

    <!-- 编辑页面 -->
    <article-modify v-else @editClose="editClose" :editArticleID="editArticleID"></article-modify>
  </div>
</template>

<script>
import article from '@/model/article'
import LinTable from '@/component/base/table/lin-table'
import ArticleModify from './article-modify'
import ParseTime from '@/lin/util/parse-time'

export default {
  components: {
    LinTable,
    ArticleModify,
  },
  data() {
    return {
      tableColumn: [{ prop: 'title', label: '新闻标题' }],
      tableData: [],
      operate: [],
      showEdit: false,
      editArticleID: 1,
    }
  },
  async created() {
    this.loading = true
    await this.getArticles()
    this.operate = [
      { name: '编辑', func: 'handleEdit', type: 'primary' },
      {
        name: '删除',
        func: 'handleDelete',
        type: 'danger',
        permission: '删除新闻',
      },
    ]
    this.loading = false
  },
  methods: {
    async getArticles() {
      try {
        const articles = await article.getArticles()
        this.tableData = articles
      } catch (error) {
        if (error.code === 10020) {
          this.tableData = []
        }
      }
    },
    handleEdit(val) {
      console.log('val', val)
      this.showEdit = true
      this.editArticleID = val.row.id
    },
    handleDelete(val) {
      this.$confirm('此操作将永久删除该新闻, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(async () => {
        const res = await article.deleteArticle(val.row.id)
        if (res.code < window.MAX_SUCCESS_CODE) {
          this.getArticles()
          this.$message({
            type: 'success',
            message: `${res.message}`,
          })
        }
      })
    },
    rowClick() {},
    editClose() {
      this.showEdit = false
      this.getArticles()
    },
    // 导出表格
    exprotExcel() {
      // 动态导入
      import('@/lin/util/export-excel').then(excel => {
        const tHeader = ['timestamp', 'title', 'label', 'importance', 'status']
        const filterVal = ['timestamp', 'title', 'label', 'importance', 'status']
        const data = this.formatJson(filterVal)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list',
        })
      })
    },
    // 将表单格式化为json数据
    formatJson(filterVal) {
      return this.tableData.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return ParseTime(v[j])
        }
        return v[j]
      }),)
    },
  },
}
</script>

<style lang="scss" scoped>
.container {
  padding: 0 30px;

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;

    .title {
      height: 59px;
      line-height: 59px;
      color: $parent-title-color;
      font-size: 16px;
      font-weight: 500;
    }
  }

  .pagination {
    display: flex;
    justify-content: flex-end;
    margin: 20px;
  }
}
</style>
