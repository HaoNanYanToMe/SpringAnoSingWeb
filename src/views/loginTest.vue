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
				let params = {
					"uid": '4d098ae060b54b1baf0d6d8d77b39563'
				}

				this.$http.exFunction(this,
					"sysUser",
					"getRouter",
					1, params, 0, res => {
						setTimeout(() => {
							Cookies.set('user', '超级管理员');
							Cookies.set('uuid', '4d098ae060b54b1baf0d6d8d77b39563');
							Cookies.set('RID','b5942a6c590546e59fcf4bd51b6a7c0f--');
							Cookies.set('RTYPE',0);
							localStorage.setItem('rout', JSON.stringify(res.data));
							_this.$store.commit('updateMenulist');
							this.$router.push({
								name: 'home_index'
							});

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
