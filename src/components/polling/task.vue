<template>
  <div
    class="module-container"
    v-loading="loading"
    element-loading-text="拼命加载中"
    element-loading-spinner="el-icon-loading"
    element-loading-background="rgba(0, 0, 0, 0.6)">
    <div class="module-header">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>巡检巡查</el-breadcrumb-item>
        <el-breadcrumb-item>巡检任务</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="module-main">
      <div class="main-search main-search-multi">
        <div class="search-row">
          <div class="item">
            <span>任务名称</span>
            <el-input style="width: 160px;" v-model.trim="nowSearch.name"></el-input>
          </div>
          <div class="item">
            <span>执行部门</span>
            <el-select v-model="nowSearch.sector" style="width: 160px;" clearable filterable placeholder="请选择执行部门">
              <el-option
                v-for="item in sectorOptions"
                :key="item.base_id"
                :label="item.name"
                :value="item.base_id">
              </el-option>
            </el-select>
          </div>
          <div class="item">
            <span>完成状态</span>
            <el-select v-model="nowSearch.state" clearable style="width: 160px;" placeholder="请选择完成状态">
              <el-option v-for="item in stateOptions" :key="item.value" :label="item.label" :value="item.value"></el-option>
            </el-select>
          </div>
          <div class="operate">
            <el-button type="primary" @click="searchList">搜索</el-button>
            <el-button type="primary" :disabled="downDisabled" v-if="authority.indexOf(67) !== -1" @click="downFile">导出</el-button>
          </div>
        </div>
        <div class="search-row">
          <div class="item">
            <span>执行人员</span>
            <el-select v-model="nowSearch.crews" style="width: 160px;" multiple collapse-tags placeholder="请选择执行人员">
              <el-option
                v-for="item in crewOptions"
                :key="item.user_id"
                :label="item.user_name"
                :value="item.user_id">
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
          <div class="operate" v-if="authority.indexOf(69) !== -1">
            <el-button type="primary" @click="pushDialog = true">推送设置</el-button>
          </div>
        </div>
      </div>
      <el-table class="list-table" :data="tableData" border style="width: 100%">
        <el-table-column type="index" width="50" label="序号"></el-table-column>
        <el-table-column label="任务名称" :show-overflow-tooltip="true">
          <template slot-scope="scope">
            <a href="javascript:void(0);" class="name" @click="detClick(scope.row.id_id)">{{ scope.row.plan_name }}</a>
          </template>
        </el-table-column>
        <el-table-column prop="po_size" width="80" label="点位数"></el-table-column>
        <el-table-column label="执行部门">
          <template slot-scope="scope">
            <span v-if="scope.row.ogz_id">{{ scope.row.ogz_name }}</span>
            <span v-else>-</span>
          </template>
        </el-table-column>
        <el-table-column width="200" label="开始时间">
          <template slot-scope="scope">
            <span>{{ scope.row.start_time | formatDate }}</span>
          </template>
        </el-table-column>
        <el-table-column width="200" label="结束时间">
          <template slot-scope="scope">
            <span>{{ scope.row.end_time | formatDate }}</span>
          </template>
        </el-table-column>
        <el-table-column label="已完成">
          <template slot-scope="scope">
            <span class="blue" v-if="scope.row.continue_size === 0">0</span>
            <el-popover
              v-else
              placement="top"
              title="已完成点位"
              trigger="click"
              :content="siteFinishContent">
              <a href="javascript:void(0);" slot="reference" class="blue" @click="detFinishClick(scope.row.id_id)">{{ scope.row.continue_size }}</a>
            </el-popover>
          </template>
        </el-table-column>
        <el-table-column label="未完成">
          <template slot-scope="scope">
            <span class="red" v-if="scope.row.not_size === 0">0</span>
            <el-popover
              v-else
              placement="top"
              title="未完成点位"
              trigger="click"
              :content="siteUndoneContent">
              <a href="javascript:void(0);" slot="reference" class="red" @click="detUndoneClick(scope.row.id_id)">{{ scope.row.not_size }}</a>
            </el-popover>
          </template>
        </el-table-column>
        <el-table-column label="执行组/执行人" :show-overflow-tooltip="true">
          <template slot-scope="scope">
            <el-popover
              placement="top"
              :title="scope.row.group_name"
              trigger="click"
              :content="groupContent"
              v-if="scope.row.group_id">
              <a href="javascript:void(0);" slot="reference" class="blue" @click="getGrouoCrew(scope.row.group_id)">{{ scope.row.group_name }}</a>
            </el-popover>
            <span v-else-if="scope.row.user_id">{{ scope.row.user_name }}</span>
            <span v-else>-</span>
          </template>
        </el-table-column>
        <el-table-column label="任务进度">
          <template slot-scope="scope">
            <span v-if="scope.row.continue_state === 1">未领取</span>
            <span v-else>已进行{{ scope.row.continue_process | formatPercent }}</span>
          </template>
        </el-table-column>
        <el-table-column width="80" label="是否及时完成">
          <template slot-scope="scope">
            <span v-if="scope.row.oo_time === 0">是</span>
            <span v-else-if="scope.row.oo_time === 1">否</span>
          </template>
        </el-table-column>
        <el-table-column label="审批状态">
          <template slot-scope="scope">
            <span v-if="scope.row.approval_state === 1">审批中</span>
            <span v-else-if="scope.row.approval_state === 2">通过</span>
            <span v-else-if="scope.row.approval_state === 3">不通过</span>
            <span v-else>-</span>
          </template>
        </el-table-column>
        <el-table-column width="260" label="操作">
          <template slot-scope="scope">
            <a href="javascript:void(0);" class="operate com" @click="drawClick(scope.row.id_id)" v-if="scope.row.draw === 1 && authority.indexOf(64) !== -1">领取</a>
            <span class="operate forbid" v-else-if="scope.row.draw === 2 && authority.indexOf(64) !== -1">领取</span>
            <span class="operate forbid" v-else-if="scope.row.draw === 3 && authority.indexOf(64) !== -1">已领取</span>
            <a href="javascript:void(0);" class="operate com" @click="dispatchClick(scope.row.id_id, scope.row.ogz_id)" v-if="scope.row.dispatch === 1 && authority.indexOf(63) !== -1">派遣</a>
            <span class="operate forbid" v-else-if="scope.row.dispatch === 2 && authority.indexOf(63) !== -1">派遣</span>
            <a href="javascript:void(0);" class="operate com" @click="tradeClick(scope.row.id_id, scope.row.ogz_id)" v-if="scope.row.trade === 1 && authority.indexOf(65) !== -1">换人</a>
            <span class="operate forbid" v-else-if="scope.row.trade === 2 && authority.indexOf(65) !== -1">换人</span>
            <a href="javascript:void(0);" class="operate com" @click="comClick(scope.row.id_id)" v-if="scope.row.com === 1 && authority.indexOf(66) !== -1">维护</a>
            <span class="operate forbid" v-if="scope.row.com === 2 && authority.indexOf(66) !== -1">维护</span>
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
      :parentPro="projectId"
      :parentId="itemId"
      @parentClose="detClose">
    </det-module>
    <!-- 人员 -->
    <crew-module
      :parentDialog="crewDialog"
      :parentId="itemId"
      :parentSector="itemSector"
      :parentType="disType"
      @parentUpdata="crewUpdata"
      @parentCancel="crewCancel">
    </crew-module>
    <!-- 推送设置 -->
    <push-module
      :parentDialog="pushDialog"
      :parentPro="projectId"
      :parentSector="sectorOptions"
      @parentClose="pushClose">
    </push-module>
  </div>
