<template>
  <div style="width:90%;margin: 0 auto;">
    <div style="margin-top:20px;">
      <el-row :gutter="5">
        <el-col :span="6">
          <el-button type="danger" @click="batchremote">批量删除</el-button>
          <el-button type="success" @click="addsemester">新增</el-button>
        </el-col>
        <el-col :span="18">
          <el-form :inline="true" style="float:right;">
            <el-form-item>
              <el-input v-model="conditions.search.posname" placeholder="请输入关键字" @keyup.enter.native="conditionsearch"/>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="conditionsearch">查询</el-button>
            </el-form-item>
          </el-form>
        </el-col>
      </el-row>
    </div>
    <el-table
      :data="tableData"
      height="500"
      border
      style="width: 100%"
      @selection-change="handleSelectionChange"
      @sort-change="handlerSortchange">
      <!-- 多选 -->
      <el-table-column
        type="selection"
        width="40"
        style="text-align:center;"/>
      <el-table-column
        prop="position"
        sortable = "custom"
        label="职位"/>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button
            size="mini"
            @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页插件 -->
    <el-pagination
      :total="total"
      :page-size="pageSize"
      :current-page="currentPage"
      background
      layout="prev, pager, next"
      @current-change="handlecurrentchange"/>

    <!-- 修改模态框 -->
    <el-dialog :visible.sync="dialogFormVisible" width="50%" title="修改职务">
      <el-form :model="positionform">
        <el-form-item label-width="120px" label="职务名称">
          <el-input v-model="positionform.position" clearable placeholder="请输入职务名称"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="suremodify">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 新增模态框 -->
    <el-dialog :visible.sync="newFormVisible" width="50%" title="新增学期">
      <el-form :model="positionform">
        <el-form-item label-width="120px" label="职务名称">
          <el-input v-model="positionform.position" clearable placeholder="请输入职务名称"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="sureAddSemester">确 定</el-button>
      </div>
    </el-dialog>
  </div>

</template>

<script>
import { getPositionList, modifyPosition, removePosition, batchrmPositions, newPosition } from '../../../api/sysdirectory'
export default {
  name: 'Position',
  data() {
    return {
      currentPage: 1, // 当前页面
      total: 1000, // 总记录数
      pageSize: 10, // 每页的大小
      multipleSelection: [], // 多选数组
      // 表格具体内容
      tableData: [],
      // 查询条件
      conditions: {
        order: '',
        sort: '',
        page: 1,
        pagesize: 10,
        search: {
          posname: ''
        }
      },
      // 修改表单是否隐藏
      dialogFormVisible: false,
      // 修改表单
      positionform: {
        id: '', // 编号
        position: '',
        positionnum: ''
      },

      // 新增表单是否隐藏
      newFormVisible: false
    }
  },
  created() {
    // 抓取数据
    this.fetchData()
  },
  methods: {
    // 选择发生变化
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    // 点击跳转页面变化
    handlecurrentchange(val) {
      this.currentPage = val
      this.conditions.page = val
      this.fetchData()
    },
    // 分页抓取数据
    fetchData() {
      getPositionList(this.conditions).then(response => {
        const data = response.data
        this.tableData = data.list
        this.total = data.total
        this.currentPage = data.pageNum
      })
    },
    // 编辑
    handleEdit(index, row) {
      this.positionform = row
      this.dialogFormVisible = true
    },
    // 删除
    handleDelete(index, row) {
      const rowid = row.id
      this.$confirm('你确定要删除【' + row.position + '】吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        removePosition(rowid).then(response => {
          this.$message.success('删除成功!')
          // 重新fetchdata
          this.fetchData()
        })
      }).catch(() => {
        this.$message.info('已取消删除!')
      })
    },
    // 批量删除
    batchremote() {
      const delids = this.multipleSelection.map((e, i, arr) => {
        return e.id
      })
      if (delids.length > 0) {
        this.$confirm('你确定要删除这' + delids.length + '条记录吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          batchrmPositions(delids).then(response => {
            if (response.ok) {
              this.$message.success('删除成功')
              // 重新fetchdata
              this.fetchData()
            } else {
              this.$message.error(response.msg)
            }
          })
        }).catch(() => {
          this.$message.info('已取消删除!')
        })
      } else {
        this.$message.warning('请先选择记录!')
      }
    },
    // 处理排序
    handlerSortchange(sortobj) {
      this.conditions.sort = sortobj.prop // 待排序的字段
      this.conditions.order = (sortobj.order === 'descending') ? 'desc' : (sortobj.order === 'ascending' ? 'asc' : '')
      this.fetchData()
    },
    // 确定修改
    suremodify() {
      modifyPosition(this.positionform).then(response => {
        if (response.ok) {
          this.$message.success('修改成功')
          this.dialogFormVisible = false
          // 重新featchdada
          this.fetchData()
        } else {
          this.$message.error('修改失败')
        }
      })
    },
    // 按照条件查询
    conditionsearch() {
      this.fetchData()
    },
    // 新增
    addsemester() {
      this.newFormVisible = true
      this.positionform = {}
    },
    // 确定新增
    sureAddSemester() {
      this.positionform.id = ''
      newPosition(this.positionform).then(response => {
        if (response.ok) {
          this.$message.success('添加成功!')
          this.newFormVisible = false
          this.fetchData()
        } else {
          this.$message.error(response.msg)
        }
      })
    }
  }
}
</script>
