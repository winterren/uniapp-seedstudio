<template>
	<view>
		<view v-if="!ready">
			<zero-loading type="love"></zero-loading>
		</view>
		<view class="container" v-if="ready">
			<uni-popup ref="popup" type="dialog">
				<uni-popup-dialog mode="base" :duration="2000" :before-close="true" @close="dialogClose()" @confirm="dialogConfirm(dialogType)">{{dialog[dialogType]}}</uni-popup-dialog>
			</uni-popup>
			
			<!-- 抽屉 -->
			<uni-drawer ref="showMenu" mode="right" :mask-click="true" :width="300">
				<scroll-view class="maskMenu" style="height: 100%;" scroll-y="true" >
					<view class="maskBtnList">
						<view class="btnMaskMenu" @click="handleReset">重置</view>
						<view class="btnMaskMenu" @click="handleScore">交卷</view>
						<view class="btnMaskMenu" @click="handleQuit">退出</view>
					</view>
				
					<view class="qView">
						<view class="qTitle" v-if="singleNo.length>0">单选题</view>
						<view class="qList">
							<view class="qItem"
							v-for="(item,index) in singleNo" 
							:key="index"
							:class="[{correct:aRecordList[item.id].correct==true},{wrong:aRecordList[item.id].correct==false}]"
							@click="switchIdToIndex(item.qid)"
							>{{ item.qid-0+1 }}</view>
						</view>
						<view class="qTitle" v-if="multiNo.length>0">多选题</view>
						<view class="qList">
							<view class="qItem" 
							v-for="(item,index) in multiNo" 
							:key="index"
							:class="[{correct:aRecordList[item.id].correct==true},{wrong:aRecordList[item.id].correct==false}]"
							@click="switchIdToIndex(item.qid)"
							>{{ item.qid-0+1 }}</view>
						</view>
						<view class="qTitle" v-if="tfNo.length>0">判断题</view>
						<view class="qList">
							<view class="qItem" 
							v-for="(item,index) in tfNo" 
							:key="index"
							:class="[{correct:aRecordList[item.id].correct==true},{wrong:aRecordList[item.id].correct==false}]"
							@click="switchIdToIndex(item.qid)"
							>{{ item.qid-0+1 }}</view>
						</view>
					</view>
				</scroll-view>
			</uni-drawer>
			<!-- 页面顶部 -->
			<view class="top">
				<view class="tag"># {{parseInt(qList[currentQuestion].qid)+1}}  
					<text v-if="type==='single'">单选题</text>
					<text v-if="type==='multi'">多选题</text>
					<text v-if="type==='tf'">判断题</text>
				</view>
				<view class="iconList">
					<!-- 图标：笔记 -->
					<!-- <span class="iconfont">&#xe6e5;</span> -->
					<!-- 图标：收藏 -->
					<span class="iconfont" :class="{wrong:wrong}" @click="handleWrongQuestion">{{wrong?'&#xe6ea;':'&#xe6eb;'}}</span>
					<!-- 图标：菜单 -->
					<span class="iconfont" @click="showMenu">&#xe6ef;</span>
				</view>
			</view>
			<!-- 页面主体 -->
			<view class="subject">{{qList[currentQuestion].subject}}</view>
			<!-- （不是多选）单选/判断 -->
			<view class="ansList" v-if="type!='multi'">
				<view class="ansOpt" 
				v-for="(item,index) in qList[currentQuestion].optList" 
				@click="handleAnsClick(index)"
				
				:class="[{correct:hasAnswered&&index==qList[currentQuestion].answer}
				,{wrong:hasAnswered&&index==currentAnsOpt&&!currentAnswerIsRight}]" 
				>{{item}}</view>
				<!-- :style="randomOrder(index)" -->
			</view>
			<!-- 多选 -->
			<view class="ansList" v-if="type=='multi'">
				<view class="ansOpt" 
				v-for="(item,index) in qList[currentQuestion].optList" 
				@click="handleMultiAnsClick(index)"
				
				:class="[{active:currentMultiAnsOpt[index]===true}
				,{correct:hasAnswered&&currentAnswerIsRight&&currentMultiAnsOpt[index]===true}
				,{wrong:hasAnswered&&!currentAnswerIsRight&&currentMultiAnsOpt[index]===true}
				]"
				>{{item}}</view>
				<!-- :style="randomOrder(index)" -->
			</view>
			<view class="btnList">
				<view class="btnItem" 
				:class="{disabled:firstQ}"
				@click="changeToPre"
				>上一题</view>
				<view class="btnItem" 
				v-if="type=='multi'"
				:class="{disabled:hasAnswered}"
				@click="handleMultiSubmit"
				>确定</view>
				<view class="btnItem" 
				:class="{disabled:lastQ}"
				@click="changeToPost"
				>下一题</view>
			</view>
			<view class="analysis wrong" v-if="hasAnswered&&!currentAnswerIsRight&&type!='multi'">
				正确答案为：<li v-for="(item,index) in qList[currentQuestion].optList" v-if="index==qList[currentQuestion].answer" style="list-style: none;">{{item}}</li>
			</view>
			<view class="analysis wrong" v-if="hasAnswered&&!currentAnswerIsRight&&type=='multi'">
				正确答案为：<ol><li v-for="(item,index) in qList[currentQuestion].optList" v-if="correctMulti[index]">{{item}};</li></ol>
			</view>
			<view class="analysis right" v-if="hasAnswered&&currentAnswerIsRight">
				回答正确
			</view>
			
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				wrongList:[],
				ready:false,
				qList:[],
				emptyAnswerRecord:[],
				aRecordList:[],
				currentQuestion: 0,
				currentAnsOpt: null,
				hasAnswered: false,
				type: null,
				currentMultiAnsOpt: [],
				randomAnswer: false,
				randomSeed: 3,
				page:1,
				total:null,
				totalTemp:null,
				singleNo:[],
				multiNo:[],
				tfNo:[],
				dialog:{
					score: "您有未提交的题目，是否确认交卷？",
					reset: "您本次作答会被清空，是否要重置吗？",
					quit: "确认要退出答题吗？"
				},
				dialogType:null, //score
				isWrongQuestion: true,
			}
		},
		computed:{
			answerIsRight(){
				return function(opt){
					return opt == this.qList[this.currentQuestion].answer
				}
			},
			wrong(){
				return this.aRecordList[this.currentQuestion].wrong
			},
			currentQuestionOpt(i){
				return function(opt){
					return this.qList[this.currentQuestion].optList
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
			
			// this.loadAllPage(
			// this.$url+'api/exam-questions?populate=exam_category&filters[exam_category][id]='+option.cate+'&sort[0]=id&pagination[pageSize]=100&pagination[page]='
			// 	,function(){console.log("ok")}
			// )
			this.total = option.to - option.from + 1
			this.totalTemp = this.total
			console.log(this.total)
			this.qList = []
			this.emptyAnswerRecord = []
			this.randomSeed = Math.random()*2 + 2
			this.getData(option)

		},
		methods: {
			getData(option){
				let cate = option.cate
				let from = option.from - 1
				let to = option.to - 0 + 1
				let rq = option.rq
				let ra = option.ra
				
				let wrongArray = '';
				this.wrongList = uni.getStorageSync("exam_wrong_list")
				for(let i=0;i<this.wrongList.length;i++){
					wrongArray += '&filters[id][$in]['+i+']='+this.wrongList[i].qid
				}
				// this.wrongList.forEach((item)=>{
				// 	this.qid
				// })
				
				// filters[id][$in][0]=1&filters[id][$in][1]=2
				
				//请求题库
				uni.request({
					url: this.$url+'/api/exam-questions?populate=*'
					+wrongArray
					// +'&filters[exam_category][id]='+option.cate
					// +'&filters[id][$gt]='+from
					// +'&filters[id][$lt]='+to
					// +'&sort[0]=id'
					+'&pagination[pageSize]=100'
					+'&pagination[page]='+this.page
					,
					success: (res) => {
						if(res.data.data.length != 0){
							console.log(res.data.meta.pagination.total)
							this.total = res.data.meta.pagination.total
							if(this.total>100){
								this.totalTemp = 100
								this.total -= 100
							}else{
								this.totalTemp = this.total - 1
							}
							for(let i=0; i<this.totalTemp; i++){
								let temp = res.data.data[i].attributes
								// console.log(temp)
								let optList =  [temp.option_a, temp.option_b, temp.option_c, temp.option_d, temp.option_e, temp.option_f, temp.option_g, temp.option_h].filter(item=>{return item!=null})
								// 实例变量数组：保存题库信息
								this.qList.push({
									id: i+(this.page-1)*100,
									// nid: i+(this.page-1)*100,
									qid: res.data.data[i].id-1,
									subject: temp.subject,
									optList:optList,
									answer: temp.answer,
									type: temp.type
								})
								// 答题记录数组：保存正确答案
								this.emptyAnswerRecord.push({
									id: i+(this.page-1)*100,
									// nid: i+(this.page-1)*100,
									qid: res.data.data[i].id-1,
									answer: temp.answer,
									myAnswer: null,
									correct: null,
									type:temp.type,
									wrong: false,
								})
							}
							this.page++
							this.getData(option)
						}else{
							// rq：题目乱序
							if(rq=='true'){
								let stack1 = [];
								let stack2 = [];

								while (this.qList.length) {
								    //Math.random()：返回 [0,1) 之间的一个随机数
								    let index = parseInt(Math.random() * this.qList.length);  // 利用数组长度生成随机索引值
								    stack1.push(this.qList[index]);  // 将随机索引对应的数组元素添加到新的数组中
								    stack2.push(this.emptyAnswerRecord[index]);  // 将随机索引对应的数组元素添加到新的数组中
								    this.qList.splice(index, 1);  // 删除原数组中随机生成的元素
								    this.emptyAnswerRecord.splice(index, 1);  // 删除原数组中随机生成的元素
									
								}
								this.qList=stack1
								this.emptyAnswerRecord=stack2
								// console.log(this.qList[1])
								// console.log(this.emptyAnswerRecord[1])
							}			
									
							for(let i=0; i<this.qList.length; i++){
								this.qList[i].id = i
								this.emptyAnswerRecord[i].id = i
								// console.log(this.qList[i].id+'|'+this.qList[i].qid)
							}
							
							// 排序：单选->多选->判断
							let stackSingleQ = [];
							let stackSingleA = [];
							let stackMultiQ = [];
							let stackMultiA = [];
							let stackTfQ = [];
							let stackTfA = [];
							let singleId = 0
							let multiId = 0
							let tfId = 0
							while(this.qList.length){
								let qTemp = this.qList.shift()
								let aTemp = this.emptyAnswerRecord.shift()
								
								
								if(qTemp.type==="single"){
									stackSingleQ.push(qTemp)
									stackSingleA.push(aTemp)
									this.singleNo.push({id:singleId,qid:qTemp.qid})
									singleId++
								}
								if(qTemp.type==="multi"){
									stackMultiQ.push(qTemp)
									stackMultiA.push(aTemp)
									this.multiNo.push({id:multiId,qid:qTemp.qid})
									multiId++
								}
								if(qTemp.type==="tf"){
									stackTfQ.push(qTemp)
									stackTfA.push(aTemp)
									this.tfNo.push({id:tfId,qid:qTemp.qid})
									tfId++
								}
							}
							console.log(singleId,multiId,tfId)
							
							this.qList=[...stackSingleQ,...stackMultiQ,...stackTfQ]
							this.emptyAnswerRecord=[...stackSingleA,...stackMultiA,...stackTfA]
							
							for(let i=0;i<this.multiNo.length;i++){
								this.multiNo[i].id = singleId - 0 + i 
							}
							
							for(let i=0; i<this.qList.length; i++){
								this.qList[i].id = i
								this.emptyAnswerRecord[i].id = i
								// console.log(this.qList[i].id+'|'+this.qList[i].qid)
							}
							
							// for(let i=0;i<this.qList.length;i++){
							// 	let qTemp = this.qList[i]
							// 	let aTemp = this.aList[i]
							// 	if(qTemp.type==="single"){
							// 		this.singleNo.push({id:qTemp.id,qid:qTemp.qid})
							// 	}
							// 	if(qTemp.type==="multi"){
							// 		this.multiNo.push({id:qTemp.id,qid:qTemp.qid})
							// 	}
							// 	if(qTemp.type==="tf"){
							// 		this.tfNo.push({id:qTemp.id,qid:qTemp.qid})
							// 	}
							// }

							
							
							
							//选项乱序
							this.randomAnswer = (ra=='true')
							
							//请求答题记录：存入本地存储
							this.aRecordList = [].concat(this.emptyAnswerRecord)
							this.syncRecordList()
							
							// 找错题
							let wrongList = uni.getStorageSync("exam_wrong_list")
							if(wrongList){
								wrongList.forEach((itemWrong)=>{
									this.aRecordList.filter((itemOriginal)=>{
										return (itemOriginal.qid == itemWrong.qid) && (itemOriginal.wrong = true)
										// if(this.aRecordList.indexOf(item.qid)!=-1){return;}
										// console.log(this.aRecordList[this.aRecordList.indexOf(item.qid)])
										// this.aRecordList[this.aRecordList.indexOf(item.qid)].wrong = true
										// console.log(this.aRecordList[this.aRecordList.indexOf(item.qid)])
									})
									
								
								})
							}
							
							
							this.ready = true
							// initialize
							this.type = this.qList[0].type
						}
					},
					fail: () => {
						this.$toast("网络连接有误，请稍候重试")
					}
				})
			},
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
			switchIdToIndex(i){
				let index = this.qList.indexOf( this.qList.filter(item=>{return item.qid==i})[0] )
				this.changeToQuestion(index)
				console.log(index)
			},
			// 切换到第i题
			changeToQuestion(i){
				this.currentQuestion = i
				this.type = this.qList[i].type
				
				// console.log(this.qList[this.currentQuestion])
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
				uni.setStorageSync("exam_practice",this.aRecordList)
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
			},
			//交卷
			handleScore(){
				// 判断是否一题未答
				if(this.aRecordList.filter((item)=>{return item.correct!=null}).length==0){
					uni.showToast({
						icon:"none",
						title:"请先答题再交卷，不要着急哦"
					})
					return;
				}
				// 判断是否有未答的题
				if(this.aRecordList.filter((item)=>{return item.correct==null}).length>0){
					console.log(1)
					this.$refs.showMenu.close();
					this.dialogOpen('score')
					return;
				}
				uni.navigateTo({
					url: './score'
				})
			},
			handleQuit(){
				this.$refs.showMenu.close();
				this.dialogOpen('quit')
				
			},
			handleReset(){
				this.$refs.showMenu.close();
				this.dialogOpen('reset')
			},
			// 读取所有分页的数据并存储在数组newList中
			loadAllPage(url,suc) {
				uni.request({
					url: url+this.page,
					method: 'GET',
					success:(res)=> {
						this.newList = [...this.newList,...res.data.data]
						console.log(`第${this.page}个请求执行完毕：`,this.newList)
						console.log('该数组的长度为',res.data.data.length)
						if(res.data.data.length != 0){
							this.page++
							this.loadAllPage()
						}
					},
					fail: (err) => {
						suc()
					}
				})
			},
			dialogOpen(dialogType) {
				this.dialogType = dialogType
				this.$refs.popup.open()
			},
			/**
			 * 点击取消按钮触发
			 * @param {Object} done
			 */
			dialogClose() {
				// TODO 做一些其他的事情，before-close 为true的情况下，手动执行 close 才会关闭对话框
				// ...
				this.$refs.popup.close()
			},
			/**
			 * 点击确认按钮触发
			 * @param {Object} done
			 * @param {Object} value
			 */
			dialogConfirm(dialogType) {
				console.log(dialogType)
				
				this.$refs.popup.close()
				if(dialogType=='score'){
					uni.navigateTo({
						url: './score'
					})
				}
				if(dialogType=='reset'){
					// this.aRecordList.forEach((item)=>{
					// 	item.myAnswer= null,
					// 	item.correct= null
					// })
					// this.changeToQuestion(0)
					uni.navigateBack({
						delta: 0
					})
				}
				if(dialogType=='quit'){
					uni.navigateTo({
						url: '../../index/index'
					})
				}
			},
			// 切换错题状态
			handleWrongQuestion(){
				let wrongList = []
				if( uni.getStorageSync("exam_wrong_list").length>0 ){
					wrongList = uni.getStorageSync("exam_wrong_list")
				}
				
				let wrongItem = wrongList.filter( (item)=>{
					return item.qid === this.qList[this.currentQuestion].qid
				} )[0]
				let recordItem = this.aRecordList.filter( (item)=>{
					let result
					if(wrongItem===undefined){
						result = -1
					}else{
						result = wrongItem.qid
					}
					return item.qid === result
				})[0]
				// 如果错题，返回错题index；如果没错，返回-1
				let wrongNumber = wrongList.indexOf(wrongItem)
				let recordNumber = this.aRecordList.indexOf(recordItem)
				console.log(wrongItem,recordItem,wrongNumber,recordNumber)
				if(wrongNumber == -1){
					this.aRecordList[this.currentQuestion].wrong = true
					wrongList.push(this.aRecordList[this.currentQuestion])
				}else{
					this.aRecordList[this.currentQuestion].wrong = false
					wrongList.splice(wrongNumber,1)
				}
				uni.setStorageSync("exam_wrong_list",wrongList)
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
	.container{padding: 30rpx ; }
	
	/* 顶部 */
	.top{ height: 100rpx; display: flex; align-items: center; justify-content: space-between; }
	.tag{ font-size: 46rpx; font-weight: bold; color: #333;}
	.tag text{ font-size: 28rpx; color: #bbb; margin-left: 20rpx; }
	.iconList{ font-size: 60rpx; color: #bbb;}
	.iconfont{ margin-left: 10rpx; padding:12rpx ; }
	.iconfont.wrong{ color: var(--studioRed)}
	
	.subject{ font-size: 28rpx; line-height: 44rpx; margin: 20rpx 0; }
	.ansList{ display: flex; flex-direction: column; }
	.ansOpt{ width: 100%; line-height: 45rpx; text-align: center; background-color: #f6f6f6; border-radius: 15rpx;
	 margin-bottom: 20rpx; color: #666; padding: 30rpx; box-sizing: border-box;}
	.ansOpt.active{ background-color: #999; color: #FFF;}
	.ansOpt.correct{ background-color: #3ad6b1 !important; color: #FFF;}
	.ansOpt.wrong{ background-color: #ff7a7e !important; color: #FFF; }
	
	.btnList{height: 80rpx; display: flex; justify-content: space-between; padding: 20rpx 0;}
	.btnItem{ height: 80rpx; line-height: 80rpx; width: 160rpx; text-align: center; }
	.btnItem.disabled{ color: #bbb;}
	
	.analysis{padding: 20rpx; width: 100%; line-height: 50rpx; border-radius: 15rpx; box-sizing: border-box; color: #FFF;}
	.analysis ol{ padding-inline-start: 1em; }
	.analysis.wrong{background-color: var(--studioRed);}
	.analysis.right{background-color: var(--studioGreen);}
	
	/* 弹出菜单 */
	.maskMenu{padding: 30rpx ; box-sizing: border-box;}
	.maskBtnList{ display: flex; }
	.btnMaskMenu{ width: calc(26% + 16rpx); line-height: 80rpx; background-color: #333; color: #FFF; border-radius: 15rpx; text-align: center; margin-right: 2%; margin-top:25rpx; }
	.qTitle{ margin: 30rpx 0 20rpx; }
	.qList{ display: flex; flex-wrap: wrap; }
	.qItem{ font-size: 24rpx; width: 12%; line-height: 60rpx; color: #aaa; border-radius: 15rpx; border: 4rpx solid #aaa;text-align: center;
	  margin: 0 2% 20rpx 0;}
	.qItem.correct{border: 4rpx solid #3ad6b1; color: #3ad6b1; }
	.qItem.wrong{border: 4rpx solid #ff7a7e; color: #ff7a7e;}

</style>
