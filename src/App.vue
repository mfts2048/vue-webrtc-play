<script lang="ts" setup>
import axios from 'axios'

const failUrl = ref('webrtc://101.34.3.87/live/1686990987248')
const webrtcRef = ref<HTMLVideoElement | null>(null)
const message = ref<string[]>([])

let rtcPeerConnection: RTCPeerConnection
let offer: RTCSessionDescriptionInit

onMounted(async () => {
  webrtcRef.value?.addEventListener('play', () => {
    message.value.push('视频已被触发播放')
  })
  webrtcRef.value?.addEventListener('loadedmetadata', () => {
    message.value.push('已加载元数据。')
  })
  webrtcRef.value?.addEventListener('loadeddata', () => {
    message.value.push('media 中的首帧已经完成加载。')
  })
  webrtcRef.value?.addEventListener('playing', () => {
    message.value.push('由于缺乏数据而暂停或延迟后，播放准备开始。')
  })
})

onUnmounted(() => {
  disconnect()
})

async function disconnect() {
  message.value.push('连接断开')
  webrtcRef.value?.pause()
  rtcPeerConnection?.close()
}

let playPromise: Promise<any>

async function connect(streamurl: string) {
  if (!webrtcRef.value)
    return

  webrtcRef.value.load()

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

  message.value.push('正在准备连接...')
  const api = 'http://101.34.3.87:1985/rtc/v1/play/'

  const clientip = null

  const res = await axios.post(api, {
    api,
    streamurl,
    clientip,
    sdp: offer.sdp,
  })

  // rtcPeerConnection.Remo/

  rtcPeerConnection.setRemoteDescription(new RTCSessionDescription({
    type: 'answer',
    sdp: res.data.sdp,
  }))

  rtcPeerConnection.addEventListener('connectionstatechange', () => {
    if (!webrtcRef.value)
      return

    if (rtcPeerConnection.connectionState === 'connected') {
      message.value.push('webrtc 连接成功，正在缓冲加载中')

      webrtcRef.value?.load()
      playPromise = webrtcRef.value.play()
      playPromise.then(() => {
        message.value.push('视频加载成功')
      }).catch((err) => {
        message.value.push('视频加载过程中被中断')
        console.error(err)
      })
    }
  })
}
</script>

<template>
  <main font-sans p="x-4 y-10" text="center gray-700 dark:gray-200">
    <div>
      <TheInput v-model="failUrl" />
      <button
        p="x-4 y-2"
        text="center"
        mx-2
        bg="transparent"
        border="~ rounded gray-200 dark:gray-700"
        hover:bg-gray
        @click="connect(failUrl)"
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
    <ol mt-5>
      <li v-for="(el, index) in message" :key="index" text-2>
        <span>{{ index + 1 }}</span> = {{ el }}
      </li>
    </ol>
    <div>
      <div ma mt10 h-70 w-120 bg-black>
        <video ref="webrtcRef" controls ma h-full w-full />
      </div>
    </div>
    <TheFooter />
  </main>
</template>
