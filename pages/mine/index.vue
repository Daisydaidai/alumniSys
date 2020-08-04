<template>
	<view class="content">
		<view class="user-info-wrap">
			<view class="user-info-cont">
				<view class="avatar-wrap">
					<image :src="user_info.avatar||default_avatar" mode="" class="alumni-avatar"></image>
					<image v-if="user_info.xb=='男'" :src="male_icon" mode="" class="sex-icon"></image>
					<image v-else :src="female_icon" mode="" class="sex-icon"></image>
				</view>
				<view class="user-info">
					<view>
						<text class="username">{{user_info.name}}</text>
						<text>{{user_info.xh}}</text>
					</view>
					<text>{{user_info.nj}}级 · {{user_info.zy}}</text>
				</view>
			</view>
		</view>
		<view class="collected-wrap cur-5" v-if="!collect_none">
			<view class="collected-header">
				<view class="collected" style="color: #9FA4B5;">
					最近收藏的校友
					<!-- <image src="../../static/collect-list.png" mode="" class="collect-list"></image>
					<text>收藏的校友</text> -->
				</view>
				<!-- <u-icon name="arrow-right" color="#9FA4B5" size="28"></u-icon> -->
			</view>
			<view class="collected-swiper">
				<!-- <view class="collect-none" v-if="collect_none" >暂无收藏</view> -->
				<!-- <view class="no-wrap" v-if="collect_none">
					<image src="../../static/no_list.png" mode="" class="no-list"></image>
					暂无收藏哦
				</view> -->
				<view class="collected-item" v-for="(item, index) in alumniList" :key="index">
					<view class="avatar-wrap">
						<image :src="item.avatar||default_avatar" mode="" class="alumni-avatar"></image>
						<image v-if="item.xb=='男'" :src="male_icon" mode="" class="sex-icon"></image>
						<image v-else :src="female_icon" mode="" class="sex-icon"></image>
					</view>
					<view class="alumni-info-wrap">
						<view class="basics-info">
							<text class="alumni-name">{{item.name}}</text>
							<image v-if="!item.collected" @click="addCol(item.id, index)" src="../../static/unCollect.png" mode="" class="is-collect"></image>
							<image v-else @click="addCol(item.id, index)" src="../../static/collect.png" mode="" class="is-collect"></image>
						</view>
						<view class="alumni-class">{{item.nj}}级 · {{item.zy}}</view>
						<view class="prof-label" style="display: flex;align-items: center;">
							<view class="hy-tags">
								<text class="hy-logo">行业</text>
								{{item.hy}}
							</view>
							<!-- <view class="fenge">
							</view>
							<text class="label-item" v-for="(r, i) in item.tags">{{r.title}}</text> -->
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="collected-wrap">
			<view class="collected-header" @click="collectList">
				<view class="collected">
					<image src="../../static/collect-list.png" mode="" class="collect-list"></image>
					<text>我的收藏</text>
				</view>
				<u-icon name="arrow-right" color="#9FA4B5" size="28"></u-icon>
			</view>
		</view>
		<view class="perfect-info com-wrap"  @click="to_info">
			<view class="com-left">
				<image src="../../static/fillout.png" mode=""></image>
				<text class="com-title">完善资料</text>
			</view>
			<view class="com-right">
				<!-- <text style="margin-right: 10rpx;">方便校友快速找到你</text> -->
				<u-icon name="arrow-right" color="#9FA4B5" size="28"></u-icon>
			</view>
		</view>
		
		<view class="account-setting com-wrap" @click="to_setting">
			<view class="com-left">
				<image src="../../static/setting.png" mode=""></image>
				<text class="com-title">账号设置</text>
			</view>
			<view class="com-right">
				<u-icon name="arrow-right" color="#9FA4B5" size="28"></u-icon>
			</view>
		</view>
		<view class="service-call com-wrap">
			<view class="com-left">
				<image src="../../static/service-phone.png" mode=""></image>
				<text class="com-title">联系我们</text>
			</view>
			<view class="com-right">
				0571-88341268
			</view>
		</view>
		<view class="logout com-wrap" @click="logout">
			<view class="com-left">
				<image src="../../static/logout.png" mode=""></image>
				<text class="com-title">退出账号</text>
			</view>
		</view>
	</view>
