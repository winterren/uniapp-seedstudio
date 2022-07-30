<template>
	<view>
		<!-- 页面加载动画 -->
		<view v-if="!ready">
			<zero-loading type="love"></zero-loading>
		</view>
		<image class="headerImg" v-show="!cateIsFixed" src="https://www.seedstudio.cn:1437/uploads/bolg_Header_425b4a5484.jpg?updated_at=2022-06-18T09:46:14.572Z"></image>
		<!-- 分类 -->
		<view class="cate" :class="{cateFixed:cateIsFixed}">
			<scroll-view scroll-x="true" style="white-space: nowrap;">
				<view class="cateItems"
					:class="{cateItemSelect:item.id == cateItemId}" 
					@click="changeCateItem(item.id)" 
					v-for="(item,index) in catesList">
					{{item.attributes.name}}
				</view>
			</scroll-view>
		</view>
		<!-- 分类置顶占位 -->
		<view class="catePlaceholder" v-if="cateIsFixed"></view>
		<view class="mainBox" @touchstart="start" @touchend="end">
			<view class="main">
				<!-- 博客列表 -->
				<view class="blog">
					<view class="blogItems" v-for="(item,index) in blogList" @click="goBlogDetails(item.id)">
						<view class="title lineOne">{{item.title}}</view>
						<view class="info">
							<view class="user">{{item.user}}</view>
							<view class="time">{{item.time}}</view>
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
				pageSize: 20,
				pageCount: '',
				blogTotal: '',
				startClientX: 0,
				cateIsFixed: false
			}
		},
		methods: {
			// 获取分类
			getCates() {
				uni.request({
					url: this.$url + '/api/blog-cates',
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
					url: 'https://www.seedstudio.cn:1437/api/blogs?populate=blog_cate&populate=users_permissions_user&filters[blog_cate][id]='+ this.cateItemId +'&sort[0]=updatedAt:desc&pagination[pageSize]=' + this.pageSize + '&pagination[page]=' + this.page ,
					method: 'GET',
					success:(res) => {
						// this.blogList = [ ...this.blogList,...res.data.data  ]
						res.data.data.forEach((item)=>{
							this.blogList.push({
								id: item.id,
								title: item.attributes.title,
								user: item.attributes.users_permissions_user.data.attributes.nickname,
								time: item.attributes.created_date
							})
						})
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
			},
			// 左右滑动切换分类
			// 手指开始触摸屏幕
			start(e) {
				this.startClientX = e.changedTouches[0].clientX
			},
			// 手指停止触摸屏幕
			end(e) {
				const subX = e.changedTouches[0].clientX - this.startClientX
				if(subX > 100) {
					if(this.cateItemId > 1) {
						this.changeCateItem(--this.cateItemId)
					}
				} else if(subX < -100) {
					if(this.cateItemId < this.catesList.length) {
						this.changeCateItem(++this.cateItemId)
					}
				}
			},
			// 图片计时器
			headerImgIsShow() {
				setTimeout(() => {
					this.imgShow = false
				}, 2000)
			}
		},
		created(){
			// 获取分类
			this.getCates()
			this.headerImgIsShow()
		},
		onReachBottom() {
			if(this.page >= this.pageCount) return
			this.status = 'loading'
			this.page == ++ this.page
			this.status = 'loading'
			this.getBlogList()
		},
		onPageScroll(res) {
			let query = uni.createSelectorQuery().in(this);
			query.select('.headerImg').boundingClientRect(data => {
				let headerImgBottom = data.bottom
				if(headerImgBottom < 0) {
					this.cateIsFixed = true
				}
			}).exec();
		}
	}
</script>

<style>
	.mainBox {
		height: calc(100vh - 88rpx - 80rpx);
	}
	/* 页面 */
	/* page {
		background-color: var(--studioBg); 
		height: 100%;
	} */
	
	.headerImg {
		width: 100%;
		height: 320rpx;
	}
	
	/deep/.uni-scroll-view::-webkit-scrollbar {
		/* 隐藏滚动条，但依旧具备可以滚动的功能 */
		display: none;
	}
	
	/* 分类 */
	.cate {
		height: 80rpx;
		width: 100%;
		line-height: 80rpx;
		padding: 0 30rpx;
		background-color: #fff;
		z-index: 2;
	}
	.cateFixed {
		position: fixed;
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
	.blog {
		margin-top: 10rpx;
		height: 100%;
	}
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
		padding-top: 200rpx;
	}
	
	/* 文本一行显示 */
	.lineOne {
		overflow: hidden !important;
		text-overflow: ellipsis !important;
		display: -webkit-box !important;
		-webkit-line-clamp: 1;//文字上限行
		-webkit-box-orient: vertical;
	}
</style>