</template>

<script>
/*
* continue_state:
* */

import { mapState } from 'vuex'
// 引入详情组件
import detModule from '@/components/polling/task-det'
// 引入人员组件
import crewModule from '@/components/polling/task-crew'
// 引入人员组件
import pushModule from '@/components/polling/task-push'
export default{
  name: 'task',
  data () {
    return {
      search: {
        name: '',
        startDate: '',
        endDate: '',
        sector: '',
        state: '',
        crews: '0',
        date: []
      },
      nowSearch: {
        name: '',
        sector: '',
        state: '',
        crews: [],
        date: []
      },
      pickerOptions: {
        disabledDate (time) {
          return time.getTime() > Date.now()
        }
      },
      sectorOptions: [],
      stateOptions: [
        {
          label: '已完成',
          value: 0
        },
        {
          label: '未完成',
          value: 3
        }
      ],
      crewOptions: [],
      tableData: [],
      groupContent: '',
      siteFinishContent: '',
      siteUndoneContent: '',
      total: 0,
      nowPage: 1,
      limit: 10,
      todayStartTime: 0,
      itemId: '',
      detDialog: false,
      itemSector: '',
      disType: 1,
      crewDialog: false,
      downDisabled: false,
      loading: false,
      pushDialog: false
    }
  },
  created () {
    if (!this.modVisit) {
      this.$router.go(-1)
      return
    }
    // 获取当天日期
    const nowDate = this.$common.getNowDate('yyyy-mm-dd')
    this.search.date = [nowDate, nowDate]
    this.nowSearch.date = [nowDate, nowDate]
    // 今天开始毫秒
    let todayStartTime = nowDate + ' 00:00:00'
    this.todayStartTime = new Date(todayStartTime).getTime()
    // 获取列表数据
    this.getListData()
    // 获取部门
    this.getSectorOptions()
    // 获取项目人员
    this.getCrewOptions()
  },
  mounted () {

  },
  components: {
    detModule,
    crewModule,
    pushModule
  },
  computed: {
    ...mapState('user', [
      'userId',
      'userName',
      'sectorId'
    ]),
    ...mapState('user', {
      modVisit: state => state.modAuthority.task,
      authority: state => state.detAuthority.task
    }),
    ...mapState('other', [
      'companyId',
      'projectId',
      'projectOrgId'
    ])
  },
  methods: {
    // 搜索
    searchList () {
      let crews = this.nowSearch.crews
      if (crews.length > 0) {
        crews = crews.join(',')
      } else {
        crews = '0'
      }
      this.search = {
        name: this.nowSearch.name,
        sector: this.nowSearch.sector,
        state: this.nowSearch.state,
        crews: crews,
        date: this.nowSearch.date
      }
      // 当前页码初始化
      this.nowPage = 1
      // 获取列表数据
      this.getListData()
    },
    // 获取列表数据
    getListData () {
      let date = this.search.date || []
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        project_id: this.projectId,
        projectN_id: this.projectId,
        planN_name: this.search.name,
        startN_date: date[0] || '',
        endN_date: date[1] || '',
        ogz_id: this.search.sector,
        continueN_state: this.search.state,
        userN_id: this.search.crews,
        page: this.nowPage,
        limit1: this.limit
      }
      params = this.$qs.stringify(params)
      this.loading = true
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/inspection/v3.7.3/all/sel/selInsTaskSearch',
        data: params
      }).then((res) => {
        this.loading = false
        if (res.data.result === 'Sucess') {
          this.total = res.data.data1.total
          // 当前时间
          let nowTime = new Date().getTime()
          const nowSector = this.sectorId + ''
          let tableData = res.data.data1.insTask
          // 领取draw 1：可领取 2：不可领取 3：已领取
          // 派遣dispatch 1: 可派遣 2：不可派遣
          // 换人trade 1：可换人 2：不可换人
          // 维护com 1：可维护 2：不可维护
          tableData.forEach(item => {
            // 领取、派遣
            if (item.continue_state === 1) {
              if (item.start_time <= nowTime && item.end_time > this.todayStartTime) {
                if (nowSector === item.ogz_id || item.ogz_id === undefined) {
                  item.draw = 1
                } else {
                  item.draw = 2
                }
              } else {
                item.draw = 2
              }
              if (item.end_time >= this.todayStartTime) {
                item.dispatch = 1
              } else {
                item.dispatch = 2
              }
            } else {
              item.draw = 3
              item.dispatch = 2
            }
            // 换人
            if (item.continue_state === 0 || item.continue_state === 2) {
              if (!item.group_id) {
                if (item.end_time >= nowTime && parseFloat(item.continue_process) !== 1) {
                  item.trade = 1
                } else {
                  item.trade = 2
                }
              } else {
                item.trade = 2
              }
            } else {
              item.trade = 2
            }
            // 维护
            if (item.continue_state === 1 && item.end_time > nowTime) {
              item.com = 1
            } else {
              item.com = 2
            }
          })
          this.tableData = tableData
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
    /* 获取部门 */
    getSectorOptions () {
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
          this.sectorOptions = res.data.data1[0].children
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
    /* 项目人员 */
    getCrewOptions () {
      let params = {
        organize_id: this.projectOrgId,
        user_name: '',
        user_phone: '',
        role_id: '',
        page: 1,
        limit1: 10000
      }
      params = this.$qs.stringify(params)
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/v3.2/selUser',
        data: params
      }).then((res) => {
        if (res.data.result === 'Sucess') {
          let crewData = res.data.data1.users
          this.crewOptions = crewData
          this.ocrewOptions = crewData
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
    /* 获取组人员 */
    getGrouoCrew (id) {
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        group_id: id
      }
      params = this.$qs.stringify(params)
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/v2.6/selUserByGroupId',
        data: params
      }).then((res) => {
        if (res.data.result === 'Sucess') {
          const resData = res.data.data1
          let names = resData.map(item => {
            return item.user_name
          })
          this.groupContent = names.join('、')
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
    /* 详情 */
    detClick (id) {
      this.itemId = id
      this.detDialog = true
    },
    detClose () {
      this.detDialog = false
    },
    /* 领取 */
    drawClick (id) {
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        project_id: this.projectId,
        id_id: id,
        type: 0,
        user_name: this.userName
      }
      params = this.$qs.stringify(params)
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/inspection/reqInsTask',
        data: params
      }).then((res) => {
        if (res.data.result === 'Sucess') {
          this.$message({
            showClose: true,
            message: '领取成功',
            type: 'success'
          })
          // 刷新列表数据
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
    /* 人员 */
    // disType   1：派遣   2：换人
    // 派遣
    dispatchClick (id, sector) {
      // 类型
      this.disType = 1
      // id
      this.itemId = id
      // 部门
      this.itemSector = sector
      this.crewDialog = true
    },
    // 换人
    tradeClick (id, sector) {
      // 类型
      this.disType = 2
      // id
      this.itemId = id
      // 部门
      this.itemSector = sector
      this.crewDialog = true
    },
    // 更新
    crewUpdata () {
      this.crewDialog = false
      // 获取列表
      this.getListData()
    },
    // 取消
    crewCancel () {
      this.crewDialog = false
    },
    /* 导出 */
    downFile () {
      let date = this.search.date || []
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        project_id: this.projectId,
        projectN_id: this.projectId,
        planN_name: this.search.name,
        startN_date: date[0] || '',
        endN_date: date[1] || '',
        ogz_id: this.search.sector,
        continueN_state: this.search.state,
        userN_id: this.search.crews,
        page: 1,
        limit1: 1000
      }
      params = this.$qs.stringify(params)
      this.downDisabled = true
      setTimeout(() => {
        this.downDisabled = false
      }, 5000)
      window.location.href = this.sysetApi() + '/inspection/v2.1.02/pc/imp/inseximport?' + params
    },
    /* 维护 */
    comClick (id) {
      this.$router.push(
        {
          path: '/main/task-com',
          query: {
            id: id
          }
        }
      )
    },
    /* 地址详情 */
    // 已完成
    detFinishClick (id) {
      this.siteFinishContent = ''
      let params = {
        user_id: this.userId,
        id_id: id
      }
      params = this.$qs.stringify(params)
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/inspection/selInsTaskContinuePo',
        data: params
      }).then((res) => {
        if (res.data.result === 'Sucess') {
          const resData = res.data.data1.mes || []
          let names = resData.map(item => {
            return item.position_name
          })
          this.siteFinishContent = names.join('、')
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
    // 未完成
    detUndoneClick (id) {
      this.siteUndoneContent = ''
      let params = {
        user_id: this.userId,
        id_id: id
      }
      params = this.$qs.stringify(params)
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/inspection/selInsTaskNotContinuePo',
        data: params
      }).then((res) => {
        if (res.data.result === 'Sucess') {
          const resData = res.data.data1.mes || []
          let names = resData.map(item => {
            return item.position_name
          })
          this.siteUndoneContent = names.join('、')
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
    /* 推送设置 */
    pushClose () {
      this.pushDialog = false
    }
  },
  filters: {
    filterDate (str) {
      if (!str) { return '' }
      let value = new Date(str)
      let month = value.getMonth() + 1 + ''
      month = month.padStart(2, '0')
      let day = value.getDate() + ''
      day = day.padStart(2, '0')
      let hour = value.getHours() + ''
      hour = hour.padStart(2, '0')
      let minutes = value.getMinutes() + ''
      minutes = minutes.padStart(2, '0')
      return `${month}-${day} ${hour}:${minutes}`
    }
  }
}
</script>

<style lang="less" scoped>
  @import '../../assets/css/base-column.css';
</style>
