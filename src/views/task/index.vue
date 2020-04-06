<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <el-input v-model="query.value" clearable placeholder="输入搜索内容" style="width: 200px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <el-select v-model="query.type" clearable placeholder="类型" class="filter-item" style="width: 130px">
          <el-option v-for="item in queryTypeOptions" :key="item.key" :label="item.display_name" :value="item.key" />
        </el-select>
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission">

        <template slot="right">
          <el-button
            slot="reference"
            v-permission="permission.open"
            class="filter-item"
            type="success"
            icon="el-icon-edit"
            size="mini"
            :loading="crud.delAllLoading"
            :disabled="crud.selections.length === 0"
            @click="taskOpen(crud.selections)"
          >
            上架
          </el-button>
          <el-button
            slot="reference"
            v-permission="permission.close"
            class="filter-item"
            type="warning"
            icon="el-icon-edit"
            size="mini"
            :loading="crud.delAllLoading"
            :disabled="crud.selections.length === 0"
            @click="taskClose(crud.selections)"
          >
            下架
          </el-button>
          <el-button
            slot="reference"
            v-permission="permission.checkSuccess"
            class="filter-item"
            type="success"
            icon="el-icon-edit"
            size="mini"
            :loading="crud.delAllLoading"
            :disabled="crud.selections.length === 0"
            @click="taskSuccess(crud.selections)"
          >
            审核成功
          </el-button>
          <el-button
            slot="reference"
            v-permission="permission.checkFailure"
            class="filter-item"
            type="danger"
            icon="el-icon-edit"
            size="mini"
            :loading="crud.delAllLoading"
            :disabled="crud.selections.length === 0"
            @click="taskFail(crud.selections)"
          >
            审核失败
          </el-button>
        </template>
      </crudOperation>

      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="id" prop="id">
            <el-input v-model="form.id" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="任务类型" prop="type">
            <el-select v-model="form.type" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.task_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="支持设备">
            <el-select v-model="form.device" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.device_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="项目名称">
            <el-input v-model="form.itemName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="项目标题">
            <el-input v-model="form.taskTitle" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="项目备注">
            <el-input :rows="3" v-model="form.taskRemark" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="做单限时" prop="limitTime">
            <el-select v-model="form.limitTime" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.limit_time"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="审核时间" prop="checkTime">
            <el-select v-model="form.checkTime" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.check_time"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="悬赏主发布单价">
            <el-input v-model="form.rewardPrice" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="实际用户看到的单价">
            <el-input v-model="form.realityPrice" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="悬赏总额">
            <el-input v-model="form.totalMoney" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="任务总数量">
            <el-input v-model="form.rewardCount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="已完成数量" prop="completedCount">
            <el-input v-model="form.completedCount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建时间">
            <el-date-picker v-model="form.gmtCreate" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建人">
            <el-input v-model="form.createId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="状态">
            <el-select v-model="form.status" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.task_status"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="是否需要图片">
            <el-input v-model="form.needImg" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否需要文本">
            <el-input v-model="form.needText" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="完成时间">
            <el-input v-model="form.gmtFinish" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column v-if="columns.visible('id')" prop="id" label="id" width="180" fixed />
        <el-table-column v-if="columns.visible('type')" prop="type" label="任务类型">
          <template slot-scope="scope">
            {{ dict.label.task_type[scope.row.type] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('device')" prop="device" label="支持设备">
          <template slot-scope="scope">
            {{ dict.label.device_type[scope.row.device] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('itemName')" prop="itemName" label="项目名称" />
        <el-table-column v-if="columns.visible('taskTitle')" prop="taskTitle" label="项目标题" />
        <el-table-column v-if="columns.visible('taskRemark')" prop="taskRemark" label="项目备注" />
        <el-table-column v-if="columns.visible('limitTime')" prop="limitTime" label="做单限时">
          <template slot-scope="scope">
            {{ dict.label.limit_time[scope.row.limitTime] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('checkTime')" prop="checkTime" label="审核时间">
          <template slot-scope="scope">
            {{ dict.label.check_time[scope.row.checkTime] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('rewardPrice')" prop="rewardPrice" label="悬赏主发布单价" />
        <el-table-column v-if="columns.visible('realityPrice')" prop="realityPrice" label="实际用户看到的单价" />
        <el-table-column v-if="columns.visible('totalMoney')" prop="totalMoney" label="悬赏总额" />
        <el-table-column v-if="columns.visible('rewardCount')" prop="rewardCount" label="任务总数量" />
        <el-table-column v-if="columns.visible('completedCount')" prop="completedCount" label="已完成数量" />
        <el-table-column v-if="columns.visible('gmtCreate')" prop="gmtCreate" label="创建时间" width="180">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.gmtCreate) }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('createId')" prop="createId" label="创建人" width="180"/>
        <el-table-column v-if="columns.visible('status')" prop="status" label="状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status == 1" type="danger">{{ dict.label.task_status[scope.row.status] }}</el-tag>
            <el-tag v-if="scope.row.status == 2" type="success">{{ dict.label.task_status[scope.row.status] }}</el-tag>
            <el-tag v-if="scope.row.status == 3" type="warning">{{ dict.label.task_status[scope.row.status] }}</el-tag>
            <el-tag v-if="scope.row.status == 4" type="info">{{ dict.label.task_status[scope.row.status] }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('needImg')" prop="needImg" label="是否需要图片" />
        <el-table-column v-if="columns.visible('needText')" prop="needText" label="是否需要文本" />
        <el-table-column v-if="columns.visible('gmtFinish')" prop="gmtFinish" label="完成时间" width="180">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.gmtFinish) }}</span>
          </template>
        </el-table-column>

        <el-table-column label="操作" width="200" align="center" fixed="right" >
          <template slot-scope="scope">
             <el-button v-if="scope.row.status == 1" v-permission="permission.checkSuccess" type="success" @click="taskSuccess(scope.row)">审核成功</el-button>
             <el-button v-if="scope.row.status == 1" v-permission="permission.checkFailure" type="danger" @click="taskFail(scope.row)">审核失败</el-button>
             <el-button v-if="scope.row.status == 2" v-permission="permission.close" type="warning" @click="taskClose(scope.row)">下架</el-button>
             <el-button v-if="scope.row.status == 3" v-permission="permission.open" type="success" @click="taskOpen(scope.row)">上架</el-button>
          </template>

        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import request from '@/utils/request'
import crudTask from '@/api/task'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

// crud交由presenter持有
const defaultCrud = CRUD({ title: 'task', url: 'api/task', sort: 'id,desc', crudMethod: { ...crudTask }})
const defaultForm = { id: null, type: null, device: null, itemName: null, taskTitle: null, taskRemark: null, limitTime: null, checkTime: null, rewardPrice: null, realityPrice: null, totalMoney: null, rewardCount: null, completedCount: null, steps: null, gmtCreate: null, createId: null, status: null, needImg: null, needText: null, gmtFinish: null }
export default {
  name: 'Task',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(defaultCrud), header(), form(defaultForm), crud()],
  dicts: ['task_type', 'device_type', 'limit_time', 'check_time', 'task_status'],
  data() {
    return {
      permission: {
        add: ['admin', 'task:add'],
        edit: [''],
        del: [''],
        close: ['admin', 'task:close'],
        open: ['admin', 'task:open'],
        checkSuccess: ['admin', 'task:checkSuccess'],
        checkFailure: ['admin', 'task:checkFailure']
      },
      rules: {
        id: [
          { required: true, message: '不能为空', trigger: 'blur' }
        ],
        type: [
          { required: true, message: '任务类型不能为空', trigger: 'blur' }
        ],
        limitTime: [
          { required: true, message: '做单限时不能为空', trigger: 'blur' }
        ],
        checkTime: [
          { required: true, message: '审核时间不能为空', trigger: 'blur' }
        ],
        completedCount: [
          { required: true, message: '已完成数量不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'id', display_name: 'id' },
        { key: 'type', display_name: '任务类型' },
        { key: 'device', display_name: '支持设备' },
        { key: 'itemName', display_name: '项目名称' },
        { key: 'taskTitle', display_name: '项目标题' },
        { key: 'limitTime', display_name: '做单限时' },
        { key: 'checkTime', display_name: '审核时间' },
        { key: 'createId', display_name: '创建人' },
        { key: 'status', display_name: '状态' }
      ]
    }
  },
  methods: {
    // 获取数据前设置好接口地址
    [CRUD.HOOK.beforeRefresh]() {
      const query = this.query
      if (query.type && query.value) {
        this.crud.params[query.type] = query.value
      }
      return true
    },
    taskSuccess(data){
      request({
        url: 'api/task/checkSuccess',
        method: 'post',
        data:this.getIds(data)
      })
    },
    taskFail(data){
      request({
        url: 'api/task/checkFailure',
        method: 'post',
        data:this.getIds(data)
      })
    },
    taskOpen(data) {
      var _this = this;
      const res = request({
        url: 'api/task/open',
        method: 'post',
        data:this.getIds(data)
      }).then(() => {
        this.crud.refresh();
      })
    },
    taskClose(data) {
      var _this = this;
      request({
        url: 'api/task/close',
        method: 'post',
        data:this.getIds(data)
      }).then(() => {
        this.crud.refresh();
      })
    },
    getIds(data){
      const ids = [];
      if (data instanceof Array) {
        data.forEach(val => {
          ids.push(val.id)
        })
      } else {
        ids.push(data.id)
      }
      return ids;
    }

  }
}
</script>

<style scoped>

</style>
