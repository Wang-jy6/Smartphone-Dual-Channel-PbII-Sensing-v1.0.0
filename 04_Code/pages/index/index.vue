<template>
  <view class="container">
    <view class="card">
      <view class="big-title">Image Analyzer</view>
      <view class="section-title">Select Box Type</view>
      <view class="switch-btn-row">
        <button
          :class="'switch-btn ' + (activeBox==='exp' ? 'btn-active-exp' : '')"
          data-box="exp"
          @tap="onBoxSwitch"
        >Experimental</button>
        <button
          :class="'switch-btn ' + (activeBox==='ctrl' ? 'btn-active-ctrl' : '')"
          data-box="ctrl"
          @tap="onBoxSwitch"
        >Control</button>
      </view>
      <text class="text-sub">Drag to move, resize corners to adjust box size</text>
    </view>

    <view class="card">
      <button class="button-main" @tap="chooseImage">Choose Image</button>
    </view>

    <view class="card">
      <view class="index-canvas-wrapper">
        <canvas
          id="imgCanvas"
          class="index-canvas"
          canvas-id="imgCanvas"
          :width="canvasW"
          :height="canvasH"
          @touchstart="onTouchStart"
          @touchmove.stop.prevent="onTouchMove"
          @touchend="onTouchEnd"
        />
      </view>
      <button class="button-main" @tap="calcRGB">Calculate RGB Values</button>
      <button class="button-main" @tap="handleButtonATap">Send to Standard</button>
    </view>

    <view class="card">
      <view class="medium-title">Analysis Results</view>
      <view class="section-title">Experimental Group RGB</view>
      <view class="std-row">
        <text class="text-base">R: {{expRGB.R}}</text>
        <text class="text-base">G: {{expRGB.G}}</text>
        <text class="text-base">B: {{expRGB.B}}</text>
      </view>
      <view class="section-title">Control Group RGB</view>
      <view class="std-row">
        <text class="text-base">R: {{ctrlRGB.R}}</text>
        <text class="text-base">G: {{ctrlRGB.G}}</text>
        <text class="text-base">B: {{ctrlRGB.B}}</text>
      </view>
      <view class="section-title">Ratio Values</view>
      <view class="ratio-row">
        <text class="text-base">R Ratio: <text class="rich-formula">{{ratios.R||'--'}}</text></text>
        <text class="text-base">G Ratio: <text class="rich-formula">{{ratios.G||'--'}}</text></text>
        <text class="text-base">B Ratio: <text class="rich-formula">{{ratios.B||'--'}}</text></text>
      </view>
    </view>

    <view class="card">
      <button class="button-main" @tap="goToStandardPage">Standard Curve Fitting</button>
      <button class="button-main" @tap="goToResultPage">View Results</button>
      <button class="button-main" @tap="hisgoToResultPage">History Data</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      activeBox: 'exp', expRGB: { R: 0, G: 0, B: 0 }, ctrlRGB: { R: 0, G: 0, B: 0 },
      ratios: {},
      imagePath: '',
      canvasW: 300,
      canvasH: 300,
      boxes: {
        exp: { x: 45, y: 55, w: 96, h: 96 },
        ctrl: { x: 160, y: 55, w: 96, h: 96 }
      },
      _touchState: null,
      _canvasRect: null
    }
  },
  onReady() {
    this.setCanvasSize()
  },
  onShow() {
    this.expRGB = uni.getStorageSync('expRGB') || { R: 0, G: 0, B: 0 }
    this.ctrlRGB = uni.getStorageSync('ctrlRGB') || { R: 0, G: 0, B: 0 }
    this.ratios = uni.getStorageSync('ratios') || {}
    const boxes = uni.getStorageSync('selectionBoxes')
    if (boxes && boxes.exp && boxes.ctrl) this.boxes = boxes
    this.$nextTick(() => this.redrawCanvas())
  },
  methods: {
    onBoxSwitch(e) {
      this.activeBox = e.currentTarget.dataset.box
      this.redrawCanvas()
    },
    chooseImage() {
      uni.chooseImage({ count: 1, sizeType: ['original'], sourceType: ['album', 'camera'],
        success: (res) => {
          this.imagePath = res.tempFilePaths[0]
          this.setCanvasSize(() => this.redrawCanvas())
        }
      })
    },
    setCanvasSize(done) {
      uni.createSelectorQuery().in(this).select('.index-canvas-wrapper').boundingClientRect((rect) => {
        if (rect && rect.width) {
          const size = Math.round(rect.width)
          this.canvasW = size
          this.canvasH = size
          this._canvasRect = rect
          this.normalizeAllBoxes()
        }
        this.$nextTick(() => {
          this.redrawCanvas()
          if (typeof done === 'function') done()
        })
      }).exec()
    },
    redrawCanvas() {
      const ctx = uni.createCanvasContext('imgCanvas', this)
      ctx.setFillStyle('#FFFFFF')
      ctx.fillRect(0, 0, this.canvasW, this.canvasH)
      if (this.imagePath) ctx.drawImage(this.imagePath, 0, 0, this.canvasW, this.canvasH)
      this.drawSelectionBox(ctx, 'exp')
      this.drawSelectionBox(ctx, 'ctrl')
      ctx.draw()
    },
    drawImageOnly() {
      return new Promise((resolve) => {
        const ctx = uni.createCanvasContext('imgCanvas', this)
        ctx.setFillStyle('#FFFFFF')
        ctx.fillRect(0, 0, this.canvasW, this.canvasH)
        ctx.drawImage(this.imagePath, 0, 0, this.canvasW, this.canvasH)
        ctx.draw(false, () => resolve())
      })
    },
    drawSelectionBox(ctx, key) {
      const box = this.boxes[key]
      const active = key === this.activeBox
      const color = key === 'exp' ? '#A3D9A5' : '#70B774'
      const handle = active ? 12 : 9
      ctx.setStrokeStyle(color)
      ctx.setLineWidth(active ? 3 : 2)
      ctx.beginPath()
      ctx.rect(box.x, box.y, box.w, box.h)
      ctx.stroke()
      ctx.setFillStyle(color)
      this.getCorners(box).forEach(p => {
        ctx.fillRect(p.x - handle / 2, p.y - handle / 2, handle, handle)
      })
    },
    async calcRGB() {
      if (!this.imagePath) { uni.showToast({ title: 'Please select image first', icon: 'none' }); return }
      uni.showLoading({ title: 'Calculating...' })
      try {
        this.normalizeAllBoxes()
        await this.drawImageOnly()
        const expRGB = await this.readBoxRGB('exp')
        const ctrlRGB = await this.readBoxRGB('ctrl')
        this.expRGB = expRGB
        this.ctrlRGB = ctrlRGB
        uni.setStorageSync('expRGB', expRGB)
        uni.setStorageSync('ctrlRGB', ctrlRGB)
        uni.setStorageSync('selectionBoxes', this.boxes)
        this.calcRatios()
        this.saveHistory()
        uni.hideLoading()
        uni.showToast({ title: 'RGB values saved', icon: 'success' })
      } catch (err) {
        uni.hideLoading()
        uni.showToast({ title: 'Read failed', icon: 'none' })
      } finally {
        this.redrawCanvas()
      }
    },
    readBoxRGB(key) {
      return new Promise((resolve, reject) => {
        const box = this.getSampleBox(this.boxes[key])
        uni.canvasGetImageData({
          canvasId: 'imgCanvas',
          x: box.x,
          y: box.y,
          width: box.w,
          height: box.h,
          success: (res) => resolve(this.averageImageData(res.data)),
          fail: reject
        }, this)
      })
    },
    averageImageData(data) {
      let r = 0, g = 0, b = 0, c = 0
      for (let i = 0; i < data.length; i += 4) { r += data[i]; g += data[i + 1]; b += data[i + 2]; c++ }
      if (!c) return { R: 0, G: 0, B: 0 }
      return { R: Math.round(r / c), G: Math.round(g / c), B: Math.round(b / c) }
    },
    calcRatios() {
      this.ratios = {
        R: this.ctrlRGB.R ? (this.expRGB.R / this.ctrlRGB.R).toFixed(4) : '',
        G: this.ctrlRGB.G ? (this.expRGB.G / this.ctrlRGB.G).toFixed(4) : '',
        B: this.ctrlRGB.B ? (this.expRGB.B / this.ctrlRGB.B).toFixed(4) : ''
      }
      uni.setStorageSync('ratios', this.ratios)
    },
    saveHistory() {
      const history = uni.getStorageSync('rgbHistory') || []
      history.push({
        saveTime: Date.now(),
        expR: this.expRGB.R, ctrlR: this.ctrlRGB.R,
        expG: this.expRGB.G, ctrlG: this.ctrlRGB.G,
        expB: this.expRGB.B, ctrlB: this.ctrlRGB.B
      })
      uni.setStorageSync('rgbHistory', history)
    },
    handleButtonATap() { uni.showToast({ title: 'Sent to Standard', icon: 'success' }); uni.navigateTo({ url: '/pages/standard/standard' }) },
    goToStandardPage() { uni.navigateTo({ url: '/pages/standard/standard' }) },
    goToResultPage() { uni.navigateTo({ url: '/pages/result/result' }) },
    hisgoToResultPage() { uni.navigateTo({ url: '/pages/history-data/history-data' }) },
    onTouchStart(e) {
      const touch = { clientX: e.touches[0].clientX, clientY: e.touches[0].clientY }
      uni.createSelectorQuery().in(this).select('.index-canvas-wrapper').boundingClientRect((rect) => {
        if (rect) this._canvasRect = rect
        this.startCanvasTouch(touch)
      }).exec()
    },
    startCanvasTouch(touch) {
      const point = this.toCanvasPoint(touch)
      const target = this.findTouchedBox(point)
      if (!target) return
      this.activeBox = target.key
      this._touchState = {
        mode: target.corner ? 'resize' : 'drag',
        corner: target.corner,
        startX: point.x,
        startY: point.y,
        startBox: { ...this.boxes[target.key] }
      }
      this.redrawCanvas()
    },
    onTouchMove(e) {
      if (!this._touchState) return
      const point = this.toCanvasPoint(e.touches[0])
      const dx = point.x - this._touchState.startX
      const dy = point.y - this._touchState.startY
      const next = this._touchState.mode === 'resize'
        ? this.resizeBox(this._touchState.startBox, dx, dy, this._touchState.corner)
        : this.moveBox(this._touchState.startBox, dx, dy)
      this.boxes = { ...this.boxes, [this.activeBox]: next }
      this.redrawCanvas()
    },
    onTouchEnd() {
      this._touchState = null
      this.normalizeAllBoxes()
      uni.setStorageSync('selectionBoxes', this.boxes)
      this.redrawCanvas()
    },
    toCanvasPoint(touch) {
      const rect = this._canvasRect
      if (!rect) return { x: touch.clientX, y: touch.clientY }
      const scaleX = this.canvasW / rect.width
      const scaleY = this.canvasH / rect.height
      return {
        x: (touch.clientX - rect.left) * scaleX,
        y: (touch.clientY - rect.top) * scaleY
      }
    },
    findTouchedBox(point) {
      const keys = [this.activeBox, this.activeBox === 'exp' ? 'ctrl' : 'exp']
      for (const key of keys) {
        const corner = this.getTouchedCorner(point, this.boxes[key])
        if (corner) return { key, corner }
        if (this.isInsideBox(point, this.boxes[key])) return { key, corner: '' }
      }
      return null
    },
    getCorners(box) {
      return [
        { name: 'nw', x: box.x, y: box.y },
        { name: 'ne', x: box.x + box.w, y: box.y },
        { name: 'sw', x: box.x, y: box.y + box.h },
        { name: 'se', x: box.x + box.w, y: box.y + box.h }
      ]
    },
    getTouchedCorner(point, box) {
      const threshold = 18
      const hit = this.getCorners(box).find(c => Math.abs(point.x - c.x) <= threshold && Math.abs(point.y - c.y) <= threshold)
      return hit ? hit.name : ''
    },
    isInsideBox(point, box) {
      return point.x >= box.x && point.x <= box.x + box.w && point.y >= box.y && point.y <= box.y + box.h
    },
    moveBox(box, dx, dy) {
      return this.clampBox({ ...box, x: box.x + dx, y: box.y + dy })
    },
    resizeBox(box, dx, dy, corner) {
      const min = 24
      let left = box.x, top = box.y, right = box.x + box.w, bottom = box.y + box.h
      if (corner.includes('w')) left = Math.min(right - min, Math.max(0, left + dx))
      if (corner.includes('e')) right = Math.max(left + min, Math.min(this.canvasW, right + dx))
      if (corner.includes('n')) top = Math.min(bottom - min, Math.max(0, top + dy))
      if (corner.includes('s')) bottom = Math.max(top + min, Math.min(this.canvasH, bottom + dy))
      return this.clampBox({ x: left, y: top, w: right - left, h: bottom - top })
    },
    clampBox(box) {
      const min = 24
      const w = Math.max(min, Math.min(box.w, this.canvasW))
      const h = Math.max(min, Math.min(box.h, this.canvasH))
      const x = Math.max(0, Math.min(box.x, this.canvasW - w))
      const y = Math.max(0, Math.min(box.y, this.canvasH - h))
      return { x, y, w, h }
    },
    normalizeAllBoxes() {
      this.boxes = { exp: this.clampBox(this.boxes.exp), ctrl: this.clampBox(this.boxes.ctrl) }
    },
    getSampleBox(box) {
      const normalized = this.clampBox(box)
      return {
        x: Math.round(normalized.x),
        y: Math.round(normalized.y),
        w: Math.max(1, Math.round(normalized.w)),
        h: Math.max(1, Math.round(normalized.h))
      }
    }
  }
}
</script>

