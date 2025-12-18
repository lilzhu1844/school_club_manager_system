<template>
  <view class="page">
    <!-- 顶部个人信息卡片 -->
    <view class="profile-card">
      <!-- 头像+用户名区域 -->
      <view class="profile-header">
        <view class="avatar-wrapper">
          <image :src="avatarUrl || defaultAvatar" class="avatar-img" mode="aspectFill" />
        </view>
        <text class="username">{{ profile.name || profile.account || '未登录' }}</text>
      </view>

      <!-- 统计数据栏 -->
      <view class="stats-bar">
        <view class="stat-item">
          <text class="stat-num">{{ profile.club_count||0 }}</text>
          <text class="stat-label">已加入社团</text>
        </view>
        <view class="stat-divider"></view>
        <view class="stat-item">
          <text class="stat-num">{{ profile.activity_count||0 }}</text>
          <text class="stat-label">参与活动</text>
        </view>
      </view>
    </view>

    <!-- 基本信息面板 -->
    <view class="info-panel">
      <view class="panel-title">基本信息</view>
      <view class="info-list">
        <view class="info-item">
          <text class="item-label">姓名</text>
          <text class="item-value">{{ profile.name || '未设置' }}</text>
        </view>
        <view class="info-item">
          <text class="item-label">邮箱</text>
          <text class="item-value">{{ profile.email || '未设置' }}</text>
        </view>
        <view class="info-item">
          <text class="item-label">手机号</text>
          <text class="item-value">{{ profile.phone || '未设置' }}</text>
        </view>
        <view class="info-item">
          <text class="item-label">性别</text>
          <text class="item-value">{{ genderText(profile.gender) }}</text>
        </view>
        <view class="info-item">
          <text class="item-label">学院</text>
          <text class="item-value">{{ profile.college || '未设置' }}</text>
        </view>
      </view>
    </view>

    <!-- 操作按钮区 -->
    <view class="action-buttons">
      <button class="btn edit-btn" @tap="editProfile">修改个人信息</button>
      <button class="btn logout-btn" @tap="logout">退出登录</button>
    </view>
  </view>
</template>

<script>
import { request, clearToken } from '../../utils/request.js'
import { go, relaunchTo } from '../../utils/router.js'
export default {
  data() { return { profile: {}, navLock: false, avatarUrl: '', defaultAvatar: 'https://static-aliyun.oss-cn-hangzhou.aliyuncs.com/placeholder/avatar.png' } },
  onShow() { this.fetch(); this.loadAvatar() },
  methods: {
    async fetch() {
      try { this.profile = await request({ url: '/student/me', method: 'GET' }) || {} } catch(e) { this.profile = {} }
    },
    loadAvatar() {
      const local = uni.getStorageSync('AVATAR') || ''
      const fromProfile = this.profile && this.profile.avatar ? this.profile.avatar : ''
      const relOrAbs = fromProfile || local
      if (!relOrAbs) { this.avatarUrl = '' ; return }
      this.avatarUrl = this.fullAvatar(relOrAbs)
    },
    fullAvatar(relOrAbs) {
      if (!relOrAbs) return ''
      if (/^https?:\/\//.test(relOrAbs)) return relOrAbs
      const apiBase = uni.getStorageSync('BASE_URL') || 'http://localhost:9000/api/v1'
      const origin = apiBase.replace(/\/api\/v1$/, '')
      return origin + relOrAbs
    },
    genderText(g) { if(!g) return '未设置'; if (g==='male' || g==='M' || g==='男') return '男'; if (g==='female' || g==='F' || g==='女') return '女'; return '保密' },
    editProfile() { if (this.navLock) return; this.navLock = true; go('mineEdit') ; setTimeout(()=>{ this.navLock=false }, 320) },
    logout() {
      uni.showModal({
        title: '提示',
        content: '确认退出登录？',
        confirmText: '退出',
        confirmColor: '#e34a4a',
        success: (res) => {
          if (res.confirm) {
            clearToken()
            uni.showToast({ title: '已退出' })
            relaunchTo('login')
          }
        }
      })
    }
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

/* 页面容器：简约浅灰背景 */
.page {
  min-height: 100vh;
  background-color: #f5f7fa;
  padding: 30rpx 24rpx;
  display: flex;
  flex-direction: column;
  gap: 24rpx;
}

/* 个人信息卡片：简约白色卡片 */
.profile-card {
  background-color: #ffffff;
  border-radius: 12rpx;
  padding: 40rpx 32rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* 头像区域 */
.profile-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 32rpx;
}
.avatar-wrapper {
  width: 140rpx;
  height: 140rpx;
  border-radius: 50%;
  overflow: hidden;
  border: 2rpx solid #e5e7eb;
  margin-bottom: 16rpx;
}
.avatar-img {
  width: 100%;
  height: 100%;
}
.username {
  font-size: 32rpx;
  font-weight: 600;
  color: #333333;
}

/* 统计数据栏 */
.stats-bar {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 16rpx 0;
  background-color: #f9fafb;
  border-radius: 8rpx;
}
.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
}
.stat-num {
  font-size: 36rpx;
  font-weight: 600;
  color: #4096ff;
  margin-bottom: 4rpx;
}
.stat-label {
  font-size: 24rpx;
  color: #666666;
}
.stat-divider {
  width: 1rpx;
  height: 40rpx;
  background-color: #e5e7eb;
  margin: 0 20rpx;
}

/* 基本信息面板 */
.info-panel {
  background-color: #ffffff;
  border-radius: 12rpx;
  padding: 32rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08);
}
.panel-title {
  font-size: 28rpx;
  font-weight: 600;
  color: #333333;
  margin-bottom: 24rpx;
  padding-bottom: 12rpx;
  border-bottom: 1rpx solid #e5e7eb;
}
.info-list {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.info-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 1rpx solid #f0f2f5;
}
.info-item:last-child {
  border-bottom: none;
}
.item-label {
  font-size: 28rpx;
  color: #666666;
}
.item-value {
  font-size: 28rpx;
  color: #333333;
  text-align: right;
}

/* 操作按钮区 */
.action-buttons {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
  margin-top: 8rpx;
}
.btn {
  height: 96rpx;
  border-radius: 8rpx;
  font-size: 30rpx;
  font-weight: 600;
  transition: all 0.2s ease;
  border: none;
}
/* 修改信息按钮：主按钮（简约蓝） */
.edit-btn {
  background-color: #4096ff;
  color: #ffffff;
}
.edit-btn:active {
  background-color: #3388ee;
  transform: scale(0.98);
}
/* 退出登录按钮：次要按钮（浅灰+红色文字） */
.logout-btn {
  background-color: #f9fafb;
  color: #f53f3f;
  border: 1rpx solid #e5e7eb;
}
.logout-btn:active {
  background-color: #f0f2f5;
  transform: scale(0.98);
}
</style>