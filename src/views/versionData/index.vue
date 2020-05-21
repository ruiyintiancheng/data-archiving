/**
 * 版本数据
 * weilq
 */
<template>
  <div class="app-container"
       style="margin-left:5px; margin-right: 5px;">
    <div class="my-col"
         :style="{'width': leftWidth + 'px'}">
      <div class="row-botton clearfix">
        <div class="row-title">
          <svg-icon icon-class="tree" />
          <span>数据源树</span>
        </div>
      </div>
      <el-input placeholder="筛选"
                v-model="filterText"
                style="padding:5px 10px;width:85%;"></el-input>
      <el-tree class="filter-tree"
               :filter-node-method="filterNode"
               :data="treeData"
               :props="treeOption"
               default-expand-all
               :expand-on-click-node="false"
               @node-click="treeNodeHandle"
               empty-text=""
               ref="tree2">
      </el-tree>
    </div>
    <div class="my-col my-col-button"
         style="width: 20px; position: relative;">
      <div class="botton-body">
        <el-button @click="leftArrowRight"
                   v-show="!openLeft"
                   type="text"
                   icon="el-icon-d-arrow-right"
                   style="margin-left: 0;"></el-button>
        <el-button @click="leftArrowLeft"
                   v-show="openLeft"
                   type="text"
                   icon="el-icon-d-arrow-left"
                   style="margin-left: 0;"></el-button>
      </div>
    </div>
    <div class="my-col"
         v-centerw="{leftWidth}">
      <my-table ref="myTable"></my-table>
    </div>
  </div>
</template>
<script>
import { baseRequest } from '@/api/base'
import myTable from './components/table'

export default {
  directives: {
    centerw: {
      update: function(el, binding) {
        const reduceWidth = binding.value.leftWidth + 21
        el.style.width = 'calc(100% - ' + reduceWidth + 'px'
      }
    }
  },
  components: {
    myTable
  },
  data() {
    return {
      filterText: '',
      treeData: [], // 树数据
      treeOption: { // 树的配置信息
        label: 'name'
      },
      currentTreeNode: null,

      openLeft: true, // 数据源展开
      leftWidth: 300 // 数据源宽度
    }
  },
  created() {
    this.getTrees()
  },
  watch: {
    filterText(val) {
      this.$refs.tree2.filter(val)
    }
  },
  mounted() {
    // this.editableTabs2 = []
  },
  methods: {
    getTrees() {
      baseRequest('/basic/dataResources/getSchemaByDbCategory', { dbCategory: '2' })
        .then(response => {
          this.treeData = response.data.item
        })
    },
    filterNode(value, data) { // 树的查询
      if (!value) return true
      return data[this.treeOption.label].indexOf(value) !== -1
    },
    leftArrowLeft() { // 数据源收缩
      this.leftWidth = 0
      this.openLeft = !this.openLeft
    },
    leftArrowRight() { // 数据源展开
      this.leftWidth = 300
      this.openLeft = !this.openLeft
    },
    // 点击树
    treeNodeHandle(e, node) {
      if (node.data.children && node.data.children.length > 0) {
        return
      }
      if (!node.data.id && node.data.id !== 0) {
        return
      }
      this.currentTreeNode = node.data
      this.$refs.myTable.setOptions(this.currentTreeNode)
      // this.getOption()
    }
  }
}
</script>
<style scoped>
.app-container .code-tree {
  margin-top: 5px;
  overflow: auto;
  position: relative;
  border: 1px solid #e4e4e4;
}
.my-col {
  height: 100%;
  float: left;
  overflow: hidden;
  transition: width 0.28s linear 0s;
}
.my-col-button {
  border-left: 1px solid #ccc;
  border-right: 1px solid #ccc;
}
.botton-body {
  width: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  position: absolute;
  top: 50%;
  left: 0;
  transform: translateY(-50%);
}
</style>
