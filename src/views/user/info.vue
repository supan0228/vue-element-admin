<template>
    <div>
        <el-dialog  :visible.sync="dialogTableVisible" title="用户详情" width="80%" top="5vh">
        <el-container>
            <el-header style="height:12vh">
                <el-row>
                    <el-col :span="6"><p>ID:{{id}}</p></el-col>
                    <el-col :span="6"><p>用户名:{{user.username}}</p></el-col>
                    <el-col :span="6"><p>昵称:{{user.nickname}}</p></el-col>
                    <el-col :span="6"><p>手机号:{{user.phone}}</p></el-col>
                    <el-col :span="6"><p>锁定:{{user.status}}</p></el-col>
                    <el-col :span="6"><p>会员:{{user.user_level}}</p></el-col>
                    <el-col :span="6"><p>会员到期时间:{{user.updated_time}}</p></el-col>
                    <el-col :span="6"><p>注册时间:{{user.created_time}}</p></el-col>
                </el-row>
            </el-header>
            
        </el-container>
    </el-dialog>
    </div>
</template>