<template>
  <view class="container">
    <view class="big-title">Concentration Calculation</view>
    <view class="card">
      <view class="section-title">Latest Ratio</view>
      <text class="text-base">Exp{{channel}}/Ctrl{{channel}} = <text class="rich-formula">{{detectVal}}</text></text>
    </view>
    <view class="card">
      <view class="section-title">Calculate Concentration</view>
      <view class="ratio-row">
        <input class="text-base" placeholder="Ratio" :value="detectVal" @input="onDetectValInput" />
        <input class="unit-input" placeholder="Unit" :value="concUnit" @input="onUnitInput" />
        <button class="button-mini" @tap="calcConc">Calculate</button>
      </view>
      <view class="caption">Result: <text class="rich-formula">{{detectConc}}</text> <text>{{concUnit}}</text></view>
    </view>
    <view class="card">
      <view class="section-title">Fitted Curve Equation</view>
      <view class="rich-formula">{{curveStr}}</view>
      <view class="rich-r2">{{r2Str}}</view>
    </view>
    <view class="button-group">
      <button class="button-main" @tap="goBackHome">Back to Home</button>
      <button class="button-main" @tap="goToPrevPage">Previous</button>
    </view>
  </view>
</template>

<script>
export default {
  data() { return { channel: '', detectVal: '', detectConc: '', concUnit: '', k: 0, b: 0, curveStr: '', r2Str: '', stdUnit: '' } },
  onLoad() {
    const chIdx = uni.getStorageSync('channelIndex') || 0, ch = ['R','G','B'][chIdx] || 'R'
    const k = uni.getStorageSync('curveK') || 0, b = uni.getStorageSync('curveB') || 0
    const unit = uni.getStorageSync('curveUnit') || '', ratios = uni.getStorageSync('ratios') || {}
    this.channel = ch; this.k = k; this.b = b; this.curveStr = uni.getStorageSync('curveFormula') || ''
    this.r2Str = uni.getStorageSync('curveR2') || ''; this.concUnit = unit; this.detectVal = ratios[ch] || ''; this.stdUnit = unit
  },
  methods: {
    onDetectValInput(e) { this.detectVal = e.detail.value },
    onUnitInput(e) { this.concUnit = e.detail.value },
    calcConc() {
      if (!this.detectVal) { uni.showToast({ title: 'Please input ratio', icon: 'none' }); return }
      if (!this.k) { uni.showToast({ title: 'Please fit curve first', icon: 'none' }); return }
      this.detectConc = ((Number(this.detectVal) - this.b) / this.k).toFixed(4)
    },
    goBackHome() { uni.redirectTo({ url: '/pages/index/index' }) },
    goToPrevPage() { uni.navigateBack() }
  }
}
</script>

<style>
page, .container { font-family: "PingFang SC","Microsoft YaHei",Arial,sans-serif; color: #2E3A2B; line-height: 1.5em; padding: 24rpx; background-color: #EAF6ED; box-sizing: border-box; margin: 0; }
.big-title { font-size: 36rpx !important; font-weight: 600; color: #355E3B; margin-bottom: 24rpx; }
.section-title { font-size: 24rpx; font-weight: 500; color: #2E3A2B; margin: 16rpx 0 8rpx; }
.text-base { font-size: 24rpx; color: #2E3A2B; }
.caption { font-size: 18rpx; color: #7A8A79; }
.card { background: #F0FBF4; border-radius: 32rpx; box-shadow: 0 4rpx 16rpx rgba(106,166,105,0.2); padding: 32rpx; margin-bottom: 32rpx; }
.button-main { display: block; width: 100%; font-size: 24rpx !important; font-weight: 500; color: #FFFFFF !important; background: linear-gradient(90deg,#A3D9A5,#70B774); border: none; border-radius: 32rpx; height: 64rpx; line-height: 64rpx; margin: 24rpx 0 !important; box-shadow: 0 6rpx 20rpx rgba(112,183,116,0.3); text-align: center; }
.button-main:active { opacity: 0.9; }
.button-mini { font-size: 20rpx; height: 50rpx; line-height: 50rpx; border-radius: 28rpx; padding: 0 20rpx; background: linear-gradient(90deg,#A3D9A5,#70B774); color: #fff !important; border: none; }
input, .unit-input { font-size: 24rpx; color: #2E3A2B; border: 1rpx solid #CDE7CD; border-radius: 24rpx; padding: 0 20rpx; height: 64rpx; background: #FFFFFF; margin-bottom: 16rpx; box-sizing: border-box; }
input:focus { border-color: #70B774; }
.ratio-row { display: flex; flex-wrap: wrap; align-items: center; gap: 16rpx; }
.rich-formula { font-size: 28rpx; font-weight: 600; color: #356A48; }
.rich-r2 { font-size: 26rpx; font-weight: 500; color: #d21b3d; margin-top: 8rpx; }
.button-group { display: flex; flex-direction: column; align-items: center; gap: 10px; margin-bottom: 14px; }
</style>
