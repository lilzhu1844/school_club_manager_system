<template>
  <view class="page">
    <!-- 核心卡片容器（垂直居中+适配） -->
    <view class="edit-container">
      <view class="card">
        <!-- 卡片头部：标题+返回逻辑保留 -->
        <view class="card-header">
          <view class="page-title">编辑资料</view>
        </view>

        <!-- 头像区域 -->
        <view class="avatar-section" @tap="chooseAvatar">
          <view class="avatar-wrapper">
            <image :src="avatarUrl || defaultAvatar" class="avatar-img" mode="aspectFill" />
            <view class="avatar-mask">
              <text class="avatar-tip">更换头像</text>
            </view>
          </view>
        </view>

        <!-- 分割线 -->
        <view class="divider"></view>

        <!-- 表单区域 -->
        <view class="form-content">
          <!-- 姓名 -->
          <view class="form-item">
            <text class="form-label">姓名</text>
            <input class="form-input" v-model="form.name" placeholder="请输入姓名" />
          </view>

          <!-- 手机号 -->
          <view class="form-item">
            <text class="form-label">手机号</text>
            <input class="form-input" v-model="form.phone" placeholder="请输入手机号" />
          </view>

          <!-- 性别 -->
          <view class="form-item">
            <text class="form-label">性别</text>
            <view class="gender-selector">
              <button class="gender-btn" :class="{ active: form.gender==='male' }" @tap="form.gender='male'">男</button>
              <button class="gender-btn" :class="{ active: form.gender==='female' }" @tap="form.gender='female'">女</button>
              <button class="gender-btn" :class="{ active: form.gender==='secret' }" @tap="form.gender='secret'">保密</button>
            </view>
          </view>

          <!-- 学院 -->
          <view class="form-item">
            <text class="form-label">学院</text>
            <input class="form-input" v-model="form.college" placeholder="请输入学院" />
          </view>

          <!-- 学号 -->
          <view class="form-item">
            <text class="form-label">学号</text>
            <input class="form-input" v-model="form.student_no" placeholder="请输入学号" />
          </view>
        </view>

        <!-- 操作按钮区 -->
        <view class="action-btns">
          <button class="cancel-btn" @tap="goBack">取消</button>
          <button class="save-btn" @tap="submit">保存</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import { request, getToken } from '../../utils/request.js'
