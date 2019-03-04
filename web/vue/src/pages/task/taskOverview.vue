<template>
  <el-form label-position="left" inline class="demo-table-expand">
    <el-form-item label="任务创建时间:">
      {{record.created | formatTime}} <br>
    </el-form-item>
    <el-form-item label="任务类型:">
      {{record.level | formatLevel}} <br>
    </el-form-item>
    <el-form-item label="单实例运行:">
      {{record.multi | formatMulti}} <br>
    </el-form-item>
    <el-form-item label="超时时间:">
      {{record.timeout | formatTimeout}} <br>
    </el-form-item>
    <el-form-item label="重试次数:">
      {{record.retry_times}} <br>
    </el-form-item>
    <el-form-item label="重试间隔:">
      {{record.retry_interval | formatRetryTimesInterval}}
    </el-form-item> <br>
    <el-form-item label="任务节点">
      <div v-for="item in record.hosts" :key="item.host_id">
        {{item.alias}} - {{item.name}}:{{item.port}} <br>
      </div>
    </el-form-item> <br>
    <el-form-item label="命令:" style="width: 100%">
      {{record.command}}
    </el-form-item> <br>
    <el-form-item label="备注" style="width: 100%">
      {{record.remark}}
    </el-form-item>
  </el-form>
</template>
<script>
export default {
  props: {
    record: {
      required: true
    }
  },
  data () {
    return {
    }
  },
  filters: {
    formatLevel (value) {
      if (value === 1) {
        return '主任务'
      }
      return '子任务'
    },
    formatTimeout (value) {
      if (value > 0) {
        return value + '秒'
      }
      return '不限制'
    },
    formatRetryTimesInterval (value) {
      if (value > 0) {
        return value + '秒'
      }
      return '系统默认'
    },
    formatMulti (value) {
      if (value > 0) {
        return '否'
      }
      return '是'
    }
  }
}
</script>
