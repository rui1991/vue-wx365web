<template>
  <el-dialog title="地图标点" :visible.sync="parentDialog" width="50%" :show-close="false" :close-on-click-modal="false" custom-class="hint-dialog">
    <div class="seaech">
      <el-input style="width: 60%; margin-right: 5px;" placeholder="请输地址名称" prefix-icon="el-icon-search" v-model.trim="position"></el-input>
      <el-button type="primary" @click="seaechClick">搜索</el-button>
    </div>
    <div id="container"></div>
    <div slot="footer" class="dialog-footer">
      <el-button @click="cancelClick">取 消</el-button>
      <el-button type="primary" :disabled="disabled" @click="confirmClick">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script>
import AMap from 'AMap'
export default{
  props: ['parentDialog', 'parentCoord'],
  data () {
    return {
      map: null,
      position: '',
      disabled: true,
      marker: null
    }
  },
  created () {

  },
  methods: {
    mapInit () {
      // 清空搜索框
      this.position = ''
      // 创建地图对象
      this.map = null
      this.marker = null
      let _map = null
      let _marker = null
      _map = new AMap.Map('container', {
        resizeEnable: true,
        center: [116.434381, 39.898515],
        zoom: 16
      })

      // 判断是否有坐标
      const coordStr = this.parentCoord
      if (coordStr) {
        let pointArr = coordStr.split(',')
        const pointLng = Number.parseFloat(pointArr[0])
        const pointLat = Number.parseFloat(pointArr[1])
        // 调整地图中心点坐标和地图级别
        _map.setZoom(19)
        _map.setCenter([pointLng, pointLat])
        // 创建标注
        _marker = new AMap.Marker({
          position: [pointLng, pointLat],
          offset: new AMap.Pixel(-13, -30),
          draggable: true,
          cursor: 'move'
        })
        _marker.setMap(_map)
        this.marker = _marker
      }
      // 点击地图
      _map.on('click', e => {
        // 清除地图上的所有标点
        _map.clearMap()
        let lng = e.lnglat.getLng()
        let lat = e.lnglat.getLat()
        _marker = new AMap.Marker({
          position: [lng, lat],
          offset: new AMap.Pixel(-13, -30),
          draggable: true,
          cursor: 'move'
        })
        _marker.setMap(_map)
        this.marker = _marker
      })
      this.map = _map
    },
    seaechClick () {
      let geocoder = new AMap.Geocoder({
        city: '全国'
      })
      // 获取地址名称
      const position = this.position
      geocoder.getLocation(position, (status, result) => {
        if (status === 'complete' && result.geocodes.length) {
          let lnglat = result.geocodes[0].location
          this.map.setZoom(19)
          this.map.setCenter(lnglat)
        } else {
          this.$message({
            showClose: true,
            message: '根据地址查询位置失败',
            type: 'error'
          })
        }
      })
    },
    confirmClick () {
      let point = this.marker.getPosition()
      let coord = point.lng + ',' + point.lat
      this.$emit('parentUpdata', coord)
    },
    // 取消
    cancelClick () {
      this.$emit('parentCancel')
    }
  },
  watch: {
    parentDialog (val, oldVal) {
      if (val) {
        setTimeout(() => {
          this.mapInit()
        }, 100)
        // this.mapInit()
      }
    },
    marker (val, oldVal) {
      if (val) {
        this.disabled = false
      } else {
        this.disabled = true
      }
    }
  }
}
</script>

<style lang="less" scoped>
  .seaech{
    height: 60px;
  }
  #container{
    width: 100%;
    height: 400px;
  }
</style>
