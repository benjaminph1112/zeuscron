<template>
  <el-container>
    <!-- <task-sidebar></task-sidebar> -->
    <el-main>
      <el-form :inline="true">
        <el-row>
          <el-form-item label="任务ID">
            <el-input v-model.trim="searchParams.id"></el-input>
          </el-form-item>
          <el-form-item label="任务名称">
            <el-input v-model.trim="searchParams.name"></el-input>
          </el-form-item>
          <el-form-item label="标签">
            <el-input v-model.trim="searchParams.tag"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="执行方式">
            <el-select v-model.trim="searchParams.protocol">
              <el-option label="全部" value=""></el-option>
              <el-option v-for="item in protocolList" :key="item.value" :label="item.label" :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="任务节点">
            <el-select v-model.trim="searchParams.host_id">
              <el-option label="全部" value=""></el-option>
              <el-option v-for="item in hosts" :key="item.id" :label="item.alias + ' - ' + item.name + ':' + item.port " :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="状态">
            <el-select v-model.trim="searchParams.status">
              <el-option label="全部" value=""></el-option>
              <el-option v-for="item in statusList" :key="item.value" :label="item.label" :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="search()">搜索</el-button>
          </el-form-item>
        </el-row>
      </el-form>
      <el-row type="flex" justify="end">
        <el-col :span="2">
          <el-button type="primary" @click="toEdit(null)" v-if="this.$store.getters.user.isAdmin">新增</el-button>
        </el-col>
        <el-col :span="2">
          <el-button type="info" @click="refresh">刷新</el-button>
        </el-col>
      </el-row>
      <el-pagination background layout="prev, pager, next, sizes, total" :total="taskTotal" :page-size="20" @size-change="changePageSize" @current-change="changePage" @prev-click="changePage" @next-click="changePage">
      </el-pagination>
      <el-table :data="tasks" tooltip-effect="dark" border style="width: 100%" size="small">
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-tabs type="border-card">
              <el-tab-pane label="任务日志">
                <small-log-list :id="scope.row.id"></small-log-list>
              </el-tab-pane>
              <el-tab-pane label="任务概览">
                <task-overview :record="scope.row"></task-overview>
              </el-tab-pane>
            </el-tabs>

          </template>
        </el-table-column>
        <el-table-column prop="id" label="#" width="80">
        </el-table-column>
        <el-table-column prop="name" label="任务名称" width="150">
        </el-table-column>
        <el-table-column prop="tag" label="标签">
        </el-table-column>
        <el-table-column prop="spec" label="cron表达式" width="120">
        </el-table-column>
        <el-table-column label="下次执行时间" width="160">
          <template slot-scope="scope">
            {{scope.row.next_run_time | formatTime}}
          </template>
        </el-table-column>
        <el-table-column prop="protocol" :formatter="formatProtocol" label="执行方式">
        </el-table-column>
        <el-table-column label="状态" v-if="this.isAdmin" width="80">
          <template slot-scope="scope">
            <el-switch v-if="scope.row.level === 1" v-model="scope.row.status" :active-value="1" :inactive-vlaue="0" active-color="#13ce66" @change="changeStatus(scope.row)" inactive-color="#ff4949">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="状态" v-else width="80">
          <template slot-scope="scope">
            <el-switch v-if="scope.row.level === 1" v-model="scope.row.status" :active-value="1" :inactive-vlaue="0" active-color="#13ce66" :disabled="true" inactive-color="#ff4949">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="400" v-if="this.isAdmin">
          <template slot-scope="scope">
            <el-button-group>
              <el-button type="primary" @click="toEdit(scope.row)">编辑</el-button>
              <el-button type="success" @click="showRunTask(scope.row)">手动执行</el-button>
              <!-- <el-button type="info" @click="jumpToLog(scope.row)">查看日志</el-button> -->
              <el-button type="danger" @click="remove(scope.row)">删除</el-button>
            </el-button-group>
          </template>
        </el-table-column>
      </el-table>
    </el-main>

    <el-dialog title="手动执行" :visible.sync="dialogVisible">
      <span>
        <el-row>
          <el-col :span="16">
            <el-input type="input" size="medium" width="100" placeholder="请输入命令行参数" v-model="runRecord.command_args">
            </el-input>
          </el-col>
        </el-row>
        <el-row>
          {{runRecord.command | parseCommand(runRecord.command_args)}}
        </el-row>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="runTask">确 定</el-button>
      </span>
    </el-dialog>

  </el-container>
