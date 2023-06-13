<script lang="ts" setup>
import axios from 'axios'

const url = ref('webrtc://101.34.3.87/live/1686638972318')
const webrtcRef = ref<HTMLVideoElement | null>(null)

let rtcPeerConnection: RTCPeerConnection
let offer: RTCSessionDescriptionInit

onMounted(async () => {
  rtcPeerConnection = new RTCPeerConnection(undefined)

  rtcPeerConnection.ontrack = function (event) {
    webrtcRef.value && (webrtcRef.value.srcObject = event.streams[0])
  }

  rtcPeerConnection.addTransceiver('audio', {
    direction: 'recvonly',
  })

  rtcPeerConnection.addTransceiver('video', {
    direction: 'recvonly',
  })

  offer = await rtcPeerConnection.createOffer()
  await rtcPeerConnection.setLocalDescription(offer)
})

onUnmounted(() => {
  disconnect()
})

async function disconnect() {
  webrtcRef.value?.pause()
  rtcPeerConnection.close()
}

async function connect() {
  const api = 'http://101.34.3.87:1985/rtc/v1/play/'

  const clientip = null

  const res = await axios.post(api, {
    api,
    streamurl: url.value,
    clientip,
    sdp: offer.sdp,
  })

  rtcPeerConnection.setRemoteDescription(new RTCSessionDescription({
    type: 'answer',
    sdp: res.data.sdp,
  }))

  rtcPeerConnection.addEventListener('connectionstatechange', () => {
    if (rtcPeerConnection.connectionState === 'connected')
      webrtcRef.value?.play()
  })
}
</script>

<template>
  <main font-sans p="x-4 y-10" text="center gray-700 dark:gray-200">
    <div>
      <TheInput v-model="url" />
      <button
        p="x-4 y-2"
        text="center"
        mx-2
        bg="transparent"
        border="~ rounded gray-200 dark:gray-700"
        hover:bg-gray
        @click="connect"
      >
        Connect
      </button>
      <button
        p="x-4 y-2"
        text="center"
        bg="transparent"
        border="~ rounded gray-200 dark:gray-700"
        hover:bg-gray
        @click="disconnect"
      >
        Disconnect
      </button>
    </div>
    <div>
      <div ma mt10 h-70 w-120 bg-black>
        <video ref="webrtcRef" controls ma h-70 />
      </div>
    </div>
    <TheFooter />
  </main>
</template>
