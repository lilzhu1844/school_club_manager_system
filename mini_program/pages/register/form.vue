<template>
  <view class="page">
    <!-- 登录卡片容器（垂直居中） -->
    <view class="register-container">
      <view class="card">
        <!-- 卡片头部 -->
        <view class="card-header">
          <view class="avatar-wrapper">
            <view class="avatar-icon"></view>
          </view>
          <view class="title-group">
            <view class="page-title">注册</view>
            <view class="brand">快乐社团平台</view>
            <view class="subtitle">创建您的专属账号</view>
          </view>
        </view>

        <!-- 分割线 -->
        <view class="divider"></view>

        <!-- 表单区域 -->
        <view class="form-content">
          <!-- 用户名 -->
          <view class="form-group">
            <input class="form-input" v-model="form.account" placeholder="用户名（至少4位）" @blur="validate('account')" />
            <text v-if="errors.account" class="error-text">{{ errors.account }}</text>
          </view>

          <!-- 姓名 -->
          <view class="form-group">
            <input class="form-input" v-model="form.name" placeholder="姓名" />
          </view>

          <!-- 学院 -->
          <view class="form-group">
            <input class="form-input" v-model="extra.college" placeholder="学院" />
          </view>

          <!-- 性别选择 -->
          <view class="form-group">
            <view class="gender-selector">
              <button class="gender-btn" :class="{ active: extra.gender==='male' }" @tap="extra.gender='male'">男</button>
              <button class="gender-btn" :class="{ active: extra.gender==='female' }" @tap="extra.gender='female'">女</button>
              <button class="gender-btn" :class="{ active: extra.gender==='secret' }" @tap="extra.gender='secret'">保密</button>
            </view>
          </view>

          <!-- 密码 -->
          <view class="form-group">
            <view class="password-row">
              <input class="form-input password-input" :password="hidePwd" v-model="form.password" placeholder="密码（至少6位）" @blur="validate('password')" />
              <button class="pwd-toggle-btn" @tap="hidePwd=!hidePwd">{{ hidePwd? '显示' : '隐藏' }}</button>
            </view>
            <text v-if="errors.password" class="error-text">{{ errors.password }}</text>
          </view>

          <!-- 确认密码 -->
          <view class="form-group">
            <input class="form-input" :password="true" v-model="extra.confirm" placeholder="确认密码" @blur="validate('confirm')" />
            <text v-if="errors.confirm" class="error-text">{{ errors.confirm }}</text>
          </view>
        </view>

        <!-- 分割线 -->
        <view class="divider"></view>

        <!-- 操作区域 -->
        <view class="action-area">
          <button class="register-btn" :loading="loading" :disabled="loading" @tap="submit">注册</button>
          <view class="login-tip">
            已有账号？<text class="login-link" @tap="goLogin">立即登录</text>
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
  data() {
    return {
      form: { account: '', password: '', name: '' },
      extra: { college: '', gender: 'secret', confirm: '' },
      errors: { account: '', password: '', confirm: '' },
      hidePwd: true,
      loading: false
    }
  },
  methods: {
    validate(field) {
      if (field === 'account') {
        const v = (this.form.account || '').trim()
        this.errors.account = v.length >= 4 ? '' : '至少4位字符'
      }
      if (field === 'password') {
        const v = this.form.password || ''
        this.errors.password = v.length >= 6 ? '' : '至少6位字符'
      }
      if (field === 'confirm') {
        const v = this.extra.confirm || ''
        this.errors.confirm = v === this.form.password ? '' : '两次密码不一致'
      }
    },
    async submit() {
      this.validate('account'); this.validate('password'); this.validate('confirm')
      if (this.errors.account || this.errors.password || this.errors.confirm) return
      this.loading = true
      try {
        const data = await request({ url: '/public/register', method: 'POST', data: { account: this.form.account, password: this.form.password, name: this.form.name, gender: this.extra.gender, college: this.extra.college } })
        if (data && data.token) {
          setToken(data.token)
          uni.showToast({ title: '注册成功' })
          switchTo('clubsList')
        } else {
          uni.showToast({ title: '注册失败', icon: 'none' })
        }
      } catch(e) { uni.showToast({ title: e.msg || '注册失败', icon: 'none' }) }
      this.loading = false
    },
    goLogin() { go('login', {}, { replace: true }) }
  }
}
</script>

<style scoped>
/* 全局重置 */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* 页面容器：垂直水平居中 */
.page {
  min-height: 100vh;
  background-color: #f5f7fa; /* 替换var(--page-bg) */
  padding: 30rpx 24rpx;
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 注册卡片容器 */
.register-container {
  width: 100%;
  max-width: 500rpx;
}

/* 核心卡片：简约白色卡片 */
.card {
  background: #ffffff; /* 替换var(--card-bg) */
  border-radius: 12rpx;
  padding: 48rpx 36rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08); /* 替换var(--card-shadow) */
  display: flex;
  flex-direction: column;
  gap: 40rpx;
}

/* 卡片头部：头像+标题组合 */
.card-header {
  display: flex;
  align-items: center;
  gap: 24rpx;
}

