<script setup lang="ts">
import 'uno.css'

import { onMessage, sendMessage } from 'webext-bridge/content-script'

const copy = (str) => {
  let textarea = null
  !textarea && (textarea = document.createElement('textarea'))
  // console.log(res)
  textarea.value = str
  textarea.style.position = 'fixed'
  textarea.style.top = 0
  textarea.style.left = 0
  textarea.style.opacity = 0
  document.body.appendChild(textarea)
  textarea.select()
  document.execCommand('copy')
  document.body.removeChild(textarea)
}

onMessage('content-scipt<=background', async (msg) => {
  const { data } = msg
  copy(data.str)
  return { cool: 'Got you popup! I am cool from content-script' }
})
const getDom = async () => {
  const a = {}
  document.querySelectorAll('.td-subject >a').forEach((item) => {
    const title = item.title
    const badKeyList = ['精油', '精华', '香水', '车走', '面霜', '申删', '母婴', '隔离', '美瞳']
    if (title.includes('作业') && !badKeyList.some(key => title.includes(key)))
      a[`${Object.keys(a).length + 1}. ${title}`] = item.href
  })
  sendMessage('content-scipt=>background', a, 'background')
}
</script>

<template>
  <div class="flex font-sans m-5 top-0 left-0 leading-1em z-100 fixed items-end select-none">
    <div
      class="bg-white rounded-lg h-min shadow w-max text-gray-800"
      p="x-4 y-2"
      m="y-auto r-2"
      transition="opacity duration-300"
    >
      <button @click="getDom">
        get DOM
      </button>
    </div>
  </div>
</template>