</template>

<script>
	const app = getApp();
	export default {
		data() {
			return {
				default_avatar: '../../static/default-avatar.png',
				alumni_avatar: '',
				alumni_sex: 2,
				male_icon: '../../static/male.png',
				female_icon: '../../static/female.png',
				is_collect: 1,
				user_info: {},
				alumniList: [],
				collected: true,
				collect_none: false
			}
		},
		onLoad() {
			// console.log(app.reToken())
			if(app.reToken()) {
				// console.log(123)
				uni.getStorageSync("access_token")
			}
			this.getUserInfo();
			this.getAlumniList();
		},
		onShow() {
			if(uni.getStorageSync("reload")) {
				this.getAlumniList();
				uni.setStorageSync("reload", false);
			}
			if(uni.getStorageSync("coll_reload")) {
				this.getAlumniList();
				uni.setStorageSync("coll_reload", false);
			}
		},
		methods: {
			getUserInfo() {
				uni.request({
					url: getApp().globalData.url + 'api/user/info',
					method: 'GET',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						// console.log(res)
						this.user_info = res.data.items.info;
						uni.setStorageSync("userPhone", res.data.items.user.mobile)
					},
					fail: (res) => {
					
					}
				})
			},
			getAlumniList() {
				uni.request({
					url: getApp().globalData.url + 'api/user/collect/recent',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						// console.log(res)
						this.alumniList = res.data.items
						if(this.alumniList.length > 0) {
							this.collect_none = false
						}else {
							this.collect_none = true
						}
					},
					fail: (res) => {
						
					}
				})
			},
			addCol(id, i) {
				var _this = this;
				var requestCode = uni.request({
					url: getApp().globalData.url + 'api/user/collect',
					method: 'POST',
					data: {
						info_id: id
					},
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						if (res.data.errorCode == 0) {
							uni.showToast({
								icon: 'none',
								title: '取消收藏成功',
							});
							this.getAlumniList()
							uni.setStorageSync("coll_reload", true);
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
			collectList() {
				if(this.collect_none) {
					uni.showToast({
					    icon: 'none',
					    title: '暂无收藏哦',
					});
				}else {
					uni.navigateTo({
						url: './collectList'
					})
				}
			},
			to_info() {
				uni.navigateTo({
					url: './info'
				})
			},
			to_setting() {
				uni.navigateTo({
					url: './account'
				})
			},
			login() {
				uni.navigateTo({
					url: '../login/login'
				})
			},
			logout() {
				uni.showModal({
					title: '提示',
					content: '是否确定退出登录',
					cancelColor: '#BCC9C3',
					confirmColor: '#1DBF74',
					success: function(res) {
						if (res.confirm) {
							var requestCode = uni.request({
								url: getApp().globalData.url + 'api/logout',
								method: 'POST',
								header: {
									Authorization: 'Bearer ' + uni.getStorageSync('access_token')
								},
								success: (res) => {
									// console.log(res)
									if (res.data.errorCode == 0) {
										uni.showToast({
											icon: 'none',
											title: '退出成功',
										});
										uni.setStorageSync('access_token', false)
										uni.reLaunch({
											url: '../login/login'
										})
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
						}
					}
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.user-info-wrap {
		padding: 0 32rpx;
		
		
		.user-info-cont {
			display: flex;
			border-bottom: 1px solid #F0F2FA;
			padding: 32rpx 0;
			
			.avatar-wrap {
				display: flex;
				height: 88rpx;
				
				.alumni-avatar {
					width: 88rpx;
					height: 88rpx;
					border-radius: 50%;
					z-index: 0;
				}
				
				.sex-icon {
					width: 24rpx;
					height: 24rpx;
					position: relative;
					top: 64rpx;
					right: 30rpx;
					z-index: 99;
				}
			}
			
			.user-info {
				display: flex;
				flex-direction: column;
				justify-content: space-between;
				color: #9FA4B5;
				font-size: 24rpx;
				
				.username {
					color: #34353B;
					font-size: 32rpx;
					font-weight: bold;
					margin-right: 12rpx;
				}
			}
		}
		
	}
	
	.cur-5 {
		padding: 26rpx 0 0 0!important;
		border-bottom: 16rpx solid #F5F7FF;
		// border-top: 16rpx solid #F5F7FF;
		
		.prof-label {
			display: flex;
			flex-direction: row;
			flex-wrap: wrap;
		}
	}
	
	.collected-wrap {
		padding: 36rpx 0;
		
		.collected-header {
			display: flex;
			padding: 0 32rpx;
			justify-content: space-between;
			
			.collected {
				color: #0B102C;
				font-size: 32rpx;
				display: flex;
				align-items: center;
				
				.collect-list {
					width: 48rpx;
					height: 48rpx;
					margin-right: 10rpx;
				}
			}
		}
		
		.collected-swiper {
			display: flex;
			flex-wrap: nowrap;
			overflow-x: scroll;
			padding: 20rpx 0 32rpx 32rpx;
			
			.collect-none {
				height: 156rpx;
				line-height: 156rpx;
				text-align: center;
				width: 100%;
				color: #595E75;
				font-size: 28rpx;
			}
			
			.collected-item {
				display: flex;
				padding: 24rpx;
				// width: 388rpx;
				min-height: 156rpx;
				box-shadow:0px 8rpx 16rpx 0px rgba(0,0,0,0.08);
				border-radius:8rpx;
				margin-right: 20rpx;
				
				.avatar-wrap {
					display: flex;
					height: 64rpx;
					
					.alumni-avatar {
						width: 64rpx;
						height: 64rpx;
						border-radius: 50%;
						z-index: 0;
					}
					
					.sex-icon {
						width: 17rpx;
						height: 17rpx;
						position: relative;
						top: 50rpx;
						right: 23rpx;
						z-index: 99;
					}
				}
				
				.alumni-info-wrap {
					font-size: 20rpx;
					color: #9FA4B5;
					flex: 1;
					min-width: 264rpx;
					
					.basics-info {
						display: flex;
						
						.alumni-name {
							color: #34353B;
							font-size: 28rpx;
							margin-right: 16rpx;
						}
						
						.is-collect {
							width: 36rpx;
							height: 36rpx;
							margin-left: auto;
						}
					}
					
					.label-item {
						white-space: nowrap;
						font-size: 18rpx;
						padding: 4rpx 12rpx;
						background-color: #F5F7FF;
						border-radius: 4rpx;
						margin-top: 10rpx;
						margin-right: 12rpx;
					}
					
					.alumni-class {
						white-space: nowrap;
					}
				}
			}
		}
	}
	
	.com-wrap {
		display: flex;
		justify-content: space-between;
		padding: 36rpx 32rpx;
		
		.com-left {
			color: #0B102C;
			font-size: 32rpx;
			display: flex;
			align-items: center;
			
			image {
				width: 48rpx;
				height: 48rpx;
				margin-right: 10rpx;
			}
		}
		
		.com-right {
			color: #9FA4B5;
			font-size: 24rpx;
		}
	}
	
	.no-wrap {
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		padding-right: 32rpx;
		// margin-top: 32rpx;
		color: #9FA4B5;
		font-size: 28rpx;
		
		.no-list {
			width: 160rpx;
			height: 124rpx;
			margin-bottom: 16rpx;
		}
	}
	
	.hy-tags {
		display: flex;
		align-items: center;
		background-color: #F5F7FF;
		padding: 4rpx 8rpx 4rpx 0;
		margin-top: 10rpx;
		border-radius: 8rpx;
		font-size: 18rpx;
		
		.hy-logo {
			text-align: center;
			color: #596FFE;
			background-color: #E5E8FE;
			border-radius: 8rpx;
			margin-right: 8rpx;
			padding: 4rpx;
		}
	}
	
	.fenge {
		width: 2rpx;
		height: 36rpx;
		background-color: #F0F2FA;
		margin: 10rpx 16rpx 0;
	}
</style>
