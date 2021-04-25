<template>
  <div class="app-container">
    <div class="filter-container" style="display:flex;justify-content: space-between;">
       <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        创建秒杀
      </el-button>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
    >
<el-table-column label="秒杀内容" min-width="180px">
    <template slot-scope="{row}">
        <div style="display:flex;">
            <img :src="row.value.cover" style="width:100px;height:50px;">
            <div style="margin-left:10px;">
                <div>{{row.value.title}}</div>
                <small style="display:block;">原始价格：<span style="color:red;">￥{{row.value.price}}</span></small>
                <small style="display:block;">秒杀价格：<span style="color:red;">￥{{row.price}}</span></small>
            </div>
        </div>
    </template>
</el-table-column>
<el-table-column label="限制人数" width="100" align="center">
    <template slot-scope="{row}">
        <span>{{ row.s_num }}</span>
    </template>
</el-table-column>
<el-table-column label="已使用人数" width="120" align="center">
    <template slot-scope="{row}">
        <span>{{ row.used_num }}</span>
    </template>
</el-table-column>
<el-table-column label="秒杀状态" width="100" align="center">
    <template slot-scope="{row}">
        <span :style=" row.time_status == '秒杀中' ? 'color:red;' : 'color:#bbbbbb;'">{{ row.time_status }}</span>
    </template>
</el-table-column>
<el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
    <template slot-scope="{row,$index}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button size="mini" type="danger" :disabled="row.status == 0" @click="changeStatus(row)">下架</el-button>
    </template>
</el-table-column>
</el-table>

<pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />

<el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
    <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="90px" style="width: 700px; margin-left:50px;">
        <el-form-item label="类型" prop="type">
            <el-select v-model="temp.type" @change="temp.value = null">
                <el-option label="课程" value="course"></el-option>
                <el-option label="专栏" value="column"></el-option>
            </el-select>
        </el-form-item>
        <el-form-item :label="temp.type == 'course' ? '关联课程' : '关联专栏'" prop="value">
            <el-button type="primary" size="mini" @click="connectValue">关联</el-button>
            <div v-if="temp.value" style="display: flex;">
                <el-card shadow="always">
                    <div>
                        <img :src="temp.value.cover">
                    </div>
                    <div>{{ temp.value.title }}</div>
                    <div style="color:red;">￥{{ temp.value.price }}</div>
                </el-card>
            </div>
        </el-form-item>
        <el-form-item label="秒杀价" prop="price">
            <el-input-number v-model="temp.price" size="mini" :min="0" :step="1" :controls="true" controls-position="both">
            </el-input-number>
        </el-form-item>
        <el-form-item label="秒杀人数" prop="s_num">
            <el-input-number v-model="temp.s_num" size="mini" :min="0" :step="1" :controls="true" controls-position="both">
            </el-input-number>
        </el-form-item>
        <el-form-item label="秒杀活动开始时间-结束时间">
            <el-date-picker v-model="timerange" type="datetimerange" start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
        </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
            取消
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
            提交
        </el-button>
    </div>
</el-dialog>
<choose-course ref="chooseCourse"></choose-course>
</div>
</template>

