/*
  表结构浏览
  author: weilq
  date: 2019/10/8
 */
<template>
  <el-dialog title="表结构浏览"
             :visible.sync="mainVisible"
             :close-on-click-modal='false'
             v-el-drag-dialog
             width="77%"
             custom-class="dialog-default">
    <div class="dialog-contant-default file-download-log select">
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
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="tableVersion"
                           label="版本号"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="columnName"
                           label="字段名"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="columnComment"
                           label="字段注释"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="columnOrder"
                           label="字段顺序"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="dataLength"
                           label="数据长度"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="columnCharset"
                           label="字符编码"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="columnTypeDesc"
                           label="字段类型"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="dataScale"
                           label="数据精度"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="addTime"
                           label="添加时间"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="field_convert_map.isPk"
                           label="是否主键字段"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="field_convert_map.lengthType"
                           label="长度类型"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="field_convert_map.nullAble"
                           label="是否允许为空"
                           align="center"
                           width="160"></el-table-column>
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
      tableHeight: 100,
      tableData: []
    }
  },
  mounted() {
    // this.$nextTick(function() {
    //   this.tableHeight = document.querySelector('.dialog-contant-default.file-download-log').offsetHeight - 80
    // })
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
      baseRequest('/basic/t03ArchiveColumnVersion/selects', param)
        .then(response => {
          this.tableData = response.data.item
        })
    },
    setTabHeight() {
      this.$nextTick(function() {
        this.tableHeight = document.querySelector('.dialog-contant-default.file-download-log.select').offsetHeight - 80
      })
    }
  }
}
</script>