<template>
  <view class="page">
    <view class="card">
      <view class="avatar green"></view>
      <view class="brand">社团管理平台</view>
      <view class="subtitle">创建您的账号</view>
      <view class="group">
        <input class="input" v-model="form.account" placeholder="用户名" @blur="validate('account')" />
        <text v-if="errors.account" class="err">{{ errors.account }}</text>
      </view>
      <view class="group">
        <input class="input" v-model="form.name" placeholder="姓名" />
      </view>
      <view class="group">
        <input class="input" v-model="extra.college" placeholder="学院" />
      </view>
      <view class="group">
        <view class="gender">
          <button class="gbtn" :class="extra.gender==='male'?'on':''" @tap="extra.gender='male'">男</button>
          <button class="gbtn" :class="extra.gender==='female'?'on':''" @tap="extra.gender='female'">女</button>
          <button class="gbtn" :class="extra.gender==='secret'?'on':''" @tap="extra.gender='secret'">保密</button>
        </view>
      </view>
      <view class="group">
        <view class="row">
          <input class="input flex" :password="hidePwd" v-model="form.password" placeholder="密码（至少6位）" @blur="validate('password')" />
          <button class="toggle" @tap="hidePwd=!hidePwd">{{ hidePwd? '显示' : '隐藏' }}</button>
        </view>
        <text v-if="errors.password" class="err">{{ errors.password }}</text>
      </view>
      <view class="group">
        <input class="input" :password="true" v-model="extra.confirm" placeholder="确认密码" @blur="validate('confirm')" />
        <text v-if="errors.confirm" class="err">{{ errors.confirm }}</text>
      </view>
      <button class="primary" :loading="loading" :disabled="loading" @tap="submit">注册</button>
      <view class="tip">已有账号？<text class="action" @tap="goLogin">立即登录</text></view>
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

<style>
.page { min-height:100vh; display:flex; flex-direction:column; background:linear-gradient(135deg, #57c6e1 0%, #3da8e9 100%); padding:20px; justify-content:center; align-items:center; box-sizing:border-box }
.card { width:100%; max-width:400px; background:#fff; border-radius:24px; box-shadow: 0 20px 60px rgba(0, 0, 0, 0.15); padding:40px 20px; display:flex; flex-direction:column; align-items:stretch }
.avatar { width:90px; height:90px; border-radius:50%; background:linear-gradient(135deg, #57c6e1 0%, #3da8e9 100%); opacity:0.9; padding:3px; box-shadow: 0 6px 20px rgba(87, 198, 225, 0.3); margin:0 auto 20px }
.green { background:linear-gradient(135deg, #57c6e1 0%, #3da8e9 100%) }
.brand { margin:8px auto 6px auto; color:#333; font-weight:700; font-size:20px; letter-spacing:0.5px }
.subtitle { margin:0 auto 16px auto; color:#666; font-size:13px }
.group { width:100%; margin-bottom:16px }
.label { display:block; font-size:14px; color:#333; margin-bottom:6px }
.row { display:flex; align-items:center; gap:8px }
.input { width:100%; border:2px solid #e0e0e0; border-radius:16px; padding:14px 18px; font-size:15px; background:#fff; transition: all 0.3s ease; box-sizing: border-box; height:54px }
.input:focus { border-color:#57c6e1; background:#f0f9ff; box-shadow: 0 0 0 3px rgba(87, 198, 225, 0.1) }
.flex { flex:1 }
.toggle { font-size:12px; padding:8px 10px; background:#f0f9ff; color:#57c6e1; border-radius:10px }
.gender { width:100%; display:flex; gap:6px; justify-content:start }
.gbtn { padding:6px 10px; border:1px solid #e0f0f5; border-radius:14px; background:#fff; color:#57c6e1; font-size:12px }
.on { background:#e9f7fb }
.err { margin-top:6px; font-size:12px; color:#e34a4a }
.primary { width:100%; margin-top:12px; background:linear-gradient(135deg, #57c6e1 0%, #4bb8e6 100%); color:#fff; border:none; border-radius:16px; font-weight:700; padding:16px; box-shadow: 0 6px 16px rgba(87, 198, 225, 0.3); transition: all 0.3s ease }
.primary:active { transform: translateY(2px); box-shadow: 0 4px 12px rgba(87, 198, 225, 0.3) }
.tip { margin-top:16px; color:#666; text-align:center }
.action { color:#57c6e1; font-weight:600; transition: color 0.3s ease }
.action:active { color:#4bb8e6 }
</style>
