<template>
	<view>
		<view class="back-header">
			<u-icon name="arrow-left" color="#444444" size="40" @click="toBack"></u-icon>
			<text class="label-title">账号设置</text>
		</view>
		<view class="cont">
			<view class="cont-list" @click="change_phone">
				<text class="cont-left">更改绑定</text>
				<view class="cont-r">
					<u-icon name="lock" color="#9FA4B5" size="28" v-if="user_phone"></u-icon>
					<text class="num" v-if="!user_phone">请绑定手机号</text>
					<text class="num" v-else>{{user_phone}}</text>
					<u-icon name="arrow-right" color="#9FA4B5" size="28"></u-icon>
				</view>
			</view>
			<view class="cont-list" @click="change_password">
				<text class="cont-left">修改密码</text>
				<view class="cont-r">
					<u-icon name="arrow-right" color="#9FA4B5" size="28"></u-icon>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	const app = getApp();
	export default {
		data() {
			return {
				user_phone: ''
			};
		},
		onLoad() {
			if(app.reToken()) {
				uni.getStorageSync("access_token")
			}
			this.getUserInfo()
		},
		methods: {
			toBack() {
				uni.switchTab({
					url: './index'
				})
			},
			change_phone() {
				uni.navigateTo({
					url: './changePhone'
				})
			},
			change_password() {
				uni.navigateTo({
					url: './password'
				})
			},
			getUserInfo() {
				uni.request({
					url: getApp().globalData.url + 'api/user/info',
					method: 'GET',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						console.log(res)
						var data = res.data.items.user
						if(data.mobile) {
							var phone = data.mobile
							this.user_phone =  phone.replace(/^(\d{3})\d{4}(\d+)/,"$1****$2")
							uni.setStorageSync("phone_num", phone);
							uni.setStorageSync("user_phone", this.user_phone);
						}
						
					},
					fail: (res) => {
					
					}
				})
			},
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
		height: calc(100vh - var(--window-top) - 106rpx);
		background-color: #F5F7FF;
		padding-top: 24rpx;
		
		.cont-list {
			background-color: #fff;
			padding: 32rpx;
			display: flex;
			justify-content: space-between;
			color: #0B102C;
			font-size: 32rpx;
			
			.num {
				font-size: 24rpx;
				color: #9FA4B5;
			}
		}
	}
</style>
