<template>
  <div>
    <el-row class="main" :gutter="20">
      <el-col :span="6">
        <el-button class="button" type="primary">
          <i class="el-icon-edit"></i>新增视频</el-button
        >
      </el-col>
      <el-col :span="12">
        <el-select v-model="value" placeholder="商品状态" class="select">
          <el-option style="color:#606266">已上架 </el-option>
          <el-option style="color:#606266"> 已下架 </el-option>
        </el-select>
      </el-col>
      <el-col :span="6" class="search">
        <el-input
          v-model="input"
          placeholder="请输入内容"
          class="input"
        ></el-input>
        <el-button class="searchB" type="primary">
          <i class="el-icon-search"></i>搜索</el-button
        >
      </el-col>
    </el-row>
    <el-table :data="tableData" style="width: 100%" border>
      <el-table-column sortable prop="id" label="ID" width="100" align="center">
      </el-table-column>
      <el-table-column prop="content" label="视频内容" width="500">
          <template slot-scope="{ row }">
          <div style="display: flex">
            <img
              :src="row.cover"
              style="width: 100px; height: 50px; margin-right: 10px"
            />
            <div
              style="
                display: flex;
                flex-direction: column;
                justify-content: space-between;
              "
            >
              <span>{{ row.title }}</span>
              <span style="color: red">￥{{ row.price }}</span>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        prop="sub_count"
        label="订阅量"
        align="center"
        width="100"
      >
      </el-table-column>
      <el-table-column prop="status" label="状态" align="center" width="100">
        <template>
          <el-button size="mini" type="success" plain> 已上架 </el-button>
        </template>
      </el-table-column>
      <el-table-column prop="created_time" label="创建时间" align="center">
      </el-table-column>
      <el-table-column prop="try" label="操作" align="center">
        <template>
          <el-button size="mini" type="primary"> 编辑 </el-button>
          <el-button size="mini" type="primary"> 上架 </el-button>
          <el-button size="mini" type="danger"> 删除 </el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="block">
      <el-pagination
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage1"
        :page-sizes="[20, 40, 60, 80, 100]"
        :page-size="20"
        layout="total, sizes, prev, pager, next, jumper"
        :total="100"
      >
      </el-pagination>
    </div>
  </div>
</template>
  

<script>
import { fetchList, createMedia, updateMedia, deleteMedia } from "@/api/media";

export default {
  data() {
    return {
      currentPage1: 1,
      currentPage2: 2,
      currentPage3: 3,
      currentPage4: 4,
      currentPage5: 5,
      tableData: [],
      formInline: {
        user: "",
        region: "",
      },
    };
  },
  methods: {
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
    },
    async getlist() {
      let res = await fetchList();
      console.log(res);
      if (res.code = 20000) {
        this.tableData = res.data.items;
      }
    },
  },
  created() {
    this.getlist();
  },
};
</script>
<style scoped>
.main {
  margin: 20px;
}
.button {
  margin: 5px 0 0 5px;
}
.searchB {
  margin-right: 0;
}
.input {
  width: 200px;
  height: 36px;
}
.select {
  width: 110px;
  height: 36px;
  float: right;
}

.search {
  float: right;
}
</style>

