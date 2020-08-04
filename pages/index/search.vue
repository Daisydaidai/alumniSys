<template>
	<view class="wrapper">
		<view class="search-wrap">
			<view class="search-input">
				<!-- <u-icon name="arrow-left" color="#444444" size="40" @click="closeCur"></u-icon> -->
				<view class="select-style" style="display: flex;">
					<view class="u-demo-result-line" @click="showGrade = true">{{ curGrade || curYear }}级</view>
					<u-picker :show-time-tag="false" mode="time" :end-year="curYear" v-model="showGrade" :params="params" @confirm="confirmGrade"></u-picker>
					<u-icon @click="showGrade = true" name="arrow-down" color="#444444" size="26" style="margin-right: 8rpx;" class="arrow-down"></u-icon>
				</view>
				<view class="search-cont">
					<image src="../../static/search.png" mode="" class="search-icon"></image>
					<u-input v-model="search_word" type="text" placeholder="请输入姓名、班级等" @input="onKeyInput" :focus="focus" />
				</view>
				<text class="closeCur" @click="closeCur">取消</text>
			</view>
			<!-- 搜索联想词 -->
			<view class="on-input-wrap" v-if="on_input">
				<view class="result-sum" @click="toResPage">
					<view class="search-title">
						搜索“<text class="model-word" style="color: #405AFE;">{{search_word}}</text>”的校友
					</view>
					<view class="res-sum">
						<text style="margin-right: 10rpx;">共{{res_num}}条结果</text>
						<u-icon name="arrow-right" color="#9FA4B5" size="12"></u-icon>
					</view>
				</view>
				<view class="ass-word-wrap">
					<view class="ass-word-item" v-for="(item,index) in mhKw" :key="index" @click="mhWord(item)">
						<image src="../../static/search.png" mode="" class="search-icon"></image>
						<text>{{item.name}}</text>
					</view>
				</view>
			</view>
			<view class="search-click">
				<view class="solo-sele">
					<view class="select-style" @click="major_hy">
						<text>专业/行业</text>
						<u-icon name="arrow-down" color="#444444" size="26" class="arrow-down"></u-icon>
					</view>
					<view class="select-style">
						<view class="u-demo-result-line" @click="showRegion = true">{{ curRegion || '工作地区' }}</view>
						<u-picker mode="region" v-model="showRegion" :defaultRegion="defaultRegion" @confirm="confirmRegion"></u-picker>
						<u-icon @click="showRegion = true" name="arrow-down" color="#444444" size="26" class="arrow-down"></u-icon>
					</view>
					<view class="select-style">
						<view class="u-demo-result-line" @click="showPreRegion = true">{{ curPreRegion || '生源地' }}</view>
						<u-picker mode="region" v-model="showPreRegion" :defaultRegion="defaultRegion" @confirm="confirmPreRegion"></u-picker>
						<u-icon @click="showPreRegion = true" name="arrow-down" color="#444444" size="26" class="arrow-down"></u-icon>
					</view>
					<view class="select-style" @click="showTag">
						<text>标签</text>
						<u-icon name="arrow-down" color="#444444" size="26" class="arrow-down"></u-icon>
					</view>
				</view>
			</view>
		</view>
		<!-- 搜索无结果状态 -->
		<view class="search-no-result" v-if="searching&&res_num===0">
			没有符合该搜索条件的结果
		</view>
		
		<view class="shortcut-search-wrap" v-if="hotArea">
			<view class="shortcut-title">
				您可以试试
			</view>
			<view class="shortcut-search">
				<text class="shortcut-label" @click="addKeyword(item)" v-for="(item, index) in hotKw" :key="index">{{item.keywords}}</text>
			</view>
		</view>
		<view class="his-search" v-if="hisArea">
			<view class="his-search-top">
				<text class="his-title">历史搜索</text>
				<image src="../../static/his-del.png" @click="hisDel" mode="" class="his-del"></image>
			</view>
			<view class="his-label-wrap">
				<text class="his-label-item" @click="addKeyword(item)" v-for="(item, index) in hisKw" :key="index">{{item.keywords}}</text>
			</view>
		</view>
		
		<!-- 更多筛选 -->
		<view class="more-mask" v-if="showMajorHy||showTags"></view>
		<view class="more-sele-wrap" v-if="showMajorHy">
			<view class="more-item-cont">
				<view class="label-item">
					<text class="sele-title">专业</text>
					<view class="label-cont">
						<text class="label-text" @click="c_mojar(m, l)" :class="majorIndex===l?'active-label':''" v-for="(m, l) in majorList" :key="l">{{m.zy}}</text>
					</view>
				</view>
				<view class="label-item">
					<text class="sele-title">行业领域</text>
					<view class="label-cont">
						<text class="label-text" @click="c_hy(hy, m)" :class="hyIndex===m?'active-label':''" v-for="(hy, m) in hyList" :key="m">{{hy.title}}</text>
					</view>
				</view>
			</view>
			<view class="opt-wrap">
				<button type="default" class="reset-btn" @click="resetMajorBtn">重置</button>
				<button type="default" class="confirm-btn" @click="confirmMajorBtn">确定</button>
			</view>
		</view>
		<view class="more-sele-wrap" v-if="showTags">
			<view class="more-item-cont">
				<view class="label-item" v-for="(item, index) in dataList" :key="index">
					<text class="sele-title">{{item.title}}</text>
					<view class="label-cont">
						<text @click="labelStatus(lb)" :class="{'active-label': rSelect.indexOf(lb)!=-1}" class="label-text" v-for="(lb, i) in item.label" :key="i">{{lb.title}}</text>
					</view>
				</view>
			</view>
			<view class="opt-wrap">
				<button type="default" class="reset-btn" @click="resetBtn">重置</button>
				<button type="default" class="confirm-btn" @click="confirmBtn">确定</button>
			</view>
		</view>
		
		<!-- 搜索结果 -->
		<view class="alumni-wrap res-list" v-if="res&&res_num!=0">
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
			<view class="common more_btn" @click="getMore()" v-if="!is_all">
				加载更多
			</view>
			<view class="common more_btn" v-else>
				没有更多了~
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
				hotKw: [],
				hotArea: false,
				hisKw: [],
				hisArea: false,
				mhKw: [],
				focus: true,
				on_input: false,
				res: false,
				params: {
					year: true,
					month: false,
					day: false
				},
				showGrade: false,
				showMajor: false,
				showRegion: false,
				showPreRegion: false,
				curGrade: '',
				curYear: '',
				curMajor: '专业',
				majorList: [],
				hyList: [],
				curRegion: '',
				chooseRegin: '',
				curPreRegion: '',
				choosePreRegion: '',
				defaultRegion: ['浙江省', '杭州市', '西湖区'],
				showMajorHy: false,
				showTags: false,
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
				res_num: 0,
				f_title: '',
				f_label: [],
				s_title: '',
				s_label: [],
				choose_mojar: '',
				choose_hy: '',
				majorIndex: -1,
				hyIndex: -1,
				rSelect: [],
				selectList: [],
				selectId: [],
				searching: false,
				page: 1,
				is_all:false,
				listlength:0,
				dataList: [],
				clickMore: false
			};
		},
		watch: {
			'search_word': {
				handler(value) {
					if(this.search_word != '') {
						if (this.timer) {
							clearTimeout(this.timer)
						}
						this.timer = setTimeout(() => {
							// console.log('aaa');
							this.getAlumniList();
						}, 1000)
					}
					// if (this.search_word == '') {
					// 	this.getAlumniList()
					// }
				},
				deep: true
			}
		},
		onLoad() {
			if(app.reToken()) {
				uni.getStorageSync("access_token")
			}
			this.getHotKw();
			this.getHisKw();
			this.getMajorList();
			this.getHyList();
			this.getLabelList();
			var date=new Date;
			this.curYear = date.getFullYear()
		},
		methods: {
			sList() {
				for(var i = 0; i < this.rSelect.length; i ++) {
					this.selectId.push(this.rSelect[i].id);
					this.selectList.push(this.rSelect[i].title);
				}
				// this.getAlumniList()
			},
			getMore() {
				this.page += 1;
				this.clickMore = true;
				this.getAlumniList()
			},
			getAlumniList() {
				uni.request({
					url: getApp().globalData.url + 'api/index/index/search',
					method: 'POST',
					data: {
						keywords: this.search_word,
						nj: this.curGrade,
						zy: this.choose_mojar,
						hy: this.choose_hy,
						area: this.chooseRegin,
						syd: this.choosePreRegion,
						tags: this.selectId,
						page: this.page
					},
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						// console.log(res)
						if(this.clickMore) {
							this.alumniList = this.alumniList.concat(res.data.items.data)
							this.clickMore = false
						}else {
							this.alumniList = res.data.items.data;
						}
						// console.log(this.alumniList)
						this.res_num = this.alumniList.length
						this.searching = true
						if(this.alumniList.length > 0) {
							this.res = true
							// this.alumni_none = false
						}
						if(this.listlength == this.alumniList.length || this.alumniList.length < 15) {
							this.is_all = true
						}
						if(this.alumniList.length - this.listlength < 15) {
							this.is_all = true
						}
						this.listlength = this.alumniList.length;
						// this.page += 1
					},
					fail: (res) => {
						
					}
				})
			},
			closeCur() {
				uni.switchTab({
					url: './index'
				})
			},
			onKeyInput(e) {
				if(e!='') {
					// console.log(e)
					this.on_input = true;
					this.getAlumniList();
					this.getMhKw();
				}else {
					this.on_input = false
				}
			},
			//选择专业
			c_mojar(e, index) {
				this.choose_mojar = e.zy
				this.majorIndex = index
			},
			c_hy(e, index) {
				this.choose_hy = e.title
				this.hyIndex = index
			},
			//选择标签
			labelStatus(value) {
				if(this.rSelect.indexOf( value )!=-1){  
					this.rSelect.splice( this.rSelect.indexOf(value), 1 );  
				}else{  
					if(this.rSelect.length <= 4) {
						this.rSelect.push(value);  
					}else {
						uni.showToast({
							icon: 'none',
							title: '最多可选5个标签',
						});
					}
				}
				// console.log(this.rSelect)
			},
			confirmGrade(e) {
				this.curGrade = ''
				this.curGrade = e.year;
				// this.search_word = this.curGrade
				this.getAlumniList();
			},
			// confirmMajor(e) {
			// 	this.curMajor = '';
			// 	e.map((val, index) => {
			// 		this.curMajor = val.label
			// 	})
			// 	this.getAlumniList();
			// },
			confirmRegion(e) {
				// this.curRegion = ''
				this.chooseRegin = e.province.label + ' ' + e.city.label + ' ' + e.area.label;
				this.curRegion = e.area.label;
				// this.search_word = this.curRegion
				this.getAlumniList();
			},
			confirmPreRegion(e) {
				this.curPreRegion = ''
				this.curPreRegion = e.area.label;
				this.choosePreRegion = e.province.label + ' ' + e.city.label + ' ' + e.area.label;
				// this.search_word = this.curPreRegion
				this.getAlumniList();
			},
			// moreSele() {
			// 	this.more_sele = !this.more_sele
			// },
			major_hy() {
				// console.log(111)
				this.showMajorHy = !this.showMajorHy
				this.res = false;
				if(this.showTags == true) {
					this.showTags = false
				}
			},
			showTag() {
				this.showTags = !this.showTags
				this.res = false;
				if(this.showMajorHy == true) {
					this.showMajorHy = false
				}
			},
			resetMajorBtn() {
				this.choose_mojar = ''
				this.choose_hy = '';
				this.majorIndex = -1;
				this.hyIndex = -1;
			},
			confirmMajorBtn() {
				this.showMajorHy = false
				// this.search_word = this.choose_mojar + '，' + this.choose_hy
				this.getAlumniList()
			},
			resetBtn() {
				// this.showTags = false
				this.rSelect = []
			},
			confirmBtn() {
				this.showTags = false;
				this.sList();
				// this.search_word = this.selectList
				this.getAlumniList()
			},
			toResPage() {
				this.on_input = false
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
						console.log(res);
						uni.showToast({
							icon: 'none',
							title: '请求失败，请稍后再试',
						});
					},
				})
			},
			//获取专业
			getMajorList() {
				uni.request({
					url: getApp().globalData.url + 'api/common/zy',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						// console.log(res)
						this.majorList = res.data.items
					},
					fail: (res) => {
						
					}
				})
			},
			//获取行业
			getHyList() {
				uni.request({
					url: getApp().globalData.url + 'api/common/hy',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						this.hyList = res.data.items
					},
					fail: (res) => {
						
					}
				})
			},
			//获取标签
			getLabelList() {
				uni.request({
					url: getApp().globalData.url + 'api/common/tags',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						// console.log(res)
						// this.tagsList = res.data.items;
						// this.f_title = res.data.items[0].title
						// this.f_label = res.data.items[0].children
						// this.s_title = res.data.items[1].title
						// this.s_label = res.data.items[1].children
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
			getHotKw() {
				uni.request({
					url: getApp().globalData.url + 'api/index/index/history',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						// console.log(res)
						this.hotKw = res.data.items
						if(this.hotKw.length > 0) {
							this.hotArea = true
						}
					},
					fail: (res) => {
						
					}
				})
			},
			getHisKw() {
				uni.request({
					url: getApp().globalData.url + 'api/user/history',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						// console.log(res)
						this.hisKw = res.data.items
						if(this.hisKw.length > 0) {
							this.hisArea = true
						}
					},
					fail: (res) => {
						
					}
				})
			},
			//获取模糊搜索词
			getMhKw() {
				uni.request({
					url: getApp().globalData.url + 'api/index/index/keywords',
					method: 'get',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					data: {
						keywords: this.search_word
					},
					success: (res) => {
						// console.log(res)
						this.mhKw = res.data.items;
					},
					fail: (res) => {
						
					}
				})
			},
			mhWord(item) {
				this.search_word = item.name
				this.on_input = false
			},
			addKeyword(e) {
				this.search_word = e.keywords
			},
			//清空历史记录
			hisDel() {
				uni.request({
					url: getApp().globalData.url + 'api/user/history',
					method: 'DELETE',
					header: {
						Authorization: 'Bearer ' + uni.getStorageSync('access_token')
					},
					success: (res) => {
						this.hisArea = false
						uni.showToast({
							icon: 'none',
							title: '历史记录已全部清空',
						});
					},
					fail: (res) => {
						
					}
				})
			}
		}
	}
