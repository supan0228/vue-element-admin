<template>
  <el-form :inline="true" :model="formInline" class="demo-form-inline">
    <el-form-item>
      <el-button type="primary">新增图文</el-button>
    </el-form-item>
    <el-form-item>
      <el-select v-model="formInline.region" placeholder="商品状态">
        <el-option label="已下架" value="已下架"></el-option>
        <el-option label="已上架" value="已上架"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item>
      <el-input v-model="formInline.user" placeholder="标题"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="onSubmit">搜索</el-button>
    </el-form-item>
    <el-table :data="tableData" style="width: 100%" border>
      <el-table-column sortable prop="id" label="ID" width="100" align="center">
      </el-table-column>
      <el-table-column prop="content" label="图文内容" width="500">
      </el-table-column>
      <el-table-column
        prop="sub_count"
        label="订阅量"
        width="100"
        align="center"
      >
      </el-table-column>
      <el-table-column prop="status" label="状态" width="100" align="center">
          <template >
              <el-button
                size="mini"
                type="success" 
                plain 
              >
                已上架
              </el-button>
          </template>
      </el-table-column>
      <el-table-column prop="created_time" label="创建时间" align="center">
      </el-table-column>
      <el-table-column prop="try" label="操作" align="center">
          <template >
              <el-button
                size="mini"
                type="primary" 
              >
                编辑
              </el-button>
              <el-button
                size="mini"
                type="primary" 
                >
                上架
              </el-button>
              <el-button
                size="mini"
                type="danger" 
                >
                删除
              </el-button>
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
  </el-form>
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
      tableData: [
        {
          id: "1",
          content: "王小虎bvhdbdjvbjzdnbkjnbknkvnknk",
          sub_count: "99",
          status: "普陀区",
          created_time: "2021-4-20-13:40",
          try: 200333,
        },
        {
          id: "2",
          content: "王小虎bvhdbdjvbjzdnbkjnbknkvnknk",
          sub_count: "22",
          status: "普陀区",
          created_time: "2021-4-20-14:40",
          try: 200333,
        },
        {
          id: "3",
          content: "王小虎bvhdbdjvbjzdnbkjnbknkvnknk",
          sub_count: "37",
          status: "普陀区",
          created_time: "2021-4-20-15:40",
          try: 200333,
        },
      ],
      formInline: {
        user: "",
        region: "",
      },
    };
  },
  methods: {
    onSubmit() {
      console.log("submit!");
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
    },
  },
};
</script>
<style scoped>
</style>
