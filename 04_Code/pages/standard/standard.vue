<template>
  <view class="container">
    <view class="big-title">Standard Curve Setup</view>

    <view class="card">
      <view class="section-title">Standard Points</view>
      <view class="table-header">
        <view class="cell">Conc</view><view class="cell">Exp R</view><view class="cell">Ctrl R</view>
        <view class="cell">Exp G</view><view class="cell">Ctrl G</view><view class="cell">Exp B</view><view class="cell">Ctrl B</view>
      </view>
      <view v-for="(item, idx) in stdPoints" :key="idx" class="std-row">
        <input class="text-base" placeholder="Conc" :value="item.conc" @input="onStdInput(idx, 'conc', $event)" />
        <input class="text-base" placeholder="Exp R" :value="item.expR" @input="onStdInput(idx, 'expR', $event)" />
        <input class="text-base" placeholder="Ctrl R" :value="item.ctrlR" @input="onStdInput(idx, 'ctrlR', $event)" />
        <input class="text-base" placeholder="Exp G" :value="item.expG" @input="onStdInput(idx, 'expG', $event)" />
        <input class="text-base" placeholder="Ctrl G" :value="item.ctrlG" @input="onStdInput(idx, 'ctrlG', $event)" />
        <input class="text-base" placeholder="Exp B" :value="item.expB" @input="onStdInput(idx, 'expB', $event)" />
        <input class="text-base" placeholder="Ctrl B" :value="item.ctrlB" @input="onStdInput(idx, 'ctrlB', $event)" />
      </view>
      <button class="button-main" @tap="addStdPoint">+ Add Standard Point</button>
      <button class="button-main" @tap="clearStdPoints">Clear All Points</button>
    </view>

    <view class="card">
      <view class="section-title">Fit Curve</view>
      <picker class="picker" mode="selector" :range="channelList" :value="channelIndex" @change="onChannelChange">
        <view class="text-base">Select Channel: <text class="rich-formula">{{channelList[channelIndex]}}</text></view>
      </picker>
      <view class="caption">Concentration Unit:</view>
      <input class="unit-input" :value="unit" @input="onUnitInput" placeholder="Unit" />
      <button class="button-main" @tap="fitCurve">Fit Curve</button>
    </view>

    <view class="card">
      <view class="section-title">Standard Curve Visualization</view>
      <view class="std-canvas-container">
        <canvas canvas-id="myCanvas" :width="canvasW" :height="canvasH" :style="'width:'+canvasW+'px;height:'+canvasH+'px;display:block'" class="std-canvas" />
      </view>
    </view>

    <view class="card">
      <view class="section-title">Standard Curve &amp; R²</view>
      <view class="rich-formula">{{formula}}</view>
      <view class="rich-r2">{{r2Str}}</view>
    </view>

    <view class="button-group">
      <button class="button-main" @tap="goBackHome">Back to Home</button>
      <button class="button-main" @tap="goToPrevPage">Previous</button>
      <button class="button-main" @tap="goToResultPage">Next</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      stdPoints: [{ conc: '', expR: '', expG: '', expB: '', ctrlR: '', ctrlG: '', ctrlB: '' }, { conc: '', expR: '', expG: '', expB: '', ctrlR: '', ctrlG: '', ctrlB: '' }],
      channelList: ['R','G','B'], channelIndex: 0, unit: 'mg/L', rgbHistory: [], canvasW: 300, canvasH: 180, formula: '', r2Str: '', curveK: 0, curveB: 0
    }
  },
  onLoad() { this.loadData(); this.$nextTick(() => this.setCanvasSize()) },
  onReady() { this.setCanvasSize() },
  methods: {
    loadData() {
      const sp = uni.getStorageSync('stdPoints'); if (sp) this.stdPoints = sp
      const ci = uni.getStorageSync('channelIndex'); if (ci !== '' && ci !== undefined) this.channelIndex = Number(ci)
      const u = uni.getStorageSync('curveUnit'); if (u) this.unit = u
      this.formula = uni.getStorageSync('curveFormula') || ''; this.r2Str = uni.getStorageSync('curveR2') || ''
      this.rgbHistory = uni.getStorageSync('rgbHistory') || []
      const expRGB = uni.getStorageSync('expRGB') || {}, ctrlRGB = uni.getStorageSync('ctrlRGB') || {}
      if (expRGB.R && ctrlRGB.R) {
        let arr = uni.getStorageSync('stdPoints') || []
        const np = { conc: '', expR: expRGB.R, ctrlR: ctrlRGB.R, expG: expRGB.G, ctrlG: ctrlRGB.G, expB: expRGB.B, ctrlB: ctrlRGB.B }
        if (!arr.some(p => JSON.stringify(p) === JSON.stringify(np))) { arr.unshift(np); uni.setStorageSync('stdPoints', arr) }
        this.stdPoints = arr
      }
    },
    setCanvasSize() {
      uni.createSelectorQuery().in(this).select('.std-canvas-container').boundingClientRect((rect) => {
        if (rect && rect.width) { this.canvasW = parseInt(rect.width); this.canvasH = Math.round(rect.width * 0.6) }
      }).exec()
    },
    onStdInput(idx, field, e) { const arr = JSON.parse(JSON.stringify(this.stdPoints)); arr[idx][field] = e.detail.value; this.stdPoints = arr },
    addStdPoint() { this.stdPoints.push({ conc: '', expR: '', expG: '', expB: '', ctrlR: '', ctrlG: '', ctrlB: '' }) },
    onChannelChange(e) { this.channelIndex = Number(e.detail.value) },
    onUnitInput(e) { this.unit = e.detail.value },
    fitCurve() {
      const ch = this.channelList[this.channelIndex], ek = 'exp' + ch, ck = 'ctrl' + ch
      const pts = this.stdPoints.filter(p => p.conc !== '' && p[ek] !== '' && p[ck] !== '' && Number(p[ck]) !== 0)
      if (pts.length < 2) { uni.showToast({ title: 'At least 2 valid points required', icon: 'none' }); return }
      const n = pts.length; let sx = 0, sy = 0, sxy = 0, sxx = 0; const pd = []
      pts.forEach(p => { const x = Number(p.conc), y = Number(p[ek]) / Number(p[ck]); pd.push({ x, y }); sx += x; sy += y; sxy += x * y; sxx += x * x })
      const denominator = n * sxx - sx * sx
      if (!denominator) { uni.showToast({ title: 'Concentrations must vary', icon: 'none' }); return }
      const k = (n * sxy - sx * sy) / denominator, b = (sy - k * sx) / n, my = sy / n
      let sst = 0, sse = 0; pd.forEach(p => { const yf = k * p.x + b; sst += (p.y - my) ** 2; sse += (p.y - yf) ** 2 })
      const r2 = sst ? 1 - sse / sst : 1
      this.curveK = k; this.curveB = b; this.formula = `Ratio = ${k.toFixed(4)} × Conc + ${b.toFixed(4)}`; this.r2Str = `R² = ${r2.toFixed(4)}`
      uni.setStorageSync('stdPoints', this.stdPoints); uni.setStorageSync('channelIndex', this.channelIndex); uni.setStorageSync('curveUnit', this.unit)
      uni.setStorageSync('curveFormula', this.formula); uni.setStorageSync('curveR2', this.r2Str); uni.setStorageSync('curveK', k); uni.setStorageSync('curveB', b)
      this.drawCanvas(pd, k, b)
    },
    drawCanvas(data, k, b) {
      const ctx = uni.createCanvasContext('myCanvas', this), { canvasW: cw, canvasH: ch, unit } = this
      const xs = data.map(p => p.x), ys = data.map(p => p.y), minX = Math.min(...xs), maxX = Math.max(...xs), minY = Math.min(...ys), maxY = Math.max(...ys)
      const pad = 30, xt = 5, yt = 5, xSt = (maxX - minX) / (xt - 1) || 1, ySt = (maxY - minY) / (yt - 1) || 1
      const scX = (cw - 2 * pad) / ((maxX - minX) || 1), scY = (ch - 2 * pad) / ((maxY - minY) || 1)
      const toX = x => pad + (x - minX) * scX, toY = y => ch - pad - (y - minY) * scY
      ctx.clearRect(0, 0, cw, ch)
      ctx.setStrokeStyle('#888'); ctx.setLineWidth(1.5)
      ctx.beginPath(); ctx.moveTo(pad, pad); ctx.lineTo(pad, ch - pad); ctx.lineTo(cw - pad, ch - pad); ctx.stroke()
      ctx.setFontSize(11); ctx.setFillStyle('#444')
      for (let i = 0; i < yt; i++) { const v = minY + ySt * i, p = toY(v); ctx.beginPath(); ctx.moveTo(pad - 5, p); ctx.lineTo(pad, p); ctx.stroke(); ctx.fillText(v.toFixed(2), pad - 42, p + 4) }
      for (let i = 0; i < xt; i++) { const v = minX + xSt * i, p = toX(v); ctx.beginPath(); ctx.moveTo(p, ch - pad); ctx.lineTo(p, ch - pad + 5); ctx.stroke(); ctx.fillText(v.toFixed(2), p - 14, ch - pad + 18) }
      ctx.save(); ctx.setFontSize(14); ctx.setFillStyle('#2266cc'); ctx.translate(pad - 56, ch / 2); ctx.rotate(-Math.PI / 2); ctx.fillText('Ratio', 0, 0); ctx.restore()
      ctx.setFontSize(14); ctx.setFillStyle('#2266cc'); ctx.fillText(`Concentration (${unit})`, cw / 2 - 54, ch - pad + 36)
      ctx.setFillStyle('#1a91fa'); data.forEach(p => { ctx.beginPath(); ctx.arc(toX(p.x), toY(p.y), 4, 0, Math.PI * 2); ctx.fill() })
      ctx.setStrokeStyle('#d21b3d'); ctx.setLineWidth(2); ctx.beginPath(); ctx.moveTo(toX(minX), toY(k * minX + b)); ctx.lineTo(toX(maxX), toY(k * maxX + b)); ctx.stroke()
      ctx.draw()
    },
    clearStdPoints() {
      uni.showModal({ title: '提示', content: '确定清空所有标准点？', success: (r) => { if (r.confirm) { this.stdPoints = [{ conc: '', expR: '', expG: '', expB: '', ctrlR: '', ctrlG: '', ctrlB: '' }]; uni.setStorageSync('stdPoints', this.stdPoints); uni.showToast({ title: '已清空', icon: 'success' }) } } })
    },
    goBackHome() { uni.redirectTo({ url: '/pages/index/index' }) }, goToPrevPage() { uni.navigateBack() }, goToResultPage() { uni.navigateTo({ url: '/pages/result/result' }) }
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
input, .picker, .unit-input { font-size: 24rpx; color: #2E3A2B; border: 1rpx solid #CDE7CD; border-radius: 24rpx; padding: 0 20rpx; height: 64rpx; background: #FFFFFF; margin-bottom: 16rpx; box-sizing: border-box; }
input:focus, .picker:focus { border-color: #70B774; }
.std-row, .ratio-row { display: flex; flex-wrap: wrap; align-items: center; gap: 16rpx; }
.rich-formula { font-size: 28rpx; font-weight: 600; color: #356A48; }
.rich-r2 { font-size: 26rpx; font-weight: 500; color: #d21b3d; margin-top: 8rpx; }
.button-group { display: flex; flex-direction: column; align-items: center; gap: 10px; margin-bottom: 14px; }
.std-canvas-container { width: 100%; max-width: 350px; min-width: 220px; margin: 0 auto; background: #fff; border-radius: 24rpx; box-shadow: 0 4rpx 16rpx rgba(106,166,105,0.12); padding: 12rpx; display: flex; justify-content: center; align-items: center; }
.std-canvas { width: 100%; height: 180px; min-height: 120px; max-height: 220px; display: block; border-radius: 16rpx; background: #fff; }
.std-row input { flex: 1 1 0; min-width: 0; max-width: 70rpx; padding: 0 8rpx; height: 48rpx; font-size: 20rpx; border-radius: 14rpx; margin: 0; box-sizing: border-box; }
.table-header { display: flex; flex-direction: row; flex-wrap: nowrap; font-weight: bold; gap: 10rpx; margin-bottom: 12rpx; }
.cell { flex: 1 1 0; min-width: 0; max-width: 70rpx; text-align: center; }
</style>
