<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello'
			}
		},
		onLoad() {

		},
		methods: {

		}
	}
</script>

<style>
	/* 全局样式重置与基础定义 */
	* {
		box-sizing: border-box;
		margin: 0;
		padding: 0;
	}
	
	/* 主色调：深色科技风 */
	:root {
		--primary-color: #6366F1;
		--primary-dark: #4F46E5;
		--primary-light: #818CF8;
		--secondary-color: #EC4899;
		--accent-color: #10B981;
		--bg-primary: #0F172A;
		--bg-secondary: #1E293B;
		--bg-card: #334155;
		--text-primary: #F1F5F9;
		--text-secondary: #94A3B8;
		--text-muted: #64748B;
		--border-color: #475569;
		--success-color: #10B981;
		--error-color: #EF4444;
		--warning-color: #F59E0B;
	}
	
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		min-height: 100vh;
		background: linear-gradient(135deg, var(--bg-primary) 0%, var(--bg-secondary) 100%);
		padding: 40rpx;
	}

	.logo {
		height: 320rpx;
		width: 320rpx;
		margin-bottom: 100rpx;
		border-radius: 20rpx;
		background: var(--bg-card);
		box-shadow: 0 0 40rpx rgba(99, 102, 241, 0.3);
		padding: 60rpx;
		border: 2rpx solid var(--primary-color);
		transition: all 0.3s ease;
	}
	
	.logo:hover {
		transform: scale(1.05);
		box-shadow: 0 0 60rpx rgba(99, 102, 241, 0.5);
	}

	.text-area {
		display: flex;
		justify-content: center;
		background: var(--bg-card);
		padding: 50rpx 100rpx;
		border-radius: 20rpx;
		box-shadow: 0 20rpx 60rpx rgba(0, 0, 0, 0.3);
		border: 1rpx solid var(--border-color);
	}

	.title {
		font-size: 56rpx;
		color: var(--primary-light);
		font-weight: bold;
		letter-spacing: 12rpx;
		text-shadow: 0 0 20rpx rgba(99, 102, 241, 0.5);
	}
</style>
