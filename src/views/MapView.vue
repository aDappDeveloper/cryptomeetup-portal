<template>
  <div class="map">
    <Loading v-if="!mapLoad" loadText="loading ..." />
    <mapbox
      access-token="null"
      :map-options="{
        style: 'https://maps.tilehosting.com/c/adbc36eb-6765-4278-8c1a-b14fa25d0ae2/styles/basic-dark/style.json?key=eT7rAVG6glnuTf9iWHbK',
        zoom: 11,
        center: [116.478515, 39.889992],
      }"
      @map-load="onMapLoaded"
      @map-init="onMapInit"
    />
    <button class="position-button mapboxgl-ctrl-icon" @click="updateLocation(true)">
      <b-icon icon="crosshairs-gps" size="is-small" />
    </button>
  </div>
</template>

<script>
import Vue from 'vue';
import mapboxgl from 'mapbox-gl';
import Mapbox from 'mapbox-gl-vue';
import geolib from 'geolib';
import { ajax, analysis } from '@/util/ajax'

import RedeemCodeCopyDialog from '@/components/RedeemCodeCopyDialog.vue';
import MapMarkerLocation from '@/components/landmark/MapMarkerLocation.vue';
import MapMarkerMeetup from '@/components/landmark/MapMarkerMeetup.vue';
import Loading from '@/components/Loading.vue';
import LocationPopup from '@/components/landmark/LocationPopup.vue';
import createLocation from '@/components/landmark/createLocation.vue';
import location from './location.js'
import { mapState } from 'vuex'
import { setLocalStorage, removeLocalStorage } from '@/util/storeUtil.js'

export default {
  name: 'map-view',
  data() {
    return {
      mapLoad: false,
      showPopup: false,
    };
  },
  components: {
    Mapbox,
    Loading,
  },
  computed: {
    ...mapState(['scatterAccount'])
  },
  watch: {
    scatterAccount(val) {
      if (val) {
        this.coreLogin(val)
      } else {
        removeLocalStorage('Authorization')
        removeLocalStorage('userId')
        removeLocalStorage('name')
      }
    },
  },
  created() {
    // 这两到时候都应该删除的
    this.meetupLocation = [116.478515, 39.889992];
  },
  mounted() {
    this.jumped = false;
  },
  methods: {
    coreLogin(account) {
      if (!account || !account.name) {
        return false
      }
      const param = analysis('/auth/mobile/token', {
        account: account.name,
        nickName: account.name,
        grant_type: 'mobile',
        scope: 'server',
        channel: 'eos',
        appId: 10001
      })
      ajax.post(param, null, {headers: {
        Authorization: 'Basic bGl5YW5nOnJlZC1wYWNrZXQ='
      }}).then(resp => {
        setLocalStorage('userId', resp.data.userId)
        setLocalStorage('name', account.name)
        setLocalStorage('Authorization', `Bearer ${resp.data.access_token}`)
      })
    },
    onMapInit(map) {
      // 初始化地图
      map.resize();
    },
    onMapLoaded(map) {
      // 地图加载成功
      location.onMapLoaded(map)
      // this.map = map;
      this.mapLoad = true;
      // 渲染地标

      // this.popupComponent.$on('redeemCodeGenerated', (code) => {
      //   this.$modal.open({
      //     parent: this,
      //     component: RedeemCodeCopyDialog,
      //     hasModalCard: true,
      //     props: {
      //       code,
      //     },
      //   });
      // });

      // if ('geolocation' in navigator) {
      //   this.locationUpdateTimer = setInterval(() => this.updateLocation(), 5000);
      //   this.updateLocation();
      // }
    },

    // updateCheckInAvailability(lonLat) {
    //   if (!lonLat) {
    //     return;
    //   }
    //   if (!this.popupComponent) {
    //     return;
    //   }
    //   const distance = geolib.getDistance(
    //     { latitude: this.meetupLocation[1], longitude: this.meetupLocation[0] },
    //     { latitude: lonLat[1], longitude: lonLat[0] },
    //   );
    //   // 计算活动于用户的位置
    //   this.popupComponent.setCanCheckIn(distance <= 1000);
    // },
    // updateLocation(fly = false) {
    //   navigator.geolocation.getCurrentPosition((position) => {
    //     const coord = [position.coords.longitude, position.coords.latitude];
    //     this.updateCheckInAvailability(coord);
    //     if (fly) {
    //       this.map.flyTo({ center: coord, zoom: 13 });
    //       this.jumped = true;
    //       this.getLocation(coord);
    //     } else if (!this.jumped) {
    //       // Jump
    //       this.map.jumpTo({ center: coord });
    //       this.jumped = true;
    //     }
    //     this.updateMyLocation();

    //     if (!this.locationArr) {
    //       this.getLocation(coord);
    //     }
    //   });
    // },

  },
  destroyed() {
    if (this.locationUpdateTimer) {
      clearInterval(this.locationUpdateTimer);
    }
  },
};
</script>

<style lang="sass" scoped>
.position-button
  position: absolute
  z-index: 1
  left: 10px
  bottom: 10px
  border-radius: 4px
  outline: none

.map
  position: absolute
  left: 0
  top: 0
  width: 100%
  height: 100%
  z-index: 0

  /deep/
    #map
      position: absolute
      top: 0
      left: 0
      width: 100%
      height: 100%
      color: #222
</style>