</script>

<style lang="scss">
	.wrapper {
		position: relative;
		top: 0;
		left: 0;
	}
	
	.search-icon {
		width: 36rpx;
		height: 36rpx;
		margin-right: 8rpx;
	}
	
	.search-wrap {
		position: relative;
		top: 0;
		left: 0;
		padding: 22rpx 0;
		width: 100%;
		border-bottom: 1px solid #F0F2FA;
		background-color: #fff;
		z-index: 29999;
		
		.on-input-wrap {
			position: absolute;
			left: 0;
			top: 114rpx;
			border-top: 1px solid #F0F2FA;
			width: 100%;
			height:  calc(100vh - var(--window-top) - 114rpx);
			background-color: #fff;
			z-index: 1001;
			padding: 32rpx;
			
			.result-sum {
				display: flex;
				justify-content: space-between;
				padding-bottom: 32rpx;
				border-bottom: 1px solid #F0F2FA;
				
				.search-title {
					color: #0B102C;
					font-size: 32rpx;
				}
				
				.res-sum {
					color: #9FA4B5;
					display: flex;
					align-items: center;
				}
			}
			
			.ass-word-wrap {
				
				.ass-word-item {
					padding: 32rpx 0;
					display: flex;
					align-items: center;
					border-bottom: 1px solid #F0F2FA;
				}
			}
		}
	}
	.search-input {
		padding: 0 32rpx;
		display: flex;
		align-items: center;
		margin-bottom: 24rpx;
		
		.search-cont {
			flex: 1;
			background-color: #F5F7FF;
			border-radius: 34rpx;
			display: flex;
			align-items: center;
			height: 68rpx;
			padding: 16rpx 24rpx;
			margin: 0 10rpx;
			
			.u-input {
				height: 40rpx;
			}
		}
	}
	
	.search-click {
		display: flex;
		justify-content: space-between;
		margin-top: 16rpx;
		padding: 0 32rpx;
	
		.solo-sele {
			display: flex;
			justify-content: space-between;
			width: 100%;
	
			.select-style {
				display: flex;
				align-items: center;
				
				.u-demo-result-line {
					max-width: 112rpx;
					overflow: hidden;
					white-space: nowrap;
					text-overflow: ellipsis;
					height: 40rpx;
				}
				
				.arrow-down {
					margin-left: 8rpx;
					margin-right: 32rpx;
				}
			}
		}
		
		.more-sele {
			display: flex;
			align-items: center;
			height: 36rpx;
			line-height: 36rpx;
			.sele-more {
				width: 36rpx;
				height: 36rpx;
			}
		}
	}
	
	.search-no-result {
		height: 176rpx;
		line-height: 176rpx;
		text-align: center;
		color: #595E75;
		font-size: 28rpx;
		width: 100%;
		background-color: #fff;
		border-bottom: 10rpx solid #F5F7FF;
	}
	
	.shortcut-search-wrap {
		width: 100%;
		padding: 32rpx;
		
		.shortcut-title {
			color: #0B102C;
			font-size: 32rpx;
			font-weight: bold;
		}
		
		.shortcut-search {
			display: flex;
			flex-wrap: wrap;
			padding-top: 8rpx;
			
			.shortcut-label {
				height:64rpx;
				background:rgba(245,247,255,1);
				border-radius:8px;
				padding: 12rpx 32rpx;
				margin-right: 32rpx;
				margin-top: 24rpx;
			}
		}
	}
	
	.his-search {
		padding: 32rpx;
		
		.his-search-top {
			display: flex;
			justify-content: space-between;
			
			.his-title {
				color: #0B102C;
				font-size: 32rpx;
				font-weight: bold;
			}
			
			.his-del {
				width: 48rpx;
				height: 48rpx;
			}
		}
		
		.his-label-wrap {
			display: flex;
			flex-wrap: wrap;
			
			.his-label-item {
				display: flex;
				align-items: center;
				justify-content: center;
				height: 64rpx;
				padding: 12rpx 32rpx;
				margin-right: 30rpx;
				margin-top: 24rpx;
				border: 2rpx solid #E6E9F5;
				border-radius: 8rpx;
			}
		}
	}
	
	.more-mask {
		position: fixed;
		left: 0;
		right: 0;
		top: 0;
		bottom: 0;
		transition: opacity .5s ease,visibility .5s ease;
		background-color: rgba(0,0,0,.6);
		z-index: -1;
	}
	
	.more-sele-wrap {
		position: absolute;
		// bottom: 0;
		top: 170rpx;
		background-color: #FFF;
		z-index: 99;
		
		.more-item-cont {
			padding: 0 32rpx;
			width: 100%;
			height: 996rpx;
			overflow-y: scroll;
			
			.label-item:nth-last-child(1) {
				margin-bottom: 112rpx;
			}
			
			.label-item {
				.sele-title {
					display: block;
					color: #0B102C;
					font-size: 32rpx;
					font-weight: bold;
					margin-top: 32rpx;
				}
				
				.label-cont {
					display: flex;
					flex-wrap: wrap;
					
					.label-text {
						display: flex;
						align-items: center;
						justify-content: center;
						width: 208rpx;
						height: 64rpx;
						padding: 12rpx 0;
						margin-right: 30rpx;
						margin-top: 24rpx;
						border: 2rpx solid #E6E9F5;
						border-radius: 8rpx;
					}
					
					.label-text:nth-child(3n+3) {
						margin-right: 0;
					}
					
					.active-label {
						background:rgba(244,245,254,1);
						border:1px solid rgba(64,90,254,1);
						color: #405AFE;
					}
				}
			}
		}
		
		.opt-wrap {
			position: absolute;
			bottom: 0;
			display: flex;
			height: 88rpx;
			
			uni-button[type=default] {
				font-size: 32rpx;
			}
			
			uni-button.reset-btn {
				width: 190rpx;
				height: 100%;
				border-radius: 0;
				border-width: 0;
				color: #595E75;
				background-color: #fff;
			}
			
			.confirm-btn {
				width: 560rpx;
				border-radius: 0;
				color: #fff;
				background-color: #405AFE;
			}
			
			uni-button:after {
				border-width: 0!important;
			}
		}
	}
	
	.alumni-wrap {
		padding: 0 32rpx;
		width: 100%;
		position: absolute;
		left: 0;
		top: 170rpx;
		z-index: 1000;
		background-color: #fff;
		height: calc(100vh - var(--window-top) - 170rpx);
		overflow-y: scroll;
		
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