export default {
  data() { 
    return { 
      form: { name: '', phone: '', gender: 'secret', college: '', student_no: '' },
      avatar: '',
      avatarUrl: '',
      defaultAvatar: 'https://static-aliyun.oss-cn-hangzhou.aliyuncs.com/placeholder/avatar.png'
    } 
  },
  async onShow() {
    try {
      const p = await request({ url: '/student/me', method: 'GET' })
      this.form = { 
        name: p.name || '', 
        phone: p.phone || '', 
        gender: (p.gender==='male'||p.gender==='female'||p.gender==='secret') ? p.gender : 'secret', 
        college: p.college || '', 
        student_no: p.student_no || '' 
      }
      const localAvatar = uni.getStorageSync('AVATAR') || ''
      this.avatar = p.avatar || localAvatar || ''
      this.avatarUrl = this.fullAvatar(this.avatar)
    } catch(e) {}
  },
  methods: {
    async submit() {
      try {
        const payload = Object.assign({}, this.form, this.avatar ? { avatar: this.avatar } : {})
        await request({ url: '/student/me', method: 'PUT', data: payload })
        uni.showToast({ title: '已保存' })
        setTimeout(()=>{ uni.navigateBack({ delta: 1 }) }, 300)
      } catch(e) { uni.showToast({ title: e.msg || '保存失败', icon: 'none' }) }
    },
    goBack() { uni.navigateBack({ delta: 1 }) },
    async chooseAvatar() {
      try {
        const r = await new Promise((resolve, reject) => {
          uni.chooseImage({ count: 1, sizeType: ['compressed'], success: resolve, fail: reject })
        })
        const filePath = (r.tempFilePaths && r.tempFilePaths[0]) || (r.tempFiles && r.tempFiles[0]?.path) || ''
        if (!filePath) return
        await this.uploadAvatar(filePath)
      } catch(e) {}
    },
    fullAvatar(relOrAbs) {
      if (!relOrAbs) return ''
      if (/^https?:\/\//.test(relOrAbs)) return relOrAbs
      const apiBase = uni.getStorageSync('BASE_URL') || 'http://localhost:9000/api/v1'
      const origin = apiBase.replace(/\/api\/v1$/, '')
      return origin + relOrAbs
    },
    async uploadAvatar(filePath) {
      const apiBase = uni.getStorageSync('BASE_URL') || 'http://localhost:9000/api/v1'
      const url = apiBase.replace(/\/$/, '') + '/upload/image'
      const token = getToken()
      try {
        const res = await new Promise((resolve, reject) => {
          uni.uploadFile({
            url,
            filePath,
            name: 'file',
            header: token ? { 'Authorization': 'Bearer ' + token } : {},
            success: resolve,
            fail: reject
          })
        })
        let body = {}
        try { body = JSON.parse(res.data || '{}') } catch(e) {}
        if (body && (body.status === true) && body.data && body.data.url) {
          this.avatar = body.data.url
          this.avatarUrl = this.fullAvatar(this.avatar)
          uni.setStorageSync('AVATAR', this.avatar)
          uni.showToast({ title: '头像已更新' })
        } else {
          const msg = (body && body.msg) ? body.msg : '上传失败'
          uni.showToast({ title: msg, icon: 'none' })
        }
      } catch(e) {
        uni.showToast({ title: '上传失败', icon: 'none' })
      }
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

/* 页面容器：垂直居中+适配 */
.page {
  min-height: 100vh;
  background-color: #f5f7fa; /* 替换var(--page-bg) */
  padding: 30rpx 24rpx;
  display: flex;
  justify-content: center;
  align-items: flex-start; /* 顶部对齐，避免内容过长溢出 */
  padding-top: 60rpx;
}

/* 编辑容器：宽度限制 */
.edit-container {
  width: 100%;
  max-width: 500rpx;
}

/* 核心卡片：简约白色卡片 */
.card {
  background: #ffffff; /* 替换var(--card-bg) */
  border-radius: 12rpx;
  padding: 40rpx 32rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08); /* 替换var(--card-shadow) */
  display: flex;
  flex-direction: column;
  gap: 36rpx;
}

/* 卡片头部：标题 */
.card-header {
  text-align: center;
}
.page-title {
  font-size: 36rpx;
  font-weight: 600;
  color: #333333; /* 替换var(--text-main) */
}

/* 头像区域：简约风格 */
.avatar-section {
  display: flex;
  justify-content: center;
}
.avatar-wrapper {
  position: relative;
  width: 160rpx;
  height: 160rpx;
  border-radius: 50%;
  overflow: hidden;
  border: 2rpx solid #e5e7eb; /* 替换var(--border-normal) */
}
.avatar-img {
  width: 100%;
  height: 100%;
}
.avatar-mask {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 60rpx;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}
.avatar-tip {
  color: #ffffff;
  font-size: 24rpx;
}

/* 分割线 */
.divider {
  width: 100%;
  height: 1rpx;
  background-color: #e5e7eb; /* 替换var(--border-normal) */
  margin: 8rpx 0;
}

/* 表单区域 */
.form-content {
  display: flex;
  flex-direction: column;
  gap: 28rpx;
}

/* 表单项：移动端上下布局（更友好） */
.form-item {
  display: flex;
  flex-direction: column;
  gap: 12rpx;
}
.form-label {
  font-size: 28rpx;
  color: #333333; /* 替换var(--text-main) */
  font-weight: 500;
}
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

/* 操作按钮区：横向排列，主次分明 */
.action-btns {
  display: flex;
  gap: 20rpx;
  margin-top: 16rpx;
}
.cancel-btn, .save-btn {
  flex: 1;
  height: 96rpx;
  border-radius: 8rpx;
  font-size: 30rpx;
  font-weight: 600;
  transition: all 0.2s ease;
}
/* 取消按钮：次要按钮 */
.cancel-btn {
  background-color: #f9fafb; /* 替换var(--input-bg) */
  color: #666666; /* 替换var(--text-secondary) */
  border: 1rpx solid #e5e7eb; /* 替换var(--border-normal) */
}
.cancel-btn:active {
  background-color: #f0f2f5;
  transform: scale(0.98);
}
/* 保存按钮：主要按钮 */
.save-btn {
  background-color: #4096ff; /* 替换var(--primary) */
  color: #ffffff;
  border: none;
}
.save-btn:active {
  background-color: #3388ee; /* 替换var(--primary-hover) */
  transform: scale(0.98);
}
</style>