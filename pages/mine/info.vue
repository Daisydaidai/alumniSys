<template>
	<view class="info-cont">
		<view v-if="form_l">
			<view class="back-header">
				<u-icon name="arrow-left" color="#444444" size="40" @click="toBack"></u-icon>
				<text class="label-title">更改绑定</text>
			</view>
			<view class="dialog-info">*表示默认不公开的敏感信息，请放心填写</view>
			<form @submit="" @reset="">
				<view class="my-phone my-style">
					<text class="form-left"><text style="color: #F24141;">*</text>联系电话</text>
					<text class="form-r" v-if="!userPhone" @click="changePhone">请绑定手机号</text>
					<text class="form-r" v-else>{{userPhone}}</text>
					
				</view>
				<view class="my-work-addr my-style">
					<text class="form-left">单位地区</text>
					<view class="from-r form-c" @click="showRegion = true">
						<view class="u-demo-result-line">{{ curRegion || '请选择' }}</view>
						<u-picker mode="region" v-model="showRegion" :defaultRegion="defaultRegion" @confirm="confirmRegion"></u-picker>
						<u-icon name="arrow-right" color="#9FA4B5" size="12"></u-icon>
					</view>
				</view>
				<view class="my-work-unit my-style">
					<label class="form-left" for="workunit"><text style="color: #F24141;">*</text>工作单位</label>
					<input class="form-r" type="text" v-model="work_unit" id="workunit" placeholder="请输入工作单位"/>
				</view>
				<view class="my-post my-style">
					<label class="form-left" for="workunit"><text style="color: #F24141;">*</text>职位</label>
					<input class="form-r" type="text" v-model="my_post" id="workunit" placeholder="请输入职位"/>
				</view>
				<view class="my-ind my-style">
					<text class="form-left">所属行业</text>
					<view class="from-r form-c" @click="showInd = true">
						<view class="u-demo-result-line">{{ curInd || '请选择' }}</view>
						<u-select v-model="showInd" :list="indList" @confirm="confirmInd" label-name="title" value-name="id"></u-select>
						<u-icon name="arrow-right" color="#9FA4B5" size="12"></u-icon>
					</view>
				</view>
				<view class="my-work-addr my-style">
					<text class="form-left">生源地</text>
					<view class="from-r form-c" @click="showStuRegion = true">
						<view class="u-demo-result-line">{{ stuRegion || '请选择' }}</view>
						<u-picker mode="region" v-model="showStuRegion" :defaultRegion="defaultRegion" @confirm="confirmStuRegion"></u-picker>
						<u-icon name="arrow-right" color="#9FA4B5" size="12"></u-icon>
					</view>
				</view>
				<view class="my-pre-school my-style">
					<label class="form-left" for="workunit"><text style="color: #F24141;">*</text>原毕业学校</label>
					<input class="form-r" type="text" v-model="pre_school" id="workunit" placeholder="请输入原毕业学校"/>
				</view>
				<view class="my-intr">
					<text class="form-left">个人介绍</text>
					<textarea placeholder-style="color:#9FA4B5" placeholder="请简单介绍自己，50字以内" class="intr-fill" maxlength="50" v-model="intr"/>
				</view>
				<view class="my-label" @click="toAdd">
					<view class="to-choose">
						<text class="form-left" style="padding: 0 32rpx;">个人标签</text>
						<view style="font-size: 28rpx;color: #9FA4B5;display: flex;align-items: center;"><text style="margin-right: 10rpx;">请选择</text><u-icon name="arrow-right" color="#9FA4B5" size="12"></u-icon></view>
					</view>
					<view style="display: flex;flex-wrap: wrap;padding: 0 32rpx;">
						<view class="my_show_label" v-for="(q,w) in my_labelList" :key="w">{{q.title}}</view>
					</view>
				</view>
			</form>
		</view>
		<!-- 添加标签 -->
		<view class="all-label-wrap" v-if="label_open">
			<view class="add-labal-t">
				<u-icon name="arrow-left" color="#444444" size="40" @click="closeLabel"></u-icon>
				<text class="label-title">添加标签</text>
			</view>
			<view class="label-type">
				<view class="my_label_wrap">
					<text class="my_label_title">我的标签</text>
					<view class="dialog-text">
						最多可以添加5个标签
					</view>
					<view style="display: flex;flex-wrap: wrap;">
						<view class="my_cur_label" @click="del(w)" v-for="(q,w) in my_labelList" :key="w">{{q.title}}</view>
					</view>
				</view>
				<view class="label-item-list" v-for="(item, index) in dataList" :key="index">
					<text class="label-type-title">{{item.title}}</text>
					<view class="label-item-wrap">
						<text @click="labelStatus(lb)" class="label-item" v-for="(lb, i) in item.label" :key="i">{{lb.title}}</text>
					</view>
				</view>
			</view>
			<view class="comp">
				<view class="home-btn" @click="toHome">
					<image src="../../static/index_d.png" mode="" style="width: 48rpx;height: 48rpx;"></image>
				</view>
				<view class="comp-btn" @click="clickComp">完成</view>
			</view>
		</view>
	</view>
