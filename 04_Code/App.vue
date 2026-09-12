<template>
  <view></view>
</template>

<script>
export default {
  onLaunch: function() {},
  onShow: function() {},
  onHide: function() {}
}
</script>

<style>
page, .container {
  font-family: "PingFang SC","Microsoft YaHei",Arial,sans-serif;
  color: #2E3A2B;
  line-height: 1.5em;
  padding: 24rpx;
  background-color: #EAF6ED;
  box-sizing: border-box;
  margin: 0;
}
.big-title { font-size: 36rpx !important; font-weight: 600; color: #355E3B; margin-bottom: 24rpx; }
.medium-title { font-size: 28rpx; font-weight: 500; color: #356A48; margin-bottom: 16rpx; }
.section-title { font-size: 24rpx; font-weight: 500; color: #2E3A2B; margin: 16rpx 0 8rpx; }
.text-base { font-size: 24rpx; color: #2E3A2B; }
.text-sub { font-size: 20rpx; color: #5C6B5A; }
.caption { font-size: 18rpx; color: #7A8A79; }
.card { background: #F0FBF4; border-radius: 32rpx; box-shadow: 0 4rpx 16rpx rgba(106,166,105,0.2); padding: 32rpx; margin-bottom: 32rpx; }
.button-main { display: block; width: 100%; font-size: 24rpx !important; font-weight: 500; color: #FFFFFF !important; background: linear-gradient(90deg,#A3D9A5,#70B774); border: none; border-radius: 32rpx; height: 64rpx; line-height: 64rpx; margin: 24rpx 0 !important; box-shadow: 0 6rpx 20rpx rgba(112,183,116,0.3); text-align: center; }
.button-main:active { opacity: 0.9; }
.button-mini { font-size: 20rpx; height: 50rpx; line-height: 50rpx; border-radius: 28rpx; padding: 0 20rpx; background: linear-gradient(90deg,#A3D9A5,#70B774); color: #fff !important; border: none; }
input, .picker, .unit-input { font-size: 24rpx; color: #2E3A2B; border: 1rpx solid #CDE7CD; border-radius: 24rpx; padding: 0 20rpx; height: 64rpx; background: #FFFFFF; margin-bottom: 16rpx; box-sizing: border-box; }
input:focus, .picker:focus { border-color: #70B774; }
.std-row, .ratio-row, .table-header, .switch-btn-row { display: flex; flex-wrap: wrap; align-items: center; gap: 16rpx; }
.table-header { font-size: 24rpx; font-weight: 500; border-bottom: 1rpx solid #CDE7CD; padding-bottom: 8rpx; margin-bottom: 16rpx; }
.index-canvas-wrapper { position: relative; width: 100%; padding-top: 100%; background: #FFFFFF; border-radius: 16rpx; box-shadow: 0 4rpx 16rpx rgba(106,166,105,0.2); margin-bottom: 16rpx; }
.index-canvas, canvas[canvas-id="imgCanvas"] { position: absolute !important; top: 0; left: 0; width: 100% !important; height: 100% !important; }
.switch-btn { flex: 1; height: 64rpx; margin: 0 8rpx; background: #E6F8E9; color: #2E3A2B; border-radius: 32rpx; font-size: 24rpx; font-weight: 500; box-shadow: 0 4rpx 12rpx rgba(106,166,105,0.2); border: 2rpx solid transparent; text-align: center; line-height: 64rpx; }
.btn-active-exp { background: #CDE7CD; color: #356A48; border: 2rpx solid #A3D9A5; }
.btn-active-ctrl { background: #CDE7CD; color: #356A48; border: 2rpx solid #70B774; }
.image-upload { text-align: center; margin: 24rpx 0; }
.empty { text-align: center; color: #7A8A79; margin-top: 200rpx; }
.preview-img { display: block; width: 100%; margin-top: 16rpx; border-radius: 14rpx; }
.rich-formula { font-size: 28rpx; font-weight: 600; color: #356A48; }
.rich-r2 { font-size: 26rpx; font-weight: 500; color: #d21b3d; margin-top: 8rpx; }
.list { margin-top: 16rpx; }
.button-group { display: flex; flex-direction: column; align-items: center; gap: 10px; margin-bottom: 14px; }
</style>
