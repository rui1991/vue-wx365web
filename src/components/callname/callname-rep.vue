<template>
  <div
    class="module-container"
    v-loading="loading"
    element-loading-text="拼命加载中"
    element-loading-spinner="el-icon-loading"
    element-loading-background="rgba(0, 0, 0, 0.6)">
    <div class="module-header">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>点名管理</el-breadcrumb-item>
        <el-breadcrumb-item>点名报表</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="module-main">
      <div class="main-search main-search-multi">
        <div class="search-row">
          <div class="item">
            <span>执行部门</span>
            <el-select v-model="nowSearch.sector" style="width: 160px;" clearable filterable placeholder="请选择执行部门">
              <el-option
                v-for="item in sectorOptions"
                :key="item.id"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
          </div>
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
            <el-button type="primary" @click="searchList">搜索</el-button>
            <!--<el-button type="primary" @click="setClick">设置</el-button>-->
          </div>
        </div>
        <div class="search-row">
          <div class="item">
            <span>人员名称</span>
            <el-input style="width: 160px;" v-model.trim="nowSearch.name"></el-input>
          </div>
          <div class="item">
            <span>点名结果</span>
            <el-select v-model="nowSearch.result" clearable style="width: 160px;" placeholder="请选择点名结果">
              <el-option
                v-for="item in resultOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          </div>
          <div class="operate">
            <el-button type="primary" :disabled="downDisabled" @click="downFile" v-if="authority.indexOf(97) !== -1">导出</el-button>
            <el-button type="primary" :disabled="downAllDisabled" @click="downAllFile" v-if="authority.indexOf(97) !== -1">导出汇总</el-button>
          </div>
        </div>
      </div>
      <el-table class="list-table" :data="tableData" border style="width: 100%">
        <el-table-column type="index" width="50" label="序号"></el-table-column>
        <el-table-column prop="user_name" width="150" label="姓名" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="adate" width="180" label="日期"></el-table-column>
        <el-table-column :show-overflow-tooltip="true" label="点名地址">
          <template slot-scope="scope">
            <span v-if="scope.row.positions">{{ scope.row.positions }}</span>
            <span v-else>全部</span>
          </template>
        </el-table-column>
        <el-table-column width="150" label="点名次数">
          <template slot-scope="scope">
            <span v-if="scope.row.sf === 0">0</span>
            <a href="javascript:void(0);" class="blue" @click="detClick(scope.row)" v-else>{{ scope.row.sf }}</a>
          </template>
        </el-table-column>
        <el-table-column width="150" label="点名结果">
          <template slot-scope="scope">
            <span v-if="scope.row.over === 1">成功</span>
            <span class="red" v-else-if="scope.row.over === 0">失败</span>
          </template>
        </el-table-column>
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
    <!-- 详情 -->
    <det-module
      :parentDialog="detDialog"
      :parentUid="uid"
      :parentDate="itemDate"
      @parentClose="detClose">
    </det-module>
  </div>
</template>

