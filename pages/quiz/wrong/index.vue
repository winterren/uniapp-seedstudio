<template>
	<view>
		<view v-if="!ready">
			<zero-loading type="love"></zero-loading>
		</view>
		<view class="container" v-if="ready">
			
			<!-- 乱序 -->
			<view class="formItem">
				<view class="formItemLeft">
					题目乱序
				</view>
				<view class="formItemRight">
					<switch color="#3ad6b1" style="transform:scale(0.7)" @change="switchRandQuestion"/>
				</view>
			</view>
			<!-- 乱序 -->
			<view class="formItem">
				<view class="formItemLeft">
					选项乱序
				</view>
				<view class="formItemRight">
					<switch color="#3ad6b1" style="transform:scale(0.7)" @change="switchRandAnswer"/>
				</view>
			</view>
			<!-- 开始按钮 -->
			<view class="btnStart" @click="handleStart">练习错题</view>
			<view class="btnStart" @click="handleWrong">管理错题</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				ready: false,
				cateIndex:0,
				cateArray: [],
				fromIndex:0,
				// fromArray: [],
				toIndex:1,
				// toArray: [100],
				totalQuestionNum: 0,
				totalQuestionNumArr: [],
				flagRandQuestion: false,
				flagRandAnswer: false,
			}
		},
		onLoad(){
			this.getCate()
		},
		methods: {
			getCate(){
				uni.request({
					url:this.$url+"/api/exam-categories?filters[active]=true",
					success: (res) => {
						console.log(res.data.data)
						res.data.data.forEach((item)=>{
							this.cateArray.push({
								id: item.id,
								name: item.attributes.name
							})
						})
						
						// 获得题号
						this.getFromTo()
						
						
					},
					fail: (res)=>{
						this.$toast("网络连接有误，请稍候重试")
					}
				})
			},
			getFromTo(){
				uni.request({
					url: this.$url+"/api/exam-questions?populate=exam_category&filters[exam_category][id]="+this.cateArray[this.cateIndex].id,
					success: (res) => {
						// 防呆
						if( res.data.meta.pagination.total == 0 ){ uni.showToast({icon: "none",title: "该题库为空"});return; }
						// console.log(res.data.meta.pagination.total)
						this.totalQuestionNum = res.data.meta.pagination.total
						// this.totalQuestionNum = 778
						// 对题目总数进行处理，转化成整百数组
						let num = Math.ceil(this.totalQuestionNum/100)-1
						// console.log(Math.ceil(this.totalQuestionNum/100)-1)
						this.totalQuestionNumArr=[0]
						for(let i=0;i<num;i++){
							this.totalQuestionNumArr.push((i+1)*100)
						}
						this.totalQuestionNumArr.push(this.totalQuestionNum)
						this.toIndex = num+1
						console.log(this.totalQuestionNumArr)
						this.ready = true
					},
					fail: (res)=>{
						this.$toast("网络连接有误，请稍候重试")
					}
				})
			},
			handleCateChange(e){
				console.log('picker发送选择改变，携带值为', e.detail.value)
				this.cateIndex = e.detail.value
				this.getFromTo()
			},
			handleFromPicker(e){
				if(e.detail.value>=this.toIndex){ 
					uni.showToast({icon: "none",title: "起始位置要小于结束位置"})
					return; 
				}
				console.log('picker发送选择改变，携带值为', e.detail.value)
				this.fromIndex = e.detail.value
			},
			handleToPicker(e){
				if(e.detail.value<=this.fromIndex){ 
					uni.showToast({icon: "none",title: "起始位置要小于结束位置"})
					return; 
				}
				console.log('picker发送选择改变，携带值为', e.detail.value)
				this.toIndex = e.detail.value
			},
			switchRandQuestion(e) {
				this.flagRandQuestion = e.detail.value
				console.log('switch1 发生 change 事件，携带值为', e.detail.value)
			},
			switchRandAnswer(e){
				this.flagRandAnswer = e.detail.value
				console.log('switch1 发生 change 事件，携带值为', e.detail.value)
			},
			handleStart(){
				let cate = this.cateArray[this.cateIndex].id
				let from = this.totalQuestionNumArr[this.fromIndex]
				let to = this.totalQuestionNumArr[this.toIndex]
				let rq = this.flagRandQuestion
				let ra = this.flagRandAnswer
				console.log(cate,from,to,rq,ra)
				uni.navigateTo({
					url: '/pages/quiz/wrong/practice?cate='+cate+"&from="+from+"&to="+to
						+"&rq="+rq+"&ra="+ra
				})
			},
			handleWrong(){
				uni.navigateTo({
					url: '/pages/quiz/wrong/manage'
				})
			}
		}
	}
</script>

<style>
	page{ 
		background-color: var(--studioBg); 
		/* #ifdef H5 */
		height: 100vh;
		/* #endif */
	}
	
	.container{ padding: 30rpx; }
	
	.formItem{ line-height: 100rpx; display: flex; align-items: center;}
	.formItemLeft{ width: 150rpx; flex: 0 0 150rpx; color: #333;}
	.formItemRight{ flex: 1 1 auto;}
	.picker{ background-color: #fff; color: #666; line-height: 70rpx; height: 70rpx; font-size: 28rpx; text-align: center; border: 1rpx solid #D2D2D2;}
	
	.btnStart{ width: 100%; line-height: 100rpx; background-color: var(--studioGreen); text-align: center; color: #FFF; margin-top: 50rpx;}
</style>
