<template>
	<view>
		<view v-if="!ready">
			<zero-loading type="love"></zero-loading>
		</view>
		<view class="container" v-if="ready">
			<!-- 抽屉 -->
			<uni-drawer ref="showMenu" mode="right" :mask-click="true" :width="300">
				<scroll-view class="maskMenu" style="height: 100%;" scroll-y="true" >
					<view class="maskBtnList">
						
						<view class="btnClose">重置</view>
						<view class="btnClose">交卷</view>
						<navigator url="/pages/index/index" open-type="redirect"  class="btnClose">
							<view>退出</view>
						</navigator>
						
					</view>
				
					<view class="qView">
						<view class="qTitle">题目列表</view>
						<view class="qList">
							<view class="qItem" 
							v-for="(item,index) in aRecordList" 
							:key="index"
							:class="[{correct:item.correct===true},{wrong:item.correct===false}]"
							@click="changeToQuestion(index)"
							>{{ parseInt(item.id)+1}}</view>
						</view>
					</view>
				</scroll-view>
			</uni-drawer>
			<!-- 页面顶部 -->
			<view class="top">
				<view class="tag"># {{parseInt(qList[currentQuestion].id)+1}}  
					<text v-if="this.type=='single'">单选题</text>
					<text v-if="this.type=='multi'">多选题</text>
				</view>
				<view class="iconList">
					<span class="iconfont">&#xe6e5;</span>
					<span class="iconfont">&#xe6eb;</span>
					<span class="iconfont" @click="showMenu">&#xe6ef;</span>
				</view>
			</view>
			<!-- 页面主体 -->
			<view class="subject">{{qList[currentQuestion].subject}}</view>
			<!-- 单选 -->
			<view class="ansList" v-if="this.type!='multi'">
				<view class="ansOpt" 
				v-for="(item,index) in qList[currentQuestion].optList.filter(item => item!= null)" 
				@click="handleAnsClick(index)"
				:style="randomOrder(index)"
				:class="[{correct:hasAnswered&&answerIsRight(index)},{wrong:hasAnswered&&index==currentAnsOpt&&!currentAnswerIsRight}]" 
				>{{item}}</view>
			</view>
			<!-- 多选 -->
			<view class="ansList" v-if="this.type=='multi'">
				<view class="ansOpt" 
				v-for="(item,index) in qList[currentQuestion].optList.filter(item => item!= null)" 
				@click="handleMultiAnsClick(index)"
				:style="randomOrder(index)"
				:class="[{active:currentMultiAnsOpt[index]===true}
				,{correct:hasAnswered&&currentAnswerIsRight&&currentMultiAnsOpt[index]===true}
				,{wrong:hasAnswered&&!currentAnswerIsRight&&currentMultiAnsOpt[index]===true}
				]"
				>{{item}}</view>
			</view>
			<view class="btnList">
				<view class="btnItem" 
				:class="{disabled:firstQ}"
				@click="changeToPre"
				>上一题</view>
				<view class="btnItem" 
				v-if="this.type=='multi'"
				:class="{disabled:hasAnswered}"
				@click="handleMultiSubmit"
				>确定</view>
				<view class="btnItem" 
				:class="{disabled:lastQ}"
				@click="changeToPost"
				>下一题</view>
			</view>
			<view class="analysis" v-if="hasAnswered&&!currentAnswerIsRight&&type=='multi'">
				正确答案：<ol><li v-for="(item,index) in qList[currentQuestion].optList" v-if="correctMulti[index]">{{item}};</li></ol>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				ready:false,
				qList:[],
				aRecordList:[],
				currentQuestion: 0,
				currentAnsOpt: null,
				hasAnswered: false,
				type: null,
				currentMultiAnsOpt: [],
				randomAnswer: false,
				randomSeed: 3,
			}
		},
		computed:{
			answerIsRight(){
				return function(opt){
					return opt == this.qList[this.currentQuestion].answer
				}
			},
			currentAnswerIsRight(){
				return this.currentAnsOpt == this.qList[this.currentQuestion].answer
			},
			correctMulti(){
				return this.convertStrToArr(this.qList[this.currentQuestion].answer)
			},
			firstQ(){
				return this.currentQuestion == 0
			},
			lastQ(){
				return this.currentQuestion == this.qList.length-1
			},
			randomOrder(){
				return function(i){
					if(this.randomAnswer){
						return {
							order: Math.floor(i%this.randomSeed)
						}
					}else{
						return {
							
						}
					}
				}
			}
		},
		onLoad(option){
			let cate = option.cate
			let from = option.from
			let to = option.to
			let rq = option.rq
			let ra = option.ra
			
			let emptyAnswerRecord = []
			
			this.randomSeed = Math.random()*2 + 2
			
			//请求题库
			uni.request({
				url: this.$url+'api/exam-questions?populate=exam_category&filters[exam_category]['+option.cate+']=1',
				success: (res) => {
					console.log(res.data.data)
					console.log(from,to)
					// for(let i=0; i<res.data.data.length; i++){
					for(let i=from; i<to; i++){
						let temp = res.data.data[i].attributes
						// 实例变量数组：保存题库信息
						this.qList.push({
							id: i,
							qid: res.data.data[i].id,
							subject: temp.subject,
							optList: [ temp.option_a, temp.option_b, temp.option_c, temp.option_d, temp.option_e, temp.option_f, temp.option_g, temp.option_h ],
							answer: temp.answer,
							type: temp.type
						})
						// 答题记录数组：保存正确答案
						emptyAnswerRecord.push({
							id: i,
							qid: res.data.data[i].id,
							answer: temp.answer,
							myAnswer: null,
							correct: null,
							type:temp.type
						})
					}
					// rq：题目乱序
					if(rq=='true'){
						let stack1 = [];
						let stack2 = [];
						while (this.qList.length) {
						    //Math.random()：返回 [0,1) 之间的一个随机数
						    let index = parseInt(Math.random() * this.qList.length);  // 利用数组长度生成随机索引值
						    stack1.push(this.qList[index]);  // 将随机索引对应的数组元素添加到新的数组中
						    stack2.push(emptyAnswerRecord[index]);  // 将随机索引对应的数组元素添加到新的数组中
						    this.qList.splice(index, 1);  // 删除原数组中随机生成的元素
						    emptyAnswerRecord.splice(index, 1);  // 删除原数组中随机生成的元素
						}
						this.qList=stack1
						emptyAnswerRecord=stack2
					}
					//选项乱序
					this.randomAnswer = (ra=='true')
					//请求答题记录：存入本地存储
					this.aRecordList = emptyAnswerRecord
					this.syncRecordList()
					this.ready = true
					// initialize
					this.type = this.qList[0].type
				}
			})
		},
		methods: {
			// 模态菜单
			showMenu() {
				console.log("show menu")
				this.$refs.showMenu.open();
			},
			closeMenu() {
				this.$refs.showMenu.close();
			},
			// 单击答案
			handleAnsClick(index){
				// 判断是否已经答过题
				if(this.hasAnswered){return;}
				// 如果没答过题，计入答题情况
				this.currentAnsOpt = index
				this.hasAnswered = true
				// 计入存储
				this.aRecordList[this.currentQuestion].myAnswer=this.currentAnsOpt
				this.aRecordList[this.currentQuestion].correct=this.currentAnswerIsRight
				this.syncRecordList()
			},
			// 切换到第i题
			changeToQuestion(i){
				this.currentQuestion = i
				this.type = this.qList[i].type
				// 判断是否为答过的题
				if(this.aRecordList[i].myAnswer!=null){
					this.currentAnsOpt = this.aRecordList[i].myAnswer
					this.hasAnswered = true
					// 判断是否为多选，选上存储选项
					this.currentMultiAnsOpt=[]
					if(this.type=="multi"){
						this.currentMultiAnsOpt = this.convertStrToArr(this.aRecordList[i].myAnswer)
					}
				}else{
					this.currentAnsOpt = null,
					this.hasAnswered = false
					// 判断是否为多选，清空
					this.currentMultiAnsOpt=[]
					if(this.type=="multi"){
						this.qList[this.currentQuestion].optList.forEach((item)=>{
							if(item!=null){
								this.currentMultiAnsOpt.push(false)
							}
						})
					}
				}
				
				
			},
			// 上一题；下一题
			changeToPre(){
				if(this.firstQ){return;}
				this.changeToQuestion(this.currentQuestion-1)
			},
			changeToPost(){
				//如果是多选，先交题
				if(this.type=="multi" && this.hasAnswered==false && this.currentMultiAnsOpt.indexOf(true)!=-1){ this.handleMultiSubmit(); return; }
				if(this.lastQ){return;}
				this.changeToQuestion(this.currentQuestion+1)
			},
			// 同步答题记录到存储
			syncRecordList(){
				uni.setStorageSync("practice",this.aRecordList)
			},
			// 多选题答案
			handleMultiAnsClick(index){
				if(this.hasAnswered){return;}
				// 引用类型转值类型
				this.currentMultiAnsOpt[index] = !this.currentMultiAnsOpt[index] 
				this.currentMultiAnsOpt = [].concat(this.currentMultiAnsOpt)
			},
			// 多选题提交
			handleMultiSubmit(){
				if(this.hasAnswered){return;}
				if(this.currentMultiAnsOpt.indexOf(true)==-1){
					uni.showToast({
						icon:"none",
						title:"请在选择选项后提交"
					})
					return;
				}
				this.currentAnsOpt = this.convertArrToStr(this.currentMultiAnsOpt)
				this.hasAnswered = true
				
				// 计入存储
				this.aRecordList[this.currentQuestion].myAnswer=this.currentAnsOpt
				this.aRecordList[this.currentQuestion].correct=this.currentAnswerIsRight
				console.log(this.aRecordList[this.currentQuestion].correct)
				this.syncRecordList()
			},
			// 转换多选题选项类型
			convertArrToStr(arr){
				let result=""
				for(let i=0;i<arr.length;i++){
				    if(arr[i]){
				       result+=i 
				    }
				}
				return result
			},
			convertStrToArr(str){
				let result = []
				this.qList[this.currentQuestion].optList.forEach((item)=>{
					if(item!=null){
						result.push(false)
					}
				})
				let arr = str.split("")
				for(let i=0;i<arr.length;i++){
					result[arr[i]]=true
				}
				return result
			},
			//乱序
			ConvertToRandomArray(arr) {
			    var stack = [];
			    while (arr.length) {
			        //Math.random()：返回 [0,1) 之间的一个随机数
			        var index = parseInt(Math.random() * arr.length);  // 利用数组长度生成随机索引值
			        stack.push(arr[index]);  // 将随机索引对应的数组元素添加到新的数组中
			        arr.splice(index, 1);  // 删除原数组中随机生成的元素
			    }
			    return stack;
			}

		}
	}
