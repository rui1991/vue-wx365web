<template>
  <div
    class="module-container"
    v-loading="loading"
    element-loading-text="拼命加载中"
    element-loading-spinner="el-icon-loading"
    element-loading-background="rgba(0, 0, 0, 0.6)">
    <div class="module-header">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>报表管理</el-breadcrumb-item>
        <el-breadcrumb-item>巡检地址信息报表</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="module-main">
      <div class="main-search main-search-multi">
        <div class="search-row">
          <div class="item">
            <span>地址名称</span>
            <el-input style="width: 160px;" v-model.trim="nowSearch.name"></el-input>
          </div>
          <div class="item">
            <span>任务名称</span>
            <el-input style="width: 160px;" v-model.trim="nowSearch.task"></el-input>
          </div>
          <div class="operate">
            <el-button type="primary" @click="searchList">搜索</el-button>
          </div>
        </div>
        <div class="search-row">
          <div class="item date">
            <span>选择时段</span>
            <el-date-picker
              style="width: 280px;"
              v-model="nowSearch.date"
              type="daterange"
              value-format="yyyy-MM-dd"
              :clearable="false"
              :picker-options="pickerOptions"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期">
            </el-date-picker>
          </div>
          <div class="operate">
            <el-button type="primary" :disabled="downDisabled" @click="downFile">导出</el-button>
          </div>
        </div>
      </div>
      <el-table class="list-table" :data="tableData" border :summary-method="getSummaries" show-summary style="width: 100%">
        <el-table-column fixed type="index" width="50" label="序号"></el-table-column>
        <el-table-column fixed prop="project_name" label="项目名称" width="200"></el-table-column>
        <el-table-column prop="position_name" label="地址名称" width="200"></el-table-column>
        <el-table-column prop="plan_name" label="任务名称" width="200"></el-table-column>
        <el-table-column prop="start_time" label="开始时间"></el-table-column>
        <el-table-column prop="end_time" label="结束时间"></el-table-column>
        <el-table-column prop="normalSize" label="正常"></el-table-column>
        <el-table-column prop="abnormalSize" label="异常"></el-table-column>
      </el-table>
      <el-pagination
        background
        prev-text="上一页"
        next-text="下一页"
        :current-page="nowPage"
        layout="sizes, prev, pager, next, total"
        :page-sizes="[10, 20, 50, 100, 200, 500, 1000]"
        :page-size="limit"
        @size-change="handleSizeChange"
        @current-change="pageChang"
        :total="total">
      </el-pagination>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
export default{
  name: 'reportSite',
  data () {
    return {
      search: {
        date: [],
        name: '',
        task: ''
      },
      nowSearch: {
        date: [],
        name: '',
        task: ''
      },
      pickerOptions: {
        disabledDate (time) {
          return time.getTime() > Date.now()
        }
      },
      tableData: [],
      total: 0,
      nowPage: 1,
      limit: 10,
      detData: [],
      downDisabled: false,
      loading: false
    }
  },
  created () {

  },
  mounted () {
    if (!this.modVisit) {
      this.$router.go(-1)
      return
    }
    // 时段
    const nowDate = this.$common.getNowDate('yyyy-mm-dd')
    this.search.date = [nowDate, nowDate]
    this.nowSearch.date = [nowDate, nowDate]
    // 获取列表数据
    this.getListData()
  },
  computed: {
    ...mapState('user', [
      'userId'
    ]),
    ...mapState('user', {
      modVisit: state => state.modAuthority.reportSite
    }),
    ...mapState('report', [
      'date'
    ]),
    ...mapState('other', [
      'projectOrgId'
    ])
  },
  methods: {
    // 搜索
    searchList () {
      this.search = JSON.parse(JSON.stringify(this.nowSearch))
      // 当前页码初始化
      this.nowPage = 1
      // 获取列表数据
      this.getListData()
    },
    // 获取列表数据
    getListData () {
      if (!this.projectOrgId) return
      let date = this.search.date
      let params = {
        organize_id: this.projectOrgId,
        project_name: '',
        position_name: this.search.name,
        plan_name: this.search.task,
        start_date: date[0],
        end_date: date[1],
        page: this.nowPage,
        limit1: this.limit
      }
      params = this.$qs.stringify(params)
      this.loading = true
      this.$axios({
        method: 'post',
        url: this.reportApi() + '/v3.4/selInspectPosition',
        data: params
      }).then((res) => {
        this.loading = false
        if (res.data.result === 'Sucess') {
          this.total = res.data.data1.total
          this.tableData = res.data.data1.position
        } else {
          const errHint = this.$common.errorCodeHint(res.data.error_code)
          this.$message({
            showClose: true,
            message: errHint,
            type: 'error'
          })
        }
      }).catch(() => {
        this.loading = false
        this.$message({
          showClose: true,
          message: '服务器连接失败！',
          type: 'error'
        })
      })
    },
    // 合计表格规则设置
    getSummaries (param) {
      const { columns, data } = param
      const sums = []
      columns.forEach((column, index) => {
        if (index === 0) {
          sums[index] = '合计'
          return
        }
        if (index === 1 || index === 2 || index === 3 || index === 4 || index === 5) {
          sums[index] = '-'
          return
        }
        const vals = data.map(item => Number(item[column.property]))
        if (!vals.every(value => isNaN(value))) {
          sums[index] = vals.reduce((prev, curr) => {
            const value = Number(curr)
            if (!isNaN(value)) {
              return prev + curr
            } else {
              return prev
            }
          })
        } else {
          sums[index] = ''
        }
      })
      return sums
    },
    // 切换单页大小
    handleSizeChange (limit) {
      // 设置大小
      this.limit = limit
      // 初始化页码
      this.nowPage = 1
      // 获取列表数据
      this.getListData()
    },
    // 点击分页
    pageChang (page) {
      this.nowPage = page
      // 获取列表数据
      this.getListData()
    },
    /* 导出文件 */
    downFile () {
      const date = this.search.date
      let params = {
        organize_id: this.projectOrgId,
        project_name: '',
        position_name: this.search.name,
        plan_name: this.search.task,
        start_date: date[0],
        end_date: date[1]
      }
      params = this.$qs.stringify(params)
      this.downDisabled = true
      setTimeout(() => {
        this.downDisabled = false
      }, 5000)
      window.location.href = this.reportApi() + '/v3.4/selInspectPositionEO?' + params
    }
  }
}
</script>

<style lang="less" scoped>
  @import '../../assets/css/base-column.css';
</style>
