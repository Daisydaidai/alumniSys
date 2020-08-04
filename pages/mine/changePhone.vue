<template>
	<view>
		<view class="back-header">
			<u-icon name="arrow-left" color="#444444" size="40" @click="toBack"></u-icon>
			<text class="label-title">更改绑定</text>
		</view>
		<view class="cont">
			<image src="../../static/phone.png" mode="" class="phone"></image>
			<text class="phone-num">{{old_phone}}</text>
			<text class="cur-text">当前绑定手机号</text>
			<view class="input-phone">
				<input placeholder="请输入新手机号" v-model="phone"/>
			</view>
			<view class="verif-code-wrap">
				<input placeholder="请输入验证码"  v-model="code" class="verif-code-input"/>
				<text class="get-verif" @click="getVerCode()">{{reload}}</text>
			</view>
			<button type="default" class="confirm-btn" @click="confirm_phone()">确认绑定</button>
		</view>
		
	</view>
</template>

<script>
	const app = getApp();
	export default {
		data() {
			return {
				old_phone: '',
				phone: '',
				code: '',
				reload: '获取验证码',
				timing: false,
				timer: null,
				count: ''
			};
		},
		onLoad() {
			if(app.reToken()) {
				uni.getStorageSync("access_token")
			}
			this.old_phone = uni.getStorageSync("user_phone");
		},
		methods: {
			toBack() {
				uni.navigateTo({
					url: './account'
				})
			},
			getVerCode() {
				if (this.phone.length != 11) {
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
						console.log(res)
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
			confirm_phone() {
				uni.request({
					url: getApp().globalData.url + 'api/user/mobile/change',
					method: 'POST',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					data: {
						mobile: this.phone,
						code: this.code
					},
					success: (res) => {
						// console.log(res)
						uni.showToast({
							icon: 'none',
							position: 'bottom',
							title: '手机号修改成功',
						});
						uni.reLaunch({
							url: './account'
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
		padding: 32rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		
		.phone {
			width: 128rpx;
			height: 128rpx;
			margin-top: 48rpx;
		}
		
		.phone-num {
			color: #405AFE;
			font-weight: bold;
			font-size: 36rpx;
			margin-top: 32rpx;
		}
		
		.cur-text {
			color: #9FA4B5;
			font-size: 24rpx;
			margin-top: 12rpx;
		}
		
		.input-phone {
			padding: 20rpx 32rpx;
			background-color: #F5F7FF;
			width: 100%;
			margin-top: 32rpx;
			border-radius: 8rpx;
		}
		
		.verif-code-wrap {
			display: flex;
			justify-content: space-between;
			width: 100%;
			margin-top: 32rpx;
			
			.verif-code-input {
				padding: 20rpx 32rpx;
				background-color: #F5F7FF;
				width: 390rpx;
				border-radius: 8rpx;
			}
			
			.get-verif {
				background-color: #405AFE;
				width: 200rpx;
				height: 80rpx;
				line-height: 80rpx;
				text-align: center;
				color: #fff;
				box-shadow:0px 4rpx 8rpx 0px rgba(64,90,254,1);
				border-radius:8rpx;
			}
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
	}
</style>
