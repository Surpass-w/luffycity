<template>
    <div id="Login">
        <el-row>
            <el-col :span="8" :offset="8">
                <div style="color: green;text-align: center">
                    <h1>登录界面
                        <img src="../assets/img/login.png" alt="" style="width: 25px;height: 25px">
                    </h1>
                </div>
            </el-col>
            <el-col :span="8" :offset="8">
                <el-form :model="formLabelLogin" label-width="80px">
                    <el-form-item label="账号">
                        <el-input v-model="formLabelLogin.account" placeholder="用户名/手机号/邮箱"></el-input>
                    </el-form-item>
                    <el-form-item label="密码">
                        <el-input v-model="formLabelLogin.password" prefix-icon="el-icon-key"
                                  show-password></el-input>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button type="primary" @click="login">登录账号</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
    </div>
</template>

<script>
    export default {
        name: "Login",
        data() {
            return {
                formLabelLogin: {
                    account: '',
                    password: '',
                }
            }
        },
        methods: {
            login() {
                this.$axios({
                    url: this.$settings.base_url + '/user/login/',
                    method: 'post',
                    data: JSON.stringify({
                        'username': this.formLabelLogin.account,
                        'password': this.formLabelLogin.password
                    }),
                    headers: {
                        'Content-Type': 'application/json',
                    }
                }).then(response => {
                    console.log(response.data)
                }).catch(errors => {
                    console.log(errors)
                })
            }
        }
    }
</script>

<style scoped>

</style>