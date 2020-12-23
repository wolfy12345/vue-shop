<template>
    <el-container class="home-container">
        <el-header>
            <div>
                <img src="../assets/admin-logo.jpg" alt="">
                <span>电商管理系统</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <el-container>
            <el-aside :width="isCollapse ? '64px' : '200px'">
                <div class="toggle-button" @click="toggleCollapse">|||</div>
                <el-menu
                        background-color="#333744"
                        text-color="#fff"
                        active-text-color="#409eff" unique-opened :collapse="isCollapse" :collapse-transition="false" router>
                    <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
                        <template slot="title">
                            <i class="el-icon-location"></i>
                            <span>{{item.authName}}</span>
                        </template>

                        <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id">
                            <template slot="title">
                                <i class="el-icon-menu"></i>
                                <span>{{subItem.authName}}</span>
                            </template>
                        </el-menu-item>
                    </el-submenu>
                </el-menu>
            </el-aside>
            <el-main>
                <router-view></router-view>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
    export default {
        name: "Home",
        data() {
            return {
                menuList: [],
                isCollapse: false
            }
        },
        created() {
            this.getMenuList();
        },
        methods: {
            logout() {
                window.sessionStorage.removeItem("token")
                this.$router.push("/login")
            },
            getMenuList() {
                this.$http.get("menus").then(res => {
                    // console.log(res);
                    const { data } = res;
                    if(data.meta.status !== 200)    return this.$message.error(data.meta.msg)
                    this.menuList = data.data
                })
            },
            toggleCollapse() {
                this.isCollapse = !this.isCollapse
            }
        }
    }
</script>

<style lang="less" scoped>
    .home-container {
        height: 100%;
    }

    .el-header {
        background-color: #373d41;
        display: flex;
        justify-content: space-between;
        padding-left: 0;
        align-items: center;
        color: #fff;
        font-size: 20px;

        > div {
            display: flex;
            align-items: center;
            span {
                margin-left: 15px;
            }
        }

    }
    .el-aside {
        background-color: #333744;

        .el-menu {
            border-right: none;
        }
    }
    .el-main {
        background-color: #eaedf1;
    }

    .toggle-button {
        background-color: #4a5064;
        font-size: 10px;
        line-height: 24px;
        color: #fff;
        text-align: center;
        letter-spacing: 0.2em;
        cursor: pointer;
    }
</style>