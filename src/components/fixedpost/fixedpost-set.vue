<template>
  <div class="module-container">
    <div class="module-header">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>固定岗管理</el-breadcrumb-item>
        <el-breadcrumb-item><router-link to="/main/fixedpost-rep">固定岗打卡报表</router-link></el-breadcrumb-item>
        <el-breadcrumb-item>固定岗设置</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="module-main">
      <el-table class="list-table" :data="tableData" border style="width: 100%">
        <el-table-column type="index" width="50" label="序号"></el-table-column>
        <el-table-column label="岗位地址" width="160">
          <template slot-scope="scope">
            <a href="javascript:void(0);" class="name" @click="detClick(scope.row.message)">{{ scope.row.position_name }}</a>
          </template>
        </el-table-column>
        <el-table-column label="部门" width="120" prop="ogz_name" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column label="人员" prop="user_names" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column label="时间段" :show-overflow-tooltip="true">
          <template slot-scope="scope">
            <span>{{ scope.row.message | filterTimeFrame }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="160">
          <template slot-scope="scope">
            <a href="javascript:void(0);" class="operate com" @click="comClick(scope.row)">设置</a>
            <a href="javascript:void(0);" class="operate del" @click="delClick(scope.row.position_id)">清除</a>
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
      :parentTimes="times"
      @parentClose="detClose">
    </det-module>
    <!-- 编辑 -->
    <com-module
      :parentDialog="comDialog"
      :parentId="itemId"
      :parentForm="itemForm"
      :parentTimes="times"
      @parentUpdata="comUpdata"
      @parentCancel="comCancel">
    </com-module>
    <!-- 删除 -->
    <del-module
      :parentDialog="delDialog"
      :parentId="itemId"
      :parentTimes="times"
      @parentUpdata="delUpdata"
      @parentCancel="delCancel">
    </del-module>

  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex'
// 引入详情组件
import detModule from '@/components/fixedpost/fixedpost-det2'
// 引入编辑组件
import comModule from '@/components/fixedpost/fixedpost-com'
// 引入删除组件
import delModule from '@/components/fixedpost/fixedpost-del'
export default{
  name: 'fixedpostSet',
  data () {
    return {
      tableData: [],
      total: 0,
      nowPage: 1,
      limit: 10,
      itemId: '',
      itemForm: {
        sector: '',
        crewName: '',
        crewId: []
      },
      times: '',
      detDialog: false,
      comDialog: false,
      delDialog: false
    }
  },
  created () {
    if (this.authority.indexOf(83) === -1) {
      this.$router.go(-1)
      return
    }
    // 设置全局项目禁用
    this.setProDisabled(true)
    // 查询列表数据
    this.getListData()
  },
  components: {
    detModule,
    comModule,
    delModule
  },
  computed: {
    ...mapState('user', [
      'userId'
    ]),
    ...mapState('user', {
      authority: state => state.detAuthority.fixedpostRep
    }),
    ...mapState('other', [
      'companyId',
      'projectId'
    ])
  },
  methods: {
    ...mapActions('other', [
      'setProDisabled'
    ]),
    // 获取列表数据
    getListData () {
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        project_id: this.projectId,
        page: this.nowPage,
        limit1: this.limit
      }
      params = this.$qs.stringify(params)
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/v1.0/selPermanentSet',
        data: params
      }).then((res) => {
        if (res.data.result === 'Sucess') {
          this.total = res.data.data1.total
          this.tableData = res.data.data1.sp
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
    detClick (timeStr) {
      let times = []
      if (timeStr) {
        times = JSON.parse(timeStr)
      }
      this.times = times
      this.detDialog = true
    },
    detClose () {
      this.detDialog = false
    },
    /* 设置 */
    comClick (data) {
      // 地址id
      this.itemId = data.position_id
      // 人员
      let names = data.user_names
      let crewName = ''
      if (names) {
        crewName = names.replace(/,/g, '、')
      }
      let uids = []
      if (data.user_ids) {
        uids = data.user_ids.split(',')
      }
      let crewId = uids.map(item => {
        return Number.parseInt(item)
      })
      this.itemForm = {
        sector: data.ogz_id || '',
        crewName: crewName,
        crewId: crewId
      }
      // 时段
      let times = []
      const timeStr = data.message
      if (timeStr) {
        times = JSON.parse(timeStr)
      }
      this.times = times
      this.comDialog = true
    },
    comUpdata () {
      this.comDialog = false
      // 更新列表
      this.getListData()
    },
    comCancel () {
      this.comDialog = false
    },
    /* 删除 */
    delClick (id) {
      this.itemId = id
      this.delDialog = true
    },
    delUpdata () {
      this.delDialog = false
      // 更新列表
      this.getListData()
    },
    delCancel () {
      this.delDialog = false
    }
  },
  filters: {
    filterTimeFrame (str) {
      if (!str) { return '' }
      const timeData = JSON.parse(str)
      let timeStr = ''
      timeData.forEach(item => {
        timeStr += item.time + '，' + item.frequency + '次； '
      })
      return timeStr
    }
  },
  beforeDestroy () {
    // 设置全局项目禁用
    this.setProDisabled(false)
  }
}
</script>

<style lang="less" scoped>
  @import '../../assets/css/base-column.css';
</style>
