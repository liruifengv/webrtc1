<template>
  <div class="home">
    <h1>receiver</h1>
    <div class='video-box'>
      <div>
        <span>local</span>
        <video id="local" autoplay></video>
      </div>
      <div>
        <span>remote</span>
        <video id="remote" autoplay></video>
      </div>
    </div>
    <button @click="newPeer">call</button>
  </div>
</template>

<script>
// @ is an alias to /src
/* eslint-disable */
import { desktopCapturer } from 'electron'
import Peer from 'peerjs';

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
    newPeer () {
      const peer = new Peer('receiver', { host: 'localhost', port: 9000, path: '/' })
      peer.on('connection', (conn) => {
        console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
        console.log('conn:', conn)
        console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
        conn.on('data', (data) => {
          console.log(data);
        })
      })
      console.log('peer:', peer)
      peer.on('call', call => {
        console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
        console.log('call:', call)
        console.log('■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■')
        var yourVideo = document.getElementById('local')
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
              // yourVideo.srcObject = stream
              call.answer(stream)
              call.on('stream', remoteStream => {
                document.querySelector('video#remote').srcObject = remoteStream
              })
            } catch (e) {
              this.handleError(e)
            }
          })
        } else {
          alert('没有userMedia API')
        }
      })
    },
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
            call.answer(localStream)
            call.on('stream', remoteStream => {
              document.querySelector('video#remote').srcObject = remoteStream
            })
            // if (this.hasRTCPeerConnection()) {
            //   // 稍后我们实现 startPeerConnection
            //   this.startPeerConnection(stream)
            // } else {
            //   alert('没有RTCPeerConnection API')
            // }
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
.video-box div {
  display: flex;
  flex-direction: column;
}
#local {
  width: 500px;
  border: 1px solid #108ee9;
}
#remote {
  width: 500px;
  border:1px solid #108ee9;
}
</style>
