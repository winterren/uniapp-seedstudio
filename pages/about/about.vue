<template>
	<view class="container">
		<view class="bg">
			<view class="bg-green">
				<ul class="bg-bubbles">
					<li></li><li></li><li></li><li></li><li></li><li></li><li></li><li></li><li></li><li></li><li></li>
				</ul>
			</view>
		</view>
		<view class="title"><text>团队</text>成员</view>
		<view class="card" v-for="(item,index) in memberList">
			<view class="card-left">
				<image class="avatar" :src="item.img" mode=""></image>
			</view>
			<view class="card-right">
				<view class="card-right-title"><text class="name">{{item.name}} </text>	<text class="tag">{{item.tag}}</text></view>
				<view class="card-right-intro" v-html="item.intro"></view>
			</view>
			
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				memberList: [
					{ name:"任远", tag:"团队教师", intro:"指导教师",img:"/static/img/about/avatar-teacher-ry.jpg" },
					{ 
						name:"包宸嘉", 
						tag:"2018级学生", 
						intro:"前端技术栈：HTML/CSS/JS/Vue<br/>后端技术栈：Java/NodeJS/ASP",
						img:"/static/img/about/avatar-18-bcj.jpg" ,
					},
					{
						name:"付禹辰", 
						tag:"2018级学生", 
						intro:"前端技术栈：HTML/CSS/JS/Vue<br/>后端技术栈：Java/NodeJS/ASP",
						img:"/static/img/about/avatar-18-fyc.jpg" ,
					},
					{
						name:"刘峻豪", 
						tag:"2018级学生", 
						intro:"前端技术栈：HTML/CSS/JS/Vue<br/>后端技术栈：Java/NodeJS/ASP",
						img:"/static/img/about/avatar-19-ljh.jpg" ,
					},
					{
						name:"孟子杰", 
						tag:"2018级学生", 
						intro:"前端技术栈：HTML/CSS/JS/Vue<br/>后端技术栈：Java/NodeJS/ASP",
						img:"/static/img/about/avatar-19-mzj.jpg" ,
					},
					{
						name:"胡俊杰", 
						tag:"2018级学生", 
						intro:"前端技术栈：HTML/CSS/JS/Vue<br/>后端技术栈：Java/NodeJS/ASP",
						img:"/static/img/about/avatar-19-hjj.jpg" ,
					},
					{
						name:"陈志豪", 
						tag:"2018级学生", 
						intro:"前端技术栈：HTML/CSS/JS/Vue<br/>后端技术栈：Java/NodeJS/ASP",
						img:"/static/img/about/avatar-19-czh.jpg" ,
					},

				]
			}
		},
		methods: {
			
		}
	}
</script>

<style>
	.container{padding-top: 30rpx; padding-bottom: 30rpx;}

	.title{ padding: 60rpx 30rpx 90rpx; font-size: 40rpx; font-weight: normal; letter-spacing: 10rpx; text-align: center; color: #333;} 
	.title text{ font-weight: bold; }
	.card{ background-color: rgba(255,255,255,.9); margin: 30rpx 30rpx 0; padding: 30rpx; border: 1rpx solid #D2D2D2; display: flex; align-items: center;}
	.card-left{}
	.card-left .avatar{ width: 150rpx; height: 150rpx; border-radius: 50%;}
	.card-right{ padding-left: 30rpx;}
	.card-right .card-right-title{ margin-bottom: 10rpx; }
	.card-right .card-right-title .name{ font-size: 30rpx; font-weight: bold; line-height: 50rpx; margin-bottom: 10rpx; letter-spacing: 2rpx;}
	.card-right .card-right-title .tag{ font-size: 24rpx; font-weight: normal; line-height: 50rpx; color: #999; margin-left: 20rpx;}
	.card-right .card-right-intro{ font-size: 24rpx; line-height: 38rpx;}
	
	/* bg */
	.bg{ width: 100%; height: 100vh; background-color: var(--studioBg); position: fixed; top: 0; left: 0; z-index: -10;}
	.bg-green{  width: 100%; height: 400rpx; background-color: var(--studioGreen); position: absolute; top: 0; left: 0;  }
	.bg-bubbles {
	  position: absolute;
	  top: 0;
	  left: 0;
	  width: 100%;
	  height: 100%;
	  z-index: 1;
	}
	.bg-bubbles li {
	  position: absolute;
	  list-style: none;
	  display: block;
	  width: 40px;
	  height: 40px;
	  background-color: rgba(255, 255, 255, 0.15);
	  bottom: -160px;
	  -webkit-animation: square 25s infinite;
	  animation: square 25s infinite;
	  transition-timing-function: linear;
	}
	.bg-bubbles li:nth-child(1) {
	  left: 10%;
	}
	.bg-bubbles li:nth-child(2) {
	  left: 20%;
	  width: 80px;
	  height: 80px;
	  -webkit-animation-delay: 2s;
	          animation-delay: 2s;
	  -webkit-animation-duration: 17s;
	          animation-duration: 17s;
	}
	.bg-bubbles li:nth-child(3) {
	  left: 25%;
	  -webkit-animation-delay: 4s;
	          animation-delay: 4s;
	}
	.bg-bubbles li:nth-child(4) {
	  left: 40%;
	  width: 60px;
	  height: 60px;
	  -webkit-animation-duration: 22s;
	          animation-duration: 22s;
	  background-color: rgba(255, 255, 255, 0.25);
	}
	.bg-bubbles li:nth-child(5) {
	  left: 70%;
	}
	.bg-bubbles li:nth-child(6) {
	  left: 80%;
	  width: 120px;
	  height: 120px;
	  -webkit-animation-delay: 3s;
	          animation-delay: 3s;
	  background-color: rgba(255, 255, 255, 0.2);
	}
	.bg-bubbles li:nth-child(7) {
	  left: 32%;
	  width: 160px;
	  height: 160px;
	  -webkit-animation-delay: 7s;
	          animation-delay: 7s;
	}
	.bg-bubbles li:nth-child(8) {
	  left: 55%;
	  width: 20px;
	  height: 20px;
	  -webkit-animation-delay: 15s;
	          animation-delay: 15s;
	  -webkit-animation-duration: 40s;
	          animation-duration: 40s;
	}
	.bg-bubbles li:nth-child(9) {
	  bottom: -30px;
	  left: 25%;
	  width: 10px;
	  height: 10px;
	  -webkit-animation-delay: 2s;
	          animation-delay: 2s;
	  -webkit-animation-duration: 40s;
	          animation-duration: 40s;
	  background-color: rgba(255, 255, 255, 0.3);
	}
	.bg-bubbles li:nth-child(10) {
	  left: 90%;
	  width: 160px;
	  height: 160px;
	  -webkit-animation-delay: 11s;
	          animation-delay: 11s;
	}
	.bg-bubbles li:nth-child(11) {
	  bottom: 0;
	  left: 67%;
	  width: 100px;
	  height: 100px;
	  -webkit-animation-delay: 0s;
	          animation-delay: 0s;
	  -webkit-animation-duration: 20s;
	          animation-duration: 20s;
	  background-color: rgba(255, 255, 255, 0.15);
	}
	@-webkit-keyframes square {
	  0% {
	    transform: translateY(0);
	  }
	  100% {
	    transform: translateY(-700px) rotate(600deg);
	  }
	}
	@keyframes square {
	  0% {
	    transform: translateY(0);
	  }
	  100% {
	    transform: translateY(-700px) rotate(600deg);
	  }
	}
</style>
