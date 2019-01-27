<template>
  <div class="picker-root">
    <input ref="pickerClrcode" class="picker-clrcode"
           type="text" :value="rgbaCode" :style=clrCodeStyle
           @blur="rgbaCodeInput" @keyup.13="rgbaCodeInput" @focus="focusAndSelect">
    <div ref="pickerContainer" class="picker-container"
         :class="pickerContainerClass" :style=pickerContainerStyle
         v-if="showPicker"
         @mouseup="selectEnd" @mousedown="touchPicker"
         @mouseleave="selectEnd" @mousemove="dragPicker">
      <div class="picker-inner">
        <div class="picker-base">
          <div class="picker_clr" @mousedown="clrSelectStart" :style="clrStyle">
            <div class="picker-clr-csr" :style="clrCsrStyle"></div>
          </div>
        </div>
        <div class="picker-base">
          <div class="picker-hue picker-gage" :style="hueGageStyle" @mousedown="hueSelectStart">
            <div class="picker-hue-csr picker-csr" :style="hueCsrStyle"></div>
          </div>
        </div>
        <div class="picker-base">
          <div class="picker-alp picker-gage" :style="alpGageStyle" @mousedown="alpSelectStart">
            <div class="picker-alp-csr picker-csr" :style="alpCsrStyle"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'VueSimpleColorPicker',

  data: function () {
    return {
      hsv: {
        h: 360,
        s: 100,
        v: 100
      },

      hueChangeFlg: false,
      alpChangeFlg: false,
      clrChangeFlg: false,
      pickerTouchFlg: false,

      showPicker: false
    }
  },

  watch: {
    hsv: {
      handler: function (newVal, _oldVal) {
        this.localRgb = this.__hsv2rgb(newVal)
      },
      deep: true,
      immediate: true
    }
  },

  props: {
    rgb: {
      require: false,
      type: Object,
      default: function () {
        return { r: 255, g: 255, b: 255 }
      },
      validator: function (val) {
        return 'r' in val && 'g' in val && 'b' in val
      }
    },
    alpha: {
      require: false,
      type: Number,
      default: 1,
      validator: function (val) {
        return val >= 0 && val <= 1
      }
    },
    pickerWidth: {
      require: false,
      type: Number,
      default: 300
    },
    pickerPosition: {
      require: false,
      type: String,
      validator: function (val) {
        return val === 'top' ||
               val === 'bottom' ||
               val === 'left_top' ||
               val === 'left_middle' ||
               val === 'left_bottom' ||
               val === 'right_top' ||
               val === 'right_middle' ||
               val === 'right_bottom'
      }
    }
  },

  methods: {

    hueSelectStart: function (event) {
      this.hueChangeFlg = true

      this.__setHuePos()

      this.pickerTouchFlg = true
      var $form = this.$refs.pickerClrcode
      $form.focus()
    },

    hueDrag: function (event) {
      this.__setHuePos()
    },

    alpSelectStart: function (event) {
      this.alpChangeFlg = true

      this.__setAlpPos()

      this.pickerTouchFlg = true
      var $form = this.$refs.pickerClrcode
      $form.focus()
    },

    alpDrag: function (event) {
      this.__setAlpPos()
    },

    clrSelectStart: function (event) {
      this.clrChangeFlg = true

      this.__setClrPos()

      this.pickerTouchFlg = true
      var $form = this.$refs.pickerClrcode
      $form.focus()
    },

    clrDrag: function (event) {
      this.__setClrPos()
    },

    dragPicker: function (event) {
      if (this.hueChangeFlg) { this.hueDrag(event) }
      if (this.alpChangeFlg) { this.alpDrag(event) }
      if (this.clrChangeFlg) { this.clrDrag(event) }

      var $form = this.$refs.pickerClrcode
      $form.focus()
    },

    touchPicker: function () {
      this.pickerTouchFlg = true
      var $form = this.$refs.pickerClrcode
      $form.focus()
    },

    selectEnd: function () {
      this.hueChangeFlg = false
      this.alpChangeFlg = false
      this.clrChangeFlg = false
      this.pickerTouchFlg = false
    },

    rgbaCodeInput: function (event) {
      if (this.pickerTouchFlg) {
        return false
      }

      this.showPicker = false

      var rgbaCode = event.target.value

      rgbaCode = rgbaCode.replace(/rgba?/, '')
      rgbaCode = rgbaCode.replace(/\(/, '')
      rgbaCode = rgbaCode.replace(/\)/, '')
      rgbaCode = rgbaCode.replace(/ /g, '')
      var rgbaCodeAry = rgbaCode.split(',')

      if (rgbaCodeAry.length < 3 || rgbaCodeAry.length > 4) {
        return false
      }

      var rgb = { r: 0, g: 0, b: 0 }
      rgb.r = this.__limit(parseInt(rgbaCodeAry[0], 10), 0, 255)
      rgb.g = this.__limit(parseInt(rgbaCodeAry[1], 10), 0, 255)
      rgb.b = this.__limit(parseInt(rgbaCodeAry[2], 10), 0, 255)

      var alpStr = rgbaCodeAry[3] ? rgbaCodeAry[3] : '1'
      this.localAlpha = this.__limit(parseFloat(alpStr), 0, 1)

      this.hsv = this.__rgb2hsv(rgb)
    },

    focusAndSelect: function (event) {
      this.showPicker = true
      event.target.select()
    },

    // -------------------------------------------------------------------------

    __setHuePos: function () {
      var $container = this.$refs.pickerContainer
      var hueCsrPos = this.__limit(this.__relpos(event.pageY, $container.offsetTop), 0, this.pickerSize)
      this.hsv.h = Math.round(Math.abs(this.hueUnit * hueCsrPos - 360))
    },

    __setAlpPos: function () {
      var $container = this.$refs.pickerContainer
      var alpCsrPos = this.__limit(this.__relpos(event.pageY, $container.offsetTop), 0, this.pickerSize)
      this.localAlpha = Math.round(Math.abs(this.alpUnit * alpCsrPos - 1) * 100) / 100
    },

    __setClrPos: function () {
      var $container = this.$refs.pickerContainer
      var clrCsrPosX = this.__limit(this.__relpos(event.pageX, $container.offsetLeft), 0, this.pickerSize)
      var clrCsrPosY = this.__limit(this.__relpos(event.pageY, $container.offsetTop), 0, this.pickerSize)

      this.hsv.s = Math.round(clrCsrPosX * this.clrUnit)
      this.hsv.v = Math.round(Math.abs(clrCsrPosY * this.clrUnit - 100))
    },

    __relpos: function (pagePos, offset) {
      return pagePos - (this.pickerPadding + offset)
    },

    __limit: function (target, min, max) {
      return Math.min(Math.max(target, min), max)
    },

    __objectMap: function (obj, mapFn) {
      return Object.keys(obj).reduce(function (result, key) {
        result[key] = mapFn(obj[key])
        return result
      }, {})
    },

    __gageCsrStyle: function (csrPos, csrSize) {
      var csrHalfSize = Math.round(csrSize / 2)
      var csrWidth = Math.round(csrSize / 1.4)
      return {
        top: csrPos + 'px',
        right: -csrWidth + 'px',
        borderTopWidth: csrHalfSize + 'px',
        borderBottomWidth: csrHalfSize + 'px',
        borderRightWidth: csrWidth + 'px'
      }
    },

    __gageStyle: function () {
      return {
        height: this.pickerSize + 'px',
        width: this.gageWidth + 'px'
      }
    },

    __rgbaCode: function () {
      if (this.localAlpha === 1) {
        return 'rgb(' + this.localRgb.r + ', ' + this.localRgb.g + ', ' + this.localRgb.b + ')'
      } else {
        return 'rgba(' + this.localRgb.r + ', ' + this.localRgb.g + ', ' + this.localRgb.b + ', ' + this.localAlpha + ')'
      }
    },

    __rgb2hsv: function (rgb) {
      var nrmRgb = this.__objectMap(rgb, val => val / 255)
      var max = Math.max(...Object.values(nrmRgb))
      var min = Math.min(...Object.values(nrmRgb))

      var h = max - min
      if (h > 0) {
        if (max === nrmRgb.r) {
          h = (nrmRgb.g - nrmRgb.b) / h
          if (h < 0) {
            h += 6
          }
        } else if (max === nrmRgb.g) {
          h = 2 + (nrmRgb.b - nrmRgb.r) / h
        } else {
          h = 4 + (nrmRgb.r - nrmRgb.g) / h
        }
      }
      h *= 60

      var s = max - min
      if (max !== 0) {
        s /= max
      }
      s *= 100

      var v = max * 100

      return { h: h, s: s, v: v }
    },

    __hsv2rgb: function (hsv) {
      var nrmHsv = { h: hsv.h / 360, s: hsv.s / 100, v: hsv.v / 100 }
      var rgb = { r: nrmHsv.v, g: nrmHsv.v, b: nrmHsv.v }

      if (nrmHsv.s > 0) {
        var h = (nrmHsv.h * 6) % 6
        var i = Math.floor(h)
        var f = h - i

        var v1 = 1 - nrmHsv.s
        var v2 = 1 - nrmHsv.s * f
        var v3 = 1 - nrmHsv.s * (1 - f)

        switch (i) {
          case 0: rgb.g *= v3; rgb.b *= v1; break
          case 1: rgb.b *= v1; rgb.r *= v2; break
          case 2: rgb.b *= v3; rgb.r *= v1; break
          case 3: rgb.r *= v1; rgb.g *= v2; break
          case 4: rgb.r *= v3; rgb.g *= v1; break
          case 5: rgb.g *= v1; rgb.b *= v2; break
        }
      }

      return this.__objectMap(rgb, val => Math.round(val * 255))
    }
  },

  computed: {

    localRgb: {
      get: function () { return this.rgb },
      set: function (val) { this.$emit('update:rgb', val) }
    },
    localAlpha: {
      get: function () { return this.alpha },
      set: function (val) { this.$emit('update:alpha', val) }
    },

    // ------------------------------------------------------------------------

    pickerSize: function () {
      return Math.round(this.pickerWidth / 2)
    },
    pickerPadding: function () {
      return Math.round(this.pickerSize / 12)
    },
    pickerHeight: function () {
      return this.pickerSize + this.pickerPadding * 2
    },
    gageWidth: function () {
      return Math.round(this.pickerWidth / 8)
    },
    hueCsrSize: function () {
      return Math.round(this.pickerWidth / 70) + 4
    },
    alpCsrSize: function () {
      return Math.round(this.pickerWidth / 70) + 4
    },
    clrCsrSize: function () {
      return Math.round(this.pickerWidth / 70) + 4
    },
    hueUnit: function () {
      return 360 / this.pickerSize
    },
    alpUnit: function () {
      return 1 / this.pickerSize
    },
    clrUnit: function () {
      return 100 / this.pickerSize
    },

    pickerContainerStyle: function () {
      var $form = this.$refs.pickerClrcode
      var rect = $form.getBoundingClientRect()
      var posTop = 0
      var posLeft = 0

      if (this.pickerContainerClass.includes('--top')) {
        posTop = rect.top + window.pageYOffset - this.pickerHeight - 20
        posLeft = rect.left
      }
      if (this.pickerContainerClass.includes('--bottom')) {
        posTop = rect.bottom + window.pageYOffset + 20
        posLeft = rect.left
      }
      if (this.pickerContainerClass.includes('--left')) {
        posLeft = rect.left - this.pickerWidth - 20
      }
      if (this.pickerContainerClass.includes('--right')) {
        posLeft = rect.right + 20
      }

      var formMiddle = Math.round(rect.top + rect.height / 2 + window.pageYOffset)
      if (this.pickerContainerClass.includes('_top')) {
        posTop = formMiddle - Math.round(this.pickerHeight * 0.9) + 10
      }
      if (this.pickerContainerClass.includes('_middle')) {
        posTop = formMiddle - Math.round(this.pickerHeight * 0.5)
      }
      if (this.pickerContainerClass.includes('_bottom')) {
        posTop = formMiddle - Math.round(this.pickerHeight * 0.1) - 10
      }

      var pad = this.pickerPadding

      return {
        width: (this.pickerWidth - pad * 2 - this.alpCsrSize / 2) + 'px',
        padding: pad + 'px ' + (pad + this.alpCsrSize / 2) + 'px ' + pad + 'px ' + pad + 'px',
        top: posTop + 'px',
        left: posLeft + 'px'
      }
    },

    pickerContainerClass: function () {
      if (!this.showPicker) { return '' }
      if (this.pickerPosition !== undefined) {
        return 'picker-container--' + this.pickerPosition
      }

      var $form = this.$refs.pickerClrcode
      var rect = $form.getBoundingClientRect()
      if (rect.top > this.pickerHeight + 20) {
        return 'picker-container--top'
      } else {
        return 'picker-container--bottom'
      }
    },

    hueCsrStyle: function () {
      var csrPos = Math.round(Math.abs(this.hsv.h - 360) / this.hueUnit - this.hueCsrSize / 2)
      return this.__gageCsrStyle(csrPos, this.hueCsrSize)
    },

    alpCsrStyle: function () {
      var csrPos = Math.round(Math.abs(this.localAlpha - 1) / this.alpUnit - this.alpCsrSize / 2)
      return this.__gageCsrStyle(csrPos, this.alpCsrSize)
    },

    clrCsrStyle: function () {
      var clrCsrX = this.hsv.s / this.clrUnit - this.clrCsrSize / 2
      var clrCsrY = Math.abs(this.hsv.v / this.clrUnit - this.pickerSize) - this.clrCsrSize / 2
      return {
        left: clrCsrX + 'px',
        top: clrCsrY + 'px',
        width: this.clrCsrSize + 'px',
        height: this.clrCsrSize + 'px'
      }
    },

    hueGageStyle: function () {
      return this.__gageStyle()
    },

    alpGageStyle: function () {
      return this.__gageStyle()
    },

    clrStyle: function () {
      return {
        backgroundColor: 'hsl(' + this.hsv.h + ', 100%, 50%)',
        opacity: this.localAlpha,
        width: this.pickerSize + 'px',
        height: this.pickerSize + 'px'
      }
    },

    clrCodeStyle: function () {
      var clr = 'white'
      if ((this.hsv.v * this.localAlpha + 100 * (1 - this.localAlpha)) > 50) {
        clr = 'black'
      }

      return {
        backgroundColor: this.rgbaCode,
        color: clr
      }
    },

    rgbaCode: function () {
      return this.__rgbaCode()
    }
  }
}
</script>

