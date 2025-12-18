<template>
  <view class="page">
    <!-- 背景装饰元素 -->
    <view class="bg-decor dot-1"></view>
    <view class="bg-decor dot-2"></view>
    <view class="bg-decor line-1"></view>

    <!-- 登录卡片 -->
    <view class="login-container">
      <view class="card">
        <!-- 卡片头部 -->
        <view class="card-header">
          <view class="avatar-wrapper">
            <image src="/static/guet_logo.png" class="avatar-img" mode="aspectFill" />
          </view>
          <view class="brand-wrapper">
            <view class="brand">快乐社团平台</view>
            <view class="brand-desc">打造专属你的社团体验</view>
          </view>
        </view>

        <!-- 装饰分割线 -->
        <view class="decor-line"></view>

        <!-- 表单区域 -->
        <view class="form-content">
          <input class="input-item" placeholder="用户名" v-model="form.account" />
          <input class="input-item" placeholder="密码" password v-model="form.password" />
        </view>

        <!-- 操作区域 -->
        <view class="action-area">
          <button class="login-btn" @tap="submit">登录</button>
          <view class="register-tip">
            还没有账号？<text class="register-link" @tap="goRegister">立即注册</text>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import { request, setToken } from '../../utils/request.js'
import { switchTo, go } from '../../utils/router.js'
export default {
  data() { return { form: { account: '', password: '' } } },
  methods: {
    async submit() {
      try {
        const data = await request({ url: '/public/login', method: 'POST', data: this.form })
        if (data && data.token) {
          setToken(data.token)
          uni.showToast({ title: '登录成功' })
          switchTo('clubsList')
        } else {
          uni.showToast({ title: '登录失败', icon: 'none' })
        }
      } catch(e) { uni.showToast({ title: e.msg || '登录失败', icon: 'none' }) }
    },
    goRegister() { go('register', {}, { replace: true }) }
  }
}
</script>

<style scoped>
/* 全局样式重置与基础定义（保留主题色） */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

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

/* 页面容器 - 重构为flex垂直水平居中 */
.page {
  min-height: 100vh;
  background: linear-gradient(45deg, var(--bg-primary) 0%, var(--bg-secondary) 100%);
  /* 新增背景纹理 */
  background-image: 
    repeating-linear-gradient(0deg, rgba(99, 102, 241, 0.05) 0px, rgba(99, 102, 241, 0.05) 1px, transparent 1px, transparent 20px),
    repeating-linear-gradient(90deg, rgba(99, 102, 241, 0.05) 0px, rgba(99, 102, 241, 0.05) 1px, transparent 1px, transparent 20px);
  padding: 40rpx 30rpx;
  position: relative;
  overflow: hidden;
  /* 核心：设置flex让子元素垂直水平居中 */
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 背景装饰元素 */
.bg-decor {
  position: absolute;
  border-radius: 50%;
  background: rgba(99, 102, 241, 0.1);
  z-index: 0;
}
.dot-1 {
  width: 300rpx;
  height: 300rpx;
  top: 10%;
  right: -100rpx;
  filter: blur(80rpx);
}
.dot-2 {
  width: 250rpx;
  height: 250rpx;
  bottom: 15%;
  left: -80rpx;
  filter: blur(70rpx);
}
.line-1 {
  width: 600rpx;
  height: 2rpx;
  background: linear-gradient(90deg, transparent, var(--primary-color), transparent);
  top: 50%;
  left: -100rpx;
  transform: rotate(-15deg);
  border-radius: 0;
  filter: blur(2rpx);
}

/* 登录容器 - 仅做宽度限制，依托page的flex实现居中 */
.login-container {
  width: 100%;
  max-width: 550rpx;
  position: relative;
  z-index: 1;
  /* 移除原有的top-padding，避免影响居中 */
}

/* 卡片样式 - 完全重构 */
.card {
  background: var(--bg-card);
  border-radius: 16rpx;
  padding: 40rpx 30rpx;
  box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.15);
  border: 1rpx solid rgba(71, 85, 105, 0.5);
  display: flex;
  flex-direction: column;
  gap: 30rpx;
  /* 确保卡片自身宽度占满容器 */
  width: 100%;
}

/* 卡片头部 - 头像+品牌横向布局 */
.card-header {
  display: flex;
  align-items: center;
  gap: 20rpx;
  padding-bottom: 10rpx;
}

/* 头像样式重构 */
.avatar-wrapper {
  width: 100rpx;
  height: 100rpx;
  border-radius: 16rpx;
  overflow: hidden;
  background: linear-gradient(45deg, var(--primary-color), var(--primary-dark));
  display: flex;
  align-items: center;
  justify-content: center;
}
.avatar-img {
  width: 80%;
  height: 80%;
  opacity: 0.9;
}

/* 品牌文字样式重构 */
.brand-wrapper {
  flex: 1;
}
.brand {
  color: var(--text-primary);
  font-weight: 600;
  font-size: 32rpx;
  line-height: 1.2;
}
.brand-desc {
  color: var(--text-secondary);
  font-size: 20rpx;
  margin-top: 4rpx;
}

/* 装饰分割线 */
.decor-line {
  height: 1rpx;
  background: linear-gradient(90deg, transparent, var(--border-color), transparent);
  margin: 10rpx 0;
}

/* 表单区域 */
.form-content {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}

/* 输入框样式重构 */
.input-item {
  width: 100%;
  border: 1rpx solid var(--border-color);
  border-radius: 8rpx;
  padding: 32rpx 20rpx;
  font-size: 28rpx;
  background: rgba(220, 241, 253, 0.9);
  color: var(--text-primary);
  transition: all 0.2s ease;
}
.input-item:focus {
  border-color: var(--primary-color);
  background: var(--bg-secondary);
  outline: none;
  box-shadow: 0 0 0 4rpx rgba(99, 102, 241, 0.1);
}

/* 操作区域 */
.action-area {
  display: flex;
  flex-direction: column;
  gap: 24rpx;
  margin-top: 10rpx;
}

/* 登录按钮样式重构 */
.login-btn {
  width: 100%;
  background: linear-gradient(90deg, var(--primary-color), var(--primary-dark));
  color: var(--text-primary);
  border: none;
  border-radius: 8rpx;
  padding: 0rpx 0;
  font-weight: 600;
  font-size: 30rpx;
  transition: all 0.2s ease;
}
.login-btn:active {
  transform: scale(0.98);
  opacity: 0.9;
}

/* 注册提示样式重构 */
.register-tip {
  text-align: center;
  color: var(--text-secondary);
  font-size: 26rpx;
}
.register-link {
  color: var(--primary-light);
  font-weight: 500;
  padding: 4rpx 8rpx;
  border-radius: 4rpx;
  transition: all 0.2s ease;
}
.register-link:active {
  background: rgba(129, 140, 248, 0.2);
}
</style>