<template>
    <div id="Register">
        <el-row>
            <el-col :span="8" :offset="8">
                <div style="color: green;text-align: center">
                    <h1>注册页面
                        <img src="../assets/img/register.png" alt="" style="width: 25px;height: 25px">
                    </h1>
                </div>
            </el-col>
            <el-col :span="8" :offset="8">
                <el-form :model="formLabelRegister" label-width="80px">
                    <el-form-item label="用户名">
                        <el-input v-model="formLabelRegister.username"></el-input>
                    </el-form-item>
                    <el-form-item label="密码">
                        <el-input v-model="formLabelRegister.password" prefix-icon="el-icon-key"
                                  show-password></el-input>
                    </el-form-item>
                    <el-form-item label="确认密码">
                        <el-input v-model="formLabelRegister.re_password" prefix-icon="el-icon-key"
                                  show-password></el-input>
                    </el-form-item>
                    <el-form-item label="手机号">
                        <el-input v-model="formLabelRegister.telephone" prefix-icon="el-icon-phone-outline"></el-input>
                    </el-form-item>
                    <el-form-item label="邮箱">
                        <el-input v-model="formLabelRegister.email"></el-input>
                    </el-form-item>
                    <el-form-item style="text-align: center">
                        <el-button type="primary" @click="register">注册账号</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
    </div>
</template>

<script>
    export default {
        name: "Register",
        data() {
            return {
                formLabelRegister: {
                    username: '',
                    password: '',
                    re_password: '',
                    telephone: '',
                    email: ''
                }
            }
        },
        methods: {
            register() {
                this.$axios({
                    url: this.$settings.base_url + '/user/register/',
                    method: 'post',
                    data: JSON.stringify({
                        'username': this.formLabelRegister.username,
                        'password': this.formLabelRegister.password,
                        're_password': this.formLabelRegister.re_password,
                        'telephone': this.formLabelRegister.telephone,
                        'email': this.formLabelRegister.email
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