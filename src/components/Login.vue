<template>
    <div class="login">
        <div class="box">
            <i class="el-icon-close" @click="close_login"></i>
            <div class="content">
                <div class="nav">
                    <span :class="{active: login_method === 'is_pwd'}"
                          @click="change_login_method('is_pwd')">密码登录</span>
                    <span :class="{active: login_method === 'is_code'}"
                          @click="change_login_method('is_code')">验证登录</span>
                </div>
                <el-form v-if="login_method === 'is_pwd'">
                    <el-input
                            placeholder="用户名/手机号/邮箱"
                            prefix-icon="el-icon-user"
                            v-model="username"
                            clearable>
                    </el-input>
                    <el-input
                            placeholder="密码"
                            prefix-icon="el-icon-key"
                            v-model="password"
                            clearable
                            show-password>
                    </el-input>
                    <el-button type="primary" @click="login_password">登录</el-button>
                </el-form>
                <el-form v-if="login_method === 'is_code'">
                    <el-input
                            placeholder="手机号/邮箱"
                            prefix-icon="el-icon-user"
                            v-model="account"
                            clearable
                            @blur="check_account">  <!--当失去焦点的时候，触发check_mobile的执行-->
                    </el-input>
                    <el-input
                            placeholder="验证码"
                            prefix-icon="el-icon-chat-line-round"
                            v-model="code"
                            clearable>
                        <template slot="append">
                            <span class="code" @click="send_code">{{ code_interval }}</span>
                        </template>
                    </el-input>
                    <el-button type="primary" @click="code_login">登录</el-button>
                </el-form>
                <div class="foot">
                    <span @click="go_register">立即注册</span>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Login",
        data() {
            return {
                username: '',
                password: '',
                account: '',
                code: '',
                login_method: 'is_pwd',
                code_interval: '获取验证码',
                is_send: false,
            }
        },
        methods: {
            close_login() {
                this.$emit('close')
            },
            go_register() {
                this.$emit('go')
            },
            change_login_method(method) {
                this.login_method = method;
            },
            check_account() {
                if (!this.account) return;
                //字符串.match(/正则表达式/)
                if (!this.account.match(/^1[3-9][0-9]{9}$/) && !this.account.match(/^.+@.+$/)) {
                    this.$message({
                        message: '账号格式有误',
                        type: 'warning',
                        duration: 1000,
                        onClose: () => {
                            this.account = '';
                        }
                    });
                    return false;
                }
                //发送axios请求,去后端校验手机号/邮箱
                this.$axios.get(this.$settings.base_url + '/user/check_account/', {params: {'account': this.account}}).then(response => {
                    console.log(response.data.code);
                    if (response.data.code === 100) {
                        this.is_send = true;
                    } else {
                        this.$message({
                            message: '账号号不存在 ',
                            type: 'warning',
                            duration: 1000,
                            onClose: () => {
                                this.account = '';
                            }
                        });
                    }
                }).catch(errors => {
                    console.log(errors);
                });
            },
            send_code() {
                //this.is_send  是否允许点击按钮
                if (!this.is_send) return;
                this.is_send = false;
                let code_interval_time = 60;
                this.code_interval = "发送中...";
                this.$axios({
                    url: this.$settings.base_url + '/user/send_code/',
                    method: 'get',
                    params: {
                        'account': this.account,
                    }
                }).then(response => {
                    if (response.data.code === 1) {
                        this.$message({
                            message: '发送验证码成功',
                            type: 'success',
                            duration: 1000,
                        });
                    }
                });

                let timer = setInterval(() => { //定时器
                    if (code_interval_time <= 1) {
                        clearInterval(timer);  //如果小于等于1，清除定时器
                        this.code_interval = "获取验证码";
                        this.is_send = true; // 重新回到可以点击的状态
                    } else {
                        code_interval_time -= 1;
                        this.code_interval = `${code_interval_time}秒后再发`;
                    }
                }, 1000); //每隔一秒触发一次
            },

            login_password() {
                if (this.username && this.password) {
                    //发送axios请求
                    this.$axios({
                        url: this.$settings.base_url + '/user/login/',
                        method: 'post',
                        data: JSON.stringify({'username': this.username, 'password': this.password}),
                        headers: {
                            'Content-Type': 'application/json',
                        },
                    }).then(response => {
                        //把用户信息保存到cookie中
                        if (response.data.code === 100) {
                            this.$cookies.set('token', response.data.data.token, '7d');
                            this.$cookies.set('username', response.data.data.username, '7d');
                            //关闭登录窗口
                            this.$emit('close');
                            //给父组件传递一个事件，让它从cookie中取出token和username
                            this.$emit('login_success');
                        } else {
                            this.$message({
                                message: '账号不存在,或者密码错误',
                                type: 'warning',
                            });
                            this.username = '';
                            this.password = '';
                        }
                    }).catch(errors => {
                        console.log(errors);
                    })
                } else {
                    this.$message({
                        message: '用户名或密码不能为空',
                        type: 'warning'
                    })
                }
            },

            code_login() {
                if (this.account && this.code) {
                    //发送axios请求
                    this.$axios({
                        url: this.$settings.base_url + '/user/code_login/',
                        method: 'post',
                        data: JSON.stringify({'account': this.account, 'code': this.code}),
                        headers: {
                            'Content-Type': 'application/json'
                        }
                    }).then(response => {
                        //把用户信息保存到cookie中\
                        console.log(response.data.data);
                        this.$cookies.set('token', response.data.data.token, '7d');
                        this.$cookies.set('username', response.data.data.username, '7d');
                        //关闭登录窗口
                        this.$emit('close');
                        //给父组件传递一个事件，让它从cookie中取出token和username
                        this.$emit('login_success');
                    }).catch(errors => {
                        console.log(errors);
                    })
                } else {
                    this.$message({
                        message: '验证码不能为空',
                        type: 'warning'
                    })
                }
            }
        }
    }
</script>

<style scoped>
    .login {
        width: 100vw;
        height: 100vh;
        position: fixed;
        top: 0;
        left: 0;
        z-index: 10;
        background-color: rgba(0, 0, 0, 0.3);
    }

    .box {
        width: 400px;
        height: 420px;
        background-color: white;
        border-radius: 10px;
        position: relative;
        top: calc(50vh - 210px);
        left: calc(50vw - 200px);
    }

    .el-icon-close {
        position: absolute;
        font-weight: bold;
        font-size: 20px;
        top: 10px;
        right: 10px;
        cursor: pointer;
    }

    .el-icon-close:hover {
        color: darkred;
    }

    .content {
        position: absolute;
        top: 40px;
        width: 280px;
        left: 60px;
    }

    .nav {
        font-size: 20px;
        height: 38px;
        border-bottom: 2px solid darkgrey;
    }

    .nav > span {
        margin: 0 20px 0 35px;
        color: darkgrey;
        user-select: none;
        cursor: pointer;
        padding-bottom: 10px;
        border-bottom: 2px solid darkgrey;
    }

    .nav > span.active {
        color: black;
        border-bottom: 3px solid black;
        padding-bottom: 9px;
    }

    .el-input, .el-button {
        margin-top: 40px;
    }

    .el-button {
        width: 100%;
        font-size: 18px;
    }

    .foot > span {
        float: right;
        margin-top: 20px;
        color: orange;
        cursor: pointer;
    }

    .code {
        color: orange;
        cursor: pointer;
        display: inline-block;
        width: 70px;
        text-align: center;
        user-select: none;
    }
</style>