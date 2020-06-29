<template>
	<view class="container">
		<!-- 小程序头部兼容 -->
		<!-- #ifdef MP -->
		<view class="mp-search-box" @click="search()">
			<input class="ser-input" type="text" value="输入关键字搜索" />
			</view>
		<!-- #endif -->

		<!-- 头部轮播 -->
		<view class="carousel-section">
			<!-- 标题栏和状态栏占位符 -->
			<view class="titleNview-placing"></view>
			<!-- 背景色区域 -->
			<view class="titleNview-background" :style="{ backgroundColor: titleNViewBackground }"></view>
			<swiper class="carousel" circular @change="swiperChange">
				<swiper-item v-for="(item, index) in carouselList" :key="index" class="carousel-item" @click="navToDetailPage({ title: '轮播广告' })">
					<image :src="item.pic" />
				</swiper-item>
			</swiper>
			<!-- 自定义swiper指示器 -->
			<view class="swiper-dots">
				<text class="num">{{ swiperCurrent + 1 }}</text>
				<text class="sign">/</text>
				<text class="num">{{ swiperLength }}</text>
			</view>
		</view>
		<!-- 分类 -->
		<view class="cate-section">
			<view class="cate-item" @click="navToTabPage('/pages/product/groupActivityList')">
				<image src="/static/temp/c3.png"></image>
				<text>团购活动</text>
			</view>
			<view class="cate-item" @click="navToTabPage('/pages/product/groupList')">
				<image src="/static/temp/c5.png"></image>
				<text>拼团活动</text>
			</view>
			<view class="cate-item" @click="navToTabPage('/pages/product/giftList')">
				<image src="/static/temp/c6.png"></image>
				<text>积分商城</text>
			</view>
			<view class="cate-item" @click="navToTabPage('/pages/index/secskill')">
				<image src="/static/temp/c7.png"></image>
				<text>限时秒杀</text>
			</view>
			<view class="cate-item" @click="navToTabPage('/pages/product/paiMaiList')">
				<image src="/static/temp/c8.png"></image>
				<text>拍卖商品</text>
			</view>
		</view>
		<view class="ad-1"><image src="/static/temp/ad1.jpg" mode="scaleToFill"></image></view>
		<!-- 优惠券  https://s.click.taobao.com/OPh3c1w -->
		<coupon v-for="(item, index) in couponList" :key="index" v-bind:item="item" theme="#ff0000"></coupon>
		<!-- 精选店铺 -->
		<view class="f-header m-t">
			<image src="/static/temp/h1.png"></image>
			<view class="tit-box">
				<text class="tit">精选店铺</text>
				<text class="tit2">Competitive Products For You</text>
			</view>
			<text class="yticon icon-you"></text>
		</view>
		<view>
			<scroll-view class="type-list" scroll-x>
				<view class="scoll-wrapper">
					<view v-for="(item, index) in typeList" :key="index" class="floor-item" @click="navToDetailPage(item)">
						<image :src="item.logo" mode="aspectFill"></image>
						<text class="title clamp">{{ item.name }}</text>
					</view>
					<view class="more" @click="navToList()">
						<text>查看全部</text>
						<text>More+</text>
					</view>
				</view>
			</scroll-view>
		</view>

		<!-- 猜你喜欢 -->
		<view class="f-header m-t">
			<image src="/static/temp/h1.png"></image>
			<view class="tit-box">
				<text class="tit">新品上市</text>
				<text class="tit2">Guess You Like It</text>
			</view>
			<text class="yticon icon-you"></text>
		</view>

		<view class="guess-section">
			<view v-for="(item, index) in newProductList" :key="index" class="guess-item" @click="navToDetailPage(item)">
				<view class="image-wrapper"><image :src="item.pic" mode="aspectFill"></image></view>
				<text class="title clamp">{{ item.name }}</text>
				<text class="price">￥{{ item.price }}</text>
			</view>
		</view>
		<view>
			<uni-load-more :status="status" :content-text="contentText" color="#007aff"  />
		</view>
	</view>
</template>