<script>
import { mapState } from 'vuex'
// 引入详情组件
import detModule from '@/components/callname/callname-det'
export default{
  name: 'callnameRep',
  data () {
    return {
      search: {
        sector: '',
        date: [],
        name: '',
        result: ''
      },
      nowSearch: {
        sector: '',
        date: [],
        name: '',
        result: ''
      },
      pickerOptions: {
        disabledDate (time) {
          return time.getTime() > Date.now() - 8.64e7
        }
      },
      sectorOptions: [],
      orgids: 0,
      resultOptions: [
        {
          label: '成功',
          value: 1
        },
        {
          label: '失败',
          value: 0
        }
      ],
      tableData: [],
      loading: false,
      total: 0,
      nowPage: 1,
      limit: 10,
      detDialog: false,
      uid: 0,
      itemDate: '',
      downDisabled: false,
      downAllDisabled: false
    }
  },
  created () {
    if (!this.modVisit) {
      this.$router.go(-1)
      return
    }
    const nowDate = this.$common.getBeforeDate()
    this.search.date = [nowDate, nowDate]
    this.nowSearch.date = [nowDate, nowDate]
    // 全部项目
    const allProject = this.allProject
    const projectId = this.projectId
    const nowProject = allProject.find(item => {
      return item.project_id === projectId
    })
    if (nowProject.ogzs === undefined) {
      // 获取项目所有部门
      this.getProAllSector()
    } else {
      let sectorOptions = []
      let ids = []
      nowProject.ogzs.forEach(item => {
        sectorOptions.push({
          id: item.ogz_id,
          name: item.organize_name
        })
        ids.push(item.ogz_id)
      })
      ids = ids.join(',')
      this.orgids = ids
      this.sectorOptions = sectorOptions
      // 获取列表数据
      this.getListData()
    }
  },
  mounted () {

  },
  components: {
    detModule
  },
  computed: {
    ...mapState('user', [
      'userId'
    ]),
    ...mapState('user', {
      modVisit: state => state.modAuthority.callnameRep,
      authority: state => state.detAuthority.callnameRep
    }),
    ...mapState('other', [
      'companyId',
      'allProject',
      'projectId',
      'projectOrgId'
    ])
  },
  methods: {
    // 搜索
    searchList () {
      let date = this.search.date || []
      const startDate = date[0] || ''
      const endDate = date[1] || ''
      const fate = this.getDateDiff(startDate, endDate)
      if (fate) {
        this.$message({
          showClose: true,
          message: '查询时长不能超过31天',
          type: 'warning'
        })
        return
      }
      this.search = JSON.parse(JSON.stringify(this.nowSearch))
      // 当前页码初始化
      this.nowPage = 1
      // 获取列表数据
      this.getListData()
    },
    // 获取列表数据
    getListData () {
      let date = this.search.date || []
      // 部门ID
      const orgids = this.orgids
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        project_id: this.projectId,
        ogz_ids: this.search.sector || orgids,
        start_date: date[0] || '',
        end_date: date[1] || '',
        user_name: this.search.name,
        over: this.search.result,
        page: this.nowPage,
        limit1: this.limit
      }
      params = this.$qs.stringify(params)
      this.loading = true
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/v2.0/selRollCallResult',
        data: params
      }).then((res) => {
        this.loading = false
        if (res.data.result === 'Sucess') {
          this.total = res.data.data1.total
          this.tableData = res.data.data1.rcs
          // 检验是否列表为空
          if (this.tableData.length === 0 && this.nowPage > 1) {
            this.nowPage--
            this.getListData()
          }
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
    /* 详情 */
    detClick (data) {
      this.uid = data.user_id
      this.itemDate = data.adate
      this.detDialog = true
    },
    detClose () {
      this.detDialog = false
    },
    // 获取跨越天数
    getDateDiff (startDate, endDate) {
      let startTime = new Date(Date.parse(startDate.replace(/-/g, '/'))).getTime()
      let endTime = new Date(Date.parse(endDate.replace(/-/g, '/'))).getTime()
      const seaDuration = endTime - startTime
      const maxDuration = 1000 * 60 * 60 * 24 * 31
      if (seaDuration > maxDuration) {
        return true
      } else {
        return false
      }
    },
    /* 项目所有部门 */
    getProAllSector () {
      let params = {
        organize_id: this.projectOrgId
      }
      params = this.$qs.stringify(params)
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/v3.2/selOrganizeTree',
        data: params
      }).then((res) => {
        if (res.data.result === 'Sucess') {
          const nodeData = res.data.data1[0].children || []
          let sectorOptions = []
          let ids = []
          nodeData.forEach(item => {
            ids.push(item.base_id)
            sectorOptions.push({
              id: item.base_id,
              name: item.name
            })
          })
          ids = ids.join(',')
          this.orgids = ids
          this.sectorOptions = sectorOptions
          // 获取列表数据
          this.getListData()
        } else {
          const errHint = this.$common.errorCodeHint(res.data.error_code)
          this.$message({
            showClose: true,
            message: errHint,
            type: 'error'
          })
        }
      }).catch(() => {
        this.$message({
          showClose: true,
          message: '服务器连接失败！',
          type: 'error'
        })
      })
    },
    /* 导出 */
    downFile () {
      let date = this.search.date || []
      const orgids = this.orgids
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        project_id: this.projectId,
        ogz_ids: this.search.sector || orgids,
        start_date: date[0] || '',
        end_date: date[1] || '',
        user_name: this.search.name,
        over: this.search.result
      }
      params = this.$qs.stringify(params)
      this.downDisabled = true
      setTimeout(() => {
        this.downDisabled = false
      }, 5000)
      window.location.href = this.sysetApi() + '/v2.0/rollCallResultEO?' + params
    },
    downAllFile () {
      let date = this.search.date || []
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        project_id: this.projectId,
        user_name: this.search.name,
        over: this.search.result,
        start_date: date[0] || '',
        end_date: date[1] || ''
      }
      params = this.$qs.stringify(params)
      this.downAllDisabled = true
      setTimeout(() => {
        this.downAllDisabled = false
      }, 5000)
      window.location.href = this.sysetApi() + '/v2.0/selRollCallReportSummaryEO?' + params
    },
    /* 点名次数 */
    /* 设置 */
    setClick () {
      this.$router.push({ path: '/main/callname-set' })
    }
  }
}
</script>

<style lang="less" scoped>
  @import '../../assets/css/base-column.css';
</style>
