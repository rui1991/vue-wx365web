<template>
  <div class="module-container">
    <div class="module-header">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item>定位服务</el-breadcrumb-item>
        <el-breadcrumb-item>车辆管理</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="module-main">
      <div class="main-search main-search-multi">
        <div class="search-row">
          <div class="item">
            <span>车牌号</span>
            <el-input style="width: 160px;" v-model.trim="nowSearch.mark"></el-input>
          </div>
          <div class="item date">
            <span>所属部门</span>
            <el-select v-model="nowSearch.sector" style="width: 160px;" clearable placeholder="请选择所属部门">
              <el-option
                v-for="item in sectorOptions"
                :key="item.base_id"
                :label="item.name"
                :value="item.base_id">
              </el-option>
            </el-select>
          </div>
          <div class="operate">
            <el-button type="primary" @click="searchList">搜索</el-button>
            <el-button type="primary" @click="addDialog = true" v-if="authority.indexOf(175) !== -1">新增</el-button>
          </div>
        </div>
        <div class="search-row">
          <div class="item date">
            <span>车辆类型</span>
            <el-select v-model="nowSearch.type" style="width: 160px;" clearable placeholder="请选择车辆类型">
              <el-option
                v-for="item in typeOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          </div>
          <div class="operate">
            <el-button type="primary" @click="upDialog = true" v-if="authority.indexOf(175) !== -1">导入</el-button>
            <el-button type="primary" @click="alarmDialog = true" v-if="authority.indexOf(178) !== -1">告警推送人</el-button>
          </div>
        </div>
      </div>
      <el-table class="list-table" :data="tableData" border style="width: 100%">
        <el-table-column type="index" width="50" label="序号"></el-table-column>
        <el-table-column :show-overflow-tooltip="true" prop="car_number" label="车牌号"></el-table-column>
        <el-table-column width="120" label="车种">
          <template slot-scope="scope">
            <span>{{ scope.row.car_type | filterType }}</span>
          </template>
        </el-table-column>
        <el-table-column :show-overflow-tooltip="true" prop="gps_number" label="设备号"></el-table-column>
        <el-table-column width="180" :show-overflow-tooltip="true" prop="ogz_name" label="所属部门"></el-table-column>
        <el-table-column width="240" :show-overflow-tooltip="true" prop="person_user_name" label="负责人"></el-table-column>
        <el-table-column label="定位频率">
          <template slot-scope="scope">
            <span v-if="scope.row.fqcy">{{ scope.row.fqcy }}秒</span>
            <span v-else>60秒</span>
          </template>
        </el-table-column>
        <el-table-column width="320" label="操作">
          <template slot-scope="scope">
            <a href="javascript:void(0);" class="operate blue" @click="trackClick(scope.row.gps_number)" v-if="authority.indexOf(179) !== -1">轨迹</a>
            <a href="javascript:void(0);" class="operate blue" @click="freqClick(scope.row.gps_number, scope.row.fqcy)" v-if="companyId === 1">定位频率</a>
            <a href="javascript:void(0);" class="operate blue" @click="comClick(scope.row)" v-if="authority.indexOf(176) !== -1">编辑</a>
            <a href="javascript:void(0);" class="operate red" @click="delClick(scope.row.id, scope.row.car_number)" v-if="authority.indexOf(177) !== -1">删除</a>
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
    <!-- 新增 -->
    <add-module
      :parentDialog="addDialog"
      :parentSectorOptions="sectorOptions"
      @parentUpdata="addUpdata"
      @parentCancel="addCancel">
    </add-module>
    <!-- 编辑 -->
    <com-module
      :parentDialog="comDialog"
      :parentSectorOptions="sectorOptions"
      :parentId="itemId"
      :parentForm="formData"
      @parentUpdata="comUpdata"
      @parentCancel="comCancel">
    </com-module>
    <!-- 删除 -->
    <del-module
      :parentDialog="delDialog"
      :parentId="itemId"
      :parentCarNumber="carNumber"
      @parentUpdata="delUpdata"
      @parentCancel="delCancel">
    </del-module>
    <!-- 轨迹 -->
    <track-module
      :parentDialog="trackDialog"
      :parentDeviceNum="itemDeviceNum"
      @parentClose="trackClose">
    </track-module>
    <!-- 定位频率 -->
    <freq-module
      :parentDialog="freqDialog"
      :parentGpsType="'car'"
      :parentDeviceNum="itemDeviceNum"
      :parentFreqNum="itemFreqNum"
      @parentUpdata="freqUpdata"
      @parentCancel="freqCancel">
    </freq-module>
    <!-- 告警推送人 -->
    <alarm-module
      :parentDialog="alarmDialog"
      @parentClose="alarmClose">
    </alarm-module>
    <!-- 导入 -->
    <up-module
      :parentDialog="upDialog"
      @parentUpdata="upUpdata"
      @parentClose="upClose">
    </up-module>
  </div>
</template>

