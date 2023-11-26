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

const toastMsg = ref('')

onMessage('content-scipt<=background', async (msg) => {
  const { data } = msg
  copy(data.str)
  toastMsg.value = '🎉 已复制列表到剪贴板'
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

onMounted(() => {
  // 判断详情页，自动remove无效评论 - 更适合油猴插件
  let count = 0
  if (location.pathname.includes('/group/topic/')) {
    // 自动跳转评论最后一页
    if (!location.search.includes('start=')) {
      // 获取class为paginator的div
      const divPaginator = document.querySelector('.paginator')
      if (divPaginator) {
        // 获取div下的所有a标签
        const aTags = divPaginator.getElementsByTagName('a')
        // 获取最后一个a标签
        const lastATag = aTags[aTags.length - 2]
        // 触发a标签的点击事件
        lastATag.click()
        return
      }
    }

    // 移除无效评论dom
    document.querySelectorAll('.reply-content').forEach((item) => {
      const filterText = /(d{2,})|谢谢姐妹|滴滴|谢谢|!|！|\s/gi // 过滤2个以上的d和谢谢
      const content = item.innerText.replace(filterText, '')
      if (!content || ['d', 'D', '牛', '，'].includes(content)) {
        count++
        item.parentElement?.parentElement?.remove()
      }
      else { item.innerText = content }
    })
    toastMsg.value = `✨ 已移除无效评论${count}条`
  }
})
</script>

<template>
  <div class="flex font-sans m-5 bottom-10 right-0 leading-1em z-100 fixed items-end select-none">
    <div
      class="bg-white rounded-lg h-min shadow w-max text-gray-800"
      p="x-4 y-2"
      m="y-auto r-2"
      transition="opacity duration-300"
    >
      <p v-if="toastMsg">
        {{ toastMsg }}
      </p>
      <button @click="getDom">
        get DOM
      </button>
    </div>
  </div>
</template>
