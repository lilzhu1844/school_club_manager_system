<template>
  <view class="page">
    <view class="card">
      <view class="avatar">
        <image src="/static/guet_logo.png" class="avatarimg" mode="aspectFill" />
      </view>
      <view class="brand">社团管理平台</view>
      <input class="ipt" placeholder="用户名" v-model="form.account" />
      <input class="ipt" placeholder="密码" password v-model="form.password" />
      <button class="primary" @tap="submit">登录</button>
      <view class="tip">还没有账号？<text class="action" @tap="goRegister">立即注册</text></view>
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
    }
    ,
    goRegister() { go('register', {}, { replace: true }) }
  }
}
</script>

<style>
/* 整体页面布局 */
.page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #57c6e1 0%, #3da8e9 100%);
  padding: 40px 20px;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
}

/* 卡片布局 */
.card {
  width: 100%;
  max-width: 400px;
  background: #fff;
  border-radius: 24px;
  padding: 40px 20px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  align-items: stretch;
  box-sizing: border-box;
}

/* 头像和品牌文字 */
.avatar {
  width: 90px;
  height: 90px;
  border-radius: 50%;
  overflow: hidden;
  background: linear-gradient(135deg, #57c6e1 0%, #3da8e9 100%);
  padding: 3px;
  margin: 0 auto 20px;
  box-shadow: 0 6px 20px rgba(87, 198, 225, 0.3);
}

.avatarimg {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: #fff;
}

.brand {
  margin: 10px auto 25px;
  color: #333;
  font-weight: 700;
  font-size: 20px;
  letter-spacing: 0.5px;
}

/* 输入框样式 */
.ipt {
  width: 100%;
  border: 2px solid #e0e0e0;
  border-radius: 16px;
  padding: 12px 16px;
  font-size: 15px;
  background: #fff;
  margin: 12px 0;
  box-sizing: border-box;
  text-align: left;
  height: 54px;
}

.ipt:focus {
  border-color: #57c6e1;
  background: #f0f9ff;
  box-shadow: 0 0 0 3px rgba(87, 198, 225, 0.1);
}

/* 按钮样式 */
.primary {
  width: 100%;
  margin-top: 20px;
  background: linear-gradient(135deg, #57c6e1 0%, #4bb8e6 100%);
  color: #fff;
  border: none;
  border-radius: 16px;
  padding: 16px;
  font-weight: 700;
  font-size: 16px;
  letter-spacing: 0.5px;
  box-shadow: 0 6px 16px rgba(87, 198, 225, 0.3);
  transition: all 0.3s ease;
}

.primary:active {
  transform: translateY(2px);
  box-shadow: 0 4px 12px rgba(87, 198, 225, 0.3);
}

/* 底部提示文字 */
.tip {
  margin-top: 20px;
  color: #666;
  font-size: 14px;
  text-align: center;
}

.action {
  color: #57c6e1;
  font-weight: 600;
  text-decoration: none;
  transition: color 0.3s ease;
}

.action:active {
  color: #4bb8e6;
}
</style>
