<template>
	<div class="login" @keydown.enter="handle">
    <div class="container">
      <p class="title">欢迎登陆</p>
      <Form ref="loginForm" :model="form" :rules="rules">
        <FormItem prop="username">
          <Input v-model="form.username" :disabled="btnDisable" placeholder="请输入用户名" size="large"></Input>
        </FormItem>
        <FormItem prop="password">
          <Input type="password" v-model="form.password" :disabled="btnDisable" placeholder="请输入密码" size="large"></Input>
        </FormItem>
        <FormItem prop="google">
          <Input v-model="form.captcha" :disabled="btnDisable" placeholder="请输入谷歌验证码" size="large"></Input>
        </FormItem>
        <FormItem style="text-align: left">
          <Checkbox v-model="remember">记住密码</Checkbox>
        </FormItem>
        <FormItem>
          <Button @click="handle" type="primary" size="large" long>登录</Button>
        </FormItem>
      </Form>
    </div>
		<!--<div class="login-con">-->
		<!--	<Card :bordered="false">-->
		<!--		<p slot="title">-->
    <!--      欢迎登录-->
		<!--		</p>-->
		<!--		<div class="form-con" v-if="!phoneNum">-->
		<!--			<Form ref="loginForm" :model="form" :rules="rules">-->
		<!--				<FormItem prop="username">-->
		<!--					<Input v-model="form.username" :disabled="btnDisable" placeholder="请输入用户名">-->
		<!--						<span slot="prepend">-->
		<!--							<Icon :size="16" type="person"></Icon>-->
		<!--						</span>-->
		<!--					</Input>-->
		<!--				</FormItem>-->

		<!--				<FormItem prop="password">-->
		<!--					<Input type="password" v-model="form.password" :disabled="btnDisable" placeholder="请输入密码">-->
		<!--						<span slot="prepend">-->
		<!--							<Icon :size="14" type="locked"></Icon>-->
		<!--						</span>-->
		<!--					</Input>-->
		<!--				</FormItem prop="captcha">-->

		<!--				<Row v-show="false">-->
		<!--					<Col span="12">-->
		<!--						<Input v-model="form.captcha" placeholder="验证码" >-->
		<!--						<span slot="prepend">-->
		<!--							<Icon :size="14" type="locked"></Icon>-->
		<!--						</span></Input>-->
		<!--					</Col>-->
		<!--					<Col span="10" offset="2">-->
		<!--						<img :src="logimg" style='vertical-align: middle;width:95%;height:95%' @click="chanloimg"/>-->
		<!--					</Col>-->
		<!--				</Row>-->

		<!--				<FormItem style='margin-top:40px'>-->
		<!--					<Button @click="handle" type="warning" long>登录</Button>-->
		<!--				</FormItem>-->

		<!--				<p style='color:red;text-align:center' v-if="messshow">{{errormessage}}</p>-->
		<!--			</Form>-->
		<!--		</div>-->
		<!--		<div v-if="!!phoneNum">-->
		<!--			<Form>-->
		<!--				<FormItem>-->
		<!--					<p class="phone-num">{{ phoneNum | hidePhoneNum }}</p>-->
		<!--				</FormItem>-->
		<!--				<FormItem>-->
		<!--					<Input placeholder="请输入验证码" v-model="code" :class="{appendBtn: count===0}">-->
		<!--					 	<Button slot="append" v-if="count>0" :disabled="count>0">{{count}}s后重新获取</Button>-->
		<!--					 	<Button slot="append" v-else-if="count===0" type="success" @click="getCodeTwice">获取验证码</Button>-->
		<!--					</Input>-->
		<!--				</FormItem>-->
		<!--				<FormItem>-->
		<!--					<Row>-->
		<!--						<Col span="11">-->
		<!--							<Button @click="handle" type="warning" long>登录</Button>-->
		<!--						</Col>-->
		<!--						<Col span="11" offset="2">-->
		<!--							<Button @click="cancelSignIn" long>取消</Button>-->
		<!--						</Col>-->
		<!--					</Row>-->
		<!--				</FormItem>-->
		<!--			</Form>-->

		<!--		</div>-->
		<!--	</Card>-->
		<!--</div>-->
		<div style="position:absolute;top: 10px;font-size: 14px;margin-top: 10px;margin-left: 30px;color:rgb(255 238 211);text-shadow: 1px 0px 2px #000;width: 400px;text-align: left;border-radius: 5px;padding: 5px 0;"></div>
	</div>
