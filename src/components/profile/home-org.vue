<template>
  <el-dialog title="选择组织机构" :visible.sync="parentDialog" :show-close="false" :close-on-click-modal="false" custom-class="medium-dialog">
    <el-input
      placeholder="输入关键字进行过滤"
      clearable
      v-model="filterText">
    </el-input>
    <el-tree
      :data="orgData"
      ref="tree"
      show-checkbox
      default-expand-all
      check-strictly
      check-on-click-node
      node-key="id"
      :filter-node-method="filterNode"
      @check-change="orgCheckChange"
      :props="defaultProps">
    </el-tree>
    <div slot="footer" class="dialog-footer">
      <el-button @click="cancelClick">取 消</el-button>
      <el-button type="primary" :disabled="disabled" @click="confirmClick">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { mapState } from 'vuex'
export default{
  props: ['parentDialog', 'parentId'],
  data () {
    return {
      orgData: [],
      filterText: '',
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      organizeId: 0,
      disabled: true
    }
  },
  created () {

  },
  computed: {
    ...mapState('other', [
      'orgTree'
    ])
  },
  mounted () {
    if (this.orgTree[0].organize_type === 0) {
      this.orgData = this.orgTree[0].children
    } else {
      this.orgData = this.orgTree
    }
  },
  methods: {
    // 初始化数据
    orgInit () {
      this.filterText = ''
      this.organizeId = this.parentId
      setTimeout(() => {
        this.$refs.tree.setCheckedKeys([this.organizeId])
      }, 100)
    },
    // 触发页面显示配置的筛选
    filterNode (value, data, node) {
      // 如果什么都没填就直接返回
      if (!value) return true
      // 如果传入的value和data中的label相同说明是匹配到了
      if (data.name.indexOf(value) !== -1) return true
      // 否则要去判断它是不是选中节点的子节点
      return this.checkBelongNode(value, data, node)
    },
    // 判断传入的节点是不是选中节点的子节点
    checkBelongNode (value, data, node) {
      const level = node.level
      // 如果传入的节点本身就是一级节点就不用校验了
      if (level === 1) return false
      // 先取当前节点的父节点
      let parentData = node.parent
      // 遍历当前节点的父节点
      let index = 0
      while (index < level - 1) {
        // 如果匹配到直接返回
        if (parentData.data.name.indexOf(value) !== -1) {
          return true
        }
        // 否则的话再往上一层做匹配
        parentData = parentData.parent
        index++
      }
      // 没匹配到返回false
      return false
    },
    // 点击机构树
    orgCheckChange (data, checked, self) {
      if (checked === true) {
        if (this.organizeId === data.id) return
        this.$refs.tree.setCheckedKeys([data.id])
        this.organizeId = data.id
      } else {
        if (this.organizeId === data.id) {
          this.$refs.tree.setCheckedKeys([data.id])
        }
      }
    },
    // 确定
    confirmClick () {
      const checkNodes = this.$refs.tree.getCheckedNodes()
      if (checkNodes.length === 0) {
        this.$message({
          showClose: true,
          message: '请选择组织机构！',
          type: 'warning'
        })
      } else {
        const orgNode = checkNodes[0]
        const id = orgNode.id
        const name = orgNode.name
        const type = orgNode.organize_type
        const baseId = orgNode.base_id
        const obj = {
          id: id,
          name: name,
          type: type,
          baseId: baseId
        }
        this.$emit('parentUpdata', obj)
      }
    },
    // 取消
    cancelClick () {
      this.$emit('parentCancel')
    }
  },
  watch: {
    parentDialog (val, oldVal) {
      if (val) {
        this.orgInit()
      }
    },
    filterText (val, oldVal) {
      this.$refs.tree.filter(val)
    },
    organizeId (val, oldVal) {
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

</style>