<style scoped lang="scss">
@import "../stylesheets/global.scss";

.picker-container {
  position: absolute;
  top: 100px;
  left: 20px;
  text-align: center;
}
.picker-container::before {
  content: '';
  position: absolute;
  z-index: 1;
  width: 20px; height: 20px;
}
.picker-container::after {
  content: '';
  position: absolute;
  z-index: 2;
  top: 0; left: 0;
  width: 100%; height: 100%;
}
.picker-container>* {
  position: relative;
  z-index: 3;
}
.picker-container,
.picker-container::after {
  border-radius: 5px;
}
.picker-container,
.picker-container::before {
  box-shadow: 0 0 10px 0 rgba(163,163,163,0.50);
}
.picker-container,
.picker-container::before,
.picker-container::after {
  background: rgb(53, 53, 53);
}
.picker-container--bottom::before {
  top: -10px; left: 10%;
  transform: rotate(45deg) skew(10deg,10deg);
}
.picker-container--left_bottom::before {
  right: -10px; top: 10%;
  transform: rotate(45deg) skew(-10deg,-10deg);
}
.picker-container--left_middle::before {
  right: -10px; top: 50%;
  transform: translateY(-50%) rotate(45deg) skew(-10deg,-10deg);
}
.picker-container--left_top::before {
  right: -10px; bottom: 10%;
  transform: rotate(45deg) skew(-10deg,-10deg);
}
.picker-container--top::before {
  bottom: -10px; left: 10%;
  transform: rotate(45deg) skew(10deg,10deg);
}
.picker-container--right_bottom::before {
  left: -10px; top: 10%;
  transform: rotate(45deg) skew(-10deg,-10deg);
}
.picker-container--right_middle::before {
  left: -10px; top: 50%;
  transform: translateY(-50%) rotate(45deg) skew(-10deg,-10deg);
}
.picker-container--right_top::before {
  left: -10px; bottom: 10%;
  transform: rotate(45deg) skew(-10deg,-10deg);
}

