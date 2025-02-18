<script setup lang="ts">
import { ref, watch } from 'vue'
import type { UploadFileInfo } from 'naive-ui'
import { NButton, NCard, NColorPicker, NInput, NPopconfirm, NSelect, NSlider, NSwitch, NUpload, NUploadDragger, useMessage } from 'naive-ui'
import { useAuthStore, usePanelState } from '@/store'
import { set as setUserConfig } from '@/api/panel/userConfig'

const authStore = useAuthStore()
const panelState = usePanelState()
const ms = useMessage()

const isSaveing = ref(false)

const iconTypeOptions = [
  {
    label: '详情图标',
    value: 0,
  },
  {
    label: '小图标',
    value: 1,
  },
]

watch(panelState.panelConfig, () => {
  if (!isSaveing.value) {
    isSaveing.value = true

    setTimeout(() => {
      panelState.recordState()// 本地记录
      setUserConfig({ panel: panelState.panelConfig }).then((res) => {
        if (res.code === 0)
          ms.success('配置已同步到云端')
        else
          ms.error(`配置同步到云端失败${res.msg}`)
        isSaveing.value = false
      })
    }, 1000)
  }
})

function handleUploadBackgroundFinish({
  file,
  event,
}: {
  file: UploadFileInfo
  event?: ProgressEvent
}) {
  const res = JSON.parse((event?.target as XMLHttpRequest).response)
  panelState.panelConfig.backgroundImageSrc = res.data.imageUrl
  return file
}

function uploadCloud() {
  setUserConfig({ panel: panelState.panelConfig }).then((res) => {
    if (res.code === 0)
      ms.success('配置已同步到云端')
    else
      ms.error(`配置同步到云端失败${res.msg}`)
  })
}

function resetPanelConfig() {
  panelState.resetPanelConfig()
  uploadCloud()
}
</script>

<template>
  <div class="bg-slate-200 rounded-[10px] p-[8px] h-[500px] overflow-auto">
    <NCard style="border-radius:10px" size="small">
      <div class="text-slate-500 mb-[5px]">
        LOGO
      </div>
      <NInput v-model:value="panelState.panelConfig.logoText" type="text" show-count :maxlength="20" placeholder="请输入文字" />
    </NCard>

    <NCard style="border-radius:10px" class="mt-[10px]" size="small">
      <div class="text-slate-500 mb-[5px]">
        时钟
      </div>
      <div class="flex items-center mt-[5px]">
        <span class="mr-[10px]">显示秒</span>
        <NSwitch v-model:value="panelState.panelConfig.clockShowSecond" />
      </div>
    </NCard>

    <NCard style="border-radius:10px" class="mt-[10px]" size="small">
      <div class="text-slate-500 mb-[5px]">
        壁纸
      </div>
      <NUpload
        action="/api/file/uploadImg"
        :show-file-list="false"
        name="imgfile"
        :headers="{
          token: authStore.token as string,
        }"
        :directory-dnd="true"
        @finish="handleUploadBackgroundFinish"
      >
        <NUploadDragger>
          <div
            class="h-[150px] w-[280px] border bg-slate-100 flex justify-center items-center cursor-pointer rounded-[10px]"
            :style="{ background: `url(${panelState.panelConfig.backgroundImageSrc}) no-repeat`, backgroundSize: 'cover' }"
          >
            <div class="text-shadow text-white">
              点击上传替换图片或拖拽到框内
            </div>
          </div>
        </NUploadDragger>
      </NUpload>

      <div class="flex items-center mt-[5px]">
        <span class="mr-[10px]">模糊处理</span>
        <NSlider v-model:value="panelState.panelConfig.backgroundBlur" class="max-w-[200px]" :step="2" :max="20" />
      </div>
    </NCard>

    <NCard style="border-radius:10px" class="mt-[10px]" size="small">
      <div class="text-slate-500 mb-[5px]">
        图标
      </div>
      <div>
        样式
      </div>
      <div class="flex items-center mt-[5px]">
        <NSelect v-model:value="panelState.panelConfig.iconStyle" :options="iconTypeOptions" />
      </div>
      <div>
        文字颜色
      </div>
      <div class="flex items-center mt-[5px]">
        <NColorPicker
          v-model:value="panelState.panelConfig.iconTextColor"
          :show-alpha="false"
          size="small"
          :modes="['hex']"
          :swatches="[
            '#000000',
            '#ffffff',
            '#18A058',
            '#2080F0',
            '#F0A020',
          ]"
        />
      </div>
    </NCard>

    <NCard style="border-radius:10px" class="mt-[10px]" size="small">
      <NPopconfirm
        @positive-click="resetPanelConfig"
      >
        <template #trigger>
          <NButton size="small" quaternary type="error">
            重置
          </NButton>
        </template>
        确定要重置这些样式吗？
      </NPopconfirm>

      <NButton size="small" quaternary type="success" class="ml-[10px]" @click="uploadCloud">
        立即同步到云端
      </NButton>
    </NCard>
  </div>
</template>

<style scoped>
.text-shadow{
  text-shadow: 0px 0px 5px gray;
}
</style>
