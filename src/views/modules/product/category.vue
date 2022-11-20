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
            @click="() => addTreeNode(node)">
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

    <!--  弹出对话框  -->
    <el-dialog
      :title="categoryInfo.catId === '' ? '新增' : '更新'"
      :visible.sync="dialogShowFlag"
      width="30%">
      <el-form ref="form" :model="categoryInfo" label-width="80px">
        <el-form-item label="品牌名称">
          <el-input v-model="categoryInfo.name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogShowFlag = false">取 消</el-button>
        <el-button type="primary" @click="turnConfirm">确 定</el-button>
      </span>
    </el-dialog>
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
      },
      // 表示品牌信息
      categoryInfo: {
        name: "",
        catId: "",
        parentCid: "",
        catLevel: 0
      },
      // 表示弹出是否显示
      dialogShowFlag: false
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

    turnConfirm() {
      const {name = "", catId, parentCid, catLevel} = this.categoryInfo
      const isUpdateFlag = !!catId
      this.$http({
        url: this.$http.adornUrl(`/pms/category/${isUpdateFlag ? 'update' : 'save'}`),
        method: 'post',
        data: this.$http.adornData(
          Object.assign(
            {},
            {name},
            isUpdateFlag ?
              {catId} :
              {parentCid, catLevel: catLevel + 1, showStatus: 1, sort: 0, productUnit: 0}))
      }).then(() => {
        this.$message.success(isUpdateFlag ? "编辑成功" : "添加成功")

        // 清空数据
        this.resetCategoryInfo();
        this.dialogShowFlag = false;

        // 设置默认展开的节点 重新查询
        this.defaultExpandedKeys = [parentCid]
        this.getCategory()
      })
    },

    /**
     * @author lihh
     * @description 重置品牌信息
     */
    resetCategoryInfo() {
      this.categoryInfo.name = "";
      this.categoryInfo.catId = "";
      this.categoryInfo.parentCid = "";
    },

    /**
     * @author lihh
     * @description 进行节点的添加
     * @param node 要添加节点
     */
    addTreeNode(node) {
      const {data} = node
      const {catId, catLevel} = data

      this.resetCategoryInfo();

      this.dialogShowFlag = true
      this.categoryInfo.parentCid = catId;
      this.categoryInfo.catLevel = catLevel;
    },

    /**
     * @author lihh
     * @description 表示编辑节点
     * @param node node节点信息 表示保存数据
     */
    editTreeNode(node) {
      const {data} = node
      const {name = "", catId, parentCid, catLevel} = data

      this.categoryInfo.name = name
      this.categoryInfo.catId = catId
      this.categoryInfo.parentCid = parentCid;
      this.categoryInfo.catLevel = catLevel;
      this.dialogShowFlag = true
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
