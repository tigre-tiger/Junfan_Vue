<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button type="primary" icon="plus" @click="showCreate">添加
      </el-button>
      <el-form style="margin-top:30px">
        <el-form-item>
          <el-row :gutter="10">
            <el-col :span="6">
              <el-form-item label="产品sku" label-width="80px" align="left">
                <el-input v-model="listQuery.sku" placeholder="请输入产品sku" />
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="产品名称" label-width="80px">
                <el-input v-model="listQuery.name" placeholder="请输入产品中文名称" />
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="仓库">
                <el-select v-model="listQuery.warehouseid" clearable placeholder="请选择">
                  <el-option
                    v-for="item in warehouses"
                    :key="item.warehouseid"
                    :label="item.warehousename"
                    :value="item.warehouseid"
                  />
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item>
                <el-button type="primary" icon="el-icon-search" @click="getList">查询
                </el-button>
                <el-button type="primary" icon="el-icon-search" @click="exportAll">导出
                </el-button>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form-item>
      </el-form>
    </div>
    <el-table
      v-loading.body="listLoading"
      height="1000"
      :data="list"
      element-loading-text="拼命加载中"
      border
      fit
      max-height="650"
      highlight-current-row
    >
      <el-table-column align="center" label="管理" width="180">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" icon="edit" @click="showUpdate(scope.$index)">修改</el-button>
          <el-button type="danger" size="mini" icon="delete" @click="removeProduct(scope.$index)">删除
          </el-button>
        </template>
      </el-table-column>
      <el-table-column align="center" label="仓库" prop="warehouseName" width="120" />
      <el-table-column align="center" label="图片" width="90">
        <template slot-scope="scope">
          <el-image
            class="table-td-thumb"
            :src="scope.row.imgurl"
            :preview-src-list="[scope.row.imgurl]"
          />
        </template>
      </el-table-column>
      <el-table-column align="center" label="产品sku" prop="skuCode" width="120" />
      <el-table-column align="center" label="中文名称" prop="cnName" width="120" />
      <el-table-column align="center" label="英文名称" prop="enName" width="120" />
      <el-table-column align="center" label="采购价" prop="cost" width="80" />
      <el-table-column align="center" label="长度(cm)" prop="registeredLength" width="80" />
      <el-table-column align="center" label="宽度(cm)" prop="registeredWidth" width="80" />
      <el-table-column align="center" label="高度(cm)" prop="registeredHeight" width="80" /> 
      <el-table-column align="center" label="重量(g)" prop="registeredWeight" width="80" /> 
      <el-table-column align="center" label="头程类型" prop="headwayType" width="80" />
      <el-table-column align="center" label="交货期" prop="deliverytime" width="80" />
      <el-table-column align="center" label="产品等级" prop="productlevel" width="80" /> 
      <el-table-column align="center" label="销售组" prop="salegroupName" width="120" />
      <el-table-column align="center" label="产品组" prop="productgroupName" width="120" /> 
      <el-table-column align="center" label="供应商" prop="supplierName" width="120" />
      <el-table-column label="电池" align="center" prop="scope.row.battery">
        <template slot-scope="scope">
          <el-tag
            :type="scope.row.battery==='Y'?'success':''" 
          >{{ scope.row.battery==='Y'?'带电':'不带电' }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="新品" align="center" prop="scope.row.isNew">
        <template slot-scope="scope">
          <el-tag
            :type="scope.row.isNew==='Y'?'success':''"
          >{{ scope.row.isNew==='Y'?'新品':'旧品' }}</el-tag>
        </template>
      </el-table-column>
  
    </el-table>


    <el-pagination
      :current-page="listQuery.pageIndex"
      :page-size="listQuery.pageSize"
      :total="totalCount"
      :page-sizes="[10, 20, 50, 100]"
      layout="total, sizes, prev, pager, next, jumper"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  
    <el-dialog
      width="30%"
      title="国家选择"
      :visible.sync="innerVisible"
      append-to-body
    >
      <el-select v-model="countryCode" filterable placeholder="请选择">
        <el-option
          v-for="item in countries"
          :key="item.value"
          :label="item.name"
          :value="item.value"
        />
      </el-select>
      <div slot="footer" class="dialog-footer">
        <el-button @click="innerVisible = false">取 消</el-button>
        <el-button type="success" @click="addCountry">确认</el-button>
      </div>
    </el-dialog>
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" :fullscreen="true">
      <el-form
        class="small-space"
        :model="tempProduct"
        label-position="left"
        label-width="120px"
      >
        <el-row :gutter="40">
          <el-col :span="6">
            <el-form-item label="商品编码" required>
              <el-input v-model="tempProduct.skuCode" type="text" />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="中文名称" required>
              <el-input v-model="tempProduct.cnName" type="text" />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="英文名称" required>
              <el-input v-model="tempProduct.enName" type="text" />
            </el-form-item>
          </el-col>
                   
          <el-col :span="6">
            <el-form-item label="品牌">
              <el-input v-model="tempProduct.brandedName" type="text" />
            </el-form-item>
          </el-col>
        </el-row>
          
        <el-row :gutter="40">
          <el-col :span="24">
            <el-form-item label="商品特性">
              <el-radio v-model="tempProduct.battery" label="Y" @click.native.prevent="radioClick1(Y)">带电池</el-radio>
              <el-radio v-model="tempProduct.isPlus" label="Y" @click.native.prevent="radioClick2(Y)">带插座</el-radio>
              <el-radio v-model="tempProduct.branded" label="Y" @click.native.prevent="radioClick3(Y)">品牌</el-radio>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="40">
          <el-col :span="6">
            <el-form-item label="注册长(cm)">
              <el-input v-model="tempProduct.registeredLength" type="text" />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="注册宽(cm)">
              <el-input v-model="tempProduct.registeredWidth" type="text" />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="注册高(cm)">
              <el-input v-model="tempProduct.registeredHeight" type="text" />
            </el-form-item>
          </el-col>
        </el-row>
          
        <el-row :gutter="40">
          <el-col :span="6">
            <el-form-item label="图片url">
              <el-input v-model="tempProduct.imgurl" type="text" />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="头程类型" required>
              <el-select v-model="tempProduct.headwayType" placeholder="请选择">
                <el-option
                  v-for="item in headways"
                  :key="item.value"
                  :label="item.name"
                  :value="item.value"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="产品等级" required>
              <el-select v-model="tempProduct.productlevel" clearable placeholder="请选择">
                <el-option
                  v-for="item in productlevels"
                  :key="item.value"
                  :label="item.name"
                  :value="item.value"
                />
              </el-select>
            </el-form-item>
          </el-col>
                
        </el-row>

        <el-row :gutter="40">
          <el-col :span="6">
            <el-form-item label="销售组" required>
              <el-select v-model="tempProduct.salegroupName" value-key="salegroupid" placeholder="请选择" @change="changesalegroup">
                <el-option
                  v-for="item in salegroups"
                  :key="item.salegroupid"
                  :label="item.salegroupname"
                  :value="item"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="产品组" required>
              <el-select v-model="tempProduct.productgroupName" value-key="productgroupid" placeholder="请选择" @change="changeproductgroup">
                <el-option
                  v-for="item in productgroups"
                  :key="item.productgroupid"
                  :label="item.productgroupname"
                  :value="item"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="目的仓库" required>
              <el-select v-model="tempProduct.warehouseName" value-key="warehouseid" placeholder="请选择" @change="changewarehouse">
                <el-option
                  v-for="item in warehouses"
                  :key="item.warehouseid"
                  :label="item.warehousename"
                  :value="item"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="供应商" required>
              <el-select v-model="tempProduct.supplierName" clearable value-key="supplierID" filterable placeholder="请选择" @change="changesupplier">
                <el-option
                  v-for="item in suppliers"
                  :key="item.supplierID"
                  :label="item.supplierName"
                  :value="item"
                />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="40">
          <el-col :span="6">
            <el-form-item label="交货周期" required>
              <el-input v-model="tempProduct.deliverytime" type="text" />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="采购价">
              <el-input v-model="tempProduct.cost" type="text" />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="参考链接">
              <el-input v-model="tempProduct.displayPageUrl" type="text" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="40">
          <el-col :span="12">
            <el-form-item label="备注">
              <el-input v-model="tempProduct.remark" type="textarea" />
            </el-form-item>
          </el-col>
                  
                 
        </el-row>
          
            
        
        <el-row :gutter="5">
          <el-form-item label="供应商链接1:">
            <el-input v-model="tempProduct.supplierlink1" type="text" />
          </el-form-item>
        </el-row>
        <el-row :gutter="5">
          <el-form-item label="供应商链接2:">
            <el-input v-model="tempProduct.supplierlink2" type="text" />
          </el-form-item>
        </el-row>
        <el-row :gutter="5">
          <el-form-item label="供应商链接3:">
            <el-input v-model="tempProduct.supplierlink3" type="text" />
          </el-form-item>
        </el-row>
        <el-row :gutter="5">
          <el-button type="primary" icon="edit" @click="innerVisible=true">添加国家</el-button>
        </el-row>
        <el-row :gutter="5">
          <el-table :data="tempProduct.countries" border fit highlight-current-row>
            <el-table-column align="center" label="出口国家" prop="countrycode" />
            <el-table-column align="center" label="申报价值">
              <template slot-scope="scope">
                <el-input v-model="scope.row.importprice" />
              </template>
            </el-table-column>
            <el-table-column align="center" label="关税率">
              <template slot-scope="scope">
                <el-input v-model="scope.row.importrate" />
              </template></el-table-column>
          </el-table>
        </el-row>
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>预计销量填写</span>
          </div>
          <el-row :gutter="20">
            <el-col :span="2">
              <el-form-item label="1月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month1" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="2月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month2" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="3月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month3" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="4月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month4" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="5月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month5" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="6月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month6" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="7月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month7" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="8月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month8" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="9月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month9" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="10月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month10" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="11月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month11" type="text" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item label="12月" label-width="50px">
                <el-input v-model="tempProduct.productSell.month12" type="text" />
              </el-form-item>
            </el-col>
          </el-row>
        </el-card>
                
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-if="dialogStatus=='create'" type="success" @click="createProduct">创 建</el-button>
        <el-button v-else type="primary" @click="updateProduct">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import { mapGetters } from 'vuex'
import request from '../../utils/request';
import checkPermission from '@/utils/permission';
  export default {
    data() {
      return {
        totalCount: 0, // 分页组件--数据总条数
        list: [], // 表格的数据
        listLoading: false, // 数据加载等待动画
        listQuery: {
          pageIndex: 1, // 页码
          pageSize: 50, // 每页条数
          sku: '',
          name: '',
          warehouseid: ''
        },
        innerVisible: false,
        countryCode: '',
        warehouse: {},
        salegroup: {},
        productgroup: {},
        mother: {},
        warehouses: [], // 仓库列表
        dialogStatus: 'create',
        dialogFormVisible: false,
        textMap: {
          update: '编辑',
          create: '新建产品'
        },
        headways: [
          {
            value: '空运',
            name: '空运'
          },
          {
            value: '海运',
            name: '海运'
          },
           {
            value: '直邮',
            name: '直邮'
          },
            {
            value: '铁路',
            name: '铁路'
          }
        ],
        countries: [
          {
            value: 'UK',
            name: '英国'
          },
          {
            value: 'DE',
            name: '德国'
          },
           {
            value: 'US',
            name: '美国'
          }
        ],
        productlevels: [
          {
            value: '在售',
            name: '在售'
          },
          {
            value: '停购',
            name: '停购'
          },
           {
            value: '结束',
            name: '结束'
          }
        ],
        productgroups: [],
        salegroups: [],
        mothers: [],
        supplier: {},
        suppliers: [],
        tempProduct: {
          productID: '',
          skuCode: '',
          cnName: '',
          enName: '',
          registeredLength: '',
          registeredWidth: '',
          registeredHeight: '',
          branded: '',
          brandedName: '',
          displayPageUrl: '',
          remark: '',
          inportDeclaredvalue: '',
          exportDeclaredvalue: '',
          battery: '',
          registeredWeight: '',
          isNew: '',
          warehouseID: '',
          warehouseName: '',
          headwayType: '',
          productlevel: '',
          salegroupID: '',
          salegroupName: '',
          productgroupID: '',
          productgroupName: '',
          isActive: '',
          isDelete: '',
          isPlus: '',
          registeredVolume: '',
          imgurl: '',
          cost: '',
          mid: '',
          mcode: '',
          supplierID: '',
          supplierName: '',
          supplierlink1: '',
          supplierlink2: '',
          supplierlink3: '',
          deliverytime: '',
          countries: [],
          productSell: {}
        }
      }
    },
    created() {
      this.getList();
      this.getAllWarehouses();
      this.getSupplier();
      this.getSaleGroup();
      this.getProductGroup();
    },
    computed: {
      ...mapGetters([
        'userId'
      ])
    },
    methods: {
      exportAll() {
          return request({
          url: "/product/excel",
          method: "get",
          responseType: 'blob',
          data: this.listQuery
        }).then(response => {
        const fileName = '产品列表.xlsx';
        // 获取文件名
        console.log("data:" + response.data);
         const objectUrl = URL.createObjectURL(new Blob([response.data]))
        // 文件地址
         const link = document.createElement('a')
          link.download = fileName
        link.href = objectUrl
        link.click()
        }).catch()
      },
    addCountry() {
      var countries = this.tempProduct.countries;
      var code = this.countryCode;
      if (countries !== []) {
        for (var i = 0; i < countries.length; i++) {
        if (countries[i].countrycode === code) {
          return;
        }
      }
      } else {
        countries = [];
      }
      
      const object = {};
      this.$set(object, 'countrycode', code);
      this.$set(object, 'importprice', 0);
      this.$set(object, 'importrate', 0);
      countries.push(object)
      this.tempProduct.countries = countries;
    },
    changeMother(value) {
        this.tempProduct.mid = value.mid;
      },
changeproductgroup(value) {
        this.tempProduct.productID = value.productgroupid;
        this.tempProduct.productgroupName = value.productgroupname;
      },

      changesalegroup(value) {
        this.tempProduct.salegroupID = value.salegroupid;
        this.tempProduct.salegroupName = value.salegroupname;
      },
      changewarehouse(value) {
        this.tempProduct.warehouseID = value.warehouseid;
        this.tempProduct.warehouseName = value.warehousename;
      },


       changesupplier(value) {
        this.tempProduct.supplierID = value.supplierID;
        this.tempProduct.supplierName = value.supplierName;
      },
      getAllWarehouses() {
        return request({
          url: "/warehouse/list",
          method: "get"
        }).then(data => {
          this.warehouses = data.data;
        })
      },
      getAllMotherCode() {
        return request({
          url: "/mother/list",
          method: "get"
        }).then(data => {
          this.mothers = data.data;
        })
      },
      // 获取供应商
       getSupplier() {
        return request({
          url: "/supplier/list",
          method: "get"
        }).then(data => {
          this.suppliers = data.data;
        })
      },
      // 获取销售组
       getSaleGroup() {
        return request({
          url: "/salegroup/list",
          method: "get"
        }).then(data => {
          this.salegroups = data.data;
        })
      },
      // 获取产品组
       getProductGroup() {
        return request({
          url: "/productgroup/list",
          method: "get"
        }).then(data => {
          this.productgroups = data.data;
        })
      },
  
      getList() {
        // 查询列表
        this.listLoading = true;
        request({
          url: "/product/list",
          method: "post",
          data: this.listQuery
        }).then(data => {
          this.listLoading = false;
          this.list = data.data.records;
          this.totalCount = data.data.totalCount;
        })
      },
      handleSizeChange(val) {
        // 改变每页数量
        this.listQuery.pageSize = val
        this.handleFilter();
      },
      handleCurrentChange(val) {
        // 改变页码
        this.listQuery.pageIndex = val
        this.getList();
      },
      handleFilter() {
        // 查询事件
        this.listQuery.pageIndex = 1
        this.getList()
      },
      getIndex($index) {
        // 表格序号
        return (this.listQuery.pageIndex - 1) * this.listQuery.pageSize + $index + 1
      },
      radioClick1() {
        if (this.tempProduct.battery === 'Y') {
      this.tempProduct.battery = 'N';
    } else {
      this.tempProduct.battery = 'Y';
    }
      },
       radioClick2() {
       if (this.tempProduct.isPlus === 'Y') {
      this.tempProduct.isPlus = 'N';
    } else {
      this.tempProduct.isPlus = 'Y';
    }
      },
       radioClick3() {
         if (this.tempProduct.branded === 'Y') {
      this.tempProduct.branded = 'N';
    } else {
      this.tempProduct.branded = 'Y';
    }
      },
      showCreate() {
        // 显示新增对话框
         this.tempProduct.productID = '';
        this.tempProduct.skuCode = '';
        this.tempProduct.cnName = '';
        this.tempProduct.enName = '';
        this.tempProduct.registeredLength = '';
        this.tempProduct.registeredHeight = '';
        this.tempProduct.registeredWidth = '';
        this.tempProduct.branded = '';
        this.tempProduct.brandedName = '';
        this.tempProduct.displayPageUrl = '';
        this.tempProduct.remark = '';
        this.tempProduct.battery = '';
        this.tempProduct.isNew = '';
        this.tempProduct.warehouseID = '';
        this.tempProduct.warehouseName = '';
        this.tempProduct.headwayType = '';
        this.tempProduct.productlevel = '';
        this.tempProduct.salegroupID = '';
        this.tempProduct.salegroupName = '';
        this.tempProduct.productgroupID = '';
        this.tempProduct.isActive = '';
        this.tempProduct.isDelete = '';
        this.tempProduct.isPlus = '';
        this.tempProduct.registeredVolume = '';
        this.tempProduct.imgurl = '';
        this.tempProduct.cost = '';
        this.tempProduct.mid = '';
        this.tempProduct.mcode = '';
        this.tempProduct.supplierID = '';
        this.tempProduct.supplierName = '';
        this.tempProduct.supplierlink1 = '';
        this.tempProduct.supplierlink2 = '';
        this.tempProduct.supplierlink3 = '';
        this.tempProduct.deliverytime = '';
        this.tempProduct.productSell = {};
        this.tempProduct.countries = [];
        this.dialogStatus = "create"
        this.dialogFormVisible = true
      },
      showUpdate($index) {
        const product = this.list[$index];
        this.tempProduct.productID = product.productID;
        this.tempProduct.skuCode = product.skuCode;
        this.tempProduct.cnName = product.cnName;
        this.tempProduct.enName = product.enName;
        this.tempProduct.registeredLength = product.registeredLength;
        this.tempProduct.registeredWidth = product.registeredWidth;
        this.tempProduct.registeredHeight = product.registeredHeight;
        this.tempProduct.branded = product.branded;
        this.tempProduct.brandedName = product.brandedName;
        this.tempProduct.displayPageUrl = product.displayPageUrl;
        this.tempProduct.remark = product.remark;
        this.tempProduct.battery = product.battery;
        this.tempProduct.registeredWeight = product.registeredWeight;
        this.tempProduct.isNew = product.isNew;
        this.tempProduct.warehouseID = product.warehouseID;
        this.tempProduct.productgroupName = product.productgroupName;
        this.tempProduct.warehouseName = product.warehouseName;
        this.tempProduct.headwayType = product.headwayType;
        this.tempProduct.productlevel = product.productlevel;
        this.tempProduct.salegroupID = product.salegroupID;
        this.tempProduct.salegroupName = product.salegroupName;
        this.tempProduct.productgroupID = product.productgroupID;
        this.tempProduct.isActive = product.isActive;
        this.tempProduct.isDelete = product.isDelete;
        this.tempProduct.isPlus = product.isPlus;
        this.tempProduct.registeredVolume = product.registeredVolume;
        this.tempProduct.imgurl = product.imgurl;
        this.tempProduct.cost = product.cost;
        this.tempProduct.mid = product.mid;
        this.tempProduct.mcode = product.mcode;
        this.tempProduct.supplierID = product.supplierID;
        this.tempProduct.supplierName = product.supplierName;
        this.tempProduct.supplierlink1 = product.supplierlink1;
        this.tempProduct.supplierlink2 = product.supplierlink2;
        this.tempProduct.supplierlink3 = product.supplierlink3;
        this.tempProduct.deliverytime = product.deliverytime;
        this.tempProduct.productSell = product.productSell;
        this.tempProduct.countries = product.countries;
        this.dialogStatus = "update"
        this.dialogFormVisible = true
      },
      createProduct() {
        this.tempProduct.productID = '';
        var countries = this.tempProduct.countries;
        if (countries === []) {
            countries = [];
            const object = {};
            this.$set(object, 'countrycode', 'CN');
            this.$set(object, 'importprice', 0);
            this.$set(object, 'importrate', 0);
            countries.push(object);
            this.tempProduct.countries = countries
        }
        // 添加新产品
        return request({
          url: "/product/add",
          method: "post",
          data: this.tempProduct
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      removeProduct($index) {
        const _vue = this;
        this.$confirm('确定删除此产品?', '提示', {
          confirmButtonText: '确定',
          showCancelButton: false,
          type: 'warning'
        }).then(() => {
          const product = this.list[$index];
          const productid = product.productID;
          request({
            url: "/product/delete",
            method: "post",
            params: {
              productID: productid
            }
          }).then(() => {
            _vue.getList()
          }).catch(e => {
          })
        })
      },
      updateProduct() {
        // 修改产品信息
        return request({
          url: "/product/update",
          method: "post",
          data: this.tempProduct
        }).then(() => {
          this.dialogFormVisible = false
           this.getList();
        })
      }
    }
  }
 
</script>
