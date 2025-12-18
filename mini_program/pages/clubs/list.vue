<template>
  <view class="container">
    <!-- 筛选区域 -->
    <view class="filters">
      <!-- 搜索框 -->
      <view class="search-wrapper">
        <input class="search-input" placeholder="搜索社团" v-model="keyword" @confirm="doSearch" />
      </view>
      <!-- 分类标签栏 -->
      <scroll-view class="category-scroll" scroll-x show-scrollbar="false">
        <view class="category-list">
          <button 
            v-for="c in chipList" 
            :key="c.key" 
            class="category-tag" 
            :class="{ active: c.key === selectedChipKey }" 
            @tap="setCat(c)"
          >
            {{ c.name }}
          </button>
        </view>
      </scroll-view>
    </view>

    <!-- 社团列表 -->
    <view class="club-grid">
      <view 
        v-for="item in list" 
        :key="item.id" 
        class="club-card" 
        @tap="goDetail(item.id)"
      >
        <!-- 社团logo -->
        <view class="card-header">
          <image :src="item.logo" mode="aspectFill" class="club-logo" />
        </view>
        <!-- 社团信息 -->
        <view class="card-body">
          <text class="club-name">{{ item.name }}</text>
          <text class="club-intro">{{ item.intro }}</text>
          <!-- 底部信息+操作按钮 -->
          <view class="card-footer">
            <text class="activity-count">{{ (item.activities||[]).length }}个活动</text>
            <button 
              class="action-btn" 
              :class="[
                btnState(item.id)==='join' ? 'join-btn' : 
                btnState(item.id)==='exit' ? 'exit-btn' : 'pending-btn'
              ]" 
              @tap.stop="btnState(item.id)==='join' ? join(item.id) : btnState(item.id)==='exit' ? exitClub(item.id) : ''"
            >
              {{ btnState(item.id)==='join' ? '加入' : btnState(item.id)==='exit' ? '退出' : '审批中' }}
            </button>
          </view>
        </view>
      </view>
    </view>

    <!-- 加载状态 -->
    <view v-if="loading" class="loading-tip">
      <text>正在加载中...</text>
    </view>

    <!-- 空状态 -->
    <view v-if="!loading && list.length===0" class="empty-state">
      <view class="empty-icon">🏫</view>
      <text class="empty-text">暂无社团</text>
      <text class="empty-desc">换个分类或关键词试试吧</text>
    </view>
  </view>
</template>

