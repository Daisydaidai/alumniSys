<script>
	export default {
		onLaunch: function() {
			// console.log('App Launch')
		},
		onShow: function() {
			// console.log('App Show')
		},
		onHide: function() {
			// console.log('App Hide')
		},
		globalData: {
			url: 'https://mbaxy.demoo.com.cn/'
		},
		methods: {
			reToken() {
				// console.log("er")
				var curTime = new Date();
				// uni.setStorageSync("cur_time", JSON.stringify(curTime));
				var cur = curTime.getTime();
				var expires = uni.getStorageSync('expires_in')
				// console.log("现在的时间")
				// console.log(cur)
				// console.log("登录过期时间")
				// console.log(expires)
				if(cur > expires) {
					// console.log("reee")
					uni.request({
						url: getApp().globalData.url + 'api/user/refresh',
						method: 'POST',
						header: {
							'Authorization': 'Bearer ' + uni.getStorageSync('access_token')
						},
						success: (res) => {
							// console.log("refresh")
							// console.log(res)
							// console.log("refresh 刷新token success")
							if (res.data.errorCode == 500) {
								// console.log("refresh 刷新token errorcode 500")
								uni.showToast({
									icon: 'none',
									title: '登录已过期',
								});
								// console.log("refresh 登录过期");
								uni.setStorageSync("access_token", false);
								uni.setStorageSync('open_id', false)
								uni.reLaunch({
									url: '../../pages/login/login'
								});
								// return false;
							} else {
								// console.log("refresh shuxin token 成功 重新写入缓存")
								uni.setStorageSync("access_token", res.data.items.access_token);
								uni.setStorageSync("expires_in", res.data.items.expires_in);
								return true;
							}
						},
						fail: (res) => {
							// console.log("refresh 刷新token fail 回调")
							// console.log(res)
							uni.showToast({
								icon: 'none',
								title: '登录已过期',
							});
							uni.reLaunch({
								url: '../../pages/login/login'
							});
						}
					})
				}
			},
		}
	}
</script>

<style lang="scss">
	@import "uview-ui/index.scss";
	/*每个页面公共css */
	uni-input {
		min-height: 100%!important;
		
		.uni-input-wrapper {
			line-height: 100%;
			
			.uni-input-placeholder {
				color: #B9BBCA!important;
			}
		}
	}
	
	.u-collapse {
		width: 573rpx;
		
		.u-collapse-head {
			color: #595E75!important;
		}
	}
	
	.uni-collapse-cell--open[data-v-3394cd2a] {
		background-color: #fff!important;
	}
	
	.uni-collapse-cell[data-v-3394cd2a] {
		border-width: 0!important;
	}
	
	.uni-collapse-cell__title-text[data-v-3394cd2a] {
		font-size: 32rpx!important;
		color: #0B102C!important;
		font-weight: bold;
	}
</style>
