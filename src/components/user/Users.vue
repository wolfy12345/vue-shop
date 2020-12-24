<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" class="input-with-select" v-model="queryInfo.query" clearable
                              @clear="queryUserList">
                        <el-button slot="append" icon="el-icon-search" @click="queryUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>

            <el-table
                    :data="userList" stripe border>
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column
                        prop="username"
                        label="姓名">
                </el-table-column>
                <el-table-column
                        prop="email"
                        label="邮箱">
                </el-table-column>
                <el-table-column
                        prop="mobile"
                        label="电话">
                </el-table-column>
                <el-table-column
                        prop="role_name"
                        label="角色">
                </el-table-column>
                <el-table-column
                        label="状态">
                    <template v-slot="scope">
                        <el-switch
                                v-model="scope.row.mg_state"
                                active-color="#13ce66" @change="userStateChange(scope.row)">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column
                        label="操作">
                    <template v-slot="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini"></el-button>
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="queryInfo.pagenum"
                    :page-sizes="[1, 2, 5, 10]"
                    :page-size="queryInfo.pagesize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total">
            </el-pagination>
        </el-card>

        <el-dialog
                title="提示"
                :visible.sync="addDialogVisible"
                width="50%" @close="addDialogClosed">

            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Users",
        data() {
            var checkEmail = (rule, value, callback) => {
                const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
                if (regEmail.test(value)) {
                    return callback()
                }

                callback(new Error("请输入合法邮箱"))
            }
            var checkMobile = (rule, value, callback) => {

                const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
                if (regMobile.test(value)) {
                    return callback()
                }
                callback(new Error("请输入合法手机"))
            }

            return {
                queryInfo: {
                    query: '',
                    pagenum: 1,
                    pagesize: 5
                },
                userList: [],
                total: 0,
                addDialogVisible: false,
                addForm: {
                    username: '',
                    password: '',
                    email: '',
                    mobile: ''
                },
                addFormRules: {
                    username: [
                        {required: true, message: '请输入用户名', trigger: 'blur'},
                        {min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur'}
                    ],
                    password: [
                        {required: true, message: '请输入密码', trigger: 'blur'},
                        {min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur'}
                    ],
                    email: [
                        {required: true, message: '请输入邮箱', trigger: 'blur'},
                        {validator: checkEmail, trigger: 'blur'}
                    ],
                    mobile: [
                        {required: true, message: '请输入手机', trigger: 'blur'},
                        {validator: checkMobile, trigger: 'blur'}
                    ]
                }
            }
        },
        created() {
            this.getUserList();
        },
        methods: {
            getUserList() {
                this.$http.get("users", {params: this.queryInfo}).then(res => {
                    const {data} = res;
                    if (data.meta.status !== 200) return this.$message.error("获取用户列表失败")
                    this.userList = data.data.users;
                    this.total = data.data.total;
                })
            },
            handleSizeChange(newSize) {
                this.queryInfo.pagesize = newSize;
                this.getUserList();
            },
            handleCurrentChange(newPage) {
                this.queryInfo.pagenum = newPage;
                this.getUserList();
            },
            userStateChange(userInfo) {
                // console.log(userInfo);
                this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`).then(res => {
                    console.log(res);
                    if (res.data.meta.status !== 200) {
                        userInfo.mg_state = !userInfo.mg_state
                        return this.$message.error(res.data.meta.msg)
                    }
                    this.$message.success(res.data.meta.msg)
                }).catch(err => {
                    console.log(err);
                })
            },
            queryUserList() {
                this.queryInfo.pagenum = 1;
                this.getUserList();
            },
            addDialogClosed() {
                this.$refs.addFormRef.resetFields();
            },
            addUser() {
                this.$refs.addFormRef.validate(valid => {
                    // console.log(valid);
                    if (!valid) return;

                    this.$http.post('users', this.addForm).then(res => {
                        console.log(res);
                        if (res.data.meta.status !== 201) {
                            return this.$message.error(res.data.meta.msg)
                        }
                        this.$message.success("添加用户成功")
                        this.addDialogVisible = false
                        this.getUserList();
                    }).catch(err => {
                        console.log(err);
                    })
                })
            }
        }
    }
</script>

<style scoped>

</style>