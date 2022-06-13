<template>
	<view class="main">
		<!-- 页面加载动画 -->
		<view v-if="!ready">
			<zero-loading type="love"></zero-loading>
		</view>
		<!-- 分类 -->
		<view class="cate">
			<scroll-view scroll-x="true" class="u-line-1" style="white-space: nowrap;">
				<view class="cateItems"
					:class="{cateItemSelect:item.id == cateItemId}" 
					@click="changeCateItem(item.id)" 
					v-for="(item,index) in catesList">
					{{item.attributes.name}}
				</view>
			</scroll-view>
		</view>
		<!-- 分类置顶占位 -->
		<view class="catePlaceholder"></view>
		<!-- 博客列表 -->
		<view class="blog">
			<view class="blogItems" v-for="(item,index) in blogList" @click="goBlogDetails(item.id)">
				<view class="title u-line-1">{{item.attributes.title}}</view>
				<view class="info">
					<view class="user">{{item.attributes.users_permissions_user.data.attributes.nickname}}</view>
					<view class="time">{{item.attributes.created_date}}</view>
				</view>
			</view>
		</view>
		<!-- 文章数量 -->
		<view class="loadmore" v-show="loadmoreShow">
			<u-loadmore :status="status" :load-text="loadText"/>
		</view>
		<!-- 文章为空提示 -->
		<view class="empty" v-show="emptyShow">
			<u-empty text="该分类暂时没有博文,敬请期待!"></u-empty>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				ready: false,
				catesList: [],
				cateItemId: 1,
				blogList: [],
				emptyShow: false,
				status: 'loadmore',
				loadText: {
					loadmore: '轻轻上拉',
					loading: '努力加载中',
					nomore: '没有更多了'
				},
				loadmoreShow: false,
				page: 1,
				pageSize: 10,
				pageCount: '',
				blogTotal: ''
			}
		},
		methods: {
			// 获取分类
			getCates() {
				uni.request({
					url: 'https://www.seedstudio.cn:1437/api/blog-cates',
					method: 'GET',
					success:(res) => {
						this.catesList = res.data.data
						this.getBlogList()
					}
				})
			},
			// 改变分类
			changeCateItem(id) {
				this.cateItemId = id
				this.loadmoreShow = false
				this.emptyShow = false
				this.blogList = []
				this.page = 1
				this.ready = false
				this.getBlogList()
			},
			// 获取博客列表
			getBlogList() {
				uni.request({
					url: 'https://www.seedstudio.cn:1437/api/blogs?populate=blog_cate&populate=users_permissions_user&filters[blog_cate][id]='+ this.cateItemId +'&sort[0]=id&pagination[pageSize]=' + this.pageSize + '&pagination[page]=' + this.page,
					method: 'GET',
					success:(res) => {
						this.blogList = [ ...this.blogList,...res.data.data  ]
						this.pageCount = res.data.meta.pagination.pageCount
						this.blogTotal = res.data.meta.pagination.total
						if(this.blogList.length == 0) {
							this.emptyShow = true
						} else {
							this.loadmoreShow = true
							this.estimateBlogList()
						}
						this.ready = true
					}
				})
			},
			// 判断博客条数
			estimateBlogList() {
				if(this.blogList.length == this.blogTotal) {
					this.status = 'nomore'
				}
			},
			// 前往博客详情
			goBlogDetails(id) {
				uni.navigateTo({
					url: './blogDetails?id=' + id
				})
			}
		},
		created(){
			// 获取分类
			this.getCates()
		},
		onReachBottom() {
			if(this.page >= this.pageCount) return
			this.status = 'loading'
			this.page == ++ this.page
			this.status = 'loading'
			this.getBlogList()
		}
	}
</script>

<style>
	/* 页面 */
	page {
		background-color: var(--studioBg); 
		height: 100%;
	}
	
	.main{
		background-color: var(--studioBg); 
	}
	
	/* 分类 */
	.cate {
		height: 60rpx;
		width: 100%;
		line-height: 60rpx;
		padding: 0 30rpx;
		position: fixed;
		background-color: #fdfaed;
		z-index: 2;
	}
	.cateItems {
		font-size: 30rpx;
		display: inline-block;
		width: 200rpx;
		text-align: center;
		color: #939296;
	}
	.cateItemSelect {
		color: var(--studioGreen);
		font-size: 32rpx;
		font-weight: bold;
		border-bottom: 1rpx solid var(--studioGreen);
	}
	
	/* 分类置顶占位 */
	.catePlaceholder {
		height: 60rpx;
	}
	
	/* 博客列表 */
	.blog {}
	.blogItems {
		padding: 20rpx 10rpx;
		margin: 0 30rpx;
		border-bottom: 1rpx #ececec solid;
	}
	.title {
		font-size: 34rpx;
		line-height: 60rpx;
	}
	.info {
		display: flex;
		font-size: 24rpx;
		color: #838383;
	}
	.user {
		margin-right: 30rpx;
	}
	.time {}
	
	/* 文章数量 */
	.loadmore {
		padding: 20rpx 0;
	}
	
	/* 页面为空 */
	.empty {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%,-50%);
		z-index: 1;
		width: 100%;
		height: 100%;
	}
</style>
