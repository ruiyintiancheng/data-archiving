/*
 * @Author: Wk 
 * @Date: 2019-10-08 16:49:32 
 * @Last Modified by: 1k
 * @Last Modified time: 2019-10-08 18:24:35
 * @Description:  数据导出
 */
<template>
  <div class="app-container">
    <div class="base-row">
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
      <search-form v-show="searchToggle"
                   ref="searchForm"
                   :searchFormData="configData"></search-form>
    </div>
    <div class="base-row">
      <div class="row-botton clearfix">
        <div class="row-title">
          <svg-icon icon-class="ul" />
          <span>数据列表</span>
        </div>
        <div class="row-option">
          <el-button icon="el-icon-plus"
                     @click="addParent">添加</el-button>
          <a @click="tableToggle=false"
             v-if="tableToggle">
            <svg-icon icon-class="up" />&nbsp;收起</a>
          <a @click="tableToggle=true"
             v-else>
            <svg-icon icon-class="down" />&nbsp;展开</a>
        </div>
      </div>
      <basic-table v-show="tableToggle"
                   ref="basicTable"
                   :tableOption="configData"
                   :pagenation="true"
                   :rowNum="true">
        <el-table-column slot="optionColumn"
                         label="操作"
                         align="center"
                         fixed="right"
                         :width="250">
          <template slot-scope="scope">
            <el-button v-if="scope.row.field_convert_map.exportMode==='指定表'"
                       type="info"
                       plain
                       size="mini"
                       @click="setupOption(scope.row)">设置</el-button>
            <el-button v-if="scope.row"
                       type="primary"
                       plain
                       size="mini"
                       @click="modifyOption(scope.row)">修改</el-button>
            <el-button type="danger"
                       plain
                       size="mini"
                       @click="deleteForm(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </basic-table>
    </div>
    <el-dialog title="添加"
               v-el-drag-dialog
               :visible.sync="parentFormVisible"
               width="40%"
               custom-class="dialog-default">
      <div class="dialog-contant-default">
        <el-form :rules="outsideRules"
                 class="baseUpdate-form"
                 ref="formOutside"
                 :model="updateFormData"
                 label-width="120px">
          <el-form-item prop="configName"
                        label="配置名称">
            <el-input class="form-input"
                      v-model="updateFormData.configName"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="源用户选择"
                        prop="schema">
            <el-cascader :options="options"
                         v-model="updateFormData.schema"
                         :props="SetKesDept"
                         placeholder=""
                         style="width:155px">
              <template slot-scope="{ node, data }">
                <span>{{data.name}}</span>
              </template>
            </el-cascader>
          </el-form-item>
          <el-form-item label="目标户选择"
                        prop="targetSchema">
            <el-cascader :options="thirdOptions"
                         v-model="updateFormData.targetSchema"
                         :props="SetKesDept"
                         placeholder=""
                         style="width:155px">
              <template slot-scope="{ node, data }">
                <span>{{data.name}}</span>
              </template>
            </el-cascader>
          </el-form-item>
          <el-form-item prop="exportMode"
                        label="导出方式">
            <el-select class="form-input"
                       placeholder=""
                       v-model="updateFormData.exportMode"
                       clearable>
              <el-option label="所有表"
                         value="1"></el-option>
              <el-option label="指定表"
                         value="2"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="parentFormVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="saveOperate()">保存</el-button>
      </div>
    </el-dialog>

    <el-dialog title="修改"
               v-el-drag-dialog
               :visible.sync="modifyFormVisible"
               width="40%"
               custom-class="dialog-default">
      <div class="dialog-contant-default">
        <el-form :rules="outsideRules"
                 class="baseUpdate-form"
                 ref="form"
                 :model="updateFormDate"
                 label-width="120px">
          <el-form-item prop="configName"
                        label="配置名称">
            <el-input class="form-input"
                      v-model="updateFormDate.configName"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="源用户名称">
            <el-input class="form-input"
                      :disabled="true"
                      v-model="updateFormDate.schemaName"
                      clearable></el-input>
          </el-form-item>

          <el-form-item label="目标用户名称">
            <el-input class="form-input"
                      :disabled="true"
                      v-model="updateFormDate.targetSchemaName"
                      clearable></el-input>
          </el-form-item>
          <el-form-item prop="archiveMode"
                        label="导出方式">
            <el-select class="form-input"
                       placeholder=""
                       v-model="updateFormDate.exportMode"
                       clearable>
              <el-option label="所有表"
                         value="1"></el-option>
              <el-option label="指定表"
                         value="2"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="modifyFormVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="saveOperater()">保存</el-button>
      </div>
    </el-dialog>

    <el-dialog title="设置"
               v-el-drag-dialog
               top="5vh"
               :visible.sync="setupTableVisible"
               width="75%"
               custom-class="dialog-default dialog-default-more export-data">
      <div class="dialog-contant-default">
        <div class="base-row">
          <div class="row-botton clearfix">
            <div class="row-title">
              <svg-icon icon-class="search" />
              <span>筛选查询</span>
            </div>
            <div class="row-option">
              <!-- <a href="javascript:void(0)" class="button" @click="searchOption">查询</a> -->
              <el-button icon="el-icon-search"
                         @click="setupOption()"
                         type="primary">查询</el-button>
              <a @click="searchTogg=false"
                 v-if="searchTogg">
                <svg-icon icon-class="up" />&nbsp;收起</a>
              <a @click="searchTogg=true"
                 v-else>
                <svg-icon icon-class="down" />&nbsp;展开</a>
            </div>
          </div>
          <div v-show="searchTogg"
               class="form-search">
            <el-form :inline="true"
                     :model="searchData"
                     class="demo-table-expand">
              <div class="input-both-3">
                <el-form-item>
                  <span class="input-label">表名</span>
                  <el-input style="width:250px"
                            v-model=" searchData.tableName"
                            clearable></el-input>
                </el-form-item>
              </div>
            </el-form>
          </div>
        </div>
        <div class="base-row">
          <div class="row-botton clearfix">
            <div class="row-title">
              <svg-icon icon-class="ul" />
              <span>数据列表</span>
            </div>
            <div class="row-option">
              <el-button icon="el-icon-edit"
                         @click="setupModification">添加</el-button>
              <el-button icon="el-icon-error"
                         @click="batchDeletion">删除</el-button>
              <a @click="tableToggle=false"
                 v-if="tableToggle">
                <svg-icon icon-class="up" />&nbsp;收起</a>
              <a @click="tableToggle=true"
                 v-else>
                <svg-icon icon-class="down" />&nbsp;展开</a>
            </div>
          </div>
          <div>
            <el-table :data="setupData"
                      v-show="tableToggle"
                      border
                      stripe
                      style="width:100%"
                      highlight-current-row
                      element-loading-text="给我一点时间"
                      :height="getDefaultHeight()"
                      @selection-change="handleSelectionChange"
                      header-cell-class-name="ai-el-table-header">
              <el-table-column type="selection"
                               width="55">
              </el-table-column>
              <el-table-column :min-width="100"
                               align='center'
                               label="表名">
                <template slot-scope="scope">
                  <el-tooltip :disabled="textJug(scope.row.tableName)"
                              :content="scope.row.tableName"
                              placement="top"
                              visible-arrow
                              effect="light">
                    <div class="basic-table-column">
                      {{scope.row.tableName}}
                    </div>
                  </el-tooltip>
                </template>
              </el-table-column>
              <el-table-column :min-width="60"
                               align='center'
                               label="表注释">
                <template slot-scope="scope">
                  {{scope.row.tableComment}}
                </template>
              </el-table-column>
            </el-table>
            <el-pagination background
                           @size-change="handleSizeChange"
                           @current-change="handleCurrentChange"
                           :current-page="pageNo"
                           :total="total"
                           layout="total, sizes, prev, pager, next, jumper"
                           :page-sizes="[10,15,20]"
                           :page-size="pageSize">
            </el-pagination>
          </div>
        </div>
      </div>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="setupTableVisible = false">关闭</el-button>
      </div>
    </el-dialog>

    <!-- 设置的添加 -->
    <el-dialog title="添加"
               v-el-drag-dialog
               top="5vh"
               :visible.sync="setupFormVisible"
               width="70%"
               custom-class="dialog-default dialog-default-more">
      <div class="dialog-contant-default">
        <div class="base-row">
          <div class="row-botton clearfix">
            <div class="row-title">
              <svg-icon icon-class="search" />
              <span>筛选查询</span>
            </div>
            <div class="row-option">
              <!-- <a href="javascript:void(0)" class="button" @click="searchOption">查询</a> -->
              <el-button icon="el-icon-search"
                         @click="setupModification()"
                         type="primary">查询</el-button>
              <a @click="searchTogg=false"
                 v-if="searchTogg">
                <svg-icon icon-class="up" />&nbsp;收起</a>
              <a @click="searchTogg=true"
                 v-else>
                <svg-icon icon-class="down" />&nbsp;展开</a>
            </div>
          </div>
          <div v-show="searchTogg"
               class="form-search">
            <el-form :inline="true"
                     :model="setupFormDate"
                     class="demo-table-expand">
              <div class="input-both-3">
                <el-form-item>
                  <span class="input-label">表名</span>
                  <el-input style="width:250px"
                            v-model=" setupFormDate.tableName"
                            clearable></el-input>
                </el-form-item>
              </div>
            </el-form>
          </div>
        </div>
        <div class="base-row">
          <div class="row-botton clearfix">
            <div class="row-title">
              <svg-icon icon-class="ul" />
              <span>数据列表</span>
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
          <div>
            <el-table :data="additionalList"
                      v-show="tableToggle"
                      border
                      stripe
                      style="width:100%"
                      highlight-current-row
                      element-loading-text="给我一点时间"
                      :height="getHeight()"
                      @selection-change="handSelectionChange"
                      header-cell-class-name="ai-el-table-header">
              <el-table-column type="selection"
                               width="55">
              </el-table-column>
              <el-table-column :min-width="100"
                               align='center'
                               label="表名">
                <template slot-scope="scope">
                  <el-tooltip :disabled="textJug(scope.row.tablename)"
                              :content="scope.row.tablename"
                              placement="top"
                              visible-arrow
                              effect="light">
                    <div class="basic-table-column">
                      {{scope.row.tablename}}
                    </div>
                  </el-tooltip>
                </template>
              </el-table-column>
              <el-table-column :min-width="60"
                               align='center'
                               label="用户名称">
                <template slot-scope="scope">
                  {{scope.row.schemaname}}
                </template>
              </el-table-column>
              <el-table-column :min-width="60"
                               align='center'
                               label="临时表标志">
                <template slot-scope="scope">
                  {{scope.row.field_convert_map.imterimflag}}
                </template>
              </el-table-column>
            </el-table>
            <el-pagination background
                           @size-change="handSizeChange"
                           @current-change="handCurrentChange"
                           :current-page="pageno"
                           :total="totall"
                           layout="total, sizes, prev, pager, next, jumper"
                           :page-sizes="[10,15,20]"
                           :page-size="pagesize">
            </el-pagination>
          </div>
        </div>
      </div>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="setupFormVisible = false">关闭</el-button>
        <el-button @click="setupSave"
                   type="primary">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import SearchForm from 'search-form-ry'
