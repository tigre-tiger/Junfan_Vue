<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button v-permission="['productNew:add']" type="primary" icon="plus" @click="showCreate">添加
      </el-button>
      <el-form style="margin-top:30px">
        <el-form-item>
          <el-row :gutter="10">
            <el-col :span="6">
              <el-form-item label="产品名称" label-width="80px">
                <el-input v-model="listQuery.cnName" placeholder="请输入产品中文名称" />
              </el-form-item>
            </el-col>
            <el-col :span="2">
              <el-form-item>
                <el-button type="primary" icon="el-icon-search" @click="getList">查询
                </el-button>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="10" style="margin-top:30px">
            <el-col :span="12">
              <el-radio-group v-model="listQuery.status" @change="getList">
                <el-radio-button border label="1">待提交</el-radio-button>
                <el-radio-button border label="2">待审核</el-radio-button>
                <el-radio-button border label="3">已通过</el-radio-button>
                <el-radio-button border label="4">已驳回</el-radio-button>
              </el-radio-group>
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
      <el-table-column v-permission="['productNew:update']" align="center" width="150" label="管理">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" icon="edit" @click="showUpdate(scope.$index)">修改</el-button>
          <el-button v-if="scope.row.status<3" size="mini" type="danger" icon="delete" @click="removeNewproduct(scope.$index)">删除
          </el-button>
          <el-button
            v-if="scope.row.status==3"
            size="mini"
            type="primary"
            @click="showUploadProduct(scope.$index)"
          >发布
          </el-button>
        </template>
      </el-table-column>
      <el-table-column align="center" label="图片" width="100">
        <template slot-scope="scope">
          <el-image
            class="table-td-thumb"
            :src="scope.row.imgurl1"
            :preview-src-list="[scope.row.imgurl1]"
          />
        </template>
      </el-table-column>
      <el-table-column align="center" label="中文名称" prop="cnname" />
      <el-table-column align="center" label="英文名称" prop="enname" />
      <el-table-column align="center" label="采购价" prop="cost" />
      <el-table-column align="center" label="长度(cm)" prop="length" />
      <el-table-column align="center" label="宽度(cm)" prop="width" />
      <el-table-column align="center" label="高度(cm)" prop="height" />
      <el-table-column align="center" label="重量(g)" prop="weight" />
      <el-table-column align="center" label="头程类型" prop="headwaytype" />
      <el-table-column align="center" label="交货期" prop="delivertime" />
      <el-table-column align="center" label="供应商" prop="suppliername" />

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
    <el-dialog
      width="30%"
      title="新品发布"
      :visible.sync="innerVisible"
      append-to-body
    >
      <el-form>
        <el-form-item label="产品sku" label-width="80px">
          <el-input v-model="product.sku" placeholder="请输入产品sku" />
        </el-form-item>
        <el-form-item label="供应商" label-width="80px" required>
          <el-select v-model="product.supplierID" clearable filterable placeholder="请选择">
            <el-option
              v-for="item in suppliers"
              :key="item.supplierID"
              :label="item.supplierName"
              :value="item.supplierID"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="选择国家" label-width="80px" required>
          <el-select v-model="product.country" clearable placeholder="请选择">
            <el-option
              v-for="item in countries"
              :key="item.value"
              :label="item.name"
              :value="item.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="目的仓库" label-width="80px" required>
          <el-select v-model="product.warehouseID" clearable placeholder="请选择">
            <el-option
              v-for="item in warehouses"
              :key="item.warehouseid"
              :label="item.warehousename"
              :value="item.warehouseid"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="产品等级" label-width="80px" required>
          <el-select v-model="product.productlevel" clearable placeholder="请选择">
            <el-option
              v-for="item in productlevels"
              :key="item.value"
              :label="item.name"
              :value="item.value"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="innerVisible = false">取 消</el-button>
        <el-button type="success" @click="addNewSku">确认</el-button>
      </div>
    </el-dialog>
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" :fullscreen="true">
      <el-form class="small-space" :model="productnew" label-position="left">
        <div>
          <el-row :gutter="50">
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>中文名称：</p></el-col>
                <el-col :span="14"><el-input v-model="productnew.cnname" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>英文名称：</p></el-col>
                <el-col :span="14"><el-input v-model="productnew.enname" /></el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>供应商产品链接1：</p></el-col>
                <el-col :span="14"><el-input v-model="productnew.linkurl1" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>供应商产品链接2：</p></el-col>
                <el-col :span="14"><el-input v-model="productnew.linkurl2" /></el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>供应商名称：</p></el-col>
                <el-col :span="14"><el-input v-model="productnew.suppliername" /></el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>图片链接1：</p></el-col>
                <el-col :span="14">
                  <el-input v-model="productnew.imgurl1" />
                  <el-image style="width: 100px; height: 100px" :src="productnew.imgurl1" fit="fit" />
                </el-col>
              </el-row>
            </div></el-col>
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>图片链接2：</p></el-col>
                <el-col :span="14">
                  <el-input v-model="productnew.imgurl2" />
                  <el-image style="width: 100px; height: 100px" :src="productnew.imgurl2" fit="fit" />
                </el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>产品尺寸：</p></el-col>
                <el-col :span="4"><el-input v-model="productnew.length" placeholder="长" /></el-col>
                <el-col :span="2"><p /></el-col>
                <el-col :span="4"><el-input v-model="productnew.width" placeholder="宽" /></el-col>
                <el-col :span="2"><p /></el-col>
                <el-col :span="4"><el-input v-model="productnew.height" placeholder="高" /></el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="11"><div class="grid-content ">
              <el-row>
                <el-col :span="8"><p>备注：</p></el-col>
                <el-col :span="14">
                  <el-input v-model="productnew.remark" type="textarea" placeholder="备注" />
                </el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>重量(g)：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.weight" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>头程类型：</p></el-col>
                <el-col :span="12"><el-select v-model="productnew.headwaytype" placeholder="请选择">
                  <el-option
                    v-for="item in headways"
                    :key="item.value"
                    :label="item.name"
                    :value="item.value"
                  />
                </el-select></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>交货期(天)：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.delivertime" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>采购价格(元)：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.cost" /></el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>UK参考链接：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.opplinkUk" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>UK预计日销量：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.exceptsellUk" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>UK利润率：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.profitrateUk" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>UK价格(元)：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.priceUk" /></el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>DE参考链接：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.opplinkDe" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>DE预计日销量：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.exceptsellDe" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>DE利润率：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.profitrateDe" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>DE价格(元)：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.priceDe" /></el-col>
              </el-row>
            </div></el-col>
          </el-row>
          <el-row :gutter="50" class="widthheight">
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>US参考链接：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.opplinkUs" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>US预计日销量：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.exceptsellUs" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>US利润率：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.profitrateUs" /></el-col>
              </el-row>
            </div></el-col>
            <el-col :span="6"><div class="grid-content ">
              <el-row>
                <el-col :span="12"><p>US价格(元)：</p></el-col>
                <el-col :span="12"><el-input v-model="productnew.priceUs" /></el-col>
              </el-row>
            </div></el-col>
          </el-row>
         
        </div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-if="dialogStatus=='create'" type="success" @click="createNewproduct">创 建</el-button>
        <el-button v-else type="primary" @click="updateNewproduct">修 改</el-button>
        <el-button v-if="productnew.status==1" type="success" @click="submitNewproduct">提 交</el-button>
        <el-button v-if="productnew.status==2" type="success" @click="verifyNewproduct">审 核</el-button>
        <el-button v-if="productnew.status==3" type="success" @click="rejectNewproduct">驳 回</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import { mapGetters } from 'vuex'
  import request from '../../utils/request';
    export default {
        name: "Newproduct",
      data() {
        return {
          totalCount: 0, // 分页组件--数据总条数
          list: [], // 表格的数据
          listLoading: false, // 数据加载等待动画
          listQuery: {
            pageNum: 1, // 页码
            pageRow: 50, // 每页条数
            cnName: '',
            status: '1'
          },
          warehouses: [], // 仓库列表
          dialogStatus: 'create',
          dialogFormVisible: false,
          textMap: {
            update: '编辑',
            create: '添加新品'
          },
          suppliers: [],
          innerVisible: false,
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
          productnew: {
            productid: '',
            cnname: '',
            enname: '',
            linkurl1: '',
            linkurl2: '',
            width: '',
            weight: '',
            length: '',
            height: '',
            imgurl1: '',
            imgurl2: '',
            remark: '',
            status: '',
            operatestatus: '',
            headwaytype: '',
            delivertime: '',
            cost: '',
            exceptsellUk: '',
            profitrateUk: '',
            exceptsellDe: '',
            profitrateDe: '',
            exceptsellUs: '',
            profitrateUs: '',
            opplinkDe: '',
            opplinkUk: '',
            opplinkUs: '',
            priceUk: '',
            priceDe: '',
            priceUs: '',
            suppliername: '',
            account: ''

          },
          product: {
            productID: '',
            country: '',
            sku: '',
            supplierID: '',
            warehouseID: '',
            productlevel: ''
            
          }
        }
      },
      created() {
        this.getList();
        this.getAllWarehouses();
        this.getSupplier();
      },
      computed: {
        ...mapGetters([
          'userId'
        ])
      },
      methods: {
        getAllWarehouses() {
          return request({
            url: "/warehouse/list",
            method: "get"
          }).then(data => {
            this.warehouses = data.list;
          })
        },
         // 获取供应商
       getSupplier() {
        return request({
          url: "/supplier/list",
          method: "get"
        }).then(data => {
          this.suppliers = data.list;
        })
      },
      addNewSku() {
            return request({
            url: "/productNew/uploadProduct",
            method: "post",
            data: this.product
          }).then(() => {
            this.innerVisible = false
             this.$message({
              message: "发布成功",
              type: 'success'
            })
          })
      },
        
        openimgurl1() {
          this.$prompt('输入图片链接', {
            confirmButtonText: '确定',
            cancelButtonText: '取消'
          }).then(({ value }) => {
              this.productnew.imgurl1 = value;
          }) 
},
            openimgurl2() {
          this.$prompt('输入图片链接', {
            confirmButtonText: '确定',
            cancelButtonText: '取消'
          }).then(({ value }) => {
              this.productnew.imgurl2 = value;
          }) 
},
        getList() {
          // 查询列表
          this.listLoading = true;
          return request({
            url: "/productNew/list",
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
        showCreate() {
          // 显示新增对话框
          this.productnew.productid = "";
          this.productnew.cnname = "";
          this.productnew.enname = "";
          this.productnew.linkurl1 = "";
          this.productnew.linkurl2 = "";
          this.productnew.suppliername = "";
          this.productnew.length = "";
          this.productnew.width = "";
          this.productnew.height = "";
          this.productnew.weight = "";
          this.productnew.headwaytype = "";
          this.productnew.delivertime = "";
          this.productnew.cost = "";
          this.productnew.priceUk = "";
          this.productnew.opplinkUk = "";
          this.productnew.exceptsellUk = "";
          this.productnew.profitrateUk = "";
          this.productnew.priceDe = "";
          this.productnew.opplinkDe = "";
          this.productnew.exceptsellDe = "";
          this.productnew.profitrateDe = "";
          this.productnew.opplinkUs = "";
          this.productnew.priceUs = "";
          this.productnew.exceptsellUs = "";
          this.productnew.profitrateUs = "";
          this.productnew.remark = "";
          this.productnew.imgurl1 = "";
          this.productnew.imgurl2 = "";
          // 还有图片链接和账期没传
          this.dialogStatus = "create"
          this.dialogFormVisible = true
        },
        showUpdate($index) {
          const productnew = this.list[$index];
          this.productnew.productid = productnew.productid;
          this.productnew.cnname = productnew.cnname;
          this.productnew.enname = productnew.enname;
          this.productnew.linkurl1 = productnew.linkurl1;
          this.productnew.linkurl2 = productnew.linkurl2;
          this.productnew.suppliername = productnew.suppliername;
          this.productnew.length = productnew.length;
          this.productnew.width = productnew.width;
          this.productnew.height = productnew.height;
          this.productnew.weight = productnew.weight;
          this.productnew.headwaytype = productnew.headwaytype;
          this.productnew.delivertime = productnew.delivertime;
          this.productnew.cost = productnew.cost;
          this.productnew.opplinkUs = productnew.opplinkUs;
          this.productnew.exceptsellUs = productnew.exceptsellUs;
          this.productnew.profitrateUs = productnew.profitrateUs;
          this.productnew.priceUs = productnew.priceUs;
          this.productnew.opplinkDe = productnew.opplinkDe;
          this.productnew.exceptsellDe = productnew.exceptsellDe;
          this.productnew.profitrateDe = productnew.profitrateDe;
          this.productnew.priceDe = productnew.priceDe;
          this.productnew.opplinkUk = productnew.opplinkUk;
          this.productnew.exceptsellUk = productnew.exceptsellUk;
          this.productnew.profitrateUk = productnew.profitrateUk;
          this.productnew.priceUk = productnew.priceUk;
          this.productnew.remark = productnew.remark;
          this.productnew.status = productnew.status;
          this.productnew.imgurl1 = productnew.imgurl1;
          this.productnew.imgurl2 = productnew.imgurl2;
          this.dialogStatus = "update"
          this.dialogFormVisible = true
        },
        showUploadProduct($index) {
            const productnew = this.list[$index];
            this.product.productID = productnew.productid;
            this.product.country = '';
            this.product.sku = '';
            this.product.supplierID = '';
             this.product.productlevel = '';
            this.product.warehouseID = '';
            this.innerVisible = true;
        },
        createNewproduct() {
          return request({
            url: "/productNew/add",
            method: "post",
            data: this.productnew
          }).then(() => {
            this.getList();
            this.dialogFormVisible = false
          })
        },
        updateNewproduct() {
          // 修改用户信息
          const _vue = this;
          return request({
            url: "/productNew/update",
            method: "post",
            data: this.productnew
          }).then(() => {
            let msg = "修改成功";
            this.dialogFormVisible = false
            if (this.productid === this.productnew.productid) {
              msg = '修改成功,部分信息重新登录后生效'
            }
            this.$message({
              message: msg,
              type: 'success',
              duration: 1 * 1000,
              onClose: () => {
                _vue.getList();
              }
            })
          })
        },
        submitNewproduct() {
          return request({
            url: "productNew/submit",
            method: "get",
            params: { productID: this.productnew.productid }
          }).then(() => {
            this.getList();
            this.dialogFormVisible = false
          })
        },
        verifyNewproduct() {
          return request({
            url: "/productNew/verify",
            method: "get",
            params: { productID: this.productnew.productid }
          }).then(() => {
            this.getList();
            this.dialogFormVisible = false
          })
        },
        rejectNewproduct() {
          return request({
            url: "/productNew/reject",
            method: "get",
            params: { productID: this.productnew.productid }
          }).then(() => {
            this.getList();
            this.dialogFormVisible = false
          })
        },
        removeNewproduct($index) {
          const _vue = this;
          this.$confirm('确定删除此用户?', '提示', {
            confirmButtonText: '确定',
            showCancelButton: false,
            type: 'warning'
          }).then(() => {
            const productnew = _vue.list[$index];
            _vue.api({
              url: "/productNew/delete",
              method: "get",
              params: { productID: productnew.productid }
            }).then(() => {
              _vue.getList()
            }).catch(() => {
              _vue.$message.error("删除失败")
            })
          })
        },
        // 处理电池
        batterController(val) {
          if (val.battery == 'N') {
            return '不带电'
          } else {
            return '带电'
          }
        }
      }
    }
</script>

<style scoped>
 .widthheight{
   margin-top: 5px;
 }
</style>
