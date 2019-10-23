<template>
  <div class="home">
    <div class='video-box'>
      <video id="yours" autoplay></video>
      <video id="theirs" autoplay></video>
    </div>
    <button @click="play">获取</button>
  </div>
</template>

<script>
// @ is an alias to /src
/* eslint-disable */
import { desktopCapturer } from 'electron'

export default {
  name: 'home',
  data () {
    return {
      yourConnection: null,
      theirConnection: null
    }
  },
  components: {
  },
  mounted () {
  },
  methods: {
    play () {
      var yourVideo = document.getElementById('yours')

      if (this.hasUserMedia()) {
        desktopCapturer.getSources({ types: ['window', 'screen'] }).then(async sources => {
          try {
            const stream = await navigator.mediaDevices.getUserMedia({
              audio: false,
              video: {
                mandatory: {
                  chromeMediaSource: 'desktop'
                }
              }
            })
            console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
            console.log('stream:', stream)
            console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
            yourVideo.srcObject = stream
            if (this.hasRTCPeerConnection()) {
              this.startPeerConnection(stream)
            } else {
              alert('没有RTCPeerConnection API')
            }
          } catch (e) {
            this.handleError(e)
          }
        })
      } else {
        alert('没有userMedia API')
      }
    },
    hasUserMedia () {
      navigator.getUserMedia = navigator.getUserMedia || navigator.msGetUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia
      return !!navigator.getUserMedia
    },
    hasRTCPeerConnection () {
      window.RTCPeerConnection = window.RTCPeerConnection || window.webkitRTCPeerConnection || window.mozRTCPeerConnection || window.msRTCPeerConnection
      return !!window.RTCPeerConnection
    },
    handleError (err) {
      console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
      console.log('err:', err)
      console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
    },
    startPeerConnection (stream) {
      var that = this
      // 这里使用了几个公共的stun协议服务器
      var config = {
        // 'iceServers': [{ 'url': 'stun:stun.services.mozilla.com' }, { 'url': 'stun:stunserver.org' }, { 'url': 'stun:stun.l.google.com:19302' }]
      }
      this.yourConnection = new RTCPeerConnection(config)
      this.theirConnection = new RTCPeerConnection(config)
      this.yourConnection.onicecandidate = function (e) {
        if (e.candidate) {
          that.theirConnection.addIceCandidate(new RTCIceCandidate(e.candidate))
        }
      }
      this.theirConnection.onicecandidate = function (e) {
        if (e.candidate) {
          that.yourConnection.addIceCandidate(new RTCIceCandidate(e.candidate))
        }
      }

      this.theirConnection.onaddstream = function(e) {
        var theirVideo = document.getElementById('theirs')
        console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
        console.log('e.stream:', e.stream)
        console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
        theirVideo.srcObject = e.stream
      }
      this.yourConnection.addStream(stream)
      //本方产生了一个offer
      this.yourConnection.createOffer().then(offer => {
        that.yourConnection.setLocalDescription(offer);
        //对方接收到这个offer
        that.theirConnection.setRemoteDescription(offer);
        console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
        console.log('offer:', offer)
        console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
        //对方产生一个answer
        that.theirConnection.createAnswer().then(answer => {
          that.theirConnection.setLocalDescription(answer);
          console.log('answer:', answer)
          //本方接收到一个answer
          that.yourConnection.setRemoteDescription(answer);
        })
      })
    }
  }
}
</script>

<style scoped>
.video-box {
  display: flex;
}
#theirs {
  width: 500px;
  border: 1px solid #108ee9;
}
#yours {
  width: 500px;
  border:1px solid #108ee9;
}
</style>
