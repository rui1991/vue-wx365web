<template>
  <div>
    <el-dialog title="编辑组" :visible.sync="parentDialog" :show-close="false" :close-on-click-modal="false" custom-class="medium-dialog">
      <el-form class="divide-from" :model="formData" :rules="rules" ref="ruleForm" :label-width="formLabelWidth">
        <el-form-item label="组名称" prop="name">
          <el-input v-model.trim="formData.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="部门" prop="sector">
          <el-select v-model="formData.sector" :disabled="sectorDisabled" placeholder="请选择部门" @change="sectionChang">
            <el-option
              v-for="item in sectorOptions"
              :key="item.base_id"
              :label="item.name"
              :value="item.base_id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="人员" prop="crewName">
          <el-input :disabled="true" type="textarea" v-model="formData.crewName"></el-input>
          <el-button type="primary" :disabled="crewDisabled" @click="crewClick">选择人员</el-button>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancelClick">取 消</el-button>
        <el-button type="primary" :disabled="disabled" @click="submitForm('ruleForm')">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 人员 -->
    <crew-module
      :parentDialog="crewDialog"
      :parentOrgid="orgId"
      :parentIds="formData.crewId"
      @parentUpdata="crewUpdata"
      @parentCancel="crewCancel">
    </crew-module>
  </div>
</template>

<script>
import { mapState } from 'vuex'
// 引入人员组件
import crewModule from '@/components/public/crew-checkbox2'
export default{
  props: ['parentDialog', 'parentId', 'parentForm'],
  data () {
    return {
      formLabelWidth: '100px',
      sectorOptions: [],
      rules: {
        name: [
          { required: true, message: '请输入组名称', trigger: 'blur' }
        ],
        sector: [
          { required: true, message: '请选择部门', trigger: 'change' }
        ],
        crewName: [
          { required: true, message: '请选择组人员', trigger: 'change' }
        ]
      },
      formData: {
        name: '',
        sector: '',
        crewName: '',
        crewId: []
      },
      sectorDisabled: true,
      crewDisabled: true,
      disabled: false,
      orgId: 0,
      crewDialog: false
    }
  },
  components: {
    crewModule
  },
  computed: {
    ...mapState('user', [
      'userId'
    ]),
    ...mapState('other', [
      'companyId',
      'projectId',
      'projectOrgId'
    ])
  },
  methods: {
    // 初始化数据
    comInit () {
      this.formData = JSON.parse(JSON.stringify(this.parentForm))
      const sector = this.formData.sector
      if (sector) {
        this.sectorDisabled = true
      } else {
        this.sectorDisabled = false
      }
      if (this.sectorOptions.length === 0) {
        // 获取部门
        this.getSectorOptions()
      }
    },
    // 验证表单
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.sendRequest()
        } else {
          return false
        }
      })
    },
    // 重置表单
    resetForm (formName) {
      this.$refs[formName].resetFields()
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
    },
    // 选择部门
    sectionChang () {
      this.formData.crewName = ''
      this.formData.crewId = []
    },
    // 提交
    sendRequest () {
      let crewId = this.formData.crewId
      crewId = crewId.join(',')
      let sector = this.formData.sector
      let params = {
        company_id: this.companyId,
        user_id: this.userId,
        project_id: this.projectId,
        group_id: this.parentId,
        group_name: this.formData.name,
        ogz_id: sector,
        userN_ids: crewId
      }
      params = this.$qs.stringify(params)
      this.disabled = true
      this.$axios({
        method: 'post',
        url: this.sysetApi() + '/v2.6/altDutyGroup',
        data: params
      }).then((res) => {
        this.disabled = false
        if (res.data.result === 'Sucess') {
          // 重置表单
          this.resetForm('ruleForm')
          // 刷新列表
          this.$emit('parentUpdata')
        } else {
          const errHint = this.$common.errorCodeHint(res.data.error_code)
          this.$message({
            showClose: true,
            message: errHint,
            type: 'error'
          })
        }
      }).catch(() => {
        this.disabled = false
        this.$message({
          showClose: true,
          message: '服务器连接失败！',
          type: 'error'
        })
      })
    },
    // 取消
    cancelClick () {
      // 重置表单
      this.resetForm('ruleForm')
      this.$emit('parentCancel')
    },
    /* 人员 */
    crewClick () {
      // 组织ID
      const sector = this.formData.sector
      const nowSector = this.sectorOptions.find(item => {
        return item.base_id === sector
      })
      this.orgId = nowSector.id
      this.crewDialog = true
    },
    crewUpdata (data) {
      this.formData.crewName = data.names
      this.formData.crewId = data.ids
      this.crewDialog = false
    },
    crewCancel () {
      this.crewDialog = false
    }
  },
  watch: {
    parentDialog (val, oldVal) {
      if (val) {
        this.comInit()
      }
    },
    'formData.sector' (val, oldVal) {
      if (val) {
        this.crewDisabled = false
      } else {
        this.crewDisabled = true
      }
    }
  }
}
</script>

<style lang="less" scoped>

</style>
