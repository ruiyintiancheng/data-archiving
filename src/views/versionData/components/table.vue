/*
  表格展示
  author: weilq
  date: 2019/10/8
 */
<template>
  <div v-loading="loading">
    <div>
      <div class="row-botton clearfix">
        <div class="row-title">
          <svg-icon icon-class="search" />
          <span>筛选查询</span>
        </div>
        <div class="row-option">
          <el-button icon="el-icon-search"
                     @click="searchOption"
                     type="primary">查询</el-button>
          <el-button icon="el-icon-refresh"
                     @click="getOption">重置</el-button>
          <a @click="searchToggle=false"
             v-if="searchToggle">
            <svg-icon icon-class="up" />&nbsp;收起</a>
          <a @click="searchToggle=true"
             v-else>
            <svg-icon icon-class="down" />&nbsp;展开</a>
        </div>
      </div>
      <div v-show="searchToggle">
        <div class="form-search">
          <el-form :inline="true"
                   :model="serachForm"
                   class="demo-form-inline demo-table-expand"
                   style="font-size: 0px">
            <div class="input-both-3">
              <el-form-item>
                <span class="input-label">表名</span>
                <el-input style="width:250px"
                          v-model="serachForm.tableName"
                          clearable></el-input>
              </el-form-item>
            </div>
            <div class="input-both-3">
              <el-form-item>
                <span class="input-label">注释</span>
                <el-input style="width:250px"
                          v-model="serachForm.tableComment"
                          clearable></el-input>
              </el-form-item>
            </div>
            <div class="input-both-3">
              <el-form-item class="version-data-date"
                            prop="tableVersion"
                            label="版本日期"
                            :rules="[{ type: 'date', required: true, message: '请选择日期', trigger: 'change' }]">
                <!-- <span class="input-label">版本日期</span> -->
                <el-date-picker v-model="serachForm.tableVersion"
                                type="date"
                                value-format="yyyyMMdd"
                                placeholder="选择日期"></el-date-picker>
              </el-form-item>
            </div>
          </el-form>
        </div>
      </div>
    </div>
    <div>
      <div class="row-botton clearfix">
        <div class="row-title">
          <svg-icon icon-class="ul" />
          <span>{{ tableTitle }}</span>
        </div>
        <div class="row-option">
          <a @click="tableToggle=false"
             v-if="tableToggle">
            <svg-icon icon-class="up" />&nbsp;收起</a>
          <a @click="tableToggle=true"
             v-else>
            <svg-icon icon-class="down" />&nbsp;展开</a>
        </div>
      </div>
      <div v-show="tableToggle">
        <el-table :data="tableData"
                  style="width: 100%;"
                  v-loading="listLoading"
                  element-loading-text="给我一点时间"
                  :height="tableHeight"
                  :border="true"
                  :fit="true">
          <el-table-column fixed
                           type="index"
                           width="50"
                           label="序号"
                           align="center"></el-table-column>
          <el-table-column prop="tablename"
                           label="表名"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="tablecomment"
                           label="表注释"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="tableversion"
                           label="版本号"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="schemaname"
                           label="用户名"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="schemades"
                           label="用户描述"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="datacount"
                           label="归档记录数"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column prop="addtime"
                           label="添加时间"
                           align="center"
                           width="160"></el-table-column>
          <el-table-column label="操作"
                           fixed="right"
                           align="center"
                           width="220">
            <!-- width="320" -->
            <template slot-scope="scope">
              <!-- <el-button type="primary" plain size="mini" @click="handleLook(scope.row)">浏览数据</el-button> -->
              <el-button type="primary"
                         plain
                         size="mini"
                         @click="handleTableSelect(scope.row)">表结构浏览</el-button>
              <el-button type="primary"
                         plain
                         size="mini"
                         @click="handleTableChange(scope.row)">表结构变化</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="pagination-container">
          <el-pagination background
                         @size-change="handleSizeChange"
                         @current-change="handleCurrentChange"
                         :current-page="serachForm.pageNo"
                         :page-sizes="[10,20,30,50]"
                         :page-size="serachForm.pageSize"
                         :total="total"
                         layout="total, sizes, prev, pager, next, jumper">
          </el-pagination>
        </div>
      </div>
    </div>
    <table-selects ref="tableSelects"></table-selects>
    <table-change ref="tableChange"></table-change>
  </div>
