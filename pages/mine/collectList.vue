<template>
	<view class="content">
		<view class="back-header">
			<u-icon name="arrow-left" color="#444444" size="40" @click="toBack"></u-icon>
			<text class="label-title">收藏的校友</text>
		</view>
		<view class="dialog-wrap" v-if="dialogCont">
			<view class="dialog-cont">
				<image src="../../static/dialog.png" mode="" class="dialog-icon"></image>
				<view class="dialog-text">
					<text class="main-text">相关敏感信息(手机号/职业等)可向学院校友部门获取</text>
					<text class="other-txt">联系电话：0571-88896662</text>
					<text class="other-txt">工作时间：周一至周五 08:30-12:00 14:00-16:30</text>
				</view>
			</view>
			<image src="../../static/dialog-close.png" mode="" class="dislog-close" @click="close_dialog"></image>
		</view>
		<view class="no-wrap" v-if="collect_none">
			<image src="../../static/no_list.png" mode="" class="no-list"></image>
			收藏已被清空啦
		</view>
		<view class="alumni-wrap">
			<view class="alumni-item" v-for="(item, index) in alumniList" :key="index">
				<view class="avatar-wrap">
					<image :src="item.avatar||default_avatar" mode="" class="alumni-avatar"></image>
					<image v-if="item.xb=='男'" :src="male_icon" mode="" class="sex-icon"></image>
					<image v-else :src="female_icon" mode="" class="sex-icon"></image>
				</view>
				<view class="alumni-info-wrap">
					<view class="basics-info">
						<view>
							<text class="alumni-name">{{item.name}}</text>
							<text class="alumni-class">{{item.nj}}级 · {{item.zy}}</text>
						</view>
						<image v-if="!item.collected" @click="addCol(item.id, index)" src="../../static/unCollect.png" mode="" class="is-collect"></image>
						<image v-else @click="addCol(item.id, index)" src="../../static/collect.png" mode="" class="is-collect"></image>
					</view>
					<view class="prof-label" style="display: flex;align-items: center;">
						<view class="hy-tags">
							<text class="hy-logo">行业</text>
							{{item.hy}}
						</view>
						<view class="fenge" v-if="item.tags.length !== 0">
						</view>
						<text class="label-item" v-for="(r, i) in item.tags">{{r.title}}</text>
					</view>
					<view class="self-intr">
						{{item.jl || '这位同学很懒，什么也没留下'}}
					</view>
					<view class="addr">
						<view class="now-addr">
							<text style="color: #9FA4B5;font-size: 24rpx;margin-right: 16rpx;width: 100rpx;display: inline-block;">单位地区</text>
							{{item.company_address || '未填写'}}
						</view>
						<text class="origin-stu"><text style="color: #9FA4B5;font-size: 24rpx;margin-right: 16rpx;width: 100rpx;display: inline-block;">生源地</text>{{item.syd || '未填写'}}</text>
					</view>
				</view>
			</view>
		</view>
		<!-- <view class="collect-none" v-if="collect_none">
			暂无收藏哦，快去收藏校友吧
		</view> -->
	</view>
</template>