</template>
<script>
import Cookies from "js-cookie";
import store from "../store";

import { setStore, getStore, removeStore } from "@/config/storage";
import { otherRouter, appRouter } from "@/router/router.js";
import { BASICURL, login, signIn, getCodeAgain } from "@/service/getData";

export default {
  data() {
    return {
			timer: '',
			btnDisable: false,
			count: 0,
      form: {
        username: null,
        password: null,
        captcha: null
			},
      remember: false,
			code: null,
			phoneNum: null,
      messshow: false,
      errormessage: null,
      logimg: `${BASICURL}/admin/captcha?cid=ADMIN_LOGIN`,
      rules: {
        username: [{ required: true, message: "不能为空", trigger: "blur" }],
        captcha: [{ required: true, message: "不能为空", trigger: "blur" }],
        password: [{ required: true, message: "不能为空", trigger: "blur" }]
      },
      permissions: {}
    };
  },
  methods: {
		cancelSignIn() {
			Cookies.remove('userPhone');
			window.location.reload();
		},
		getCodeTwice() {
			this.count = 60;
			getCodeAgain({phone: this.phoneNum})
			.then(res => {
			})
			.catch(err => {console.log(err)})
		},
    chanloimg() {
      this.logimg = `${BASICURL}admin/captcha?cid=ADMIN_LOGIN&a=${Math.random().toFixed(
        2
      )}`;
    },
    processPermission(menu) {
      if (menu.name != "") this.permissions[menu.name] = 1;
      if (menu.subMenu != null && menu.subMenu.length > 0) {
        for (var i = 0; i < menu.subMenu.length; i++) {
          this.processPermission(menu.subMenu[i]);
        }
      }
		},
    handle() {
		login(this.form)
				.then(res => {
					if (!res.code) {
						Cookies.set('user', res.data.admin.username, { expires: 7 });
						Cookies.set('userInfo', res.data.admin, { expires: 7 });
						setStore("leftSidebarList", res.data.permissions);
						localStorage.setItem("admin-auth-token", res.data.authToken);
						this.$router.push({ name: "home_index" });
              	   } else this.$Message.error(res.message);
				})
				.catch(err => console.log(err));
    }
  },
  beforeRouteLeave(to, from, next) {
		clearInterval(this.timer)
    window.location.reload();
    next();
  },
  created() {
		this.phoneNum = Cookies.get('userPhone');
		clearInterval(this.timer)
    this.logimg = `${BASICURL}admin/captcha?cid=ADMIN_LOGIN`;
    this.form.username = localStorage.getItem('username') || ''
    this.form.password = localStorage.getItem('password') || ''
    this.remember = Boolean(localStorage.getItem('password'))
	},
	filters: {
		hidePhoneNum(val) {
			return val.split('').fill('*',3,7).join('');
		}
	},
	watch: {
		count(newVal, oldVal) {
			if(newVal>0){
				this.timer = setTimeout(()=> {
					this.count--;
				}, 1000)
			}else{
				clearInterval(this.timer)
			}
		},
		phoneNum(newVal, oldVal) {
			if(!!newVal) this.count = 0;
		},
    remember(val) {
      if (val) {
        localStorage.setItem('username', this.form.username)
        localStorage.setItem('password', this.form.password)
      } else {
        localStorage.removeItem('username')
        localStorage.removeItem('password')
      }
    }
	}
};
</script>
<style scoped lang="less">
.login {
  width: 100%;
  height: 100%;
  position: relative;
  text-align: center;
  background: url("../images/background.png") no-repeat 100% center;
  background-size: 100%;
  .container {
    width: 358px;
    height: 419px;
    background: #FFFFFF;
    box-shadow: 0px 10px 18px 6px rgba(4, 19, 47, 0.15);
    border-radius: 20px;
    position: absolute;
    right: 10%;
    top: 30%;
    padding: 0 30px;
    .title {
      font-size: 24px;
      font-weight: 400;
      color: #000000;
      margin: 20px auto 45px auto;
    }
  }
}
</style>
<style lang="less">
.login {
  .ivu-checkbox-inner {
    border-radius: 100px;
  }
  .ivu-form-item {
    margin-bottom: 20px;
  }
}
</style>