/* 头像：简约风格 */
.avatar-wrapper {
  width: 80rpx;
  height: 80rpx;
  border-radius: 12rpx;
  background-color: #f0f4ff;
  display: flex;
  align-items: center;
  justify-content: center;
}
.avatar-icon {
  width: 48rpx;
  height: 48rpx;
  background-color: #4096ff; /* 替换var(--primary) */
  border-radius: 8rpx;
}

/* 标题组 */
.title-group {
  flex: 1;
}
.page-title {
  font-size: 36rpx;
  font-weight: 600;
  color: #333333; /* 替换var(--text-main) */
  margin-bottom: 6rpx;
}
.brand {
  font-size: 28rpx;
  color: #333333; /* 替换var(--text-main) */
  margin-bottom: 4rpx;
}
.subtitle {
  font-size: 22rpx;
  color: #666666; /* 替换var(--text-secondary) */
}

/* 表单区域 */
.form-content {
  display: flex;
  flex-direction: column;
  gap: 24rpx;
}

/* 表单项组 */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 8rpx;
}

/* 输入框之间的分割线 */
.form-group + .form-group {
  position: relative;
}

.form-group + .form-group::before {
  content: '';
  position: absolute;
  top: -12rpx;
  left: 0;
  right: 0;
  height: 1rpx;
  background-color: #93e1ef;
}

/* 输入框：简约轻量化 */
.form-input {
  height: 88rpx;
  padding: 0 24rpx;
  border: 1rpx solid #e5e7eb; /* 替换var(--border-normal) */
  border-radius: 8rpx;
  background-color: #f9fafb; /* 替换var(--input-bg) */
  color: #333333; /* 替换var(--text-main) */
  font-size: 28rpx;
  transition: all 0.2s ease;
}
.form-input::placeholder {
  color: #999999; /* 替换var(--text-placeholder) */
}
.form-input:focus {
  border-color: #4096ff; /* 替换var(--border-active) */
  background-color: #ffffff; /* 替换var(--card-bg) */
  outline: none;
  box-shadow: 0 0 0 4rpx rgba(64, 150, 255, 0.1); /* 替换var(--focus-shadow) */
}

/* 密码行 */
.password-row {
  display: flex;
  align-items: center;
  gap: 12rpx;
}
.password-input {
  flex: 1;
}

/* 密码显示/隐藏按钮 */
.pwd-toggle-btn {
  height: 88rpx;
  padding: 0 24rpx;
  background-color: #f9fafb; /* 替换var(--input-bg) */
  color: #4096ff; /* 替换var(--primary) */
  border: 1rpx solid #e5e7eb; /* 替换var(--border-normal) */
  border-radius: 8rpx;
  font-size: 26rpx;
  transition: all 0.2s ease;
}
.pwd-toggle-btn:active {
  background-color: #e8f3ff;
  border-color: #69b1ff; /* 替换var(--primary-light) */
}

/* 性别选择器 */
.gender-selector {
  display: flex;
  gap: 16rpx;
}
.gender-btn {
  flex: 1;
  height: 88rpx;
  background-color: #f9fafb; /* 替换var(--input-bg) */
  border: 1rpx solid #e5e7eb; /* 替换var(--border-normal) */
  border-radius: 8rpx;
  color: #333333; /* 替换var(--text-main) */
  font-size: 28rpx;
  transition: all 0.2s ease;
}
.gender-btn.active {
  background-color: #e8f3ff;
  border-color: #4096ff; /* 替换var(--primary) */
  color: #4096ff; /* 替换var(--primary) */
}

/* 错误提示文字 */
.error-text {
  font-size: 24rpx;
  color: #f53f3f; /* 替换var(--text-error) */
  padding-left: 4rpx;
}

/* 分割线样式 */
.divider {
  height: 1rpx;
  background-color: #03d1ff;
  margin: 30rpx 0;
}

/* 性别选择器分割线 */
.gender-selector {
  position: relative;
}
.gender-selector::before,
.gender-selector::after {
  content: '';
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 1rpx;
  height: 50rpx;
  background-color: #e5e7eb; /* 替换var(--border-normal) */
}
.gender-selector::before {
  left: 33.33%;
}
.gender-selector::after {
  left: 66.66%;
}

/* 操作区域 */
.action-area {
  display: flex;
  flex-direction: column;
  gap: 30rpx;
  margin-top: 8rpx;
}

/* 注册按钮 */
.register-btn {
  height: 96rpx;
  background-color: #4096ff; /* 替换var(--primary) */
  color: #ffffff;
  border: none;
  border-radius: 8rpx;
  font-size: 30rpx;
  font-weight: 600;
  transition: all 0.2s ease;
}
.register-btn:active {
  background-color: #3388ee; /* 替换var(--primary-hover) */
  transform: scale(0.98);
}
.register-btn[disabled] {
  background-color: #b3d1ff;
  transform: none;
}

/* 登录提示 */
.login-tip {
  text-align: center;
  font-size: 26rpx;
  color: #666666; /* 替换var(--text-secondary) */
}
.login-link {
  color: #4096ff; /* 替换var(--primary) */
  font-weight: 500;
  padding: 4rpx 8rpx;
  border-radius: 4rpx;
  transition: all 0.2s ease;
}
.login-link:active {
  background-color: #e8f3ff;
}
</style>