<script>
	const app = getApp();
	export default {
		data() {
			return {
				search_word: '',
				dialogCont: true,
				default_avatar: '../../static/default-avatar.png',
				alumni_avatar: '../../static/alumni_avatar.jpg',
				alumni_sex: 2,
				male_icon: '../../static/male.png',
				female_icon: '../../static/female.png',
				is_collect: 1,
				hoverClass: 'hover2',
				itemStyle: {
					color: '#595E75',
					fontSize: '28rpx',
					marginBottom: '18rpx'
				},
				headStyle: {
					color: '#595E75'
				},
				alumniList: [],
				collected: true,
				collect_none: false
			}
		},
		onLoad() {
			if(app.reToken()) {
				uni.getStorageSync("access_token")
			}
			this.getAlumniList();
		},
		methods: {
			getAlumniList() {
				uni.showLoading()
				uni.request({
					url: getApp().globalData.url + 'api/user/collect',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						uni.hideLoading()
						// console.log(res)
						this.alumniList = res.data.items
						// console.log(this.alumniList)
						if(this.alumniList.length === 0) {
							this.collect_none = true;
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
			toSearch() {
				uni.navigateTo({
					url: './search'
				})
			},
			close_dialog() {
				this.dialogCont = false;
			},
			toBack() {
				uni.switchTab({
					url: './index'
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		width: 100%;
	}
	
	.search-wrap {
		padding: 22rpx 32rpx;
		width: 100%;
		border-bottom: 1px solid #F0F2FA;
		
		.search-cont {
			background-color: #F5F7FF;
			border-radius: 34rpx;
			display: flex;
			align-items: center;
			height: 68rpx;
			padding: 16rpx 24rpx;
			
			.search-icon {
				width: 36rpx;
				height: 36rpx;
				margin-right: 8rpx;
			}
			
			.u-input {
				height: 40rpx;
			}
		}
	}
	
	.dialog-wrap {
		width: 100%;
		background-color: #F6F7FE;
		padding: 32rpx;
		display: flex;
		align-items: flex-start;
		justify-content: space-between;
		
		.dialog-cont {
			display: flex;
			align-items: flex-start;
			
			.dialog-icon {
				width: 32rpx;
				height: 32rpx;
				margin-right: 8rpx;
			}
			
			.dialog-text {
				display: flex;
				flex-direction: column;
				width: 600rpx;
				color: #595E75;
				font-size: 24rpx;
				// line-height: 36rpx;
				
				.main-text {
					color: #0B102C;
					margin-bottom: 8rpx;
				}
			}
		}
		
		.dislog-close {
			width: 36rpx;
			height: 36rpx;
		}
	}
	
	.alumni-wrap {
		padding: 0 32rpx;
		width: 100%;
		
		.alumni-item {
			border-bottom: 1px solid #F0F2FA;
			display: flex;
			padding: 32rpx 0;
			width: 100%;
			
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
			
			.alumni-info-wrap {
				font-size: 24rpx;
				color: #9FA4B5;
				flex: 1;
				
				.basics-info {
					display: flex;
					
					.alumni-name {
						color: #34353B;
						font-size: 32rpx;
						font-weight: bold;
						margin-right: 16rpx;
					}
					
					.is-collect {
						width: 36rpx;
						height: 36rpx;
						margin-left: auto;
					}
				}
				
				.label-item {
					display: inline-block;
					font-size: 20rpx;
					padding: 4rpx 12rpx;
					background-color: #F5F7FF;
					border-radius: 4rpx;
					margin-top: 10rpx;
					margin-right: 12rpx;
				}
			}
			
			.self-intr {
				width: 100%;
				font-size: 24rpx;
				color: #595E75;
				padding: 16rpx 0;
			}
			
			.addr {
				display: flex;
				flex-direction: column;
				// justify-content: space-between;
				font-size: 24rpx;
				color: #595E75;
				
				.now-addr:before {
					content: '';
					display: inline-block;
					width: 36rpx;
					height: 36rpx;
					background-image: url(../../static/now-addr.png);
					background-size: 100% 100%;
					margin-right: 12rpx;
				}
				
				.now-addr {
					display: flex;
					align-items: center;
					margin-bottom: 10rpx;
					// width: 280rpx;
					// overflow: hidden;
					// flex-wrap: nowrap;
				}
				
				.origin-stu:before {
					content: '';
					display: inline-block;
					width: 36rpx;
					height: 36rpx;
					background-image: url(../../static/pre-sch.png);
					background-size: 100% 100%;
					margin-right: 12rpx;
				}
				
				.origin-stu {
					display: flex;
					align-items: center;
				}
			}
		}
	}
	
	.back-header  {
		width: 100%;
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
	
	.collect-none {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		height: 100rpx;
	}
	
	.no-wrap {
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 220rpx;
		color: #9FA4B5;
		font-size: 28rpx;
		
		.no-list {
			width: 240rpx;
			height: 214rpx;
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
