<template>
  <div>
    <!-- 新增 -->
    <el-dialog title="新增设备" :visible.sync="parentDialog" :show-close="false" :close-on-click-modal="false" custom-class="medium-dialog">
      <el-form class="divide-from" :model="formData" :rules="rules" ref="ruleForm" :label-width="formLabelWidth">
        <el-form-item label="设备名称" prop="name">
          <el-input v-model.trim="formData.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="DevEui" prop="DevEui">
          <el-input v-model.trim="formData.DevEui" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="设备类型" prop="type">
          <el-select v-model="formData.type" placeholder="请选择设备类型">
            <el-option
              v-for="item in typeOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="设置温度" style="width: 440px;" v-show="formData.type === '温湿度'">
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最低温度" type="number" v-model.number="formData.htempMin">
                <template slot="append">℃</template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最高温度" type="number" v-model.number="formData.htempMax">
                <template slot="append">℃</template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="设置湿度" style="width: 440px;" v-show="formData.type === '温湿度'">
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最低湿度" type="number" v-model.number="formData.moisMin">
                <template slot="append">%</template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最高湿度" type="number" v-model.number="formData.moisMax">
                <template slot="append">%</template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="设置规则" style="width: 440px;" v-show="formData.type === '无线水浸'">
          <el-radio-group v-model="formData.waterRule">
            <el-radio label="wet">浸水自动报警</el-radio>
            <el-radio label="dry">无水自动报警</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="设置压力" style="width: 440px;" v-show="formData.type === '无线压力变送器'">
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最小压力" type="number" v-model.number="formData.presMin">
                <template slot="append">kPa</template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最大压力" type="number" v-model.number="formData.presMax">
                <template slot="append">kPa</template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="设置液位" style="width: 440px;" v-show="formData.type === '无线液位变送器'">
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最小液位" type="number" v-model.number="formData.liquMin">
                <template slot="append">cm</template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最大液位" type="number" v-model.number="formData.liquMax">
                <template slot="append">cm</template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="设置温度" style="width: 440px;" v-show="formData.type === '安全用电'">
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最低温度" type="number" v-model.number="formData.etempMin">
                <template slot="append">℃</template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最高温度" type="number" v-model.number="formData.etempMax">
                <template slot="append">℃</template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="设置电压" style="width: 440px;" v-show="formData.type === '安全用电'">
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最低电压" type="number" v-model.number="formData.voltageMin">
                <template slot="append">V</template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最高电压" type="number" v-model.number="formData.voltageMax">
                <template slot="append">V</template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="设置电流" style="width: 440px;" v-show="formData.type === '安全用电'">
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最低电流" type="number" v-model.number="formData.electMin">
                <template slot="append">A</template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item>
              <el-input style="width: 140px;" placeholder="最高电流" type="number" v-model.number="formData.electMax">
                <template slot="append">A</template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="设备位置" prop="posName">
          <el-input :disabled="true" v-model="formData.posName"></el-input>
          <el-button type="primary" @click="posDialog = true">选择位置</el-button>
        </el-form-item>
<!--        <el-form-item label="心跳间隔" prop="beat">-->
<!--          <el-input placeholder="请输入心跳间隔" type="number" v-model="formData.beat">-->
<!--            <template slot="append">分钟</template>-->
<!--          </el-input>-->
<!--        </el-form-item>-->
        <el-form-item label="描述" prop="describe">
          <el-input type="textarea" maxlength="100" placeholder="文本长度不得超过100个字符" v-model.trim="formData.describe"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancelClick">取 消</el-button>
        <el-button type="primary" :disabled="disabled" @click="submitForm('ruleForm')">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 位置 -->
    <pos-module
      :parentDialog="posDialog"
      @parentSelect="posSelect"
      @parentCancel="posCancel">
    </pos-module>
  </div>
</template>