import BasicTable from 'basic-table-ry'
import { baseSearch } from '@/api/base'
import { getSearchParam } from '@/utils/index'
import { baseRequest } from '@/api/base'
import { saveUpdate } from '@/utils/validate'
const url = '/basic/t03ExportDbConfig/selects'
export default {
  name: 'dataExportIndex',
  components: {
    SearchForm,
    BasicTable
  },
  data() {
    return {
      addListSelection: [], // 设置添加多选选中
      additionalList: [], // 添加列表
      multipelSelection: [], // 多选选中
      configid: '',
      pageNo: 1,
      total: null,
      pageSize: null,
      pageno: 1,
      totall: null,
      pagesize: null,
      listLoading: false, // 加载圆圈是否显示
      setupData: null, // 设置的列表
      searchData: {
        tableName: '', // 设置的筛选
        configId: ''
      },
      addParem: {}, // 添加发送请求的参数集
      configData: {}, // 模板数据
      searchToggle: true,
      searchTogg: true,
      tableToggle: true,
      parentFormVisible: false, // 添加开关
      modifyFormVisible: false, // 修改开关
      setupFormVisible: false, // 设置添加开关
      setupTableVisible: false, // 设置开关
      updateFormData: {
        exportMode: '',
        schema: [],
        targetSchema: [],
        configName: ''
      },
      updateFormDate: {
        exportMode: '',
        schemaname: '',
        targetSchemaName: '',
        configName: ''
      },
      setupFormDate: {},
      options: [], // 源数据
      secondOptions: [], // 归档数据
      thirdOptions: [], // 查询数据
      outsideRules: { // 校验
        configName: [{
          required: true, message: '必填!!'
        }],
        schema: [{
          required: true, message: '必填!!'
        }],
        targetSchema: [{
          required: true, message: '必填!!'
        }],
        exportMode: [{
          required: true, message: '必填!!'
        }],
        archiveFlag: [{
          required: true, message: '必填!!'
        }],
        archiveType: [{
          required: true, message: '必填!!'
        }]
      },
      SetKesDept: {// 自定义  级联选择器value label
        value: 'name',
        label: 'name',
        children: 'children'
      }
    }
  },
  created() {
    this.getOption()
  },
  methods: {
    setupSave() { // 设置添加的保存
      const table = []
      if (this.addListSelection.length !== 0) {
        for (const flag of this.addListSelection) {
          table.push(flag.tableid)
        }
        baseRequest('/basic/t03ExportDbTable/batchInsert', { configId: this.configid, tableIds: table }).then(response => {
          if (response.code === 1) {
            this.$Message({ message: '操作成功', type: 'success' })
            this.setupOption()
            this.setupFormVisible = false
          }
        })
      } else {
        this.$Message.error('未勾选操作行')
      }
    },
    setupModification() { // 打开设置添加
      this.listLoading = true
      this.setupFormDate.configId = this.configid
      baseSearch('/basic/tables/getTableNotInExport', this.setupFormDate).then(response => {
        this.additionalList = response.data.item
        this.totall = response.data.total
        this.pagesize = response.data.pageSize
        this.listLoading = false
        this.setupFormVisible = true
      })
    },
    batchDeletion() { // 设置批量删除
      const table = []
      if (this.multipelSelection.length !== 0) {
        for (const flag of this.multipelSelection) {
          table.push(flag.tableId)
        }
        baseRequest('/basic/t03ExportDbTable/batchDelete', { configId: this.configid, tableIds: table }).then(response => {
          if (response.code === 1) {
            this.$Message({ message: '操作成功', type: 'success' })
            this.setupOption()
          }
        })
      } else {
        this.$Message.error('未勾选操作行')
      }
    },
    handleSelectionChange(val) { // 设置多选选中
      this.multipelSelection = val
    },
    handSelectionChange(val) { // 设置添加多选选中
      this.addListSelection = val
    },
    setupOption(row) { // 设置
      this.listLoading = true
      if (row) {
        this.configid = row.configId
        this.searchData.configId = this.configid
        this.setupFormDate.schemaId = row.schemaId
      }
      this.searchData.pageNo = this.pageNo
      this.searchData.pageSize = this.pageSize
      baseSearch('/basic/t03ExportDbTable/selects', this.searchData).then(response => {
        this.setupData = response.data.item
        this.total = response.data.total
        this.pageSize = response.data.pageSize
        this.listLoading = false
        this.setupTableVisible = true
      })
    },
    handleSizeChange(val) { // 分页
      this.pageSize = val
      this.setupOption()
    },
    handleCurrentChange(val) { // 分页
      this.pageNo = val
      this.setupOption()
    },
    handSizeChange(val) { // 分页
      this.pagesize = val
      this.setupOption()
    },
    handCurrentChange(val) { // 分页
      this.pageno = val
      this.setupOption()
    },
    getDefaultHeight() { // 获取表格高度
      return this.$store.state.app.containHeight - 280
    },
    getHeight() {
      return this.$store.state.app.containHeight - 340
    },
    searchOption() {
      this.$refs.basicTable.getData(url, this.$refs.searchForm.searchParam())
    },
    getOption() {
      baseRequest(url, { paramId: '6' }).then(response => {
        const result = response.data
        result.formConfig = JSON.parse(result.formConfig)
        result.tableConfig = JSON.parse(result.tableConfig)
        this.configData = result
        const param = getSearchParam(result.formConfig)
        this.$refs.basicTable.getData(url, param)
      })
    },
    addParent() { // 添加
      baseRequest('/basic/dataResources/getSchemaByDbCategory', { dbCategory: '0' }).then(response => {
        this.thirdOptions = response.data.item
      })

      baseRequest('/basic/dataResources/getSchemaByDbCategory', { dbCategory: '2' }).then(response => {
        this.options = response.data.item
      })
      for (var i in this.updateFormData) {
        this.updateFormData[i] = ''
      }
      this.parentFormVisible = true
      this.$nextTick(_ => {
        this.$refs.formOutside.clearValidate()
      })
    },
    saveOperate() { // 添加保存
      this.addParem.configName = this.updateFormData.configName
      this.addParem.exportMode = this.updateFormData.exportMode
      for (const source of this.options[0].children) {
        if (source.name === this.updateFormData.schema[1]) {
          this.addParem.schemaId = source.id
        }
      }

      for (const query of this.thirdOptions[0].children) {
        if (query.name === this.updateFormData.targetSchema[1]) {
          this.addParem.targetSchemaId = query.id
        }
      }
      saveUpdate('/basic/t03ExportDbConfig/add', this.addParem, this.outsideRules, this.$refs.formOutside).then(() => {
        this.parentFormVisible = false
        this.$Message.success('操作成功')
        this.searchOption()
      })
    },
    modifyOption(row) { // 修改
      baseRequest('/basic/t03ExportDbConfig/select', { configId: row.configId }).then(response => {
        this.updateFormDate = response.data.item
        this.updateFormDate.schemaName = row.schemaName
        this.updateFormDate.targetSchemaName = row.targetSchemaName
        this.modifyFormVisible = true
      })
    },
    saveOperater() { // 修改保存
      const parem = {
        configId: this.updateFormDate.configId,
        configName: this.updateFormDate.configName,
        schemaId: this.updateFormDate.schemaId,
        targetSchemaId: this.updateFormDate.targetSchemaId,
        exportMode: this.updateFormDate.exportMode
      }
      saveUpdate('/basic/t03ExportDbConfig/update', parem, this.outsideRules, this.$refs.form).then(() => {
        this.modifyFormVisible = false
        this.$Message.success('操作成功')
        this.searchOption()
      })
    },
    // 删除
    deleteForm(row) {
      const param = { configId: row.configId }
      this.$confirm('此操作将永久删除该条数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        baseRequest('/basic/t03ExportDbConfig/delete', param).then(response => {
          this.searchOption()
          this.$Message.success('操作成功')
        })
      })
    },
    textJug(text) {
      if (text) {
        if (text.length > 11) {
          return false
        } else {
          return true
        }
      } else {
        return true
      }
    }
  }
}
</script>
<style lang="scss" >
.export-data {
  .el-dialog__body {
    height: 100%;
  }
  .el-dialog__footer {
    display: none;
    height: 0;
  }
}
</style>