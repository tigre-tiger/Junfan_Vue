<template>
  <div class="app-container">
    <div class="filter-container">
      <el-form ref="form">
        <el-form-item>
          <el-button v-if="hasPerm('mother:add')" type="primary" icon="plus" @click="showCreate">添加
          </el-button>
        </el-form-item>
      </el-form>

      <el-form style="margin-top:30px">
        <el-form-item>
          <el-row :gutter="20">
            <el-col :span="8">
              <el-form-item label="母代码名称" label-width="120px" align="left">
                <el-input v-model="listQuery.name" placeholder="请输入母代码名称" />
              </el-form-item>
            </el-col>
             
            <el-col :span="2">
              <el-form-item label-width="80px">
                <el-button type="primary" icon="el-icon-search" @click="getList">查询
                </el-button>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form-item>
      </el-form>
    </div>
     
    <el-table
      v-loading.body="listLoading"
      :data="list"
      element-loading-text="拼命加载中"
      border
      fit
      max-height="650"
      highlight-current-row
    >
      <el-table-column v-if="hasPerm('mother:update')" align="center" label="管理" width="150">
        <template slot-scope="scope">
          <el-button v-if="hasPerm('mother:update')" size="mini" type="primary" icon="edit" @click="showUpdate(scope.$index)">修改</el-button>
          <el-button
            v-if="hasPerm('mother:delete')"
            size="mini"
            type="danger"
            icon="delete" 
            @click="removeMother(scope.$index)"
          >删除
          </el-button>
        </template>
      </el-table-column>
      <el-table-column align="center" label="名称" width="150" prop="mname" />
      <el-table-column label="拥有sku">
        <template slot-scope="scope">
          <el-table
            :data="scope.row.products"
            border
            fit
            max-height="650"
            highlight-current-row
          >
            <el-table-column label="图片" width="120">
              <template slot-scope="scope">
                <el-image
                  class="table-td-thumb"
                  :src="scope.row.imgurl"
                  :preview-src-list="[scope.row.imgurl]"
                />
              </template>
            </el-table-column>
            <el-table-column align="center" prop="skucode" label="产品sku" width="200" />
            <el-table-column align="center" prop="cnname" label="中文名称" width="200" />
          </el-table> 
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      :current-page="listQuery.pageNum"
      :page-size="listQuery.pageRow"
      :total="totalCount"
      :page-sizes="[10, 20, 50, 100]"
      layout="total, sizes, prev, pager, next, jumper"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form class="small-space" :model="mother" label-position="right" label-width="100px">
        <el-form-item label="母代码名称" required>
          <el-input v-model="mother.mname" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-if="dialogStatus=='create'" type="success" @click="addMother">创 建</el-button>
        <el-button v-else type="primary" @click="updateMother">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import { mapGetters } from 'vuex'
  import request from '../../utils/request';
  export default {
    data() {
      return {
        totalCount: 0, // 分页组件--数据总条数
        list: [], // 表格的数据
        listLoading: false, // 数据加载等待动画
        listQuery: {
          pageNum: 1, // 页码
          pageRow: 50, // 每页条数
          name: ''
        },
        dialogFormVisible: false,
        dialogStatus: 'create',
         textMap: {
          update: '编辑',
          create: '新建母代码'
        },
        mother: {
          mid: '',
          mname: ''
        }
      }
    },
    created() {
      this.getList();
    },

    computed: {
      ...mapGetters([
        'userId'
      ])
    },
    methods: {
      getList() {
        // 查询列表
        this.listLoading = true;
        return request({
          url: "/mother/list",
          method: "post",
          data: this.listQuery
        }).then(data => {
          this.listLoading = false;
          this.list = data.list;
          this.totalCount = data.totalCount;
        })
      },
      handleSizeChange(val) {
        // 改变每页数量
        this.listQuery.pageRow = val
        this.handleFilter();
      },
      handleCurrentChange(val) {
        // 改变页码
        this.listQuery.pageNum = val
        this.getList();
      },
      handleFilter() {
        // 查询事件
        this.listQuery.pageNum = 1
        this.getList()
      },
      getIndex($index) {
        // 表格序号
        return (this.listQuery.pageNum - 1) * this.listQuery.pageRow + $index + 1
      },
      showUpdate($index) {
        const mother = this.list[$index];
        this.mother.mid = mother.mid;
        this.mother.mname = mother.mname;
        this.dialogStatus = "update";
        this.dialogFormVisible = true;
        },
     updateMother() {
         return request({
          url: "/mother/update",
          method: "post",
          data: this.mother
        }).then(data => {
           this.$message({
          message: '更新成功',
          type: 'success'
        });
         this.getList();
          this.dialogFormVisible = false;
        })
     },
     addMother() {
         return request({
          url: "/mother/insert",
          method: "post",
          data: this.mother
        }).then(data => {
           this.$message({
          message: '新增成功',
          type: 'success'
        });
          this.getList();
           this.dialogFormVisible = false;
        })
     },
     showCreate() {
        // 显示新增对话框
        this.mother.mid = '';
        this.mother.mname = '';
        this.dialogStatus = "create"
        this.dialogFormVisible = true
      },
     removeMother($index) {
       const mother = this.list[$index];
         return request({
          url: "/mother/delete",
          method: "get",
          params: { mid: mother.mid }
        }).then(data => {
           this.$message({
          message: '删除成功',
          type: 'success'
        });
         this.getList();
         this.dialogFormVisible = false;
        })
     }
    
    }
  }
</script>