</script>

<style>
	/* font */
	@font-face {
	  font-family: 'iconfont';
	  src: url('/static/font_quiz/iconfont.ttf?t=1654520426845') format('truetype');
	}
	.iconfont {
	  font-family: "iconfont" !important;
	  /* font-size: 16px; */
	  font-style: normal;
	  -webkit-font-smoothing: antialiased;
	  -moz-osx-font-smoothing: grayscale;
	}
	
	/* page */
	.container{padding: 30rpx; }
	
	/* 顶部 */
	.top{ height: 100rpx; display: flex; align-items: center; justify-content: space-between; }
	.tag{ font-size: 46rpx; font-weight: bold; color: #333;}
	.tag text{ font-size: 28rpx; color: #bbb; margin-left: 20rpx; }
	.iconList{ font-size: 60rpx; color: #bbb;}
	.iconfont{ margin-left: 20rpx; }
	
	.subject{ font-size: 28rpx; line-height: 44rpx; margin: 20rpx 0; }
	.ansList{ display: flex; flex-direction: column; }
	.ansOpt{ width: 100%; line-height: 80rpx; text-align: center; background-color: #f6f6f6; border-radius: 15rpx;
	 margin-bottom: 20rpx; color: #666;}
	.ansOpt.active{ background-color: #999; color: #FFF;}
	.ansOpt.correct{ background-color: #3ad6b1 !important; color: #FFF;}
	.ansOpt.wrong{ background-color: #ff7a7e !important; color: #FFF; }
	
	.btnList{height: 80rpx; display: flex; justify-content: space-between; padding-top: 20rpx;}
	.btnItem{  }
	.btnItem.disabled{ color: #bbb;}
	
	.analysis{padding: 20rpx; width: 100%; line-height: 50rpx; background-color: #3ad6b1; border-radius: 15rpx; box-sizing: border-box; color: #FFF;}
	.analysis ol{ padding-inline-start: 1em; }
	
	/* 弹出菜单 */
	.maskMenu{padding: 30rpx; box-sizing: border-box;}
	.maskBtnList{ display: flex; }
	.btnClose{ width: calc(26% + 16rpx); line-height: 60rpx; background-color: #333; color: #FFF; border-radius: 15rpx; text-align: center; margin-right: 2%; }
	.qTitle{ margin: 30rpx 0 20rpx; }
	.qList{ display: flex; flex-wrap: wrap; }
	.qItem{ font-size: 24rpx; width: 12%; line-height: 60rpx; color: #aaa; border-radius: 15rpx; border: 4rpx solid #aaa;text-align: center;
	  margin: 0 2% 20rpx 0;}
	.qItem.correct{border: 4rpx solid #3ad6b1; color: #3ad6b1; }
	.qItem.wrong{border: 4rpx solid #ff7a7e; color: #ff7a7e;}

</style>
