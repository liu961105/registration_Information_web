<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <el-input v-model="query.name" clearable placeholder="输入名称搜索" style="width: 200px" class="filter-item"
          @keyup.enter.native="crud.toQuery" />
        <date-range-picker v-model="query.createTime" class="date-item" />
        <rrOperation />
      </div>
      <crudOperation :permission="permission">
        <el-button slot="left" v-permission="['admin','app:add']" :disabled="!currentRow" class="filter-item" size="mini"
          type="primary" icon="el-icon-plus" @click="copy">复制</el-button>
      </crudOperation>
    </div>
    <!--表单组件-->
    <el-dialog append-to-body :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0"
      :title="crud.status.title" width="800px">
      <el-form ref="form" :model="form" :rules="rules" size="small" label-width="100px">
        <el-form-item label="姓名" prop="userName">
          <el-input v-model="form.userName" style="width: 670px" placeholder="学生姓名" />
        </el-form-item>
        <el-form-item label="学号" prop="userNumber">
          <el-input-number v-model.number="form.port" placeholder="例如：8080" />
        </el-form-item>
        <el-form-item label="年级" prop="userGrade">
          <el-input v-model="form.uploadPath" style="width: 670px" placeholder="例如: /opt/upload" />
        </el-form-item>
        <el-form-item label="性别" prop="userGender">
          <el-select v-model="form.userGrade" style="width: 670px" placeholder="请选择">
            <el-option value="1" label="男">男</el-option>
            <el-option value="2" label="女">女</el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="电话" prop="userPhone">
          <el-input v-model="form.backupPath" style="width: 670px" placeholder="例如: /opt/backup" />
        </el-form-item>
        <el-form-item label="地址" prop="userAddress">
          <el-input v-model="form.deployScript" :rows="3" type="textarea" autosize style="width: 670px" placeholder="" />
        </el-form-item>
        <el-form-item label="备注" prop="userRemark">
          <el-input v-model="form.startScript" :rows="3" type="textarea" autosize style="width: 670px" placeholder="" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="text" @click="crud.cancelCU">取消</el-button>
        <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
      </div>
    </el-dialog>
    <!--表格渲染-->
    <el-table ref="table" v-loading="crud.loading" :data="crud.data" highlight-current-row style="width: 100%"
      @selection-change="crud.selectionChangeHandler" @current-change="handleCurrentChange">
      <el-table-column type="selection" width="55" />
      <el-table-column prop="userName" label="姓名" />
      <el-table-column prop="userNumber" label="学号" />
      <el-table-column prop="userGrade" label="年级" />
      <el-table-column prop="userGender" label="性别" />
      <el-table-column prop="userPhone" label="电话" />
      <el-table-column prop="userAddress" label="地址" />
      <el-table-column prop="userRemark" label="备注" />
      <el-table-column prop="createTime" label="创建时间" />
      <el-table-column v-if="checkPer(['admin','app:edit','app:del'])" label="操作" width="150px" align="center">
        <template slot-scope="scope">
          <udOperation :data="scope.row" :permission="permission" />
        </template>
      </el-table-column>
    </el-table>
    <!--分页组件-->
    <pagination />
  </div>
</template>

<script>
  import crudApp from '@/api/information/studentInfo'
  import CRUD, {
    presenter,
    header,
    form,
    crud
  } from '@crud/crud'
  import rrOperation from '@crud/RR.operation'
  import crudOperation from '@crud/CRUD.operation'
  import udOperation from '@crud/UD.operation'
  import pagination from '@crud/Pagination'
  import DateRangePicker from '@/components/DateRangePicker'

  const defaultForm = {
    id: null,
    name: null,
    port: 8080,
    uploadPath: '/opt/upload',
    deployPath: '/opt/app',
    backupPath: '/opt/backup',
    startScript: null,
    deployScript: null
  }
  export default {
    name: 'App',
    components: {
      pagination,
      crudOperation,
      rrOperation,
      udOperation,
      DateRangePicker
    },
    cruds() {
      return CRUD({
        title: '学生',
        url: '/api/studentInfo/queryUser',
        crudMethod: { ...crudApp
        }
      })
    },
    mixins: [presenter(), header(), form(defaultForm), crud()],
    data() {
      return {
        currentRow: null,
        permission: {
          add: ['admin', 'user:add'],
          edit: ['admin', 'user:edit'],
          del: ['admin', 'user:del']
        },
        rules: {
          name: [{
            required: true,
            message: '请输入应用名称',
            trigger: 'blur'
          }],
          port: [{
            required: true,
            message: '请输入应用端口',
            trigger: 'blur',
            type: 'number'
          }],
          uploadPath: [{
            required: true,
            message: '请输入上传目录',
            trigger: 'blur'
          }],
          deployPath: [{
            required: true,
            message: '请输入部署目录',
            trigger: 'blur'
          }],
          backupPath: [{
            required: true,
            message: '请输入备份目录',
            trigger: 'blur'
          }],
          startScript: [{
            required: true,
            message: '请输入启动脚本',
            trigger: 'blur'
          }],
          deployScript: [{
            required: true,
            message: '请输入部署脚本',
            trigger: 'blur'
          }]
        }
      }
    },
    methods: {
      copy() {
        for (const key in this.currentRow) {
          this.form[key] = this.currentRow[key]
        }
        this.form.id = null
        this.form.createTime = null
        this.crud.toAdd()
      },
      handleCurrentChange(row) {
        this.currentRow = JSON.parse(JSON.stringify(row))
      }
    }
  }
</script>

<style scoped>
</style>
