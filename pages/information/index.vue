<template>
	<view class="main">
		<!-- 页面加载动画 -->
		<view v-if="!ready">
			<zero-loading type="love"></zero-loading>
		</view>
		<image class="headerImg" v-show="!cateIsFixed" src="https://www.seedstudio.cn:1437/uploads/_44e8e14136.jpg"></image>
		<!-- 动态列表 -->
		<view class="items" v-for="(item,index) in informationList" @click="goDetail(item.id)">
			<image class="cover" :src="item.cover"></image>
			<view class="right">
				<view class="title lineOne">{{item.title}}</view>
				<view class="content lineOne">{{item.content}}</view>
				<view class="time">{{item.time}}</view>
			</view>
		</view>
		<!-- 文章数量 -->
		<view class="loadmore" v-show="loadmoreShow">
			<u-loadmore :status="status" :load-text="loadText"/>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				ready: false,
				informationList: [],
				page: 1,
				pageSize: 20,
				status: 'loadmore',
				loadText: {
					loadmore: '轻轻上拉',
					loading: '努力加载中',
					nomore: '没有更多了'
				},
				pageCount: '',
				informationTotal: '',
				loadmoreShow: false,
			}
		},
		methods: {
			// 获取动态
			getInformation() {
				uni.request({
					url: this.$url + '/api/informations?pagination[pageSize]=' + this.pageSize + '&pagination[page]=' + this.page + '&populate=*&sort[0]=updatedAt:desc',
					method: 'GET',
					success:(res) => {
						res.data.data.forEach((item)=>{
							this.informationList.push({
								id: item.id,
								title: item.attributes.title,
								content: item.attributes.content,
								time: item.attributes.create_date,
								cover: this.$url + item.attributes.cover.data.attributes.url
							})
						})
						this.pageCount = res.data.meta.pagination.pageCount
						this.informationTotal = res.data.meta.pagination.total
						this.ready = true
						this.loadmoreShow = true
						this.estimateBlogList()
					}
				})
			},
			// 判断动态条数
			estimateBlogList() {
				if(this.informationList.length == this.informationTotal) {
					this.status = 'nomore'
				}
			},
			goDetail(id) {
				uni.navigateTo({
					url: './detail?id=' + id
				})
			}
		},
		created() {
			this.getInformation()
		},
		onReachBottom() {
			if(this.page >= this.pageCount) return
			this.status = 'loading'
			this.page == ++this.page
			this.status = 'loading'
			this.getInformation()
		}
	}
</script>

<style>
	.main {
		padding: 0 20rpx;
	}
	.headerImg {
		margin-left: -20rpx;
		margin-bottom: -10rpx;
		width: 750rpx;
		height: 320rpx;
	}
	
	/* 文本一行显示 */
	.lineOne {
		overflow: hidden !important;
		text-overflow: ellipsis !important;
		display: -webkit-box !important;
		-webkit-line-clamp: 1;//文字上限行
		-webkit-box-orient: vertical;
	}
	
	.items {
		padding: 10rpx 0;
		height: 180rpx;
		display: flex;
		border-bottom: 1rpx #ececec solid;
	}
	.items .cover {
		margin-top: 25rpx;
		flex: 0 0 260rpx;
		height: 140rpx;
		width: 260rpx;
		border-radius: 10rpx;
	}
	.items .right {
		padding-left: 20rpx;
		width: 420rpx;
		display: flex;
		flex-direction: column;
		justify-content: space-evenly;
	}
	.title {
		font-size: 34rpx;
		line-height: 60rpx;
		font-weight: 600;
	}
	.content {
		font-size: 30rpx;
		color: #838383;
	}
	.time {
		font-size: 26rpx;
		color: #838383;
	}
	
	/* 文章数量 */
	.loadmore {
		padding: 20rpx 0;
	}
</style>
