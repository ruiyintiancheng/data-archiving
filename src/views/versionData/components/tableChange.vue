/*
  表结构变化
  author: weilq
  date: 2019/10/8
 */
<template>
  <el-dialog title="表结构变化"
             :visible.sync="mainVisible"
             :close-on-click-modal='false'
             v-el-drag-dialog
             width="77%"
             custom-class="dialog-default">
    <div class="dialog-contant-default file-download-log change">
      <div class="base-row">
        <div class="row-botton clearfix">
          <div class="row-title">
            <svg-icon icon-class="ul" />
            <span>数据列表</span>
          </div>
        </div>
        <el-table :data="tableData"
                  style="width: 100%;"
                  :height="tableHeight"
                  :border="true"
                  :fit="true">
          <el-table-column fixed
                           type="index"
                           width="50"
                           label="序号"
                           align="center"></el-table-column>
          <el-table-column prop="tableName"
                           label="表名"
                           align="center"></el-table-column>
          <el-table-column prop="tableVersion"
                           label="版本号"
                           align="center"></el-table-column>
          <el-table-column prop="columnName"
                           label="字段名"
                           align="center"></el-table-column>
          <el-table-column prop="addTime"
                           label="添加时间"
                           align="center"></el-table-column>
          <el-table-column prop="field_convert_map.changeType"
                           label="变更类型"
                           align="center"></el-table-column>
        </el-table>
      </div>
    </div>
    <div slot="footer"
         class="dialog-footer">
      <el-button @click="mainVisible = false">关闭</el-button>
    </div>
  </el-dialog>
</template>
<script>
import { baseRequest } from '@/api/base'
export default {
  data() {
    return {
      mainVisible: false,
      tableHeight: 0,
      tableData: []
    }
  },
  mounted() {
  },
  methods: {
    openDialog(data) {
      this.mainVisible = true
      this.setTabHeight()
      const param = {}
      param.tableId = data.tableid
      param.tableVersion = data.tableversion
      this.getData(param)
    },
    getData(param) {
      baseRequest('/basic/t03ArchiveTableChange/selects', param)
        .then(response => {
          this.tableData = response.data.item
        })
    },
    setTabHeight() {
      this.$nextTick(function() {
        this.tableHeight = document.querySelector('.dialog-contant-default.file-download-log.change').offsetHeight - 80
      })
    }
  }
}
</script>