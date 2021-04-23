<template>
  <div class="app-container">
    <el-row :gutter="20">
      <el-col :span="6"
        ><el-card shadow="never">
          <div slot="header">
            <span>可用余额（元）</span>
            <el-button
              style="float: right"
              type="primary"
              size="small"
              @click="dialogTableVisible = true"
              >申请提现</el-button
            >
          </div>
          <div>
            <span style="font-size: 30px; font-weight: bold">20.00</span>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="never">
          <div slot="header">
            <span>累计收入（元）</span>
          </div>
          <div>
            <span style="font-size: 30px; font-weight: bold">20.00</span>
          </div>
        </el-card></el-col
      >
      <el-col :span="6"
        ><el-card shadow="never">
          <div slot="header">
            <span>待结算金额（元）</span>
          </div>
          <div>
            <span style="font-size: 30px; font-weight: bold">21</span>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="never">
          <div slot="header">
            <span>冻结金额（元）</span>
          </div>
          <div>
            <span style="font-size: 30px; font-weight: bold">20.00</span>
          </div>
        </el-card></el-col
      >
    </el-row>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
    >
      <el-table-column label="交易时间" width="180px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.created_time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="提交账号" min-width="180px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.account }}</span>
        </template>
      </el-table-column>
      <el-table-column label="开户人" width="110px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="提现金额" align="center">
        <template slot-scope="{ row }"> ￥{{ row.price }} </template>
      </el-table-column>
      <el-table-column label="状态" class-name="status-col" width="100">
        <template slot-scope="{ row }">
          <el-tag :type="row.status ? 'success' : 'danger'">
            {{ row.status | statusFilter }}
          </el-tag>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />

    <el-dialog :visible.sync="dialogTableVisible" title="提现">
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="80px"
        style="width: 400px; margin-left: 50px"
      >
        <el-form-item label="提现金额" prop="price">
          <el-input-number
            v-model="temp.price"
            :min="0"
            label="提现金额"
          ></el-input-number>
        </el-form-item>
        <el-form-item label="提现账户" prop="payment_id">
          <el-select
            v-model="temp.payment_id"
            placeholder="请选择"
            style="width: 250px"
          >
            <el-option
              v-for="item in payments"
              :key="item.id"
              :label="item.bank"
              :value="item.id"
            >
              <span style="float: left">{{ item.bank }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{
                item.account
              }}</span>
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button type="primary" @click="createData()"> 确定 </el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { fetchAssets, fetchPayments, Cashout } from "@/api/pay";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
import waves from "@/directive/waves"; // waves directive
import Tinymce from "@/components/Tinymce";

const statusOptions = {
  0: "审核中",
  1: "提现成功",
  2: "提现失败",
};

export default {
  name: "ComplexTable",
  components: {
    Pagination,
    Tinymce,
  },
  directives: {
    waves,
  },
  filters: {
    statusFilter(status) {
      return statusOptions[status];
    },
  },
  data() {
    return {
      detail: {
        no_settle: 21,
      },
      payments:[],
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
      },
      statusOptions,
      showReviewer: false,
      dialogTableVisible: false,
      temp: {
        price: 0,
        payment_id:null
      },
      rules: {
                    price: [{
                        trigger: 'blur',
                        validator:(rule,value,callback)=>{
                            if(!value){
                                return callback(new Error('提现金额必填'))
                            }
                            if(value > this.detail.no_settle){
                                return callback(new Error('提现金额不能大于'+ this.detail.no_settle +'元'))
                            }
                            callback()
                        }
                    }],
                    payment_id: [{
                        required: true,
                        message: '提现账户不能为空',
                        trigger: 'change'
                    }],
                },
    };
  },
  methods: {
    getList() {
      this.listLoading = true;
      fetchAssets(this.listQuery).then((response) => {
        this.list = response.data.items;
        this.total = response.data.total;

        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
     resetTemp(){
         this.temp = {
                    price:0,
                    payment_id:null
                }
     },
     handleCreate() {
                this.resetTemp()
                this.dialogFormVisible = true
                this.$nextTick(() => {
                    this.$refs['dataForm'].clearValidate()
                })
            },
            createData() {
                this.$refs['dataForm'].validate((valid) => {
                    if (valid) {
                        Cashout(this.temp).then((res) => {
                            this.list.unshift(res.data)
                            this.dialogFormVisible = false
                            this.$notify({
                                title: 'Success',
                                message: '申请成功，等待审核',
                                type: 'success',
                                duration: 2000
                            })
                        })
                    }
                })
            },
  },
  created(){
      this.getList()

      fetchPayments().then(res=>{
          this.payments = res.data.items
      })
  }
};
</script>

<style scoped>
.el-row {
  margin-bottom: 20px;
}
.el-col {
  border-radius: 4px;
}
</style>