</template>

<script>
	const app = getApp();
	import uniCollapse from '@/components/uni-collapse/uni-collapse.vue'
	import uniCollapseItem from '@/components/uni-collapse-item/uni-collapse-item.vue'
	export default {
		components: {uniCollapse,uniCollapseItem},
		data() {
			return {
				form_l: true,
				curRegion: '',
				defaultRegion: ['浙江省', '杭州市', '西湖区'],
				showRegion: false,
				stuRegion: '',
				showStuRegion: false,
				showInd: false,
				curInd: '',
				indList: [],
				label_open: false,
				f_open: true,
				work_unit: '',
				my_post: '',
				pre_school: '',
				intr: '',
				label_a: -1,
				rSelect:[],
				selectList: [],
				selectId: [],
				userPhone: '',
				dataList: [],
				my_labelList: [],
				active_label_id: []
			};
		},
		onLoad() {
			if(app.reToken()) {
				uni.getStorageSync("access_token")
			}
			this.getUserInfo();
			this.getIndList();
			this.getLabelList();
			this.userPhone = uni.getStorageSync("userPhone")
		},
		onUnload() {
			this.changeUserInfo()
			//返回上一页提交表单
		},
		methods: {
			changeUserInfo() {
				uni.request({
					url: getApp().globalData.url + 'api/user/info',
					method: 'PUT',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					data: {
						company: this.work_unit,
						company_address: this.curRegion,
						zw: this.my_post,
						hy: this.curInd,
						syd: this.stuRegion,
						graduated_school: this.pre_school,
						jl: this.intr,
						tags: this.selectId
					},
					success: (res) => {
						// console.log(res)
					},
					fail: (res) => {
					
					}
				})
			},
			getIndList() {
				uni.request({
					url: getApp().globalData.url + 'api/common/hy',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						this.indList = res.data.items
					},
					fail: (res) => {
						
					}
				})
			},
			getLabelList() {
				uni.request({
					url: getApp().globalData.url + 'api/common/tags',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						var data = res.data.items
						for(var i = 0; i < data.length; i ++) {
							this.dataList.push({
								title: data[i].title,
								label: data[i].children
							})
						}
					},
					fail: (res) => {
						
					}
				})
			},
			del(index) {
				this.my_labelList.splice(index, 1)
				this.selectId.splice(index, 1)
			},
			labelStatus(item) {
				var id_tags = item.id
				var id_tags_str = id_tags.toString()
				
				if(this.selectId.indexOf(id_tags_str) !== -1) {
					uni.showToast({
						icon: 'none',
						title: '不可重复选择标签哦',
					});
				}	
				if(this.my_labelList.length >= 5) {
					uni.showToast({
						icon: 'none',
						title: '最多可选5个标签',
					});
				}
				if(this.my_labelList.length < 5 && this.selectId.indexOf(id_tags_str) == -1) {
					this.my_labelList.push(item)
					this.selectId.push(id_tags_str)
					this.dataList.label
				}
				
			},
			getUserInfo() {
				uni.request({
					url: getApp().globalData.url + 'api/user/info',
					method: 'GET',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						this.user_info = res.data.items.info;
						this.work_unit = this.user_info.company;
						this.curRegion = this.user_info.company_address;
						this.curRegion = this.user_info.company_address;
						this.my_post = this.user_info.zw,
						this.curInd = this.user_info.hy,
						this.stuRegion = this.user_info.syd,
						this.pre_school = this.user_info.graduated_school,
						this.intr = this.user_info.jl
						this.my_labelList = res.data.items.tags
						for(var i = 0; i < this.my_labelList.length; i++) {
							var tags_id = this.my_labelList[i].tag_id
							if(this.selectId.indexOf(tags_id) === -1) {
								this.selectId.push(tags_id)
							}
						}
					},
					fail: (res) => {
					
					}
				})
			},
			toBack() {
				uni.switchTab({
					url: './index'
				})
			},
			confirmRegion(e) {
				this.curRegion = ''
				this.curRegion = e.province.label + ' ' + e.city.label + ' ' + e.area.label;
			},
			confirmStuRegion(e) {
				this.stuRegion = ''
				this.stuRegion = e.province.label + ' ' + e.city.label + ' ' + e.area.label;
			},
			confirmInd(e) {
				this.curInd = '';
				e.map((val, index) => {
					this.curInd = val.label
				})
			},
			toAdd() {
				this.label_open = true
				this.form_l = false
			},
			closeLabel() {
				this.form_l = true
				this.label_open = false
			},
			toHome() {
				uni.switchTab({
					url: '../index/index'
				})
				this.changeUserInfo()
			},
			clickComp() {
				this.form_l = true
				this.label_open = false
				// this.sList();
			},
			changePhone() {
				uni.navigateTo({
					url: './changePhone'
				})
			}
		}
	}
</script>

