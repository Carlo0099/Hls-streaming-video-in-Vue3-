<template>
  <div class="camera_module">
    <div class="video_view">
      <video
        class="video_show"
        ref="video"
        controls="controls"
        muted
        :style="{ height: height, width: width }"
        :id="id"
      ></video>
      <div class="video_tip"></div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, nextTick } from 'vue'
import axios from 'axios'
let Hls = require('hls.js')
export default {
  name: 'HlsVideo',
  props: {
    width: {
      type: String,
      default: '1920px'
    },
    height: {
      type: String,
      default: '1080px'
    },
    gbcode: {
      type: String,
      required: true
    },
    id: Number
  },
  setup(props) {
    let data = ref({
      url: ''
    })
    let config = {
      autoStartLoad: true,
      startPosition: -1,
      capLevelToPlayerSize: false,
      debug: false,
      defaultAudioCodec: undefined,
      initialLiveManifestSize: 1,
      maxBufferLength: 30,
      maxMaxBufferLength: 600,
      maxBufferSize: 60 * 1000 * 1000,
      maxBufferHole: 0.5,
      maxSeekHole: 2,
      lowBufferWatchdogPeriod: 0.5,
      highBufferWatchdogPeriod: 3,
      nudgeOffset: 0.1,
      nudgeMaxRetry: 3,
      maxFragLookUpTolerance: 0.2,
      liveSyncDurationCount: 3,
      liveMaxLatencyDurationCount: 10,
      enableWorker: true,
      enableSoftwareAES: true,
      manifestLoadingTimeOut: 10000,
      manifestLoadingMaxRetry: 1,
      manifestLoadingRetryDelay: 500,
      manifestLoadingMaxRetryTimeout: 64000,
      startLevel: undefined,
      levelLoadingTimeOut: 10000,
      levelLoadingMaxRetry: 4,
      levelLoadingRetryDelay: 500,
      levelLoadingMaxRetryTimeout: 64000,
      fragLoadingTimeOut: 20000,
      fragLoadingMaxRetry: 6,
      fragLoadingRetryDelay: 500,
      fragLoadingMaxRetryTimeout: 64000,
      startFragPrefetch: false,
      appendErrorMaxRetry: 3,

      enableCEA708Captions: true,
      stretchShortVideoTrack: false,
      forceKeyFrameOnDiscontinuity: true,
      abrEwmaFastLive: 5.0,
      abrEwmaSlowLive: 9.0,
      abrEwmaFastVoD: 4.0,
      abrEwmaSlowVoD: 15.0,
      abrEwmaDefaultEstimate: 500000,
      abrBandWidthFactor: 0.8,
      abrBandWidthUpFactor: 0.7,
      minAutoBitrate: 0
    }
    //获取Hls地址
    const getHlsUrl = () => {
      console.log('准备请求', props.gbcode, '号视频')
      axios
        .post('', {
          resource: {
            gbcode: props.gbcode
          },
          stream: {
            procotol: 'hls'
          }
        })
        .then((res) => {
          // console.log(res,"res");
          const resData = res.data
          console.log('当前第', props.id, '号', resData, 'resData')
          // res.data.rtn === '0' &&
          if (res.data.url) {
            // console.log(resData.url, 'resData.url')
            data.value.url = resData.url
            console.log(
              '当前第',
              props.id,
              '号视频链接',
              data.value.url,
              '获取到了'
            )
          }
          if (res.data.rtn !== '0' || res.data.url == '') {
            console.log('重新获取地址')
            getHlsUrl()
          }
          nextTick(() => {
            getStream(data.value.url)
          })
        })
        .catch((error) => {
          console.log(error, 'error')
          console.log('重新获取')
          getHlsUrl()
        })
    }
    //启动Hls流视频
    const getStream = (url) => {
      const video = document.getElementById(props.id)
      //const video_view = video.parentElement
      if (Hls.isSupported()) {
        //判断是不是支持hls
        const hls = new Hls() //初始化对象

        hls.loadSource(url) //加载资源

        hls.attachMedia(video) //渲染给video对象

        hls.on(Hls.Events.MANIFEST_PARSED, (event, data) => {
          video.classList.remove('error')
          video.nextSibling.textContent = ''
          video.classList.add('success')
          /*   
          console.log('直播成功', url) */
          playVideo(video)
        })
        hls.on(Hls.Events.ERROR, (event, data) => {
          //console.log(data)
          // 监听出错事件
          if (data.fatal) {
            video.classList.remove('success')
            video.nextSibling.textContent = '设备离线'
            video.classList.add('error')
            switch (data.type) {
              case Hls.ErrorTypes.NETWORK_ERROR:
                //尝试恢复网络错误
                console.log('遇到致命网络错误,请尝试恢复')
                console.log('准备恢复', props.id, '号视频', url)
                getHlsUrl()
                // hls.startLoad()
                //getStream(url)

                break
              case Hls.ErrorTypes.MEDIA_ERROR:
                console.log(data, '遇到媒体错误')
                console.log('遇到媒体错误,请尝试恢复')
                hls.recoverMediaError()
                break
              default:
                //无法恢复
                hls.destroy()
                break
            }
          }
        })
      } else if (
        video.canPlayType('application/vnd.apple.mpegurl') ||
        video.canPlayType('application/x-mpegURL')
      ) {
        video.src = url
        video.addEventListener('loadedmetadata', (event, data) => {
          console.log('点播成功', data)
          /*  console.log(
            'manifest loaded, found ' + data.levels.length + ' quality level',
            url
          ) */
          playVideo(video)
        })
      }
    }
    //播放
    const playVideo = (video) => {
      video.play()
    }
    // 停止流
    const videoPause = () => {
      if (this.hls) {
        this.$refs.video.pause()
        this.hls.destroy()
        this.hls = null
      }
    }
    onMounted(() => {
      getHlsUrl()
    })
    return {}
  }
}
</script>

<style lang="scss" scoped>
.video_view {
  position: relative;
}
.video_tip {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 100px;
  color: red;
}
.success {
  border-left: 10px solid greenyellow;
}
.error {
  border-left: 10px solid red;
  position: relative;
}
// .error::before {
//   position: absolute;
//   content: '设备离线';
//   left: 50%;
//   top: 50%;
//   font-size: 100px;
//   color: red;
//   z-index: 9;
// }
</style>