<style>
page, .container { font-family: "PingFang SC","Microsoft YaHei",Arial,sans-serif; color: #2E3A2B; line-height: 1.5em; padding: 24rpx; background-color: #EAF6ED; box-sizing: border-box; margin: 0; }
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
.index-canvas-wrapper { position: relative; width: 100%; padding-top: 100%; background: #FFFFFF; border-radius: 16rpx; box-shadow: 0 4rpx 16rpx rgba(106,166,105,0.2); margin-bottom: 16rpx; }
.index-canvas, canvas[canvas-id="imgCanvas"] { position: absolute !important; top: 0; left: 0; width: 100% !important; height: 100% !important; }
.switch-btn { flex: 1; height: 64rpx; margin: 0 8rpx; background: #E6F8E9; color: #2E3A2B; border-radius: 32rpx; font-size: 24rpx; font-weight: 500; box-shadow: 0 4rpx 12rpx rgba(106,166,105,0.2); border: 2rpx solid transparent; text-align: center; line-height: 64rpx; }
.btn-active-exp { background: #CDE7CD; color: #356A48; border: 2rpx solid #A3D9A5; }
.btn-active-ctrl { background: #CDE7CD; color: #356A48; border: 2rpx solid #70B774; }
.std-row, .ratio-row, .switch-btn-row { display: flex; flex-wrap: wrap; align-items: center; gap: 16rpx; }
.rich-formula { font-size: 28rpx; font-weight: 600; color: #356A48; }
</style>
