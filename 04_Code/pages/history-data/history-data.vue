<template>
  <view class="container">
    <view class="section-title">RGB History</view>
    <view class="ratio-row">
      <button class="button-mini" @tap="clearHistory">Clear All</button>
      <button class="button-mini" @tap="forceReload">Refresh</button>
    </view>
    <block v-if="histories.length">
      <view class="list">
        <view class="card item" v-for="(item, index) in histories" :key="index">
          <text class="caption">#{{index+1}} · {{item.dateStr}}</text>
          <view class="std-row"><text class="text-base">Exp R: {{item.expR}}</text><text class="text-base">Ctrl R: {{item.ctrlR}}</text></view>
          <view class="std-row"><text class="text-base">Exp G: {{item.expG}}</text><text class="text-base">Ctrl G: {{item.ctrlG}}</text></view>
          <view class="std-row"><text class="text-base">Exp B: {{item.expB}}</text><text class="text-base">Ctrl B: {{item.ctrlB}}</text></view>
        </view>
      </view>
    </block>
    <view v-else class="empty"><text class="caption">No history yet</text></view>
  </view>
</template>

<script>
export default {
  data() { return { histories: [] } },
  onShow() { this.loadHistories() },
  methods: {
    loadHistories() {
      const arr = uni.getStorageSync('rgbHistory') || []
      this.histories = this.formatHistories(arr)
    },
    formatHistories(arr) {
      return arr.slice().reverse().map(h => {
        const d = new Date(h.saveTime)
        return { ...h, dateStr: `${d.getFullYear()}/${d.getMonth()+1}/${d.getDate()} ${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}` }
      })
    },
    clearHistory() {
      uni.showModal({ title: '提示', content: '确定清空所有记录？', success: (res) => { if (res.confirm) { uni.removeStorageSync('rgbHistory'); this.histories = []; uni.showToast({ title: '已清空', icon: 'success' }) } } })
    },
    forceReload() { this.loadHistories() }
  }
}
</script>

<style>
page, .container { font-family: "PingFang SC","Microsoft YaHei",Arial,sans-serif; color: #2E3A2B; line-height: 1.5em; padding: 24rpx; background-color: #EAF6ED; box-sizing: border-box; margin: 0; }
.section-title { font-size: 24rpx; font-weight: 500; color: #2E3A2B; margin: 16rpx 0 8rpx; }
.text-base { font-size: 24rpx; color: #2E3A2B; }
.caption { font-size: 18rpx; color: #7A8A79; }
.card { background: #F0FBF4; border-radius: 32rpx; box-shadow: 0 4rpx 16rpx rgba(106,166,105,0.2); padding: 32rpx; margin-bottom: 32rpx; }
.button-mini { font-size: 20rpx; height: 50rpx; line-height: 50rpx; border-radius: 28rpx; padding: 0 20rpx; background: linear-gradient(90deg,#A3D9A5,#70B774); color: #fff !important; border: none; }
.std-row { display: flex; flex-wrap: wrap; align-items: center; gap: 16rpx; }
.ratio-row { display: flex; flex-wrap: wrap; align-items: center; gap: 16rpx; margin-bottom: 24rpx; }
.empty { text-align: center; color: #7A8A79; margin-top: 200rpx; }
.item { background: #fff; margin-bottom: 24rpx; padding: 24rpx; border-radius: 12rpx; box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.05); }
.list { margin-top: 16rpx; }
</style>
