<template>
  <div class="app-container">
    <div
      class="filter-container"
      style="display: flex; justify-content: space-between"
    >
      <el-dropdown>
        <el-button type="danger" size="small">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item @click.native="changeCommentStatus(true)"
            >禁止评论</el-dropdown-item
          >
          <el-dropdown-item @click.native="changeAccessStatus(true)"
            >禁止访问</el-dropdown-item
          >
        </el-dropdown-menu>
      </el-dropdown>

      <el-input
        v-model="listQuery.title"
        placeholder="用户名"
        style="width: 200px;margin-left:850px "
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >
        搜索
      </el-button>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
      ref="multipleTable"
      @sort-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55"> </el-table-column>
      <el-table-column label="用户" min-width="180px">
        <template slot-scope="{ row }">
          <div style="display: flex;align-items:center">
            <img
              :src="row.user.avatar"
              style="width: 50px; height: 50px; margin-right: 10px;border-radius:100%"
            />
              <span>{{row.user.username}}</span>      
          </div>
        </template>
      </el-table-column>
      <el-table-column label="消费总额" width="110px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.total_consume }}</span>
        </template>
      </el-table-column>

      <el-table-column label="创建时间" width="150px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.created_time }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        width="280"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini"  @click="openInfo(row.user.id)">
            详情
          </el-button>
          <el-button
            size="mini"
            type="success"
            style="width:80px;margin-right:8px"
          >
            联系用户
          </el-button>
          <el-dropdown>
        <el-button type="danger" size="small">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item @click.native="changeCommentStatus(true)"
            >禁止评论</el-dropdown-item
          >
          <el-dropdown-item @click.native="changeAccessStatus(true)"
            >禁止访问</el-dropdown-item
          >
        </el-dropdown-menu>
      </el-dropdown>
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
    <info></info>
  </div>
</template>

<script>
import { fetchList, changeCommentStatus,changeAccessStatus } from "@/api/user";
import waves from "@/directive/waves"; // waves directive
import Tinymce from "@/components/Tinymce";
import { parseTime } from "@/utils";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
import info from "./info"


export default {
  name: "ComplexTable",
  components: { Pagination, Tinymce,info },
  directives: { waves },
  
  data() {
    return {
      tableKey: 0,
      list: [],
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        title: undefined,
      },
     multipleSelection:[]
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      fetchList(this.listQuery).then((response) => {
        this.list = response.data.items;
        this.total = response.data.total;

        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
    },
    openInfo(id){
      this.$refs.info.open(id)
    },
    handleSelectionChange(){},
    changeCommentStatus(){},
    changeAccessStatus(){},
    
//     sortChange(data) {
//       const { prop, order } = data;
//       if (prop === "id") {
//         this.sortByID(order);
//       }
//     },
//     sortByID(order) {
//       if (order === "ascending") {
//         this.listQuery.sort = "+id";
//       } else {
//         this.listQuery.sort = "-id";
//       }
//       this.handleFilter();
//     },
//     resetTemp() {
//       this.temp = {
//         id: undefined,
//         title: 1,
//         status: 1,
//         price: 0,
//         try: "",
//         content: "",
//         cover: "",
//       };
//     },
//     handleCreate() {
//       this.resetTemp();
//       this.dialogStatus = "create";
//       this.dialogFormVisible = true;
//       this.$nextTick(() => {
//         this.$refs["dataForm"].clearValidate();
//       });
//     },
//     createData() {
//       this.$refs["dataForm"].validate((valid) => {
//         if (valid) {
//           this.temp.id = parseInt(Math.random() * 100) + 1024; // mock a id
//           this.temp.author = "vue-element-admin";
//           createArticle(this.temp).then(() => {
//             this.list.unshift(this.temp);
//             this.dialogFormVisible = false;
//             this.$notify({
//               title: "Success",
//               message: "Created Successfully",
//               type: "success",
//               duration: 2000,
//             });
//           });
//         }
//       });
//     },
//     handleUpdate(row) {
//       this.temp = Object.assign({}, row); // copy obj
//       this.temp.timestamp = new Date(this.temp.timestamp);
//       this.dialogStatus = "update";
//       this.dialogFormVisible = true;
//       this.$nextTick(() => {
//         this.$refs["dataForm"].clearValidate();
//       });
//     },
//     updateData() {
//       this.$refs["dataForm"].validate((valid) => {
//         if (valid) {
//           const tempData = Object.assign({}, this.temp);
//           tempData.timestamp = +new Date(tempData.timestamp); // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
//           updateArticle(tempData).then(() => {
//             const index = this.list.findIndex((v) => v.id === this.temp.id);
//             this.list.splice(index, 1, this.temp);
//             this.dialogFormVisible = false;
//             this.$notify({
//               title: "Success",
//               message: "Update Successfully",
//               type: "success",
//               duration: 2000,
//             });
//           });
//         }
//       });
//     },
    

//     getSortClass: function (key) {
//       const sort = this.listQuery.sort;
//       return sort === `+${key}` ? "ascending" : "descending";
//     },
//     handleUploadRemove(file, fileList) {
//       console.log(file, fileList);
//     },
//     handlePictureCardPreview(file) {
//       this.dialogImageUrl = file.url;
//       this.dialogVisible = true;
//     },
//     handleUploadSuccess(response, file, fileList) {
//       console.log(response, file, fileList);
//     },
  },
};
</script>
<style scoped>
</style>