<script>
import Api from '@/common/api';
import coupon from '@/components/coolc-coupon/coolc-coupon';
import uniLoadMore from "@/components/uni-load-more/uni-load-more.vue"
import { formatDate } from '@/common/date';
let newProductListlen = 0;
export default {
	components: {
		coupon,
		uniLoadMore
	},
	data() {
		return {
			status: 'more',
			contentText: {
				contentdown: '查看更多',
				contentrefresh: '加载中',
				contentnomore: '没有更多'
			},
			keyword: '',
			titleNViewBackground: '',
			swiperCurrent: 0,
			swiperLength: 0,
			carouselList: [],
			typeList:[],
			couponList:[],
			newProductList: [],
			pageParams:{
				pageNum:1
			}
		};
	},

	onLoad(ops) {
		if (ops.invitecode){
			this.$db.set('invitecode', ops.invitecode);
		}
		this.loadData();
	},
	 onPullDownRefresh() {//下拉刷新
		this.pageParams.pageNum = 1;
	 	this.loadData();
	    },
		onReachBottom(){ //上拉加载
			this.pageParams.pageNum+=1;
			this.newProductFn()
		},
	methods: {
		/**
		 * 请求静态数据只是为了代码不那么乱
		 * 分次请求未作整合
		 */
		async newProductFn(){
			this.status = 'loading';
			// uni.showNavigationBarLoading();
			let params = {pageNum:this.pageParams.pageNum};
			let data = await Api.apiCall('get', Api.pan.newProduct, params)
			this.newProductList = this.newProductList.concat(data.records)
			if(this.newProductList.length==newProductListLen){
				this.status = 'noMore'
			}else{
				this.status = 'more'
			}
			newProductListlen = this.newProductList.length;
			// uni.hideNavigationBarLoading();
		},
		async bannerFn(){
			let data = await Api.apiCall('get', Api.pan.banner);
			this.carouselList = data || [];
			this.swiperLength = this.carouselList.length;
			this.titleNViewBackground = 'rgb(203, 87, 60)';
		},
		async typeListFn(){
			let params = {pageNum:1};
			let data = await Api.apiCall('get', Api.pan.typeList,params);
			this.typeList = data.records || [];
		},
		
		async loadData() {
			uni.startPullDownRefresh();//下拉刷新
			await this.bannerFn() //轮播图
			await this.typeListFn() //分类
			await this.newProductFn()	//新品上市
			uni.stopPullDownRefresh();//下拉刷新
			
			const userInfo = uni.getStorageSync('userInfo');
			if (userInfo) {
				uni.getLocation({
					type: 'wgs84',
					success: function(res) {
						console.log(res);
						let params = { longitude: res.longitude, latitude: res.latitude, memberId: userInfo.id, userName: userInfo.username, pic: userInfo.icon};
						Api.apiCall('post', Api.index.submitLocaltion, params);

					}
				});
			}
		},
		dateFormat(time) {
			if (time == null || time === '') {
				return 'N/A';
			}
			let date = new Date(time);
			return formatDate(date, 'yyyy-MM-dd hh:mm:ss');
		},
		//轮播图切换修改背景色
		swiperChange(e) {
			const index = e.detail.current;
			this.swiperCurrent = index;
			this.titleNViewBackground = this.carouselList[index].background;
		},
		navToTabPage(url) {
			uni.navigateTo({
				url: url
			});
		},
		//详情页
		navToDetailPage(item) {
			//测试数据没有写id，用title代替
			let id = item.id;
			uni.navigateTo({
				url: `/pages/product/product?id=${id}`
			});
		},

		navToGroupDetailPage(item) {
			console.log(item);
			//测试数据没有写id，用title代替
			let id = item.goodsId;
			let groupId = item.id;

			uni.navigateTo({
				url: `/pages/product/groupProduct?id=${id}&&groupId=${item.id}`
			});
		},
		navToSkillDetailPage(item) {
			//测试数据没有写id，用title代替
			let id = item.id;

			uni.navigateTo({
				url: `/pages/product/secskillDetail?id=${id}`
			});
		},
		navToList() {
			uni.navigateTo({
				url: `/pages/product/list`
			});
		}
	},

	search() {
		uni.navigateTo({
			url: '/pages/search/search'
		});
	},

	// 标题栏input搜索框点击
	onNavigationBarSearchInputClicked: async function(e) {
		uni.navigateTo({
			url: '/pages/search/search'
		});
	},
	//点击导航栏 buttons 时触发
	onNavigationBarButtonTap(e) {
		const index = e.index;
		if (index === 0) {
			this.$api.msg('点击了扫描');
		} else if (index === 1) {
			// #ifdef APP-PLUS
			const pages = getCurrentPages();
			const page = pages[pages.length - 1];
			const currentWebview = page.$getAppWebview();
			currentWebview.hideTitleNViewButtonRedDot({
				index
			});
			// #endif
			uni.navigateTo({
				url: '/pages/notice/notice'
			});
		}
	}
};
</script>

