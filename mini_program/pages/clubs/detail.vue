<template>
  <view class="page">

    <view class="strip">
      <text class="strip-title">{{ club.name || '未命名社团' }}</text>
    </view>
    <view class="banner">
      <image :src="club.logo" class="cover" mode="aspectFill" />
      <view class="overlay">
        <text class="btitle">{{ club.name || '未命名社团' }}</text>
        <view class="chips">
          <text class="chip">{{ club.category || '未知类别' }}</text>
          <text class="chip">{{ club.college || '未知学院' }}</text>
        </view>
      </view>
    </view>
    <view class="stats">
      <view class="stat">
        <text class="num">{{ counts.members }}</text>
        <text class="lab">成员</text>
      </view>
      <view class="stat">
        <text class="num">{{ counts.activities }}</text>
        <text class="lab">活动</text>
      </view>
      <view class="stat">
        <text class="num">{{ counts.history }}</text>
        <text class="lab">历史成员</text>
      </view>
    </view>
    <view class="card">
      <view class="ctitle">社团简介</view>
      <view class="ccontent">{{ club.intro || '暂无简介' }}</view>
    </view>
    <view class="card">
      <view class="ctitle">基本信息</view>
      <view class="row"><text class="key">社长</text><text class="val">{{ leaderName }}</text></view>
      <view class="row"><text class="key">联系电话</text><text class="val">{{ leaderPhoneText }}</text></view>
      <view class="row"><text class="key">创建时间</text><text class="val">{{ createdAtText }}</text></view>
    </view>
    <view class="bottom">
      <button class="join" :disabled="statusLoading || joinDisabled" @tap="apply">{{ statusLoading ? '加载中...' : joinButtonText }}</button>
    </view>
  </view>
  </template>
  
  <script>
  import { request, getToken } from '../../utils/request.js'
  export default {
    data() { return { id: 0, club: {}, leaders: [], activities: [], membershipStatus: '', statusLoading: false } },
    computed: {
      counts() {
        const members = this.club.member_count || this.club.members_count || 0
        const acts = Array.isArray(this.activities) ? this.activities.length : (this.club.activity_count || 0)
        const history = this.club.history_member_count || 0
        return { members, activities: acts, history }
      },
      joinDisabled() {
        return this.mergedStatus === 'pending' || this.mergedStatus === 'approved'
      },
      joinButtonText() {
        if (this.mergedStatus === 'pending') return '申请中'
        if (this.mergedStatus === 'approved') return '已加入'
        return '加入社团'
      },
      mergedStatus() {
        return this.membershipStatus || ''
      },
      leaderName() {
        const l1 = (this.leaders || []).find(x => (x.role || '').toLowerCase().includes('leader'))
        if (l1 && l1.user) return l1.user.name || ''
        if (this.leaders && this.leaders[0] && this.leaders[0].user) return this.leaders[0].user.name || ''
        return this.club.leader_name || '未知'
      },
      leaderPhoneText() {
        const l1 = (this.leaders || []).find(x => (x.role || '').toLowerCase().includes('leader'))
        if (l1 && l1.user) return l1.user.phone || '未知'
        if (this.leaders && this.leaders[0] && this.leaders[0].user) return this.leaders[0].user.phone || '未知'
        return this.club.leader_phone || '未知'
      },
      createdAtText() {
        const v = this.club.created_at || this.club.create_time || this.club.createdAt || ''
        if (!v) return '未知'
        try {
          const d = new Date(v)
          const y = d.getFullYear()
          const m = d.getMonth() + 1
          const day = d.getDate()
          return `${y}年${m}月${day}日`
        } catch(e) { return '未知' }
      }
    },
    onLoad(opts) { this.id = Number(opts.id||0); this.fetch(); this.fetchMembershipStatus() },
    methods: {
      goBack() { uni.navigateBack({ delta: 1 }) },
      async fetch() {
        if (!this.id) return
        const data = await request({ url: `/public/clubs/${this.id}`, method: 'GET' })
        this.club = {
          id: data.id,
          name: data.name,
          logo: data.logo,
          intro: data.intro,
          category: data.category,
          college: data.college || '',
          leader_name: data.leader_name || data.leaderName || '',
          leader_phone: data.leader_phone || data.leaderPhone || '',
          created_at: data.created_at || data.createdAt || '',
          member_count: data.member_count || data.members_count || 0,
          history_member_count: data.history_member_count || 0
        }
        this.leaders = data.leaders || []
        this.activities = data.activities || []
      },
      async fetchMembershipStatus() {
        if (!this.id) return
        const token = getToken()
        if (!token) { this.membershipStatus = ''; this.statusLoading = false; await this.$nextTick(); return }
        this.statusLoading = true
        try {
          const data = await request({ url: '/student/memberships/my', method: 'GET', data: { page: 1, pageSize: 200 } })
          const list = data.list || []
          const found = list.find(it => Number(it.club_id || (it.club && it.club.id)) === this.id)
          const st = found ? (found.status || '') : ''
          this.membershipStatus = st === 'approved' ? 'approved' : (st === 'pending' ? 'pending' : '')
        } catch(e) { this.membershipStatus = '' }
        this.statusLoading = false
        await this.$nextTick()
      },
      async apply() {
        if (!this.id) return
        if (this.joinDisabled) return
        try {
          await request({ url: `/student/clubs/${this.id}/apply`, method: 'POST' })
          uni.showToast({ title: '已申请' })
          this.membershipStatus = 'pending'
          await this.$nextTick()
        } catch(e) { uni.showToast({ title: e.msg || '申请失败', icon: 'none' }) }
      }
    }
  }
  </script>
  
  <style>
