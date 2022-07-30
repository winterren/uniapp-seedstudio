<template>
	<view>
		<view v-if="!ready" >
			<zero-loading type="love"></zero-loading>
		</view>
		<view class="title"><text>大赛</text>习题</view>
		<view class="container" v-if="ready">
			<bg></bg>
			<view class="card">
				<view class="cardBtn" @click="navToPage('/pages/quiz/practice/index')">
					<uni-icons class="iconfont" type="paperplane" size="30" color="#fff"></uni-icons>
					练习模式
				</view>
				<view class="cardBtn" @click="navToPage('/pages/quiz/wrong/index')">
					<uni-icons class="iconfont" type="flag" size="30" color="#fff"></uni-icons>
					我的错题
				</view>
				<view class="cardBtn" @click="handleToBeOpen">
					<uni-icons class="iconfont" type="paperplane" size="30" color="#fff"></uni-icons>
					考试模式
				</view>
				<view class="cardBtn" @click="handleToBeOpen">
					<uni-icons class="iconfont" type="flag" size="30" color="#fff"></uni-icons>
					赛题分析
				</view>
				
			</view>
		</view>
	</view>	
</template>

<script>
	import bg from '../../components/bg/bg-green-square.vue'
	export default {
		data() {
			return {
				ready:true
			}
		},
		onLoad() {
		},
		components:{
			bg
		},
		methods: {
			navToPage(str){
				console.log(str)
				uni.navigateTo({
					url: str
				})
			},
			handleToBeOpen(){
				this.$toast('该模块暂未开放，敬请期待！')
			}
		}
	}
</script>

<style>
	.container{padding-top: 0rpx; padding-bottom: 30rpx;}

	.title{ padding: 60rpx 30rpx 90rpx; font-size: 40rpx; font-weight: normal; letter-spacing: 10rpx; text-align: center; color: #333;} 
	.title text{ font-weight: bold; }

	
	/* 卡片 */
	.card{ background-color: rgba(255,255,255,.9); margin: 30rpx 30rpx 0; padding: 30rpx; border: 1rpx solid #D2D2D2; font-size: 32rpx; 
	 display: flex; flex-wrap: wrap;
	}
	.cardBtn{ width: 100%; height: 150rpx; display: flex;align-items: center;justify-content: center; color: #FFF;
	 margin-bottom: 30rpx;}
	 .cardBtn:nth-child(1){background-color: #4daadb;}
	 .cardBtn:nth-child(2){background-color: #e98181;}
	 .cardBtn:nth-child(3){background-color: #8bcf73;}
	 .cardBtn:nth-child(4){background-color: #f1d064;}
	 
	 .cardBtn:last-child{ margin-bottom: 0;}
	 
	 .cardBtn .iconfont{margin-right: 10rpx;}
	
	
	
</style>
