<template>
	<view class="content">
		<view class="search-wrap">
			<view class="search-cont">
				<image src="../../static/search.png" mode="" class="search-icon"></image>
				<u-input v-model="search_word" type="text" placeholder="请输入姓名、班级、个人介绍内容等" @focus="toSearch" />
			</view>
		</view>
		<view class="no-wrap" v-if="alumni_none">
			<image src="../../static/no_list.png" mode="" class="no-list"></image>
			暂无校友信息哦
		</view>
		<view v-else>
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
			<view class="alumni-wrap">
				<view class="alumni-item" v-for="(item, index) in alumniList" :key="index">
					<view class="avatar-wrap">
						<image :src="item.info.avatar||default_avatar" mode="" class="alumni-avatar"></image>
						<image v-if="item.info.xb=='男'" :src="male_icon" mode="" class="sex-icon"></image>
						<image v-else :src="female_icon" mode="" class="sex-icon"></image>
					</view>
					<view class="alumni-info-wrap">
						<view class="basics-info">
							<view>
								<text class="alumni-name">{{item.info.name}}</text>
								<text class="alumni-class">{{item.info.nj}}级 · {{item.info.zy}}</text>
							</view>
							<image v-if="!item.collected" @click="addCol(item.id, index)" src="../../static/unCollect.png" mode="" class="is-collect"></image>
							<image v-else @click="addCol(item.id, index)" src="../../static/collect.png" mode="" class="is-collect"></image>
						</view>
						<view class="prof-label" style="display: flex;align-items: center;">
							<view class="hy-tags">
								<text class="hy-logo">行业</text>
								{{item.info.hy}}
							</view>
							<view class="fenge" v-if="item.tags.length !== 0">
							</view>
							<text class="label-item" v-for="(r, i) in item.tags">{{r.title}}</text>
						</view>
						<view class="self-intr">
							{{item.info.jl || '这位同学很懒，什么也没留下'}}
						</view>
						<view class="addr">
							<view class="now-addr">
								<text style="color: #9FA4B5;font-size: 24rpx;margin-right: 16rpx;width: 100rpx;display: inline-block;">单位地区</text>
								{{item.info.company_address || '未填写'}}
							</view>
							<text class="origin-stu"><text style="color: #9FA4B5;font-size: 24rpx;margin-right: 16rpx;width: 100rpx;display: inline-block;">生源地</text>{{item.info.syd || '未填写'}}</text>
						</view>
					</view>
				</view>
				<view class="common more_btn" @click="getAlumniList(page)" v-if="!is_all">
					加载更多
				</view>
				<view class="common more_btn" v-else>
					没有更多了~
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
				page: 1,
				is_all:false,
				listlength:0,
				alumni_none: true
			}
		},
		onLoad(option) {
			// console.log(option)
			if (option.openid) {
				uni.setStorageSync('open_id', option.openid)
			}
			if(option.token){
				var data = option;
				uni.setStorageSync('access_token', option.token)
				// var lastTime = option.expires_in;
				// console.log(option.token)
				uni.setStorageSync("expires_in", option.expires_in);
			  } 
			if(!uni.getStorageSync('access_token')) {
				// console.log('跳转到登录')
				uni.reLaunch({
					url: '../login/login'
				})
			} else {
				app.reToken()
				this.getAlumniList(this.page)
			}
			// return;
			// if(app.reToken()) {
			// 	uni.getStorageSync("access_token")
			// }
			// app.reToken();
		},
		onShow() {
			if(uni.getStorageSync("coll_reload")) {
				this.reloadList();
				uni.removeStorageSync("coll_reload");
			}
		},
		methods: {
			getAlumniList(page) {
				uni.showLoading()
				uni.request({
					url: getApp().globalData.url + 'api/index/index/index',
					method: 'POST',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					data: {
						page: page
					},
					success: (res) => {
						uni.hideLoading()
						// console.log(res)
						if(res.data.errorCode == 0) {
							this.alumniList = this.alumniList.concat(res.data.items.data)
							// console.log(this.alumniList.length)
							// console.log(this.listlength)
							if(this.alumniList.length > 0) {
								this.alumni_none = false
							}
							if(this.listlength == this.alumniList.length || this.alumniList.length < 15) {
								this.is_all = true
							}
							if(this.alumniList.length - this.listlength < 15) {
								this.is_all = true
							}
							this.listlength = this.alumniList.length;
							this.page += 1
						}
						uni.removeStorageSync("coll_reload");
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
							_this.alumniList[i].collected = !_this.alumniList[i].collected;
							if(_this.alumniList[i].collected == true) {
								uni.showToast({
									icon: 'none',
									title: '收藏成功',
								});
							}else {
								uni.showToast({
									icon: 'none',
									title: '取消收藏成功',
								});
							}
							uni.setStorageSync("reload", true);
						}
					},
					fail: (res) => {
						// console.log(res);
						uni.showToast({
							icon: 'none',
							title: '请求失败，请稍后再试',
						});
					},
				})
			},
			reloadList() {
				this.alumniList = [];
				this.page = 1;
				this.is_all = false;
				this.listlength = 0;
				this.getAlumniList(this.page)
			},
			toSearch() {
				uni.navigateTo({
					url: 'search'
				})
			},
			close_dialog() {
				this.dialogCont = false;
			}
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
					font-size: 18rpx;
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
					height: 38rpx;
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
	
	.more_btn{
		width: 100%;
		height: 76rpx;
		background-color: #FFFFFF;
		color: #98A7A0;
		text-align: center;
		font-size: 28rpx;
		line-height: 76rpx;
		padding-left: 0px;
	}
	
	.no-wrap {
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 300rpx;
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
