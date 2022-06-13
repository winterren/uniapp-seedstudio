<template>
	<view>
		<view v-if="!ready" >
			<zero-loading type="love"></zero-loading>
		</view>
		<view class="container" v-if="ready">
			<view class="bg">
				<view class="bg-green">
					<ul class="bg-bubbles">
						<li></li><li></li><li></li><li></li><li></li><li></li><li></li><li></li><li></li><li></li><li></li>
					</ul>
				</view>
			</view>
			<view class="round">{{score}}%</view>
		
			<view class="card">
				<view class="card-item" v-for="(item,index) in scoreForm" v-if="item.numTotal>0">
					<text class="txt-label">{{item.lbl}}</text>
					<text class="txt-correct">{{item.numCorrect}}</text>
					<text class="txt-split">/</text>
					<text class="txt-total">{{item.numTotal}}</text>
					<text class="lbl pass" v-if="item.pass==true">达标</text>
					<text class="lbl fail" v-else>未达标</text>
				</view>
				<view class="btn red" v-show="!saved" @click="handleSave">保存到错题本</view>
				<view class="btn grey" v-show="saved">已保存错题本</view>
				<view class="btn green" @click="handleQuit">退回主页页面</view>
			</view>
		</view>
	</view>	
</template>

<script>
	export default {
		data() {
			return {
				ready:false,
				scoreForm:[
					{id:0,lbl:"总正确率",numCorrect:300,numTotal:500,pass:true},
					{id:1,lbl:"单选题",numCorrect:1255,numTotal:1500,pass:true},
					{id:2,lbl:"多选题",numCorrect:20,numTotal:50,pass:false},
					{id:3,lbl:"判断题",numCorrect:100,numTotal:500,pass:false},
				],
				saved: false,
				aRecordList:[],
				score: 0
			}
		},
		onLoad() {
			this.aRecordList = uni.getStorageSync("exam_practice")
			// 总正确率
			this.scoreForm[0].numTotal = this.aRecordList.filter( (item)=>{ return item.correct!=null } ).length
			this.scoreForm[0].numCorrect = this.aRecordList.filter( (item)=>{ return item.correct==true } ).length
			this.scoreForm[0].pass = (this.scoreForm[0].numCorrect / this.scoreForm[0].numTotal)>0.6
			this.score = Math.floor((this.scoreForm[0].numCorrect / this.scoreForm[0].numTotal) * 1000) / 10
			// 单选正确率
			this.scoreForm[1].numTotal = this.aRecordList.filter( (item)=>{ return item.type=='single' && item.correct!=null } ).length
			this.scoreForm[1].numCorrect = this.aRecordList.filter( (item)=>{ return item.type=='single' && item.correct==true } ).length
			this.scoreForm[1].pass = (this.scoreForm[1].numCorrect / this.scoreForm[1].numTotal)>0.6
			// 多选正确率
			this.scoreForm[2].numTotal = this.aRecordList.filter( (item)=>{ return item.type=='multi' && item.correct!=null } ).length
			this.scoreForm[2].numCorrect = this.aRecordList.filter( (item)=>{ return item.type=='multi' && item.correct==true } ).length
			this.scoreForm[2].pass = (this.scoreForm[2].numCorrect / this.scoreForm[2].numTotal)>0.6
			// 判断正确率
			this.scoreForm[3].numTotal = this.aRecordList.filter( (item)=>{ return item.type=='tf' && item.correct!=null } ).length
			this.scoreForm[3].numCorrect = this.aRecordList.filter( (item)=>{ return item.type=='tf' && item.correct==true } ).length
			this.scoreForm[3].pass = (this.scoreForm[3].numCorrect / this.scoreForm[3].numTotal)>0.6
			this.ready = true;
		},
		methods: {
			handleSave(){
				let wrongList = this.aRecordList.filter( (item)=>{ return item.correct==false } )
				let oriWrongList = uni.getStorageSync("exam_wrong_list")
				let newWrongList = wrongList.filter( (item)=>{
					return oriWrongList.indexOf(item.qid)==-1
				} )
				uni.setStorageSync("exam_wrong_list",[...oriWrongList,...newWrongList])
				this.saved = true
			},
			handleQuit(){
				// uni.navigateBack({
				// 	delta:2
				// })
				uni.reLaunch({
					url: '/pages/index/index'
				});
			}
		}
	}
</script>

<style>
	.container{padding-top: 30rpx; padding-bottom: 30rpx;}

	.title{ padding: 60rpx 30rpx 90rpx; font-size: 40rpx; font-weight: normal; letter-spacing: 10rpx; text-align: center; color: #333;} 
	.title text{ font-weight: bold; }

	/* 圆形 */
	.round{ width: 130rpx;height: 130rpx; background-color: rgba(255,255,255,.9); margin: 0 auto; border-radius: 50%;
	text-align: center;line-height: 130rpx;font-size: 40rpx;font-weight: bold;color: #299178;border: 15rpx solid #299178; }
	
	/* 卡片 */
	.card{ background-color: rgba(255,255,255,.9); margin: 30rpx 30rpx 0; padding: 30rpx; border: 1rpx solid #D2D2D2; font-size: 28rpx;  }
	.card-item{ display: flex; align-items: center; height: 60rpx; justify-content: space-around; }
	.card-item text{ flex: 1 1; text-align: center;}
	.card-item .txt-split{ flex: 0 0 20rpx;}
	.lbl{ font-size: 24rpx;line-height: 40rpx;height: 40rpx; width: 90rpx; text-align: center;color:#333 ;border-radius: 10rpx;}
	.lbl.pass{border: 2rpx solid var(--studioGreen);color: var(--studioGreen); }
	.lbl.fail{border: 2rpx solid var(--studioRed);color: var(--studioRed); }
	.btn{ width: 100%; line-height: 80rpx; color: #FFF;text-align: center; border: 1rpx solid #D2D2D2; margin-top: 20rpx;}
	.btn.red{ background-color: var(--studioRed); }
	.btn.green{ background-color: var(--studioGreen); }
	.btn.grey{ background-color: #D2D2D2; }
	
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
