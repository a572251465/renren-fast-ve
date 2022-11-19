<template>
  <div>
    <el-tree
      :data="data"
      node-key = "catId"
      :expand-on-click-node="false"
      :props="defaultProps"
      :default-expanded-keys = "defaultExpandedKeys">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            v-if="data.catLevel !== 3"
            @click="() => addTreeNode(data)">
            添加
          </el-button>
          <el-button
            type="text"
            size="mini"
            v-if="data.children && data.children.length === 0"
            @click="() => removeTreeNode(node, data)">
            删除
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => editTreeNode(node, data)">
            编辑
          </el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  data() {
    return {
      data: [],
      // 表示默认展开的节点
      defaultExpandedKeys: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },
  methods: {
    getCategory() {
      this.$http({
        url: this.$http.adornUrl('/pms/category/listWithTree'),
        method: 'get'
      }).then(({data}) => {
        this.data = data.data
      })
    },

    addTreeNode() {

    },

    editTreeNode() {

    },

    /**
     * @author lihh
     * @description 表示删除节点
     * @param node 表示包装后的node节点
     */
    removeTreeNode(node) {
      const {data} = node
      const {name, catId, parentCid} = data

      const callback = () => {
        this.$http({
          url: this.$http.adornUrl('/pms/category/delete'),
          method: 'post',
          data: this.$http.adornData([catId], false)
        }).then(() => {
          this.$message.success("删除成功")

          // 设置默认展开的节点 重新查询
          this.defaultExpandedKeys = [parentCid]
          this.getCategory()
        })
      }

      this.$confirm(`是否删除【${name}】分类`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(callback).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    }
  },
  created() {
    this.getCategory();
  }
};
</script>

<style></style>