</template>

<script>
import taskSidebar from './sidebar'
import taskService from '../../api/task'
import { default as taskOverview } from './taskOverview'
import { default as smallLogList } from '../taskLog/smallList'

export default {
  name: 'task-list',
  components: {
    taskSidebar,
    taskOverview,
    smallLogList
  },
  data () {
    return {
      dialogVisible: false,
      runRecord: {},
      tasks: [],
      hosts: [],
      taskTotal: 0,
      searchParams: {
        page_size: 20,
        page: 1,
        id: '',
        protocol: '',
        name: '',
        tag: '',
        host_id: '',
        status: ''
      },
      isAdmin: this.$store.getters.user.isAdmin,
      protocolList: [
        {
          value: '1',
          label: 'http'
        },
        {
          value: '2',
          label: 'shell'
        }
      ],
      statusList: [
        {
          value: '2',
          label: '激活'
        },
        {
          value: '1',
          label: '停止'
        }
      ]
    }
  },
  created () {
    const hostId = this.$route.query.host_id
    if (hostId) {
      this.searchParams.host_id = hostId
    }

    this.search()
  },
  methods: {
    changeStatus (item) {
      if (item.status) {
        taskService.enable(item.id)
      } else {
        taskService.disable(item.id)
      }
    },
    formatProtocol (row, col) {
      if (row[col.property] === 2) {
        return 'shell'
      }
      if (row.http_method === 1) {
        return 'http-get'
      }
      return 'http-post'
    },
    changePage (page) {
      this.searchParams.page = page
      this.search()
    },
    changePageSize (pageSize) {
      this.searchParams.page_size = pageSize
      this.search()
    },
    search (callback = null) {
      taskService.list(this.searchParams, (tasks, hosts) => {
        this.tasks = tasks.data
        this.taskTotal = tasks.total
        this.hosts = hosts
        if (callback) {
          callback()
        }
      })
    },
    showRunTask (item) {
      this.dialogVisible = true
      this.runRecord = item
    },
    runTask () {
      let item = this.runRecord
      let args = item.command_args
      this.$appConfirm(() => {
        taskService.run(item.id, args, () => {
          this.$message.success('任务已开始执行')
        })
      }, true)
    },
    remove (item) {
      this.$appConfirm(() => {
        taskService.remove(item.id, () => {
          this.refresh()
        })
      })
    },
    jumpToLog (item) {
      this.$router.push(`/task/log?task_id=${item.id}`)
    },
    refresh () {
      this.search(() => {
        this.$message.success('刷新成功')
      })
    },
    toEdit (item) {
      let path = ''
      if (item === null) {
        path = '/task/create'
      } else {
        path = `/task/edit/${item.id}`
      }
      this.$router.push(path)
    }
  },
  filters: {
    parseCommand (a, b) {
      if (typeof a === 'undefined' || typeof b === 'undefined') {
        return ''
      }
      try {
        let args = b.split('|')
        let cmd = a
        args.forEach((ele, index) => {
          cmd = cmd.replace('{$' + (index + 1) + '}', ele)
        })
        return cmd
      } catch (e) {
        this.$alert(e)
      }
    }
  }
}
</script>
<style scoped>
.demo-table-expand {
  font-size: 0;
}
.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}
.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 50%;
}
</style>
