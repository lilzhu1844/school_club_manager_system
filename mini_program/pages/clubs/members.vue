<template>
  <view class="page">
    <!-- 列表容器 -->
    <view class="list-container">
      <!-- 加载状态提示 -->
      <view v-if="loading" class="loading-tip">
        <text>正在加载中...</text>
      </view>

      <!-- 数据列表 -->
      <view v-else class="data-list">
        <!-- 列表项卡片 -->
        <view v-for="u in list" :key="u.id" class="list-card">
          <view class="card-header">
            <!-- 头像占位（视觉优化，不影响功能） -->
            <view class="avatar-placeholder">
              <text class="avatar-text">{{ (u.name || '')[0] || '#' }}</text>
            </view>
            <view class="user-info">
              <text class="user-name">{{ u.name }}</text>
              <text class="user-meta">{{ u.student_no || '未填写学号' }} · {{ u.phone || '未填写手机号' }}</text>
            </view>
          </view>
        </view>

        <!-- 空状态 -->
        <view v-if="list.length===0" class="empty-state">
          <view class="empty-icon">📄</view>
          <text class="empty-text">暂无数据或无权限查看</text>
          <text class="empty-desc">请确认您的访问权限或稍后重试</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import { request } from '../../utils/request.js'
export default {
  data() { return { id: 0, list: [], loading: false } },
  onLoad(opts) { this.id = Number(opts.id||0); this.fetch() },
  methods: {
    async fetch() {
      if (!this.id) return
      this.loading = true
      try {
        const data = await request({ url: `/leader/clubs/${this.id}/users`, method: 'GET', data: { page: 1, pageSize: 100 } })
        this.list = data.list || []
      } catch(e) { this.list = [] }
      this.loading = false
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
  padding: 24rpx;
}

/* 列表容器 */
.list-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 16rpx;
}

/* 加载提示 */
.loading-tip {
  padding: 60rpx 0;
  text-align: center;
  color: #666666;
  font-size: 28rpx;
}

/* 数据列表 */
.data-list {
  display: flex;
  flex-direction: column;
  gap: 16rpx;
}

/* 列表项卡片：现代卡片样式 */
.list-card {
  background-color: #ffffff;
  border-radius: 12rpx;
  padding: 24rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.06);
  transition: all 0.2s ease;
}

.list-card:active {
  transform: scale(0.98);
  box-shadow: 0 1rpx 4rpx rgba(0, 0, 0, 0.08);
}

/* 卡片头部：头像+信息布局 */
.card-header {
  display: flex;
  align-items: center;
  gap: 20rpx;
}

/* 头像占位（视觉优化） */
.avatar-placeholder {
  width: 80rpx;
  height: 80rpx;
  border-radius: 50%;
  background-color: #e8f3ff;
  display: flex;
  align-items: center;
  justify-content: center;
}

.avatar-text {
  font-size: 32rpx;
  font-weight: 600;
  color: #4096ff;
}

/* 用户信息 */
.user-info {
  display: flex;
  flex-direction: column;
  gap: 8rpx;
}

.user-name {
  font-size: 30rpx;
  font-weight: 600;
  color: #333333;
}

.user-meta {
  font-size: 24rpx;
  color: #666666;
  line-height: 1.4;
}

/* 空状态 */
.empty-state {
  padding: 80rpx 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16rpx;
}

.empty-icon {
  font-size: 80rpx;
  margin-bottom: 8rpx;
}

.empty-text {
  font-size: 28rpx;
  color: #333333;
  font-weight: 500;
}

.empty-desc {
  font-size: 24rpx;
  color: #999999;
}
</style>