<script>
    import {
        fetchFlashsale,
        createFlashsale,
        updateFlashsale,
    } from '@/api/marketing'
    import chooseCourse from '@/components/chooseCourse/index.vue'
    import waves from '@/directive/waves' // waves directive
    import {
        parseTime,
    } from '@/utils'

    import util from '@/utils/util.js'
    import Pagination from '@/components/Pagination' // secondary package based on el-pagination

    let timeStatusOptions = {
        0: "未开始",
        1: "秒杀中",
        2: "已结束",
        3: "已下架"
    }

    export default {
        name: 'ComplexTable',
        components: {
            Pagination,
            chooseCourse
        },
        directives: {
            waves
        },
        data() {
            return {
                tableKey: 0,
                list: null,
                total: 0,
                listLoading: true,
                listQuery: {
                    page: 1,
                    limit: 20,
                },
                temp: {
                    id: undefined,
                    type: 'course',
                    // 关联课程/专栏
                    value: null,
                    price: 0.00,
                    s_num: 2,
                    start_time: '',
                    end_time: '',
                },
                dialogFormVisible: false,
                dialogStatus: '',
                textMap: {
                    update: '修改',
                    create: '新增'
                },
                rules: {
                    type: [{
                        required: true,
                        message: '类型不能为空',
                        trigger: 'change'
                    }],
                    // 关联课程/专栏
                    value: [{
                        required: true,
                        message: '关联课程/专栏不能为空',
                        trigger: 'blur'
                    }],
                    price: [{
                        required: true,
                        message: '秒杀价不能为空',
                        trigger: 'blur'
                    }],
                    s_num: [{
                        required: true,
                        message: '秒杀人数不能为空',
                        trigger: 'blur'
                    }],
                },
            }
        },
        created() {
            this.getList()
        },
        computed: {
            timerange: {
                get() {
                    return [this.temp.start_time, this.temp.end_time]
                },
                set(val) {
                    this.temp.start_time = val[0]
                    this.temp.end_time = val[1]
                }
            }
        },
        methods: {
            // 关联课程/专栏
            connectValue() {
                this.$refs.chooseCourse.open((val) => {
                    let item = val[0]
                    this.temp.value = {
                        id: item.id,
                        title: item.title,
                        cover: item.cover,
                        price: item.price,
                    }
                    this.$refs.dataForm.clearValidate()
                }, 1)
            },
            getList() {
                this.listLoading = true
                fetchFlashsale(this.listQuery).then(response => {
                    console.log(response)
                    this.list = response.data.items.map(item => {
                        let k = util.formatGroupStatus(item)
                        item.time_status = timeStatusOptions[k]
                        return item
                    })
                    this.total = response.data.total

                    // Just to simulate the time of the request
                    setTimeout(() => {
                        this.listLoading = false
                    }, 1.5 * 1000)
                })
            },
            handleFilter() {
                this.listQuery.page = 1
                this.getList()
            },
            resetTemp() {
                this.temp = {
                    id: undefined,
                    type: 'course',
                    // 关联课程/专栏
                    value: null,
                    price: 0.00,
                    s_num: 2,
                    start_time: '',
                    end_time: '',
                }
            },
            handleCreate() {
                this.resetTemp()
                this.dialogStatus = 'create'
                this.dialogFormVisible = true
                this.$nextTick(() => {
                    this.$refs['dataForm'].clearValidate()
                })
            },
            createData() {
                this.$refs['dataForm'].validate((valid) => {
                    if (this.temp.start_time == '' || this.temp.end_time == '') {
                        return this.$message({
                            message: '活动时间不能为空',
                            type: 'warning',
                        });
                    }
                    if (valid) {
                        createFlashsale(this.temp).then(() => {
                            this.dialogFormVisible = false
                            this.$notify({
                                title: '成功',
                                message: '创建成功',
                                type: 'success',
                                duration: 2000
                            })
                            this.getList()
                        })
                    }
                })
            },
            handleUpdate(row) {
                this.temp = Object.assign({}, row) // copy obj
                this.temp.timestamp = new Date(this.temp.timestamp)
                this.dialogStatus = 'update'
                this.dialogFormVisible = true
                this.$nextTick(() => {
                    this.$refs['dataForm'].clearValidate()
                })
            },
            updateData() {
                this.$refs['dataForm'].validate((valid) => {
                    if (valid) {
                        const tempData = Object.assign({}, this.temp)
                        updateFlashsale(tempData).then(() => {
                            this.dialogFormVisible = false
                            this.$notify({
                                title: '成功',
                                message: '修改成功',
                                type: 'success',
                                duration: 2000
                            })
                        })
                    }
                })
            },
            // 下架
            changeStatus(row) {
                this.$confirm('是否要下架该活动？', '提示', {
                    confirmButtonText: '下架',
                    cancelButtonText: '取消',
                    type: 'warning',
                }).then(action => {
                    row.status = 0
                    row.time_status = '已下架'
                    this.$message({
                        message: '下架成功',
                        type: 'success',
                    });

                });
            }
        }
    }
</script>