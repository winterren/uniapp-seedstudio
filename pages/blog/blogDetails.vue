<template>
	<view class="main">
		<!-- 页面加载动画 -->
		<view v-if="!ready">
			<zero-loading type="love"></zero-loading>
		</view>
		<!-- 内容详情 -->
		<view class="detail" v-if="ready">
			<view class="title">{{details.title}}</view>
			<view class="info">
				<view class="name">{{details.name}}</view>
				<view class="time">{{details.time}}</view>
			</view>
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
				details: {},
				content: '',
				name: ''
			}
		},
		methods: {
			// 获取博客详情
			getBlogDetail(id) {
				uni.request({
					url: this.$url + '/api/blogs/' + id + '?populate=*',
					method: 'GET',
					success:(res) => {
						this.details = {
							title: res.data.data.attributes.title,
							content: res.data.data.attributes.content,
							name: res.data.data.attributes.users_permissions_user.data.attributes.nickname,
							time: res.data.data.attributes.created_date
						}
						console.log(this.details)
						this.content = marked(this.replaceContent(this.details.content))
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
			// 获取博客详情
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
	.info {
		display: flex;
		justify-content: center;
		font-size: 28rpx;
		color: #838383;
		padding: 20rpx 0;
	}
	.name {
		padding-right: 60rpx;
	}
	
</style>
