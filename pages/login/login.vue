<template>
	<view>
		<view class="cont">
			<!-- <text class="theme-title">欢迎MBA校友登录</text> -->
			<image src="../../static/logo.png" mode="" class="logo"></image>
			<text class="theme-reas">这里有你想认识的校友</text>
			<form action="" class="form-wrap">
				<view class="phone form-item">
					<label class="form-left" for="phoneNum">
						<image class="img-size" src="../../static/self.png" mode=""></image>
						学号
					</label>
					<input class="form-r" type="text" v-model="account" id="phoneNum" placeholder="请输入学号"/>
				</view>
				<view class="password form-item">
					<label class="form-left" for="passwordNum">
						<image class="img-size" src="../../static/self.png" mode=""></image>
						密码
					</label>
					<input class="form-r" type="password" v-model="password" id="passwordNum" placeholder="请输入密码"/>
				</view>
				<text class="text-red">*初始密码为身份证号后6位</text>
				<button type="default" class="confirm-btn" @click="toLogin">登录</button>
			</form>
			
		</view>
		<view class="bottom-wrap">
			<text>Copyright©2015-2020 浙江工商大学MBA学院</text>
			<text>MBA学院电话：0571-88055297</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				account: '',
				password: '',
				open_id : ''
			};
		},
		onLoad() {
			var isWx = this.isWeiXinBrowser();
			this.open_id = uni.getStorageSync('open_id')
			if(this.open_id){
				// console.log(999)
				isWx = false
			}
			if(this.open_id && uni.getStorageSync('access_token')) {
				// console.log("index")
				uni.reLaunch({
					url: '../index/index'
				})
			}
			if(isWx && !this.open_id) {
				console.log("wx")
				window.location.href = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxac1e986e99f1fafa&redirect_uri=https://mbaxy.demoo.com.cn/api/offical/auth&response_type=code&scope=snsapi_userinfo&state=#wechat_redirect'
			}
		},
		methods: {
			isWeiXinBrowser() {
			
			// #ifdef H5
			
			// window.navigator.userAgent属性包含了浏览器类型、版本、操作系统类型、浏览器引擎类型等信息，这个属性可以用来判断浏览器类型
			
			let ua = window.navigator.userAgent.toLowerCase()
			
			// 通过正则表达式匹配ua中是否含有MicroMessenger字符串
			
			if (ua.match(/MicroMessenger/i) == 'micromessenger') {
			
			return true
			
			} else {
			
			return false
			
			}
			
			// #endif
			
			return false
			
			},
			toLogin() {
				if(this.account == '') {
					uni.showToast({
						title: '请输入学号',
						icon: 'none',
						position: 'bottom',
					});
				} else if (this.password == '') {
					uni.showToast({
						title: '请输入密码',
						icon: 'none',
						position: 'bottom',
					});
				} else {
					this.login()
				}
			},
			login() {
				uni.showLoading({
					title: '登录中...'
				});
				
				uni.request({
					url: getApp().globalData.url + 'api/login',
					method: 'POST',
					data: {
						account: this.account,
						password: this.password,
						openid: this.open_id
					},
					success: (res) => {
						// console.log('普通登录成功')
						// console.log(res)
						uni.hideLoading();
						// console.log('普通登录成功')
						var data = res.data.items;
						if(res.data.errorCode == 0) {
							uni.setStorageSync("expires_in", data.expires_in);
							uni.setStorageSync("access_token", data.access_token);
							uni.switchTab({
								url: '../index/index'
							})
						}else {
							uni.showToast({
								icon: 'none',
								title: res.data.msg,
							});
							return false;
						}
						
						// console.log(data.expires_in)
						if (res.data.statusCode == 200 || res.statusCode == 422) {
							uni.showToast({
								icon: 'none',
								title: res.data.msg,
							});
							return false;
						}
						if (res.data.errorCode == '401'|| res.data.errorCode == '400') {
							uni.showToast({
								icon: 'none',
								title: res.data.msg,
							});
							return;
						}
						if (res.statusCode == 500) {
							uni.showToast({
								icon: 'none',
								title: '登录失败，请重试',
							});
							return false;
						}
						// else {
						// 	uni.reLaunch({
						// 		url: '../index/index'
						// 	})
						// }
					},
					fail: (res) => {
					
					}
				})
					
				
			}
		}
	}
</script>

<style lang="scss">
	.back-header  {
		padding: 32rpx 18rpx;
		color: #0B102C;
		font-size: 32rpx;
		display: flex;
		
		.label-title {
			width: 100%;
			text-align: center;
			display: inline-block;
		}
	}
	
	.cont {
		display: flex;
		flex-direction: column;
		padding: 32rpx;
		
		.logo {
			width: 500rpx;
			height: 75rpx;
			margin-bottom: 32rpx;
		}
		
		.theme-reas {
			color: #9FA4B5;
			font-size: 28rpx;
			margin-top: 8rpx;
		}
		
		.form-wrap {
			margin-top: 24rpx;
			
			.form-item {
				display: flex;
				align-items: center;
				padding: 20rpx;
				background-color: #F0F2FA;
				margin-top: 24rpx;
				border-radius: 8rpx;
				
				.form-left {
					width: 170rpx;
					color: #595E75;
					font-size: 28rpx;
					font-weight: bold;
					display: flex;
					align-items: center;
				}
				
				.form-r {
					display: flex;
					font-size: 28rpx;
					justify-content: space-between;
					width: 500rpx;
				}
			}
		}
	}
	
	.phone {
		display: flex;
	}
	
	.eye-icon {
		width: 36rpx;
		height: 36rpx;
	}
	
	.confirm-btn {
		width: 100%;
		height: 96rpx;
		line-height: 96rpx;
		text-align: center;
		color: #fff;
		background:rgba(64,90,254,1);
		box-shadow:0px 8rpx 16rpx 0px rgba(64,90,254,0.3);
		border-radius:8rpx;
		margin-top: 48rpx;
	}
	
	.img-size {
		width: 48rpx;
		height: 48rpx;
		margin-right: 8rpx;
	}
	
	.text-red {
		display: block;
		margin-top: 12px;
		color: #F24141;
		font-size: 24rpx;
	}
	
	.bottom-wrap {
		position: fixed;
		left: 0;
		bottom: 32rpx;
		width: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		color: #595E75;
		font-size: 24rpx;
		
		text {
			margin-bottom: 12rpx;
		}
	}
</style>