<style lang="scss">
.mp-search-box {
	position: absolute;
	left: 0;
	top: 30upx;
	z-index: 9999;
	width: 100%;
	padding: 0 80upx;
	.ser-input {
		flex: 1;
		height: 56upx;
		line-height: 56upx;
		text-align: center;
		font-size: 28upx;
		color: $font-color-base;
		border-radius: 20px;
		background: rgba(255, 255, 255, 0.6);
	}
}
page {
	.cate-section {
		position: relative;
		z-index: 5;
		border-radius: 16upx 16upx 0 0;
		margin-top: -20upx;
	}
	.carousel-section {
		padding: 0;
		.titleNview-placing {
			padding-top: 0;
			height: 0;
		}
		.carousel {
			.carousel-item {
				padding: 0;
			}
		}
		.swiper-dots {
			left: 45upx;
			bottom: 40upx;
		}
	}
}

page {
	background: #f5f5f5;
}
.m-t {
	margin-top: 16upx;
}
/* 头部 轮播图 */
.carousel-section {
	position: relative;
	padding-top: 10px;

	.titleNview-placing {
		height: var(--status-bar-height);
		padding-top: 44px;
		box-sizing: content-box;
	}

	.titleNview-background {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 426upx;
		transition: 0.4s;
	}
	.carousel {
		width: 100%;
		height: 350upx;
	
		.carousel-item {
			width: 100%;
			height: 100%;
			padding: 0 28upx;
			overflow: hidden;
		}
	
		image {
			width: 100%;
			height: 100%;
			border-radius: 10upx;
		}
	}
	.swiper-dots {
		display: flex;
		position: absolute;
		left: 60upx;
		bottom: 15upx;
		width: 72upx;
		height: 36upx;
		background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAABkCAYAAADDhn8LAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNi1jMTMyIDc5LjE1OTI4NCwgMjAxNi8wNC8xOS0xMzoxMzo0MCAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6OTk4MzlBNjE0NjU1MTFFOUExNjRFQ0I3RTQ0NEExQjMiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6OTk4MzlBNjA0NjU1MTFFOUExNjRFQ0I3RTQ0NEExQjMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTcgKFdpbmRvd3MpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6Q0E3RUNERkE0NjExMTFFOTg5NzI4MTM2Rjg0OUQwOEUiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6Q0E3RUNERkI0NjExMTFFOTg5NzI4MTM2Rjg0OUQwOEUiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz4Gh5BPAAACTUlEQVR42uzcQW7jQAwFUdN306l1uWwNww5kqdsmm6/2MwtVCp8CosQtP9vg/2+/gY+DRAMBgqnjIp2PaCxCLLldpPARRIiFj1yBbMV+cHZh9PURRLQNhY8kgWyL/WDtwujjI8hoE8rKLqb5CDJaRMJHokC6yKgSCR9JAukmokIknCQJpLOIrJFwMsBJELFcKHwM9BFkLBMKFxNcBCHlQ+FhoocgpVwwnv0Xn30QBJGMC0QcaBVJiAMiec/dcwKuL4j1QMsVCXFAJE4s4NQA3K/8Y6DzO4g40P7UcmIBJxbEesCKWBDg8wWxHrAiFgT4fEGsB/CwIhYE+AeBAAdPLOcV8HRmWRDAiQVcO7GcV8CLM8uCAE4sQCDAlHcQ7x+ABQEEAggEEAggEEAggEAAgQACASAQQCCAQACBAAIBBAIIBBAIIBBAIABe4e9iAe/xd7EAJxYgEGDeO4j3EODp/cOCAE4sYMyJ5cwCHs4rCwI4sYBxJ5YzC84rCwKcXxArAuthQYDzC2JF0H49LAhwYUGsCFqvx5EF2T07dMaJBetx4cRyaqFtHJ8EIhK0i8OJBQxcECuCVutxJhCRoE0cZwMRyRcFefa/ffZBVPogePihhyCnbBhcfMFFEFM+DD4m+ghSlgmDkwlOgpAl4+BkkJMgZdk4+EgaSCcpVX7bmY9kgXQQU+1TgE0c+QJZUUz1b2T4SBbIKmJW+3iMj2SBVBWz+leVfCQLpIqYbp8b85EskIxyfIOfK5Sf+wiCRJEsllQ+oqEkQfBxmD8BBgA5hVjXyrBNUQAAAABJRU5ErkJggg==);
		background-size: 100% 100%;
	
		.num {
			width: 36upx;
			height: 36upx;
			border-radius: 50px;
			font-size: 24upx;
			color: #fff;
			text-align: center;
			line-height: 36upx;
		}
	
		.sign {
			position: absolute;
			top: 0;
			left: 50%;
			line-height: 36upx;
			font-size: 12upx;
			color: #fff;
			transform: translateX(-50%);
		}
	}
}

