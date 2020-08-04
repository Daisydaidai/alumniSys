<template>
	<view>
		<view class="back-header">
			<u-icon name="arrow-left" color="#444444" size="40" @click="toBack"></u-icon>
			<text class="label-title">修改初始密码</text>
		</view>
		<view class="cont">
			<text class="theme-title">修改初始密码</text>
			<text class="theme-reas">账号更安全</text>
			<form action="" class="form-wrap">
				<view class="phone form-item">
					<label class="form-left" for="phoneNum">手机号</label>
					<input class="form-r" type="text" v-model="phone" id="phoneNum" placeholder="请输入手机号"/>
				</view>
				<view class="verif-code form-item">
					<label class="form-left" for="verifCode">验证码</label>
					<view class="form-r">
						<input type="text" v-model="code" id="verifCode" placeholder="请输入验证码"/>
						<text class="verif-btn" style="color: #1677FF;" @click="getVerCode">发送验证码</text>
					</view>
				</view>
				<view class="new-password form-item">
					<label class="form-left" for="oldPass">设置新密码</label>
					<view class="form-r">
						<input class="form-r" type="text" :password="showPassword" v-model="password" id="oldPass" placeholder="请输入新密码(至少6位)"/>
						<view  :class="[!showPassword ? 'open-eye' : 'close-eye']" @click="showPwd"></view>
					</view>
				</view>
				<view class="new-password form-item">
					<label class="form-left" for="newPass">确认新密码</label>
					<view class="form-r">
						<input class="form-r" type="text" :password="showConfirmPwd" v-model="con_password" id="newPass" placeholder="请重新输入新密码(至少6位)"/>
						<view  :class="[!showConfirmPwd ? 'open-eye' : 'close-eye']" @click="showCPwd"></view>
					</view>
				</view>
				<button type="default" class="confirm-btn" @click="confirm_psd">确定</button>
			</form>
		</view>
	</view>
</template>

<script>
	const app = getApp();
	export default {
		data() {
			return {
				my_phone: '',
				phone: '',
				code: '',
				con_password: '',
				password: '',
				showPassword: true,
				showConfirmPwd: true
			};
		},
		onLoad() {
			if(app.reToken()) {
				uni.getStorageSync("access_token")
			}
			this.my_phone = uni.getStorageSync("phone_num");
		},
		methods: {
			toBack() {
				uni.navigateTo({
					url: './account'
				})
			},
			showPwd() {
				this.showPassword = !this.showPassword;
			},
			showCPwd() {
				this.showConfirmPwd = !this.showConfirmPwd;
			},
			getVerCode() {
				if (this.phone != this.my_phone) {
					uni.showToast({
						icon: 'none',
						title: '请输入正确的手机号'
					});
					return false;
				} else if (!this.timing) {
					this.getCode();
					const TIME_COUNT = 60;
					this.reload = '重新获取(' + TIME_COUNT + 's)'
					this.count = TIME_COUNT;
					this.timing = true;
					this.timer = setInterval(() => {
						if (this.count > 0 && this.count <= TIME_COUNT) {
							this.count--;
							this.reload = '重新获取(' + this.count + 's)'
						} else {
							this.timing = false;
							clearInterval(this.timer);
							this.timer = null;
							this.reload = '重新获取'
						}
					}, 1000)
				}
			},
			//获取验证码
			getCode() {
				var requestCode = uni.request({
					url: getApp().globalData.url + 'api/sms',
					method: 'POST',
					data: {
						phone: this.phone
					},
					success: (res) => {
						// console.log(res)
						if (res.statusCode == 500) {
							uni.showToast({
								icon: 'none',
								title: '发送失败，请稍后再试',
							});
							return false;
						} else {
							uni.showToast({
								icon: 'none',
								position: 'bottom',
								title: '发送成功，请注意查收',
							});
						}
					},
					fail: (res) => {
						console.log(res);
						uni.showToast({
							icon: 'none',
							title: '请求失败，请稍后再试',
						});
					},
				})
			},
			confirm_psd() {
				uni.request({
					url: getApp().globalData.url + 'api/user/password/change',
					method: 'POST',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					data: {
						mobile: this.phone,
						code: this.code,
						password: this.password,
						password_confirmation: this.con_password
					},
					success: (res) => {
						// console.log(res)
						uni.showToast({
							icon: 'none',
							position: 'bottom',
							title: '密码修改成功,请重新登录',
						});
						uni.reLaunch({
							url: '../login/login'
						})
					},
					fail: (res) => {
						console.log(res);
						uni.showToast({
							icon: 'none',
							title: '请求失败，请稍后再试',
						});
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
		
		.theme-title {
			color: #142C21;
			font-size: 48rpx;
			font-weight: bold;
		}
		
		.theme-reas {
			color: #9FA4B5;
			font-size: 28rpx;
			margin-top: 8rpx;
		}
		
		.form-wrap {
			.form-item {
				display: flex;
				padding: 24rpx 0;
				border-bottom: 1px solid #F0F2FA;
				
				.form-left {
					width: 170rpx;
					color: #0B102C;
					font-size: 28rpx;
				}
				
				.form-r {
					display: flex;
					align-items: center;
					font-size: 28rpx;
					justify-content: space-between;
					width: 500rpx;
					
					.verif-btn {
						width: 190rpx;
						text-align: right;
					}
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
	
	.open-eye {
		width: 32rpx;
		height: 24rpx;
		background-image: url(../../static/eye.png);
		background-size: 100% 100%;
	}
	
	.close-eye {
		width: 36rpx;
		height: 36rpx;
		background-image: url(../../static/eye-close.png);
		background-size: 100% 100%;
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
</style>