:root {
  --bg-primary: #0F172A;
  --bg-secondary: #1E293B;
  --bg-card: #2A3447;
  --text-primary: #F1F5F9;
  --text-secondary: #94A3B8;
  --primary-color: #6366F1;
  --primary-dark: #4F46E5;
  --primary-light: #818CF8;
  --secondary-color: #8B5CF6;
  --secondary-dark: #7C3AED;
  --secondary-light: #A78BFA;
  --accent-color: #EC4899;
  --danger-color: #EF4444;
  --border-color: #334155;
  --border-light: #475569;
  --shadow-sm: 0 2px 8px rgba(0, 0, 0, 0.3);
  --shadow-md: 0 4px 16px rgba(0, 0, 0, 0.4);
  --shadow-lg: 0 8px 32px rgba(0, 0, 0, 0.5);
  --gradient-primary: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  --gradient-secondary: linear-gradient(135deg, var(--bg-secondary), var(--bg-card));
  --border-radius-sm: 8rpx;
  --border-radius-md: 12rpx;
  --border-radius-lg: 20rpx;
  --border-radius-full: 9999rpx;
  --transition: all 0.3s ease;
}

.page { padding:20rpx 20rpx calc(env(safe-area-inset-bottom) + 120rpx) 20rpx; background:var(--bg-primary); min-height:100vh; color:var(--text-primary) }

.strip { height:64rpx; background:var(--bg-card); border-radius:var(--border-radius-md); display:flex; align-items:center; justify-content:center; color:var(--text-secondary); margin-top:16rpx; border:2rpx solid var(--border-color); box-shadow:var(--shadow-sm) }
.strip-title { font-size:28rpx }
.banner { position:relative; margin-top:16rpx; border-radius:var(--border-radius-lg); overflow:hidden; box-shadow:var(--shadow-md); border:2rpx solid var(--border-color) }
.cover { width:100%; height:300rpx; background:var(--gradient-secondary); border-radius:var(--border-radius-lg) }
.overlay { position:absolute; left:20rpx; bottom:20rpx; right:20rpx; display:flex; flex-direction:column; align-items:flex-start; padding:16rpx 20rpx; border-radius:var(--border-radius-md); background:linear-gradient(180deg, rgba(0,0,0,0.00), rgba(0,0,0,0.6)); backdrop-filter:blur(10rpx) }
.btitle { color:#fff; font-size:40rpx; font-weight:700; text-shadow:0 4rpx 8rpx rgba(0,0,0,0.5) }
.chips { margin-top:12rpx; display:flex; gap:16rpx }
.chip { padding:10rpx 20rpx; border-radius:var(--border-radius-full); background:rgba(99, 102, 241, 0.9); color:#fff; font-size:26rpx; border:2rpx solid var(--primary-color); box-shadow:0 0 20rpx rgba(99, 102, 241, 0.3) }
.stats { margin-top:20rpx; background:var(--bg-card); border-radius:var(--border-radius-lg); display:flex; overflow:hidden; box-shadow:var(--shadow-sm); border:2rpx solid var(--border-color) }
.stat { flex:1; padding:24rpx 0; display:flex; flex-direction:column; align-items:center; border-right:2rpx solid var(--border-color) }
.stat:last-child { border-right:none }
.num { font-size:44rpx; font-weight:700; color:var(--primary-light); line-height:52rpx; text-shadow:0 0 20rpx rgba(99, 102, 241, 0.5) }
.lab { font-size:28rpx; color:var(--text-secondary); margin-top:8rpx }
.card { margin-top:20rpx; background:var(--bg-card); border-radius:var(--border-radius-lg); padding:24rpx; box-shadow:var(--shadow-sm); border:2rpx solid var(--border-color) }
.ctitle { font-weight:600; margin-bottom:16rpx; color:var(--text-primary); font-size:36rpx }
.ccontent { color:var(--text-secondary); line-height:40rpx; font-size:30rpx }
.row { display:flex; justify-content:space-between; align-items:center; padding:20rpx 0; border-top:2rpx solid var(--border-color) }
.row:first-of-type { border-top:none }
.key { color:var(--text-secondary); font-size:30rpx }
.val { color:var(--text-primary); font-size:30rpx }
.bottom { position:fixed; left:0; right:0; bottom:0; padding:20rpx 20rpx calc(env(safe-area-inset-bottom) + 20rpx); background:linear-gradient(180deg, transparent, rgba(15, 23, 42, 0.95), var(--bg-primary)) }
.join { width:100%; background:var(--gradient-primary); color:#fff; border-radius:var(--border-radius-full); padding:32rpx 0; font-size:36rpx; box-shadow:0 10rpx 30rpx rgba(99, 102, 241, 0.4); transition:var(--transition) }
.join:active { transform:scale(0.98); box-shadow:0 5rpx 15rpx rgba(99, 102, 241, 0.6) }
.join[disabled] { opacity:0.6; transform:none; box-shadow:none }
</style>