<script>
import { mapState } from 'vuex'
// 引入新增组件
import addModule from '@/components/location/vehicle-admin-add'
// 引入编辑组件
import comModule from '@/components/location/vehicle-admin-com'
// 引入删除组件
import delModule from '@/components/location/vehicle-admin-del'
// 引入轨迹组件
import trackModule from '@/components/location/vehicle-admin-track'
// 引入定位频率组件
import freqModule from '@/components/location/gps-admin-freq'
// 引入告警推送人组件
import alarmModule from '@/components/location/vehicle-admin-alarm'
// 引入导入组件
import upModule from '@/components/location/vehicle-admin-up'
export default{
  name: 'vehicleAdmin',
  data () {
    return {
      search: {
        mark: '',
        sector: '',
        type: ''
      },
      nowSearch: {
        mark: '',
        sector: '',
        type: ''
      },
      sectorOptions: [],
      typeOptions: [
        {
          label: '运输车',
          value: 1
        },
        {
          label: '洒水车',
          value: 2
        },
        {
          label: '扫地车',
          value: 3
        },
        {
          label: '压缩车',
          value: 4
        },
        {
          label: '环卫车',
          value: 5
        },
        {
          label: '垃圾车',
          value: 6
        },
        {
          label: '巡逻车',
          value: 7
        }
      ],
      tableData: [],
      total: 0,
      nowPage: 1,
      limit: 10,
      addDialog: false,
      comDialog: false,
      itemId: 0,
      carNumber: '',
      formData: {
        mark: '',
        type: '',
        sector: '',
        crewName: '',
        crewId: [],
        deviceNum: '',
        remark: ''
      },
      delDialog: false,
      trackDialog: false,
      freqDialog: false,
      itemDeviceNum: '',
      itemFreqNum: 60,
      alarmDialog: false,
      upDialog: false
    }
  },
  created () {

  },
  mounted () {
    if (!this.modVisit) {
      this.$router.go(-1)
      return
    }
    // 获取列表数据
    this.getListData()
    // 获取部门
    this.getSectorOptions()
  },
  components: {
    addModule,
    comModule,
    delModule,
    trackModule,
    freqModule,
    alarmModule,
    upModule
  },
  computed: {
    ...mapState('user', [
      'companyId',
      'userId'
    ]),
    ...mapState('user', {
      modVisit: state => state.modAuthority.vehicleAdmin,
      authority: state => state.detAuthority.vehicleAdmin
    }),
    ...mapState('other', [
      'projectId',
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
      let params = {
        project_id: this.projectId,
        car_number: this.search.mark,
        ogz_id: this.search.sector,
        car_type: this.search.type,
        page: this.nowPage,
        limit1: this.limit
      }
      params = this.$qs.stringify(params)
      this.$axios({
        method: 'post',
        url: this.gpsApi() + '/selGpsCarMes',
        data: params
      }).then((res) => {
        if (res.data.result === 'Sucess') {
          this.total = res.data.data1.total
          this.tableData = res.data.data1.mes
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
    /* 新增 */
    addUpdata () {
      this.addDialog = false
      // 更新列表
      this.getListData()
    },
    addCancel () {
      this.addDialog = false
    },
    /* 编辑 */
    comClick (data) {
      this.itemId = data.id
      const sids = data.person_user_id
      let aids = []
      if (sids) {
        aids = sids.split(',')
        aids = aids.map(item => {
          return Number.parseInt(item)
        })
      }
      this.formData = {
        mark: data.car_number,
        type: data.car_type,
        sector: data.ogz_id,
        crewName: data.person_user_name,
        crewId: aids,
        deviceNum: data.gps_number,
        remark: data.remarks || ''
      }
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
    delClick (id, num) {
      this.itemId = id
      this.carNumber = num
      this.delDialog = true
    },
    delUpdata () {
      this.delDialog = false
      // 更新列表
      this.getListData()
    },
    delCancel () {
      this.delDialog = false
    },
    /* 足迹 */
    trackClick (num) {
      this.itemDeviceNum = num
      this.trackDialog = true
    },
    trackClose () {
      this.trackDialog = false
    },
    /* 定位频率 */
    freqClick (deviceNum, freqNum) {
      this.itemDeviceNum = deviceNum
      this.itemFreqNum = freqNum
      this.freqDialog = true
    },
    freqUpdata () {
      this.freqDialog = false
      // 更新列表
      this.getListData()
    },
    freqCancel () {
      this.freqDialog = false
    },
    /* 告警推送人 */
    alarmClose () {
      this.alarmDialog = false
    },
    /* 导入 */
    upUpdata () {
      this.upDialog = false
      // 更新列表
      this.getListData()
    },
    upClose () {
      this.upDialog = false
    },
    /* 部门 */
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
    }
  },
  filters: {
    filterType (val) {
      if (!val) return ''
      let typeName = ''
      switch (val) {
        case 1:
          typeName = '运输车'
          break
        case 2:
          typeName = '洒水车'
          break
        case 3:
          typeName = '扫地车'
          break
        case 4:
          typeName = '压缩车'
          break
        case 5:
          typeName = '环卫车'
          break
        case 6:
          typeName = '垃圾车'
          break
        case 7:
          typeName = '巡逻车'
          break
      }
      return typeName
    }
  }
}
</script>

<style lang="less" scoped>
  @import '../../assets/css/base-column.css';
</style>