.picker-inner {
  display: flex;
  justify-content: space-between;
}

.picker-base {
  @include trans_base(#eee, 20px);
}

.picker_clr {
  position: relative;
  overflow: hidden;
  background-color: rgb(255, 0, 0);
  @each $prefix in -webkit-, -moz-, -ms-, -o-, '' {
    background-image:
      #{$prefix}linear-gradient(to bottom, rgba(0,   0,   0,   0),   rgba(0,   0,   0,   255)),
      #{$prefix}linear-gradient(to right,  rgba(255, 255, 255, 255), rgba(255, 255, 255, 0));
  }
}

.picker-clr-csr {
  pointer-events: none;
  position: absolute;
  left: 400px;
  top: -9px;
  box-sizing: border-box;
  border: 1px solid black;
  border-radius: 50%;
  @include css3_value(
    background-image,
    radial-gradient(transparent 0%, transparent 60%, white 60%)
  );
}

.picker-gage {
  position: relative;
  width: 40px;
}

.picker-csr {
  position: absolute;
  border-right: 10px solid rgb(255, 255, 255);
  border-top: 7px solid transparent;
  border-bottom: 7px solid transparent;
}

.picker-hue {
  @include css3_value(
    background-image,
    linear-gradient(
      hsl(360,100%,50%),
      hsl(300,100%,50%),
      hsl(240,100%,50%),
      hsl(180,100%,50%),
      hsl(120,100%,50%),
      hsl(60,100%,50%),
      hsl(0,100%,50%)
    )
  );
}

.picker-alp {
  @include css3_value(
    background-image,
    linear-gradient(to bottom, rgba(0,0,0,255), rgba(0,0,0,0))
  );
}

.picker-clrcode {
  display: block;
  width: 150px;
  margin-bottom:15px;
  padding: 8px 5px;
  font-size: 13px;
  border-radius: 2px;
  border: 1px solid #777;
  background: #555;
  color: #FFF;
}
</style>
