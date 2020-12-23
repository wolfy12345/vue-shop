<template>
    <div class="login_container">
        <div class="login_box">
            <div class="login_avatar">
                <img src="../assets/logo.png" alt="">
            </div>

            <el-form ref="loginFormRef" label-width="0px" class="login_form" :model="loginForm" :rules="loginFormRules">
                <el-form-item prop="username">
                    <el-input v-model="loginForm.username" prefix-icon="el-icon-s-custom"></el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input v-model="loginForm.password" prefix-icon="el-icon-lock" type="password"></el-input>
                </el-form-item>
                <el-form-item class="btns">
                    <el-button type="primary" @click="login">登录</el-button>
                    <el-button type="info" @click="resetLoginForm">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Login",
        data() {
            return {
                loginForm: {
                    username: 'admin',
                    password: '123456'
                },
                loginFormRules: {
                    username: [
                        { required: true, message: '请输入用户名', trigger: 'blur' },
                        { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                    ],
                    password: [
                        { required: true, message: '请输入密码', trigger: 'blur' },
                        { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
                    ]
                }
            }
        },
        methods: {
            resetLoginForm() {
                this.$refs.loginFormRef.resetFields();
            },
            login() {
                this.$refs.loginFormRef.validate(valid => {
                    if(!valid)  return;
                    this.$http.post("login", this.loginForm).then(res => {
                        // console.log(res);
                        if(res.data.meta.status == 200) {
                            this.$message.success("登录成功")

                            const token = res.data.data.token;
                            window.sessionStorage.setItem("token", token);
                            this.$router.push("/home")
                        } else {
                            this.$message.error("登录失败")
                        }
                    }).catch(err => {
                        console.log(err);
                    })
                })
            }
        }
    }
</script>

<style lang="less" scoped>
    .login_container {
        background-color: #2b4b6b;
        height: 100%;
    }

    .login_box {
        width: 450px;
        height: 300px;
        background-color: #fff;
        -webkit-border-radius: 3px;
        -moz-border-radius: 3px;
        border-radius: 3px;

        position: absolute;
        top: 50%;
        left: 50%;
        -webkit-transform: translate(-50%, -50%);
        -moz-transform: translate(-50%, -50%);
        -ms-transform: translate(-50%, -50%);
        -o-transform: translate(-50%, -50%);
        transform: translate(-50%, -50%);

        .login_avatar {
            height: 130px;
            width: 130px;
            border: 1px solid #eee;
            border-radius: 50%;
            padding: 10px;
            -webkit-box-shadow: 0px 0px 10px #ddd;
            -moz-box-shadow: 0px 0px 10px #ddd;
            box-shadow: 0px 0px 10px #ddd;
            background-color: #fff;

            /*margin: 0 auto;*/
            position: absolute;
            left: 50%;
            -webkit-transform: translate(-50%, -50%);
            -moz-transform: translate(-50%, -50%);
            -ms-transform: translate(-50%, -50%);
            -o-transform: translate(-50%, -50%);
            transform: translate(-50%, -50%);

            img {
                height: 100%;
                width: 100%;
                border-radius: 50%;
                background-color: #eee;
            }
        }
    }

    .login_form {
        position: absolute;
        bottom: 0;
        width: 100%;
        padding: 0 20px;
        -webkit-box-sizing: border-box;
        -moz-box-sizing: border-box;
        box-sizing: border-box;
    }

    .btns {
        display: flex;
        justify-content: flex-end;
    }
</style>