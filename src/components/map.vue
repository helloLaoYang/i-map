<template>
  <div class="qq-map">
    <div class="qq-map__transparent"></div>
    <div class="qq-map__container">
      <div class="qq-map__hd">
        <label class="qq-map__title">
          <div>请输入地址：</div>
          <input type="text" class="qq-map__input" v-model="keywords"
            @keydown.enter="search2Latlng"
            placeholder="请输入您大致需要查询的地址">
        </label>
        <!-- 地图盒子 -->
        <div class="qq-map__box"></div>
      </div>
    </div>
  </div>
</template>
<script>
const qq = window.qq

export default {
  name: 'i-map',
  props: {
    value: {
      type: Object,
      default () {
        return {
          lat: 30.270093,
          lng: 120.162100
        }
      }
    },
    autoLocation: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      keywords: ''
    }
  },
  computed: {
    $mapEl () {
      return this.$el.querySelector('.qq-map__box')
    }
  },
  methods: {
    // 创建新的坐标点对象
    createCenterInstace (lat, lng) {
      return new qq.maps.LatLng(lat, lng)
    },
    // 自动定位，获取当前坐标点
    getCenter () {
      const that = this
      return new Promise(function (resolve, reject) {
        const Geolocation = new qq.maps.Geolocation('KORBZ-FHLKG-LHJQ5-IM6US-L5YGQ-AEFJP', 'qq-map')
        Geolocation.getLocation(({lat, lng}) => resolve(that.createCenterInstace(lat, lng)), e => reject(e))
      })
    },
    // 创建地图实例
    createMap (center) {
      const that = this
      that.$map = new qq.maps.Map(that.$mapEl, {
        center: center,
        zoom: 13
      })
    },
    // 创建marker实例
    // 并为marker实例建立移动事件
    createMark (center) {
      const that = this
      if (this.$map == null) {
        return
      }
      if (this.$marker) {
        this.$marker.setMap(null)
      }
      this.$marker = new qq.maps.Marker({
        position: center,
        draggable: true,
        map: this.$map
      })
      qq.maps.event.addListener(this.$marker, 'position_changed', function ({target}) {
        const {position: {
          lat,
          lng
        }} = target
        that.$emit('input', {lat, lng})
      })
      return this.$marker
    },
    search2Latlng () {
      const that = this
      if (!that.keywords) {
        return
      }
      if (!that.$geocoder) {
        that.$geocoder = new qq.maps.Geocoder({
          complete ({detail}) {
            const {location} = detail
            console.log(location)
            that.$map.setCenter(location)
            const center = that.createCenterInstace(location.lat, location.lng)
            that.createMark(center)
          }
        })
      }
      that.$geocoder.getLocation(`${that.keywords}`)
    },
    // 初始化函数
    async init () {
      let center = null
      const {lat, lng} = this.value
      try {
        if (this.autoLocation) {
          center = await this.getCenter()
          return
        }
        center = this.createCenterInstace(lat, lng)
      } catch (error) {
        center = this.createCenterInstace(lat, lng)
      } finally {
        this.createMap(center)
        this.createMark(center)
      }
    }
  },
  mounted () {
    this.init()
  }
}
</script>
<style lang="css">
.qq-map__transparent {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
  background-color: rgba( 0, 0, 0, .5);
}
.qq-map__container {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
  text-align: center;
}
.qq-map__container::before {
  display: inline-block;
  content: '';
  vertical-align: middle;
  width: 0;
  height: 100%;
}
.qq-map__hd {
  display: inline-block;
  width: 75%;
  min-height: 350px;
  padding: 25px;
  border-radius: 5px;
  box-shadow: 0px 0px 30px 1px rgba(0, 0, 0, .25);
  vertical-align: middle;
  background-color: #fff;
}
.qq-map__title {
  display: block;
  width: 100%;
  text-align: left;
}
.qq-map__title::after {
  content: '';
  display: block;
  width: 0;
  height: 0;
  overflow: hidden;
  clear: both;
}
.qq-map__title div {
  display: inline-block;
}
.qq-map__input {
  padding: 2px;
  height: 14px;
  width: 25%;
  margin-left: 15px;
  overflow: hidden;
  border: 0;
  border-bottom: 1px solid #ccc;
  -webkit-appearance: none;
  outline: none;
}

.qq-map__box {
  width: 100%;
  min-height: 425px;
  margin-top: 25px;
}
</style>
