<template>
  <view class="container">
    <view class="seg">
      <text :class="['seg-item', status==='approved' ? 'active' : '']" @tap="switchStatus('approved')">已加入</text>
      <text :class="['seg-item', status==='pending' ? 'active' : '']" @tap="switchStatus('pending')">申请中</text>
    </view>
    <view v-for="it in items" :key="it.clubId" class="row" @tap="onCardTap(it.clubId)">
      <view class="info">
        <text class="title">{{ it.clubName }}</text>
        <text class="meta">社长：{{ it.leaderName || '未知' }}</text>
      </view>
      <text v-if="status==='approved'" class="enter" @tap="goClub(it.clubId)">点击进入</text>
    </view>
    <view v-if="!loading && items.length===0" class="empty">暂无社团</view>
  </view>
</template>

<script>
import { request } from '../../utils/request.js'
import { go } from '../../utils/router.js'
export default {
  data() { 
    return { 
      status: 'approved',
      items: [],
      leadersMap: {},
      loading: false
    } 
  },
  onShow() { this.fetch() },
  methods: {
    switchStatus(s) {
      if (this.status === s) return
      this.status = s
      this.fetch()
    },
    async fetch() {
      this.loading = true
      try {
        const data = await request({ url: '/student/memberships/my', method: 'GET', data: { page: 1, pageSize: 200, status: this.status } })
        const list = data.list || []
        const items = list.map(it => {
          const club = it.club || {}
          return {
            clubId: it.club_id || club.id,
            clubName: club.name || '',
            leaderName: this.leadersMap[it.club_id || club.id] || ''
          }
        }).filter(it => !!it.clubId)
        this.items = items
        const ids = Array.from(new Set(items.map(i => i.clubId))).filter(Boolean)
        await this.fetchLeaders(ids)
      } catch(e) {
        this.items = []
      }
      this.loading = false
    },
    onCardTap(id) {
      if (this.status !== 'approved') return
      this.goClub(id)
    },
    goClub(id) { if (!id) return; go('clubHome', { id }) },
    async fetchLeaders(ids) {
      const pending = ids.filter(id => !(id in this.leadersMap))
      if (!pending.length) return
      try {
        const results = await Promise.all(pending.map(id => request({ url: `/public/clubs/${id}`, method: 'GET' })))
        results.forEach((data, idx) => {
          const id = pending[idx]
          const leaders = data.leaders || []
          let leaderName = ''
          for (let i = 0; i < leaders.length; i++) {
            const m = leaders[i] || {}
            if ((m.role || '').toLowerCase() === 'leader') {
              leaderName = (m.user && m.user.name) || ''
              break
            }
          }
          if (!leaderName && leaders.length > 0) {
            leaderName = (leaders[0].user && leaders[0].user.name) || ''
          }
          this.leadersMap[id] = leaderName
        })
        this.items = this.items.map(it => ({ ...it, leaderName: this.leadersMap[it.clubId] || it.leaderName }))
      } catch(e) {}
    }
  }
}
</script>

<style>
.container {
  padding:24rpx;
  background:#0F172A;
  min-height:100vh;
  --primary-color: #6366F1;
  --primary-dark: #4F46E5;
  --primary-light: #818CF8;
  --secondary-color: #8B5CF6;
  --bg-primary: #0F172A;
  --bg-secondary: #1E293B;
  --bg-card: #1E293B;
  --text-primary: #F1F5F9;
  --text-secondary: #94A3B8;
  --border-color: #334155;
  --shadow-sm: 0 2rpx 8rpx rgba(0, 0, 0, 0.5);
  --shadow-md: 0 4rpx 16rpx rgba(0, 0, 0, 0.6);
  --shadow-lg: 0 8rpx 32rpx rgba(0, 0, 0, 0.7);
  --gradient-primary: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  --gradient-card: linear-gradient(135deg, rgba(30, 41, 59, 0.9), rgba(15, 23, 42, 0.9));
}

.seg {
  display:flex;
  gap:20rpx;
  align-items:center;
  margin-bottom:24rpx;
  background:var(--bg-secondary);
  border:2rpx solid var(--border-color);
  border-radius:16rpx;
  padding:8rpx;
  box-shadow: var(--shadow-sm);
}

.seg-item {
  flex:1;
  text-align:center;
  padding:20rpx 0;
  border-radius:12rpx;
  color:var(--text-secondary);
  font-size:28rpx;
  transition: all 0.3s ease;
}

.seg-item.active {
  background:var(--gradient-primary);
  color:var(--text-primary);
  font-weight:600;
  box-shadow: 0 0 20rpx rgba(99, 102, 241, 0.4);
}

.seg-item:active {
  transform: scale(0.95);
}

.row {
  padding:32rpx;
  background:var(--gradient-card);
  border:2rpx solid var(--border-color);
  border-radius:20rpx;
  box-shadow: var(--shadow-md);
  display:flex;
  align-items:center;
  justify-content:space-between;
  margin-bottom:24rpx;
  transition: all 0.3s ease;
}

.row:last-child {
  margin-bottom:0;
}

.row:active {
  transform: scale(0.98);
  box-shadow: var(--shadow-sm);
}

.info {
  display:flex;
  flex-direction:column;
  flex:1;
}

.title {
  font-weight:600;
  color:var(--text-primary);
  font-size:32rpx;
  margin-bottom:12rpx;
}

.meta {
  display:block;
  color:var(--text-secondary);
  margin-top:8rpx;
  font-size:26rpx;
}

.enter {
  color:var(--primary-light);
  font-weight:600;
  font-size:28rpx;
  padding:16rpx 32rpx;
  background:var(--bg-primary);
  border:2rpx solid var(--primary-color);
  border-radius:16rpx;
  box-shadow: 0 0 12rpx rgba(99, 102, 241, 0.3);
  transition: all 0.3s ease;
}

.enter:active {
  transform: scale(0.95);
}

.empty {
  text-align:center;
  color:var(--text-secondary);
  padding:80rpx;
  font-size:28rpx;
}
</style>
