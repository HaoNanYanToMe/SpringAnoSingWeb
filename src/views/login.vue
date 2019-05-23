<style lang="less">
	@import './login.less';
</style>

<template>
	<div class="login">
		<div class="login-con">
			<div class="form-con">
				<div class="logos">
					<img src="../images/login-logo.png" alt="">
					<div class="progress">
						<div class="tipTaps">
							<div class="tip">{{tip}}</div>
						</div>
						<Progress :percent="progress" :status="status"></Progress>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	var qs = require('qs');
	import md5 from 'js-md5';
	import store from '../store';
	import Cookies from 'js-cookie';
	import {
		appRouter
	} from '../router/router';

	export default {
		data() {
			return {
				progress: 0,
				tip: '正在为您初始化环境……',
				status: 'active',
				interVal: ''
			};
		},
		methods: {
			//启动定时器
			StartProgress() {
				var _this = this;
				_this.interVal = setInterval(function() {
					if (_this.progress < 100) {
						_this.progress += 1
						if (_this.progress > 0 && _this.progress < 25) {
							_this.tip = '初始化系统环境中'
						} else if (_this.progress > 25 && _this.progress < 50) {
							_this.tip = '检验账号合法性中'
						} else if (_this.progress > 50 && _this.progress < 75) {
							_this.tip = '正在配置系统菜单'
						} else if (_this.progress > 75) {
							_this.tip = '系统就绪欢迎使用'
						}
					} else {
						clearInterval(_this.interVal);
					}
				}, 50);
			},
			//配置动态路由
			appRouterSet() {
				var _this = this;
				_this.StartProgress();

				var uid = Cookies.get("UUID")
				let params = {
					"uid": uid,
				}

				this.$http.exFunction(this,
					"sysUser",
					"getRouter",
					1, params, 0, res => {
						setTimeout(() => {
							if (res.success) {
								Cookies.set('user', Cookies.get('NAME'));
								// 权限菜单过滤相关
								localStorage.setItem('rout', JSON.stringify(res.data));
								_this.$store.commit('updateMenulist');
								setTimeout(() => {
									this.$router.push({
										name: 'home_index'
									});
								}, 3000)
							} else {
								//失败返回
								this.status = 'wrong';
								this.tip = '鉴权失败,3秒后返回登录'
								clearInterval(_this.interVal);
								setTimeout(() => {
									// window.location.href = 'http://bj.xdf.cn/ch/logout'
									window.location.href = 'http://127.0.0.1:8088/logout' 
								}, 3000)
							}
						}, 2500)
					})
			},
		},
		created() {
			this.appRouterSet();
		}
	};
</script>

<style>

</style>
