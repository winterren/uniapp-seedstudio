<template>
	<view>
		<zero-loading type="love" v-show="!ready"></zero-loading>
		<view class="bg"></view>
		<view class="container" v-show="ready">
			<view class="wave-bg">
				<view class="wave0"></view>
				<view class="wave1"></view>
			</view>
			<view class="container">
				<view class="top">
					<view class="line1">
						<view>SEED</view>
						<view>STUDIO</view>
					</view>
					<view class="line2">
						<text>萌芽</text>
						<text>工作室</text>
					</view>
				</view>
				<!-- 宫格 -->
				<uni-grid class="grid" :column="3" :highlight="true" @change="change">
					<uni-grid-item class="grid-item" v-for="(item, index) in list" :index="index" :key="index">
						<view class="grid-item-box" style="background-color: #fff;">
							<uni-icons :type="item.type" :size="40" color="#777" class="grid-item-box-icon" />
							<text class="grid-item-box-text">{{item.text}}</text>
						</view>
					</uni-grid-item>
				</uni-grid>
				<!-- 动态 -->
				<view class="cardList">
					<view class="cardItem" v-for="item in news">
						<image :src="item.cover" mode="" class="cardItemImg" ></image>
						<view class="cardItemTitle">{{item.title}}</view>
						<view class="cardItemIntro">{{item.content}}</view>
					</view>
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
				news: [
					
				],
				list: [
				{
					img: '/static/c3.png',
					text: '近期动态',
					url:'/pages/information/index',
					type: "fire"
				},
				{
					img: '/static/c1.png',
					text: '技术博客',
					url:'/pages/blog/index',
					type: "chat"
				},
				{
					img: '/static/c2.png',
					text: '大赛习题',
					url:'/pages/quiz/index',
					type: "calendar",
				},
				{
					img: '/static/c4.png',
					text: '作品展示',
					url:'',
					type: "vip"
				},
				{
					img: '/static/c4.png',
					text: '2023毕设',
					url:'',
					type: "paperplane"
				},
				{
					img: '/static/c4.png',
					text: '团队介绍',
					url:'/pages/about/about',
					type: "staff"
				},
				]
			}
		},
		onLoad() {
			this.getInformation()
			
		},
		onReady() {
			this.ready=true
		},
		onShareAppMessage(){
			
		},
		onShareTimeline(){
			
		},
		methods: {
			change(e) {
				let {index} = e.detail
				if(this.list[index].url==''){
					uni.showToast({
						icon:"none",
						title:"该功能暂未开放！敬请期待！"
					})
				}
				uni.navigateTo({
					url:this.list[index].url
				})
				// uni.showToast({
				// 	title: `点击第${index+1}个宫格`,
				// 	icon: 'none'
				// })
			},
			// 获取动态
			getInformation() {
				uni.request({
					url: this.$url + '/api/informations?pagination[pageSize]=3&pagination[page]=1&populate=*&sort[0]=updatedAt:desc',
					method: 'GET',
					success:(res) => {
						res.data.data.forEach((item)=>{
							this.news.push({
								id: item.id,
								title: item.attributes.title,
								content: item.attributes.content,
								time: item.attributes.create_date,
								cover: this.$url + item.attributes.cover.data.attributes.url
							})
						})
					},
					fail: () => {
						uni.showToast({
							icon:"none",
							title:"服务器开小差了，请稍候重试"
						})
					}
				})
			},
		}
	}
</script>

<style>
	page{
		/* #ifdef MP */
		background-color: var(--studioBg);
		/* #endif */
	}
	.container{
		padding: 30rpx
	}
	/* 头部 */
	.top{
		width: 100%;
		height: 300rpx;
		text-align: center;
	}
	.top .line1{
		margin-top: 50rpx;
		font-size: 60rpx;
		font-weight: bold;
		letter-spacing: 5rpx;
	}
	.top .line2{
		margin-top: 20rpx;
		font-size: 40rpx;
		letter-spacing: 5rpx;
	}
	.top .line2 text:first-child{
		font-weight: bold;
		margin-right: 10rpx;
	}
	/* 信息卡 */
	.cardItem{
		background-color: rgba(255,255,255,0.8);
		border: 1rpx solid #D2D2D2;
		margin-top: 30rpx;
	}
	.cardItemImg{
		width: 100%;
		height: 320rpx;
	}
	.cardItemTitle{
		margin: 10rpx 20rpx 0;
		font-size: 28rpx;
		line-height: 50rpx;
	}
	.cardItemIntro{
		margin: 10rpx 20rpx 20rpx;
		color: #888;
		font-size: 24rpx;
	}
	
	/* 宫格 */
	.grid{
		margin-top: 50rpx;
	}
	.grid-item{
		background-color: rgba(255,255,255,0.8);
	}
	.grid-item-box {
		flex: 1;
		// position: relative;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 20rpx 0;
		font-size: 26rpx;
		background: none !important;
	}
	.grid-item-box-text{
		margin-top: 15rpx;
		letter-spacing: 2rpx;
	}
	
	/* 动画波浪背景 */
	.bg{
		position: fixed;
		width: 100%;
		/* height: 100vh; */
		left: 0;
		top: 0;
		bottom: 0;
		background-color: #F5F5F5;
		background-color: var(--studioBg);
		z-index: -30;
	}
	.wave-bg {
		position: absolute;
		left: 20%; 
		top: 15%;
		/* padding: 0; */
		/* border: 0; */
		width: 500rpx;
		/* height: 400rpx; */
		/* background-color: rgb(255, 255, 255); */
		/* border-radius: 50%; */
	}
	.wave0,.wave1{
		position: absolute;
		width: 1000rpx;
		height: 1000rpx;
		margin-top: -150%;
		margin-left: -50%;
		background-color: rgba(58, 214, 177,.4);
		border-radius: 45%;
		animation: spin 10s linear -0s infinite;
		z-index: -10;
	}
	.wave1{
		margin-top: -152%;
		background-color: rgb(58, 214, 177,.9);
		border-radius: 47%;
		animation: spin 30s linear -15s infinite;
		z-index: -20;
	}
	@keyframes spin{
		0%{
			transform: translate(-0%, -0%) rotate(0deg) scale(1);
		}
		25%{
			transform: translate(-1%, -1%) rotate(90deg) scale(1.0);
		}
		50%{
			transform: translate(-0%, -2%) rotate(180deg) scale(1.0);
		}
		75%{
			transform: translate(1%, -1%) rotate(270deg) scale(1.0);
		}
		100%{
			transform: translate(-0%, -0%) rotate(360deg) scale(1.0);
		}
	}

</style>
