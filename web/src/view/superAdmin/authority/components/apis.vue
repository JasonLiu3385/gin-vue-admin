<template>
  <div>
    <div class="clearflex">
      <el-button @click="authApiEnter" class="fl-right" size="small" type="primary">确 定</el-button>
    </div>
    <el-tree
      :data="apiTreeData"
      :default-checked-keys="apiTreeIds"
      :props="apiDefaultProps"
      default-expand-all
      highlight-current
      node-key="path"
      ref="apiTree"
      show-checkbox
    ></el-tree>
  </div>
</template>
<script>
import { getAllApis } from '@/api/api'
import { UpdateCasbin, getPolicyPathByAuthorityId } from '@/api/casbin'
export default {
  name: 'Apis',
  props: {
    row: {
      default: function() {
        return {}
      },
      type: Object
    }
  },
  data() {
    return {
      apiTreeData: [],
      apiTreeIds: [],
      apiDefaultProps: {
        children: 'children',
        label: 'description'
      }
    }
  },
  methods: {
    // 创建api树方法
    buildApiTree(apis) {
      const apiObj = new Object()
      apis &&
        apis.map(item => {
          if (apiObj.hasOwnProperty(item.apiGroup)) {
            apiObj[item.apiGroup].push(item)
          } else {
            Object.assign(apiObj, { [item.apiGroup]: [item] })
          }
        })
      const apiTree = []
      for (const key in apiObj) {
        const treeNode = {
          ID: key,
          description: key + '组',
          children: apiObj[key]
        }
        apiTree.push(treeNode)
      }
      return apiTree
    },
    // 关联关系确定
    async authApiEnter() {
      const checkArr = this.$refs.apiTree.getCheckedNodes(true)
      var casbinInfos = []
      checkArr&&checkArr.map(item=>{
        var casbinInfo = {
          path:item.path,
          method:item.method
        }
        casbinInfos.push(casbinInfo)
      })
      const res = await UpdateCasbin({
        authorityId: this.activeUserId,
        casbinInfos
      })
      if (res.code == 0) {
        this.$message({ type: 'success', message: res.msg })
      }
    }
  },
  async created() {
    // 获取api并整理成树结构
    const res2 = await getAllApis()
    const apis = res2.data.apis
    this.apiTreeData = this.buildApiTree(apis)

    const res = await getPolicyPathByAuthorityId({
      authorityId: this.row.authorityId
    })
    this.activeUserId = this.row.authorityId
    this.apiTreeIds = res.data.paths || []
  }
}
</script>
<style lang="scss">
</style>