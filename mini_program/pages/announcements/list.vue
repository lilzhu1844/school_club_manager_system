<template>
  <view class="container">

    <view class="filters">
      <input class="search" placeholder="搜索公告或活动" v-model="keyword" @confirm="doSearch" />
      <view class="seg">
        <button :class="['segbtn', active==='announcement'?'on':'']" @tap="setTab('announcement')">公告</button>
        <button :class="['segbtn', active==='activity'?'on':'']" @tap="setTab('activity')">活动</button>
      </view>
    </view>
    <view v-for="item in list" :key="item.id" class="card" @tap="openItem(item)">
      <view class="card-head">
        <text class="title">{{ titleOf(item) }}</text>
        <text v-if="active==='announcement'" class="tag">公开</text>
      </view>
      <text class="meta">{{ meta1Of(item) }}</text>
      <text class="meta" v-if="meta2Of(item)">{{ meta2Of(item) }}</text>
      <text class="meta" v-if="active==='activity' && item.club">{{ item.club.name }}</text>
    </view>
    <view v-if="!loading && list.length===0" class="empty">暂无数据</view>
  </view>
</template>

<script>
import { request } from '../../utils/request.js'
export default {
  data() { return { list: [], page: 1, pageSize: 10, total: 0, keyword: '', loading: false, active: 'announcement', clubId: 0 } },
  computed: { totalPages() { return Math.max(1, Math.ceil(this.total / this.pageSize)) } },
  onLoad(opts) { if (opts && opts.clubId) this.clubId = Number(opts.clubId||0); if (opts && opts.active) this.active = opts.active; },
  onShow() { this.fetch() },
  methods: {
    async fetch() {
      this.loading = true
      const url = this.active==='announcement' ? '/public/announcements' : '/public/activities'
      const data = await request({ url, method: 'GET', data: { page: this.page, pageSize: this.pageSize, keyword: this.keyword, clubId: this.clubId || '' } })
      this.list = data.list || []
      const p = data.pagination || { total: 0 }
      this.total = p.total || 0
      this.loading = false
    },
    doSearch() { this.page = 1; this.fetch() },
    setTab(t) { this.active = t; this.page = 1; this.fetch() },
    titleOf(item) { return this.active==='announcement' ? item.title : item.subject },
    meta1Of(item) {
      if (this.active==='announcement') {
        const d = item.date || item.publishDate || (item.createdAt ? String(item.createdAt).slice(0,10) : '')
        return d
      } else {
        if (item.time) return item.time
        const st = item.startTime || ''
        const et = item.endTime || ''
        if (st && et) return `${st} - ${et}`
        return st || et || ''
      }
    },
    meta2Of(item) {
      if (this.active==='announcement') {
        return item.club ? item.club.name : ''
      } else {
        return item.place || ''
      }
    },
    openItem(item) {
      if (this.active !== 'activity') return
      const id = item.id
      const clubId = item.club_id || (item.club && item.club.id) || this.clubId || ''
      const qs = [
        ['id', id],
        ['clubId', clubId],
        ['subject', item.subject || ''],
        ['time', item.time || ''],
        ['place', item.place || ''],
        ['clubName', (item.club && item.club.name) || '']
      ].map(([k,v]) => `${k}=${encodeURIComponent(v||'')}`).join('&')
      uni.navigateTo({ url: `/pages/activities/detail?${qs}` })
    }
  }
}
</script>

<style>
.container {
  padding:0 24rpx 24rpx 24rpx;
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



.filters {
  display:flex;
  flex-direction:column;
  gap:20rpx;
  margin-top:20rpx;
}

.search {
  width:100%;
  border:2rpx solid var(--border-color);
  border-radius:24rpx;
  padding:20rpx 24rpx;
  background:var(--bg-secondary);
  color:var(--text-primary);
  font-size:28rpx;
  box-shadow: var(--shadow-sm);
  box-sizing: border-box;
}

.search::placeholder {
  color:var(--text-secondary);
}

.seg {
  display:flex;
  gap:20rpx;
}

.segbtn {
  flex:1;
  border:2rpx solid var(--border-color);
  border-radius:24rpx;
  background:var(--bg-secondary);
  color:var(--text-secondary);
  font-size:28rpx;
  padding:20rpx 0;
  box-shadow: var(--shadow-sm);
  transition: all 0.3s ease;
}

.segbtn.on {
  background:var(--gradient-primary);
  color:var(--text-primary);
  border-color:var(--primary-color);
  box-shadow: 0 0 20rpx rgba(99, 102, 241, 0.4);
}

.segbtn:active {
  transform: scale(0.95);
}

.card {
  background:var(--gradient-card);
  border:2rpx solid var(--border-color);
  border-radius:20rpx;
  padding:24rpx;
  margin:20rpx 0;
  box-shadow: var(--shadow-md);
  transition: all 0.3s ease;
}

.card:active {
  transform: scale(0.98);
  box-shadow: var(--shadow-sm);
}

.card-head {
  display:flex;
  align-items:center;
  justify-content:space-between;
  margin-bottom:12rpx;
}

.title {
  font-weight:600;
  color:var(--text-primary);
  font-size:32rpx;
  flex:1;
  overflow:hidden;
  text-overflow:ellipsis;
  white-space:nowrap;
}

.meta {
  display:block;
  color:var(--text-secondary);
  margin-top:12rpx;
  font-size:26rpx;
}

.tag {
  padding:4rpx 16rpx;
  background:var(--primary-color);
  color:var(--text-primary);
  border-radius:16rpx;
  font-size:22rpx;
  box-shadow: 0 0 12rpx rgba(99, 102, 241, 0.3);
}

.empty {
  text-align:center;
  color:var(--text-secondary);
  padding:48rpx;
  font-size:28rpx;
}
</style>
