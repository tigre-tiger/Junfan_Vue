<template>
  <div class="app-container">
    <div class="filter-container">
      <el-form ref="form">
        <el-form-item>
          <el-button v-if="hasPerm('supplier:add')" type="primary" icon="plus" @click="showCreate">添加
          </el-button>
        </el-form-item>
      </el-form>

      <el-form style="margin-top:30px">
        <el-form-item>
          <el-row :gutter="20">
            <el-col :span="8">
              <el-form-item label="供应商名称" label-width="120px" align="left">
                <el-input v-model="listQuery.name" placeholder="请输入供应商名称" />
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
      <el-table-column v-if="hasPerm('user:update')" align="center" label="管理" width="150">
        <template slot-scope="scope">
          <el-button v-if="hasPerm('supplier:update')" size="mini" type="primary" icon="edit" @click="showUpdate(scope.$index)">修改</el-button>
          <el-button
            v-if="hasPerm('supplier:delete')"
            size="mini"
            type="danger"
            icon="delete" 
            @click="removeSupplier(scope.$index)"
          >删除
          </el-button>
        </template>
      </el-table-column>
      <el-table-column align="center" label="供应商名称" prop="suppliername" />
      <el-table-column label="供应商链接" target="_blank" width="90">
        <template slot-scope="scope">
          <el-link type="primary" :href="scope.row.linkurl">供应商链接</el-link>
        </template>
      </el-table-column>
      <el-table-column align="center" width="120" prop="linkperson" label="联系人" />
      <el-table-column align="center" label="联系电话" prop="linkphone" />
      <el-table-column align="center" label="创建人" prop="createname" />
      <el-table-column align="center" label="创建时间" prop="createtime" />
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
      <el-form class="small-space" :model="supplier" label-position="right" label-width="100px">
        <el-form-item label="供应商名称" required>
          <el-input v-model="supplier.suppliername" />
        </el-form-item>
        <el-form-item label="联系人" required>
          <el-input v-model="supplier.linkperson" />
        </el-form-item>
        <el-form-item label="联系电话" required>
          <el-input v-model="supplier.linkphone" />
        </el-form-item>
        <el-form-item label="供应商链接" required>
          <el-input v-model="supplier.linkurl" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-if="dialogStatus=='create'" type="success" @click="addSupplier">创 建</el-button>
        <el-button v-else type="primary" @click="updateSupplier">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import { mapGetters } from 'vuex'



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
          create: '新建供应商'
        },
        supplier: {
          supplierid: '',
          suppliername: '',
          linkperson: '',
          linkphone: '',
          status: '',
          createid: '',
          createtime: '',
          updatetime: '',
          createname: '',
          linkurl: ''
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
        this.api({
          url: "/supplier/listSupplier",
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
        const supplier = this.list[$index];
        this.supplier.supplierid = supplier.supplierid;
        this.supplier.suppliername = supplier.suppliername;
        this.supplier.linkperson = supplier.linkperson;
        this.supplier.linkphone = supplier.linkphone;
        this.supplier.status = supplier.status;
        this.supplier.createid = supplier.createid;
        this.supplier.createname = supplier.createname;
        this.supplier.createtime = supplier.createtime;
        this.supplier.updatetime = supplier.updatetime;
        this.supplier.linkurl = supplier.linkurl;
        this.dialogStatus = "update";
        this.dialogFormVisible = true;
        },
     updateSupplier() {
         this.api({
          url: "/supplier/update",
          method: "post",
          data: this.supplier
        }).then(data => {
           this.$message({
          message: '更新成功',
          type: 'success'
        });
         this.getList();
          this.dialogFormVisible = false;
        })
     },
     addSupplier() {
         this.api({
          url: "/supplier/add",
          method: "post",
          data: this.supplier
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
        this.supplier.supplierid = '';
        this.supplier.suppliername = '';
        this.supplier.linkperson = "";
        this.supplier.linkphone = "";
        this.supplier.status = "";
        this.supplier.createid = "";
        this.supplier.createtime = "";
        this.supplier.updatetime = "";
        this.supplier.createname = "";
        this.supplier.linkurl = "";
        this.dialogStatus = "create"
        this.dialogFormVisible = true
      },
     removeSupplier($index) {
       const supplier = this.list[$index];
         this.api({
          url: "/supplier/delete",
          method: "get",
          params: { supplierID: supplier.supplierid }
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
