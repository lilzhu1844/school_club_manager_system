<template>
  <view class="page">

    <view class="list">
      <view class="empty">暂未开放</view>
    </view>
  </view>
</template>

<script>
export default {
  data() { return { id: 0 } },
  onLoad(opts) { this.id = Number(opts.id||0) }
}
</script>

<style>
.page { min-height:100vh; background:#f7f8fa }

.list { padding:12px }
.empty { text-align:center; color:#888; padding:24px }
</style>