<script>
import { mapState } from 'vuex'
// 引入位置组件
import posModule from '@/components/facility/hardfac-pos'
export default{
  props: ['parentDialog'],
  data () {
    return {
      typeOptions: [
        {
          label: '温湿度',
          value: '温湿度'
        },
        {
          label: '无线水浸',
          value: '无线水浸'
        },
        {
          label: '无线压力变送器',
          value: '无线压力变送器'
        },
        {
          label: '无线液位变送器',
          value: '无线液位变送器'
        },
        {
          label: '无线烟感',
          value: '无线烟感'
        },
        {
          label: '安全用电',
          value: '安全用电'
        },
        {
          label: '一键报警',
          value: '一键报警'
        },
        {
          label: '红外报警',
          value: '红外报警'
        },
        {
          label: '燃气报警',
          value: '燃气报警'
        }
      ],
      formLabelWidth: '100px',
      rules: {
        name: [
          { required: true, message: '请输入设备名称', trigger: 'blur' }
        ],
        type: [
          { required: true, message: '请输入设备类型', trigger: 'change' }
        ],
        code: [
          { required: true, message: '请输入验证码', trigger: 'blur' }
        ],
        posName: [
          { required: true, message: '请选择设备位置', trigger: 'change' }
        ],
        DevEui: [
          { required: true, message: '请输入设备DevEui', trigger: 'blur' }
        ],
        AppEui: [
          { required: true, message: '请输入设备AppEui', trigger: 'blur' }
        ],
        AppKey: [
          { required: true, message: '请输入设备AppKey', trigger: 'blur' }
        ]
        // beat: [
        //   { required: true, message: '请输入设备心跳时间', trigger: 'blur' }
        // ]
      },
      formData: {
        name: '',
        type: '',
        htempMin: '',
        htempMax: '',
        moisMin: '',
        moisMax: '',
        waterRule: '',
        presMin: '',
        presMax: '',
        liquMin: '',
        liquMax: '',
        etempMin: '',
        etempMax: '',
        voltageMin: '',
        voltageMax: '',
        electMin: '',
        electMax: '',
        posName: '',
        posId: '',
        DevEui: '',
        // beat: '',
        describe: ''
      },
      posDialog: false,
      disabled: false
    }
  },
  created () {

  },
  components: {
    posModule
  },
  computed: {
    ...mapState('user', [
      'userId'
    ]),
    ...mapState('other', [
      'companyId',
      'projectId'
    ])
  },
  methods: {
    // 初始化数据
    addInit () {
      this.formData = {
        name: '',
        type: '',
        htempMin: '',
        htempMax: '',
        moisMin: '',
        moisMax: '',
        waterRule: '',
        presMin: '',
        presMax: '',
        liquMin: '',
        liquMax: '',
        etempMin: '',
        etempMax: '',
        voltageMin: '',
        voltageMax: '',
        electMin: '',
        electMax: '',
        posName: '',
        posId: '',
        DevEui: '',
        // beat: '',
        describe: ''
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
    // 提交
    sendRequest () {
      const type = this.formData.type
      let typeValue = 0
      // 温湿度
      let htempMinVal = ''
      const htempMin = Number.parseFloat(this.formData.htempMin)
      let htempMaxVal = ''
      const htempMax = Number.parseFloat(this.formData.htempMax)
      let moisMinVal = ''
      const moisMin = Number.parseFloat(this.formData.moisMin)
      let moisMaxVal = ''
      const moisMax = Number.parseFloat(this.formData.moisMax)
      // 无线水浸
      let waterVal = ''
      const waterRule = this.formData.waterRule
      // 无线压力变送器
      let presMinVal = ''
      const presMin = Number.parseFloat(this.formData.presMin)
      let presMaxVal = ''
      const presMax = Number.parseFloat(this.formData.presMax)
      // 无线液位变送器
      let liquMinVal = ''
      const liquMin = Number.parseFloat(this.formData.liquMin)
      let liquMaxVal = ''
      const liquMax = Number.parseFloat(this.formData.liquMax)
      // 安全用电
      let etempMinVal = ''
      const etempMin = Number.parseFloat(this.formData.etempMin)
      let etempMaxVal = ''
      const etempMax = Number.parseFloat(this.formData.etempMax)
      let voltageMinVal = ''
      const voltageMin = Number.parseFloat(this.formData.voltageMin)
      let voltageMaxVal = ''
      const voltageMax = Number.parseFloat(this.formData.voltageMax)
      let electMinVal = ''
      const electMin = Number.parseFloat(this.formData.electMin)
      let electMaxVal = ''
      const electMax = Number.parseFloat(this.formData.electMax)
      switch (type) {
        case '温湿度':
          typeValue = 45
          let htemp = this.compareSize(htempMin, htempMax)
          htempMinVal = htemp.min
          htempMaxVal = htemp.max
          let mois = this.compareSize(moisMin, moisMax)
          moisMinVal = mois.min
          moisMaxVal = mois.max
          break
        case '无线水浸':
          typeValue = 25
          waterVal = waterRule
          break
        case '无线压力变送器':
          typeValue = 36
          let pressure = this.compareSize(presMin, presMax)
          presMinVal = pressure.min
          presMaxVal = pressure.max
          break
        case '无线液位变送器':
          typeValue = 37
          let liquid = this.compareSize(liquMin, liquMax)
          liquMinVal = liquid.min
          liquMaxVal = liquid.max
          break
        case '无线烟感':
          typeValue = 39
          break
        case '安全用电':
          typeValue = 41
          let etemp = this.compareSize(etempMin, etempMax)
          etempMinVal = etemp.min
          etempMaxVal = etemp.max
          let voltage = this.compareSize(voltageMin, voltageMax)
          voltageMinVal = voltage.min
          voltageMaxVal = voltage.max
          let elect = this.compareSize(electMin, electMax)
          electMinVal = elect.min
          electMaxVal = elect.max
          break
        default:
          typeValue = 44
      }
      let params = {
        project_id: this.projectId,
        name: this.formData.name,
        type: typeValue,
        node_cn_type: type,
        min_temperature: htempMinVal,
        max_temperature: htempMaxVal,
        min_humidity: moisMinVal,
        max_humidity: moisMaxVal,
        dry_wet: waterVal,
        min_pressure: presMinVal,
        max_pressure: presMaxVal,
        min_liquid: liquMinVal,
        max_liquid: liquMaxVal,
        min_temp: etempMinVal,
        max_temp: etempMaxVal,
        min_voltage: voltageMinVal,
        max_voltage: voltageMaxVal,
        min_current: electMinVal,
        max_current: electMaxVal,
        location_id: this.formData.posId,
        devEUI: this.formData.DevEui,
        out_time: 1,
        description: this.formData.describe
      }
      params = this.$qs.stringify(params)
      this.disabled = true
      this.$axios({
        method: 'post',
        url: this.loraApi() + '/lora/addNode',
        data: params
      }).then((res) => {
        this.disabled = false
        if (res.data.result === 'Sucess') {
          // 重置表单
          this.resetForm('ruleForm')
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
    // 比较大小值
    compareSize (min, max) {
      let minResult = ''
      let maxResult = ''
      if (min && max) {
        if (min > max) {
          minResult = max
          maxResult = min
        } else {
          minResult = min
          maxResult = max
        }
      } else {
        if (min) {
          minResult = min
        }
        if (max) {
          maxResult = max
        }
      }
      return {
        min: minResult,
        max: maxResult
      }
    },
    // 取消
    cancelClick () {
      // 重置表单
      this.resetForm('ruleForm')
      this.$emit('parentCancel')
    },
    /* 位置选择 */
    posSelect (data) {
      this.formData.posName = data.name
      this.formData.posId = data.id
      this.posDialog = false
    },
    posCancel () {
      this.posDialog = false
    }
  },
  watch: {
    parentDialog (val, oldVal) {
      if (val) {
        this.addInit()
      }
    }
  }
}
</script>

<style lang="less" scoped>

</style>