<script>
import { request } from '../../utils/request.js'
import { go } from '../../utils/router.js'
export default {
  data() {
    return { list: [], page: 1, pageSize: 10, total: 0, keyword: '', chipKeyword: '', selectedChipKey: 'cat-0', categories: [], categoryId: 0, loading: false, hasMore: true, chipList: [ {id:0,key:'cat-0',name:'全部', mock:false} ], memberships: {} }
  },
  computed: {
    totalPages() { return Math.max(1, Math.ceil(this.total / this.pageSize)) },
    categoryLabel() { return this.categoryId ? (this.categories.find(c=>c.id===this.categoryId)?.name||'全部类别') : '全部类别' }
  },
  async onShow() { if (!this.categories.length) await this.fetchCategories(); await this.fetchMemberships(); this.resetAndFetch() },
  onReachBottom() { this.loadMore() },
  methods: {
    async fetchCategories() {
      try {
        const data = await request({ url: '/public/categories', method: 'GET', data: { page: 1, pageSize: 100 } })
        this.categories = data.list || []
        const more = (this.categories||[]).map(c=>({id:c.id,key:'cat-'+c.id,name:c.name, mock:false}))
        this.chipList = [{id:0,key:'cat-0',name:'全部', mock:false}, ...more]
        this.appendMockCategories()
      } catch(e) {}
    },
    appendMockCategories() {
      const presets = ['文学类','音乐类','志愿服务','社会实践','电竞类','摄影类','创业类']
      // 去重后随机取3个
      const existNames = new Set(this.chipList.map(i=>i.name))
      const candidates = presets.filter(n=>!existNames.has(n))
      for (let i=0; i<Math.min(3, candidates.length); i++) {
        const idx = Math.floor(Math.random() * candidates.length)
        const name = candidates.splice(idx,1)[0]
        this.chipList.push({ id: 0, key: 'mock-'+name, name, mock: true })
      }
    },
    async fetchMemberships() {
      try {
        const data = await request({ url: '/student/memberships/my', method: 'GET', data: { page: 1, pageSize: 200 } })
        const ms = data.list || []
        const m = {}
        ms.forEach(it=>{ m[it.club_id || (it.club && it.club.id) ] = it.status })
        this.memberships = m
      } catch(e) { this.memberships = {} }
    },
    async fetch(append=false) {
      this.loading = true
      const kw = this.keyword || this.chipKeyword
      const data = await request({ url: `/public/clubs`, method: 'GET', data: { page: this.page, pageSize: this.pageSize, keyword: kw, categoryId: this.categoryId || '' } })
      const items = data.list || []
      const p = data.pagination || { total: 0 }
      this.total = p.total || 0
      this.list = append ? (this.list.concat(items)) : items
      this.hasMore = this.list.length < this.total
      this.loading = false
    },
    loadMore() { if (this.loading || !this.hasMore) return; this.page++; this.fetch(true) },
    resetAndFetch() { this.page = 1; this.hasMore = true; this.list = []; this.fetch(false) },
    goDetail(id) { go('clubsDetail', { id }) },
    doSearch() { this.chipKeyword = ''; this.resetAndFetch() },
    onCatChange(e) { const idx = Number(e.detail.value||0); const item = this.categories[idx]; this.categoryId = item ? item.id : 0; this.resetAndFetch() },
    setCat(c) {
      this.selectedChipKey = c.key
      if (c.mock) { this.chipKeyword = c.name; this.categoryId = 0 }
      else { this.categoryId = c.id || 0; this.chipKeyword = '' }
      this.resetAndFetch()
    },
    activeChipClass(c) {
      return c.key === this.selectedChipKey ? 'on' : ''
    },
    btnState(cid) {
      const st = this.memberships[cid]
      if (!st) return 'join'
      if (st === 'approved') return 'exit'
      return 'pending'
    },
    async join(cid) {
      try { await request({ url: `/student/clubs/${cid}/apply`, method: 'POST' }); uni.showToast({ title: '已申请' }); await this.fetchMemberships() } catch(e) {}
    },
    async exitClub(cid) {
      try { await request({ url: `/student/clubs/${cid}/exit`, method: 'POST' }); uni.showToast({ title: '已退出' }); await this.fetchMemberships() } catch(e) {}
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

/* 页面容器 */
.container {
  min-height: 100vh;
  background-color: #f5f7fa;
  padding: 24rpx 24rpx 40rpx 24rpx;
}

/* 筛选区域 */
.filters {
  display: flex;
  flex-direction: column;
  gap: 16rpx;
  margin-bottom: 24rpx;
}

/* 搜索框 */
.search-wrapper {
  width: 100%;
}
.search-input {
  width: 100%;
  height: 88rpx;
  padding: 0 24rpx;
  background-color: #ffffff;
  border: 1rpx solid #e5e7eb;
  border-radius: 8rpx;
  font-size: 28rpx;
  color: #333333;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.06);
}
.search-input::placeholder {
  color: #999999;
}

/* 分类标签栏 */
.category-scroll {
  white-space: nowrap;
  padding: 8rpx 0;
}
.category-list {
  display: flex;
  gap: 12rpx;
  padding: 4rpx 2rpx;
}
.category-tag {
  padding: 12rpx 24rpx;
  background-color: #ffffff;
  border: 1rpx solid #e5e7eb;
  border-radius: 40rpx;
  font-size: 28rpx;
  color: #666666;
  flex-shrink: 0;
  transition: all 0.2s ease;
}
.category-tag.active {
  background-color: #4096ff;
  color: #ffffff;
  border-color: #4096ff;
  box-shadow: 0 2rpx 8rpx rgba(64, 150, 255, 0.2);
}
.category-tag:active {
  transform: scale(0.95);
}

/* 社团列表网格 */
.club-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 20rpx;
}

/* 社团卡片 */
.club-card {
  width: calc(50% - 10rpx);
  background-color: #ffffff;
  border-radius: 12rpx;
  overflow: hidden;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08);
  transition: all 0.2s ease;
  display: flex;
  flex-direction: column;
}
.club-card:active {
  transform: scale(0.98);
  box-shadow: 0 1rpx 6rpx rgba(0, 0, 0, 0.1);
}

/* 卡片头部（logo） */
.card-header {
  width: 100%;
  height: 180rpx;
  background-color: #f9fafb;
  overflow: hidden;
}
.club-logo {
  width: 100%;
  height: 100%;
}

/* 卡片内容 */
.card-body {
  padding: 20rpx;
  display: flex;
  flex-direction: column;
  flex: 1;
  gap: 12rpx;
}
.club-name {
  font-size: 30rpx;
  font-weight: 600;
  color: #333333;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.club-intro {
  font-size: 24rpx;
  color: #666666;
  line-height: 1.4;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  flex: 1;
}

/* 卡片底部 */
.card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 8rpx;
}
.activity-count {
  font-size: 22rpx;
  color: #999999;
}
.action-btn {
  padding: 8rpx 20rpx;
  border-radius: 40rpx;
  font-size: 24rpx;
  transition: all 0.2s ease;
  border: none;
}
/* 加入按钮 */
.join-btn {
  background-color: #e8f3ff;
  color: #4096ff;
}
/* 退出按钮 */
.exit-btn {
  background-color: #fef2f2;
  color: #f53f3f;
}
/* 审批中按钮 */
.pending-btn {
  background-color: #f9fafb;
  color: #999999;
}
.action-btn:active {
  transform: scale(0.95);
}

/* 加载提示 */
.loading-tip {
  text-align: center;
  padding: 40rpx 0;
  color: #666666;
  font-size: 28rpx;
}

/* 空状态 */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80rpx 0;
  gap: 16rpx;
}
.empty-icon {
  font-size: 80rpx;
  margin-bottom: 8rpx;
}
.empty-text {
  font-size: 30rpx;
  color: #333333;
  font-weight: 500;
}
.empty-desc {
  font-size: 24rpx;
  color: #999999;
}
</style>