/* 分类 */
.cate-section {
	display: flex;
	justify-content: space-around;
	align-items: center;
	flex-wrap: wrap;
	padding: 30upx 22upx;
	background: #fff;
	.cate-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		font-size: $font-sm + 2upx;
		color: $font-color-dark;
	}
	/* 原图标颜色太深,不想改图了,所以加了透明度 */
	image {
		width: 88upx;
		height: 88upx;
		margin-bottom: 14upx;
		border-radius: 50%;
		opacity: 0.7;
		box-shadow: 4upx 4upx 20upx rgba(250, 67, 106, 0.3);
	}
}
.ad-1 {
	width: 100%;
	height: 210upx;
	padding: 10upx 0;
	background: #fff;
	image {
		width: 100%;
		height: 100%;
	}


	padding: 4upx 30upx 24upx;
	background: #fff;
	.s-header {
		display: flex;
		align-items: center;
		height: 92upx;
		line-height: 1;
		.s-img {
			width: 140upx;
			height: 30upx;
		}
		.tip {
			font-size: $font-base;
			color: $font-color-light;
			margin: 0 20upx 0 40upx;
		}
		.timer {
			display: inline-block;
			width: 40upx;
			height: 36upx;
			text-align: center;
			line-height: 36upx;
			margin-right: 14upx;
			font-size: $font-sm + 2upx;
			color: #fff;
			border-radius: 2px;
			background: rgba(0, 0, 0, 0.8);
		}
		.icon-you {
			font-size: $font-lg;
			color: $font-color-light;
			flex: 1;
			text-align: right;
		}
	}
	.floor-list {
		white-space: nowrap;
	}
	.scoll-wrapper {
		display: flex;
		flex-wrap: wrap;
		flex-direction: row;
		justify-content: space-between;
		// align-items: flex-start;
	}
	.floor-item {
		width: 150upx;
		margin-right: 20upx;
		font-size: $font-sm + 2upx;
		color: $font-color-dark;
		line-height: 1.8;
		image {
			width: 150upx;
			height: 150upx;
			border-radius: 6upx;
		}
		.price {
			color: $uni-color-primary;
		}
	}
}
.f-header {
	display: flex;
	align-items: center;
	height: 140upx;
	padding: 6upx 30upx 8upx;
	background: #fff;
	image {
		flex-shrink: 0;
		width: 80upx;
		height: 80upx;
		margin-right: 20upx;
	}
	.tit-box {
		flex: 1;
		display: flex;
		flex-direction: column;
	}
	.tit {
		font-size: $font-lg + 2upx;
		color: #font-color-dark;
		line-height: 1.3;
	}
	.tit2 {
		font-size: $font-sm;
		color: $font-color-light;
	}
	.icon-you {
		font-size: $font-lg + 2upx;
		color: $font-color-light;
	}
}

/* 猜你喜欢 */
.guess-section {
	display: flex;
	flex-wrap: wrap;
	padding: 0 30upx;
	background: #fff;
	.guess-item {
		display: flex;
		flex-direction: column;
		width: 48%;
		padding-bottom: 40upx;
		&:nth-child(2n + 1) {
			margin-right: 4%;
		}
	}
	.image-wrapper {
		width: 100%;
		height: 330upx;
		border-radius: 3px;
		overflow: hidden;
		image {
			width: 100%;
			height: 100%;
			opacity: 1;
		}
	}
	.title {
		font-size: $font-lg;
		color: $font-color-dark;
		line-height: 80upx;
	}
	.price {
		font-size: $font-lg;
		color: $uni-color-primary;
		line-height: 1;
	}
}
.type-list{
	.scoll-wrapper{
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		align-items: center;
		padding: 30upx 22upx;
	
	.floor-item{
			padding: 10upx;
			width: 25%;
			display: flex;
			flex-direction: column;
			align-items: center;
			font-size: $font-sm;
			image{
				margin-bottom: 20upx;
				border-radius: 50%;
				width: 80upx;
				height: 80upx;
			}
	}
	.more{
		font-size: $font-sm+2upx;
	}
	}
	
}
	
	
	
</style>
