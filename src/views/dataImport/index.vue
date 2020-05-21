/*
 * @Author: 1k 
 * @Date: 2019-09-26 11:22:37 
 * @Last Modified by: 1k
 * @Last Modified time: 2019-10-08 17:39:05
 * @Description:  数据导入
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
            <el-button v-if="scope.row"
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
          <el-form-item label="归档户选择"
                        prop="archiveSchema">
            <el-cascader :options="secondOptions"
                         v-model="updateFormData.archiveSchema"
                         :props="SetKesDept"
                         placeholder=""
                         style="width:155px">
              <template slot-scope="{ node, data }">
                <span>{{data.name}}</span>
              </template>
            </el-cascader>
          </el-form-item>
          <el-form-item label="查询户选择"
                        prop="readSchema">
            <el-cascader :options="thirdOptions"
                         v-model="updateFormData.readSchema"
                         :props="SetKesDept"
                         placeholder=""
                         style="width:155px">
              <template slot-scope="{ node, data }">
                <span>{{data.name}}</span>
              </template>
            </el-cascader>
          </el-form-item>
          <el-form-item prop="archiveMode"
                        label="归档方式">
            <el-select class="form-input"
                       placeholder=""
                       v-model="updateFormData.archiveMode"
                       clearable>
              <el-option label="新表默认归档"
                         value="1"></el-option>
              <el-option label="新表默认不归档"
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
                      v-model="updateFormDate.schemaname"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="归档户名称">
            <el-input class="form-input"
                      :disabled="true"
                      v-model="updateFormDate.archiveschemaname"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="查询户名称">
            <el-input class="form-input"
                      :disabled="true"
                      v-model="updateFormDate.readschemaname"
                      clearable></el-input>
          </el-form-item>
          <el-form-item prop="archiveMode"
                        label="归档方式">
            <el-select class="form-input"
                       placeholder=""
                       v-model="updateFormDate.archiveMode"
                       clearable>
              <el-option label="新表默认归档"
                         value="1"></el-option>
              <el-option label="新表默认不归档"
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
               custom-class="dialog-default dialog-default-more import-data">
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
              <el-dropdown @command="archivalMark">
                <el-button size="mini">
                  归档标志<i class="el-icon-arrow-down el-icon--right"></i>
                </el-button>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="1">归档</el-dropdown-item>
                  <el-dropdown-item command="0">不归档</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
              <el-dropdown @command="archivedType">
                <el-button size="mini">
                  归档类型<i class="el-icon-arrow-down el-icon--right"></i>
                </el-button>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="1">全量</el-dropdown-item>
                  <el-dropdown-item command="2">增量</el-dropdown-item>
                  <el-dropdown-item command="3">变化量</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
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

              <el-table-column :min-width="100"
                               align='center'
                               label="用户名">
                <template slot-scope="scope">
                  {{scope.row.schemaName}}
                </template>
              </el-table-column>
              <el-table-column :min-width="60"
                               align='center'
                               label="归档标志">
                <template slot-scope="scope">
                  {{scope.row.field_convert_map.archiveFlag}}
                </template>
              </el-table-column>
              <el-table-column :min-width="60"
                               align='center'
                               label="归档类型">
                <template slot-scope="scope">
                  {{scope.row.field_convert_map.archiveType}}
                </template>
              </el-table-column>
              <el-table-column :min-width="100"
                               align='center'
                               label="筛选条件">
                <template slot-scope="scope">
                  <el-tooltip :disabled="textJug(scope.row.queryCondition)"
                              :content="scope.row.queryCondition"
                              placement="top"
                              visible-arrow
                              effect="light">
                    <div class="basic-table-column">
                      {{scope.row.queryCondition}}
                    </div>
                  </el-tooltip>
                </template>
              </el-table-column>
              <el-table-column label="操作"
                               width="120"
                               fixed="right"
                               align="center">
                <template slot-scope="scope">
                  <el-button v-if="scope.row"
                             type="primary"
                             plain
                             size="mini"
                             @click="setupModification(scope.row)">修改</el-button>
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
            <!-- <tree-table :data="data" v-show="tableToggle" :pagenation="true" :rowNum="true" :columns="columns" border> -->
            <!-- <tree-table :data="data" v-show="tableToggle" :pagenation="true" :rowNum="true" :columns="columns" border>
          <el-table-column label="操作" width="300" fixed="right" align="center">
            <template slot-scope="scope">
              <el-button v-if="scope.row" type="primary" plain size="mini" @click="updateOption(scope.row)">修改</el-button>
              <el-button size="mini" plain type="danger" @click="deleteForm(scope.row)">删除</el-button>
              <el-button size="mini" plain v-if="scope.row.server_config_id" v-show="false"></el-button>
              <el-button size="mini" plain v-else @click="addChildren(scope.row)">添加</el-button>
            </template>
          </el-table-column>
        </tree-table> -->
          </div>
        </div>
      </div>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="setupTableVisible = false">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 设置里的修改 -->
    <el-dialog title="修改"
               v-el-drag-dialog
               :visible.sync="setupFormVisible"
               width="40%"
               custom-class="dialog-default">
      <div class="dialog-contant-default">
        <el-form :rules="outsideRules"
                 class="baseUpdate-form"
                 ref="setupForm"
                 :model="setupFormDate"
                 label-width="120px">
          <el-form-item prop="archiveFlag"
                        label="归档标志">
            <el-select class="form-input"
                       placeholder=""
                       v-model="setupFormDate.archiveFlag">
              <el-option label="不归档"
                         value="0"></el-option>
              <el-option label="归档"
                         value="1"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item prop="archiveType"
                        label="归档类型">
            <el-select class="form-input"
                       placeholder=""
                       v-model="setupFormDate.archiveType">
              <el-option label="全量"
                         value="1"></el-option>
              <el-option label="增量"
                         value="2"></el-option>
              <el-option label="变化量"
                         value="3"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="筛选条件">
            <el-input type="textarea"
                      style="width:200px"
                      v-model="setupFormDate.queryCondition"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="setupFormVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="setupSave()">保存</el-button>
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
const url = '/basic/t03ArchiveDbConfig/selects'
export default {
  name: 'dataImportIndex',
  components: {
    SearchForm,
    BasicTable
  },
  data() {
    return {
      multipelSelection: [], // 多选选中
      configid: '',
      pageNo: 1,
      total: null,
      pageSize: null,
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
      setupFormVisible: false, // 设置修改开关
      setupTableVisible: false, // 设置开关
      updateFormData: {
        archiveMode: '',
        schema: [],
        archiveSchema: [],
        readSchema: [],
        configName: ''
      },
      updateFormDate: {
        archiveMode: '',
        schemaname: '',
        archiveschemaname: '',
        readschemaname: '',
        configName: ''
      },
      setupFormDate: {
        archiveFlag: '',
        archiveType: '',
        queryCondition: '',
        configId: null,
        tableId: null
      },
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
        archiveSchema: [{
          required: true, message: '必填!!'
        }],
        readSchema: [{
          required: true, message: '必填!!'
        }],
        archiveMode: [{
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
    setupSave() { // 设置修改的保存
      saveUpdate('/basic/t03ArchiveTable/update', this.setupFormDate, this.outsideRules, this.$refs.setupForm).then(() => {
        this.setupFormVisible = false
        this.$Message.success('操作成功')
        this.setupOption()
      })
    },
    setupModification(row) { // 打开设置修改
      this.setupFormDate.archiveFlag = row.archiveFlag
      this.setupFormDate.archiveType = row.archiveType
      this.setupFormDate.queryCondition = row.queryCondition
      this.setupFormDate.configId = this.configid
      this.setupFormDate.tableId = row.tableId
      this.setupFormVisible = true
    },
    archivedType(command) { // 归档类型批量
      const table = []
      if (this.multipelSelection.length !== 0) {
        for (const flag of this.multipelSelection) {
          table.push(flag.tableId)
        }
        baseRequest('/basic/t03ArchiveTable/batchUpdate', { configId: this.configid, archiveType: command, tableIds: table }).then(response => {
          if (response.code === 1) {
            this.$Message({ message: '操作成功', type: 'success' })
            this.setupOption()
          }
        })
      } else {
        this.$Message.error('未勾选操作行')
      }
    },
    archivalMark(command) { // 归档标志批量
      const table = []
      if (this.multipelSelection.length !== 0) {
        for (const flag of this.multipelSelection) {
          table.push(flag.tableId)
        }
        baseRequest('/basic/t03ArchiveTable/batchUpdate', { configId: this.configid, archiveFlag: command, tableIds: table }).then(response => {
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
    setupQuery() { // 设置查询/刷新

    },
    setupOption(row) { // 设置
      this.listLoading = true
      if (row) {
        this.configid = row.configid
        this.searchData.configId = this.configid
      }
      this.searchData.pageNo = this.pageNo
      this.searchData.pageSize = this.pageSize
      baseSearch('/basic/t03ArchiveTable/selects', this.searchData).then(response => {
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
    getDefaultHeight() { // 获取表格高度
      return this.$store.state.app.containHeight - 280
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
        this.options = response.data.item
      })
      baseRequest('/basic/dataResources/getSchemaByDbCategory', { dbCategory: '1' }).then(response => {
        this.secondOptions = response.data.item
      })
      baseRequest('/basic/dataResources/getSchemaByDbCategory', { dbCategory: '2' }).then(response => {
        this.thirdOptions = response.data.item
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
      this.addParem.archiveMode = this.updateFormData.archiveMode
      for (const source of this.options[0].children) {
        if (source.name === this.updateFormData.schema[1]) {
          this.addParem.schemaId = source.id
        }
      }
      for (const file of this.secondOptions[0].children) {
        if (file.name === this.updateFormData.archiveSchema[1]) {
          this.addParem.archiveSchemaId = file.id
        }
      }
      for (const query of this.thirdOptions[0].children) {
        if (query.name === this.updateFormData.readSchema[1]) {
          this.addParem.readSchemaId = query.id
        }
      }
      saveUpdate('/basic/t03ArchiveDbConfig/add', this.addParem, this.outsideRules, this.$refs.formOutside).then(() => {
        this.parentFormVisible = false
        this.$Message.success('操作成功')
        this.searchOption()
      })
    },
    modifyOption(row) { // 修改
      baseRequest('/basic/t03ArchiveDbConfig/select', { configid: row.configid }).then(response => {
        this.updateFormDate = response.data.item
        this.updateFormDate.schemaname = row.schemaname
        this.updateFormDate.archiveschemaname = row.archiveschemaname
        this.updateFormDate.readschemaname = row.readschemaname
        this.modifyFormVisible = true
      })
    },
    saveOperater() { // 修改保存
      const parem = {
        configId: this.updateFormDate.configId,
        configName: this.updateFormDate.configName,
        archiveMode: this.updateFormDate.archiveMode
      }
      saveUpdate('/basic/t03ArchiveDbConfig/update', parem, this.outsideRules, this.$refs.form).then(() => {
        this.modifyFormVisible = false
        this.$Message.success('操作成功')
        this.searchOption()
      })
    },
    // 删除
    deleteForm(row) {
      const param = { configId: row.configid }
      this.$confirm('此操作将永久删除该条数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        baseRequest('/basic/t03ArchiveDbConfig/delete', param).then(response => {
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
.import-data {
  .el-dialog__body {
    height: 100%;
  }
  .el-dialog__footer {
    display: none;
    height: 0;
  }
}
</style>