<style lang="scss">
	.info-cont {
		position: relative;
		top: 0;
		left: 0;
	}
	
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
	
	.dialog-info {
		background-color: #F5F7FF;
		padding: 26rpx 32rpx;
		color: #595E75;
	}
	
	.my-style {
		color: #333333;
		font-size: 28rpx;
		padding: 32rpx;
		display: flex;
		
		.form-r {
			color: #595E75;
			font-size: 28rpx;
			flex: 1;
			display: flex;
			justify-content: space-between;
		}
		
		.form-c {
			display: flex;
			justify-content: space-between;
			width: 502rpx;
			
			.u-demo-result-line {
				color: #9FA4B5;
			}
		}
	}
	
	.form-left {
		width: 200rpx;
		font-weight: bold;
	}
	
	.my-phone, 
	.my-ind,
	.my-pre-school {
		border-bottom: 16rpx solid #F5F7FF;
	}
	
	.my-intr {
		padding: 32rpx;
		border-bottom: 16rpx solid #F5F7FF;
		
		.intr-fill {
			margin-top: 24rpx;
			font-size: 28rpx;
			width: 100%;
			height: 140rpx;
			line-height: 40rpx;
		}
	}
	
	.my-label {
		padding: 32rpx 0;
		padding-right: 32rpx;
		display: flex;
		flex-direction: column;
		border-bottom: 16rpx solid #F5F7FF;
		
		.to-choose {
			display: flex;
			justify-content: space-between;
			align-items: center;
		}
		
		.add-label-btn {
			width: 176rpx;
			height: 64rpx;
			background-color: #405AFE;
			color: #fff;
			border-radius: 8rpx;
			text-align: center;
			line-height: 64rpx;
			font-size: 28rpx;
			margin: 32rpx auto;
		}
	}
	
	.all-label-wrap {
		position: absolute;
		left: 0;
		top: 0;
		height: calc(100vh - var(--window-top));
		width: 100%;
		background-color: #fff;
		z-index: 999;
		
		.add-labal-t {
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
		
		.dialog-text {
			padding: 12rpx 0;
			color: #595E75;
			font-size: 24rpx;
		}
	}
	
	.label-type-title {
		display: flex;
		color: #0B102C;
		font-size: 28rpx;
		font-weight: bold;
		margin-top: 32rpx;
		margin-bottom: 24rpx;
	}
	
	.label-item-wrap {
		display: flex;
		flex-wrap: wrap;
		// justify-content: space-between;
		
		.label-item {
			padding: 12rpx 24rpx;
			background-color: #F4F5FE;
			font-size: 24rpx;
			color: #9FA4B5;
			border-radius: 8rpx;
			margin-right: 16rpx;
			margin-bottom: 16rpx;
		}
		
		.active_label {
			background-color: #405AFE;
			color: #fff;
		}
	}
	
	.label-type {
		height: calc(100vh - var(--window-top) - 138rpx);
		overflow-y: scroll;
		padding-bottom: 98rpx;
	}
	
	.comp {
		position: fixed;
		bottom: 0;
		display: flex;
		align-items: center;
		// border-top: 1px solid rgba(0, 0, 0, 0.33);
		background-color: #fff;
		padding: 18rpx 32rpx;
		width: 100%;
		
		.home-btn {
			display: flex;
			// flex-direction: column;
			align-items: center;
			justify-content: center;
			margin-right: 32rpx;
			// color: #9FA4B5;
			// font-size: 20rpx;
			width: 88rpx;
			height: 88rpx;
			border-radius: 50%;
			box-shadow:0rpx 8rpx 16rpx 0rpx rgba(0,0,0,0.15);
		}
		
		.comp-btn {
			width: 566rpx;
			height: 88rpx;
			background-color: #405AFE;
			color: #fff;
			font-weight: bold;
			font-size: 32rpx;
			line-height: 88rpx;
			text-align: center;
			border-radius: 50rpx;
			box-shadow:0rpx 8rpx 16rpx 0rpx rgba(64,90,254,0.3);
		}
	}
	
	.label-item-list {
		padding: 0 32rpx;
	}
	
	.my_label_wrap {
		padding: 32rpx;
		padding-bottom: 0;
		// display: flex;
		// flex-wrap: wrap;
	}
	
	.my_label_title {
		color: #0B102C;
		font-size: 28rpx;
		font-weight: bold;
	}
	
	.my_cur_label {
		padding: 8rpx 16rpx;
		background-color: #D8DDFE;
		border-radius: 8rpx;
		margin-right: 16rpx;
		margin-top: 8rpx;
		display: flex;
		align-items: center;
		color: #405AFE;
	}
	
	.my_show_label {
		padding: 8rpx 16rpx;
		background-color: #D8DDFE;
		border-radius: 8rpx;
		margin-right: 16rpx;
		margin-top: 16rpx;
		display: flex;
		align-items: center;
		color: #405AFE;
	}
	
	.my_cur_label::after {
		content: "";
		display: inline-block;
		width: 28rpx;
		height: 28rpx;
		background-image: url(../../static/del_label.png);
		background-size: 100% 100%;
		margin-left: 8rpx;
	}
</style>