</template>
<script>
import { baseRequest } from '@/api/base'
import tableSelects from './tableSelects'
import tableChange from './tableChange'
export default {
  components: {
    tableSelects,
    tableChange
  },
  data() {
    return {
      serachForm: {
        schemaId: '',
        tableName: '', // 表名
        tableComment: '', // 注释
        tableVersion: null, // 版本日期
        pageNo: 1,
        pageSize: 10
      },
      loading: false,
      searchToggle: true,
      tableToggle: true,
      tableTitle: '数据列表',
      tableData: [], // table数据
      formHeight: 0,
      listLoading: false, // 加载
      tableParams: {}, // 查询参数
      total: 0, // 总数
      options: null // 请求参数
    }
  },
  computed: {
    tableConfig: function() { // table element属性
      return this.tableOption.tableConfig
    },
    formConfig: function() {
      return this.tableOption.formConfig
    },
    tableHeight: function() {
      return this.$store.state.app.containHeight - this.formHeight - 100
    }
  },
  watch: {
    searchToggle(val) {
      this.$nextTick(_ => {
        this.formHeight = this.getFormHeight()
      })
    }
  },
  created() {
    // this.getOption()
  },
  mounted() {
    this.$nextTick(_ => {
      this.formHeight = this.getFormHeight()
    })
  },
  methods: {
    searchOption() { // 查询
      this.getData()
      // console.log(this.serachForm)
      // this.$refs.myTable.setOptions(this.serachForm)
    },
    getOption() {
      // this.serachForm.fileName = ''
      // this.serachForm.staus = ''
      // this.$refs.myTable.clearOptions()
    },
    setOptions(option) {
      this.clearOptions()
      this.tableTitle = option.name
      this.tableData = []
      this.serachForm.schemaId = option.id
      // this.serachForm.schemaId = '1'
      // this.options = option
      this.getData()
    },
    clearOptions() {
      this.serachForm.tableName = ''
      this.serachForm.tableComment = ''
      // this.serachForm.tableVersion = this.dateFormat(new Date(), 'yyyyMMdd')
      this.serachForm.pageNo = 1
      this.serachForm.pageSize = 10
    },
    getData() { // 获取列表
      let param = {}
      param = { ...this.serachForm }

      baseRequest('/basic/t03ArchiveTableVersion/getArchiveTableBySchemaId', param)
        .then(response => {
          this.tableData = response.data.item
          this.total = response.data.total
        })
    },
    getFormHeight() {
      const formDom = document.querySelector('.form-search')
      return formDom ? formDom.offsetHeight + 52 : 52
    },
    handleSizeChange(val) { // 分页
      this.serachForm.pageSize = val
      this.getData()
    },
    handleCurrentChange(val) { // 分页
      this.serachForm.pageNo = val
      this.getData()
    },
    handleLook(row) {
      console.log(row)
    },
    handleTableSelect(row) {
      this.$refs.tableSelects.openDialog(row)
    },
    handleTableChange(row) {
      this.$refs.tableChange.openDialog(row)
    },
    dateFormat(date, fmt) {
      var o = {
        'M+': date.getMonth() + 1, // 月份
        'd+': date.getDate(), // 日
        'h+': date.getHours(), // 小时
        'm+': date.getMinutes(), // 分
        's+': date.getSeconds(), // 秒
        'q+': Math.floor((date.getMonth() + 3) / 3), // 季度
        'S': date.getMilliseconds() // 毫秒
      }
      if (/(y+)/.test(fmt)) {
        fmt = fmt.replace(RegExp.$1, (date.getFullYear() + '').substr(4 - RegExp.$1.length))
      }
      for (var k in o) {
        if (new RegExp('(' + k + ')').test(fmt)) {
          fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? (o[k]) : (('00' + o[k]).substr(('' + o[k]).length)))
        }
      }
      return fmt
    }
  }
}
</script>
<style>
.version-data-date > label {
  font-weight: 400 !important;
}
</style>
