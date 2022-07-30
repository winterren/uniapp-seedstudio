<template>
	<view>
		<view v-if="!hasWrongList" style="padding-top: 100rpx;">
			<u-empty text="当前还没有错题"></u-empty>
		</view>
		<view v-if="hasWrongList" style="padding-top: 20rpx;">
			<view class="btnList">
				<!-- <view class="btn green" @click="practiceWrong">开始练习</view> -->
				<view class="btn green" @click="deleteSelect">删除所选</view>
				<view class="btn red" @click="deleteAll">清空所有</view>
			</view>
			<checkbox-group @change="checkboxChange">
				<view v-for="(item,index) in wrongListDetail" class="question">
					<checkbox  :value="item.qid+''"  class="checkbox" />
					<view>
						<view class="subject">
							<text>{{item.id}}.</text>
							<text>{{item.subject}}</text>
						</view>
						<view class="option">
							<view class="optionItem" v-for="(item2,index2) in item.optList">
								{{item2}}
							</view>
						</view>
						<view class="answer">
							正确答案：<text v-for="(item3,index3) in item.optList" v-if="(item.type!='multi'&&index3==item.answer)||( item.type=='multi'&&item.answer.split('').indexOf(index3+'')>-1 )">{{item3}}；</text>
						</view>
					</view>
					
				</view>
			</checkbox-group>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				hasWrongList:true,
				wrongList:[],
				wrongListDetail: [],
				selectList:[]
			}
		},
		onLoad(){
			// 读取错题
			this.wrongList = uni.getStorageSync("exam_wrong_list")
			this.getWrongDetail()
		},
		methods: {
			// 同步错题
			syncWrongList(){
				uni.setStorageSync("exam_wrong_list", this.wrongList)
			},
			getWrongDetail(){
				this.wrongListDetail = []
				if(!this.wrongList || this.wrongList.length==0 ){
					console.log("当前没有错题")
					this.hasWrongList = false
					return;
				}
				this.wrongList.push({qid:-100}) //触发结束回调
				// 遍历错题
				// https://www.seedstudio.cn:1437/api/exam-questions?filters[id]=1
				
				this.wrongList.forEach((item)=>{
					this.getSingleWrongItem(item)
				})
				
				// for(let i = 0; i<this.wrongList.length; i++){
				// 	this.getSingleWrongItem(this.wrongList[i])
				// }
				
			},
			getSingleWrongItem(item){
				uni.request({
					url: this.$url + '/api/exam-questions?filters[id]='+(item.qid+1),
					success: (res) => {
						if(res.data.data.length==0){ //结束回调
							console.log("suc callback")
							this.wrongList.sort((item1,item2)=>{return item1.id-item2.id})
							console.log(this.wrongListDetail)	
							return;
						}
						// console.log(item.qid)
						// console.log(res.data.data[0].id)
						let temp = res.data.data[0].attributes
						let optList =  [temp.option_a, temp.option_b, temp.option_c, temp.option_d, temp.option_e, temp.option_f, temp.option_g, temp.option_h].filter(item=>{return item!=null})
						// 实例变量数组：保存题库信息
						this.wrongListDetail.push({
							id: item.qid-0+1,
							// nid: i+(this.page-1)*100,
							qid: item.qid,
							subject: temp.subject,
							optList:optList,
							answer: temp.answer,
							type: temp.type
						})
					}
				})
			},
			checkboxChange: function (e) {
				// var items = this.items,
					var values = e.detail.value;
					this.selectList = values
					console.log(values)
				// for (var i = 0, lenI = items.length; i < lenI; ++i) {
				// 	const item = items[i]
				// 	if(values.includes(item.value)){
				// 		this.$set(item,'checked',true)
				// 	}else{
				// 		this.$set(item,'checked',false)
				// 	}
				// }
			},
			practiceWrong(){},
			deleteSelect(){
				for(var i=0; i<this.selectList.length; i++){
					console.log(this.selectList[i])
					// 清除storage
					this.wrongList = this.wrongList.filter((item)=>{
						console.log(Number(item.qid),Number(this.selectList[i]),Number(item.qid)!=Number(this.selectList[i]))
						return Number(item.qid)!=Number(this.selectList[i])
					})
					console.log(this.wrongList)
					// var itemIndex = this.wrongList.findIndex((item)=>{return Number(item=>item.qid)==Number(this.selectList[i])})
					// console.log(itemIndex)
					// if(itemIndex>-1){
					// 	this.wrongList.splice(itemIndex,1)
					// 	console.log(this.wrongList)
					// }
					// console.log(this.wrongList.filter((item)=>{return item.qid == this.selectList[i]}))
					// console.log(this.wrongList.indexOf(this.wrongList.filter((item)=>{return item.qid == this.selectList[i]}))[0])
					// for(var j = 0; j < this.wrongList.length; j++){
					// 	if(this.wrongList[j]==item){
					// 		this.wrongList.splice(j,1)
					// 		console.log(this.wrongList)
					// 	}
					// }
				}
				this.getWrongDetail()
				this.syncWrongList()
				
			},
			deleteAll(){
				this.wrongList = []
				this.getWrongDetail()
				this.syncWrongList()
				this.hasWrongList=false
			}
		}
	}
</script>

<style>
page{padding: 0 30rpx;box-sizing: border-box;}

.btnList{ display: flex;justify-content: space-between;}
.btn{color: #FFF;padding: 20rpx;border-radius: 10rpx;
text-align: center; width:350rpx;margin-right: 20rpx;}
.btn.red{ background-color: var(--studioRed);}
.btn.green{ background-color: var(--studioGreen);}
.btn.yellow{ background-color: #f1a816;}

.question{ display: flex; margin-top: 10rpx;padding-top: 10rpx;border-top: 1rpx solid #f7f7f7;line-height: 50rpx;}
.question .checkbox{}
.question .subject{}
.question .answer{}
</style>
