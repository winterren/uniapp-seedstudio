<template>
	<view class="main">
		<!-- 页面加载动画 -->
		<view v-if="!ready">
			<zero-loading type="love"></zero-loading>
		</view>
		<!-- 内容详情 -->
		<view class="detail" v-if="ready">
			<view class="title">{{information.title}}</view>
			<view class="time">{{information.time}}</view>
			<image class="img" :src="information.cover" mode=""></image>
			<u-parse :content="content"></u-parse>
		</view>
	</view>
</template>

<script>
	import {marked} from 'marked'
	import uParse from '@/components/u-parse/u-parse.vue'
	export default {
		components: {
			uParse
		},
		data() {
			return {
				ready: false,
				information: {},
				content: '',
				name: ''
			}
		},
		methods: {
			// 获取动态详情
			getBlogDetail(id) {
				uni.request({
					url: this.$url + '/api/informations/' + id + '?populate=*',
					method: 'GET',
					success:(res) => {
						this.information = {
							id: res.data.data.id,
							title: res.data.data.attributes.title,
							content: res.data.data.attributes.content,
							time: res.data.data.attributes.create_date,
							cover: this.$url + res.data.data.attributes.cover.data.attributes.url
						}
						this.content = marked(this.replaceContent(this.information.content))
						this.ready = true
					}
				})
			},
			// 给图片增加链接
			replaceContent(string) {
				let reg = /!\[(.*?)\]\((.*?)\)/gi
				string = string.replace(reg, (v,p1,p2) => {
					p2 = '![]' + '(' + this.$url + p2 + ')'
					return p2
				})
				return string
			}
		},
		onLoad(e) {
			// 获取动态详情
			this.getBlogDetail(e.id)
		}
	}
</script>

<style>
	@import url("@/components/u-parse/u-parse.css");
	page {
		/* background-color: var(--studioBg); */
		height: 100%;		
	}

	.main {
		/* background-color: var(--studioBg); */
	}
	
	.detail {
		padding: 30rpx;
	}
	.title {
		text-align: center;
		font-size: 38rpx;
		font-weight: 600;
	}
	.img {
		width: 100%;
		height: 320rpx;
	}
	.time {
		text-align: center;
		font-size: 28rpx;
		color: #838383;
		padding: 20rpx 0;
	}
	
</style>
