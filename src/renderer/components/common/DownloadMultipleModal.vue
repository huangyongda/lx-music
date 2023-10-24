<template>
  <material-modal :show="show" :bg-close="bgClose" :teleport="teleport" @close="handleClose">
    <main :class="$style.main">
      <h2>{{ $t('download__multiple_tip', { len: list.length }) }}<br>{{ $t('download__multiple_tip2') }}</h2>
      <base-btn :class="$style.btn" @click="downfromcacheservice()">直接在缓存服务器下载128K的mp3文件到本地</base-btn>
      <base-btn ref="downloadButton" :class="$style.btn" @click="handleClick('128k')">{{ $t('download__normal') }} - 128K</base-btn>
      <base-btn :class="$style.btn" @click="handleClick('320k')">{{ $t('download__high_quality') }} - 320K</base-btn>
      <base-btn :class="$style.btn" @click="handleClick('flac')">{{ $t('download__lossless') }} - FLAC</base-btn>
      <base-btn :class="$style.btn" @click="handleClick('flac24bit')">{{ $t('download__lossless') }} - FLAC Hires</base-btn>
    </main>
  </material-modal>
</template>

<script>
import { createDownloadTasks } from '@renderer/store/download/action'
// import { useRouter } from '@common/utils/vueRouter'
import { appSetting } from '@renderer/store/setting'
import axios from 'axios'
import fs from 'fs'
// import path from 'path'

export default {
  props: {
    show: {
      type: Boolean,
      default: false,
    },
    bgClose: {
      type: Boolean,
      default: true,
    },
    list: {
      type: Array,
      default() {
        return []
      },
    },
    teleport: {
      type: String,
      default: '#root',
    },
  },
  emits: ['update:show', 'confirm'],
  mounted() {
    // const router = useRouter()
    // this.$nextTick(() => {
    // console.log('点击128k')
    // console.log(this.list)
    // updateSetting({ 'download.savePath': '/Users/huangyd/Desktop/mu' })
    // console.log(appSetting)
    // console.log(appSetting['download.savePath'])
    // console.log('serveraddr')
    // console.log(appSetting['network.proxy.serveraddr'])
    //
    //
    // console.log('all download success')
    // this.handleClick('128k')
    // this.$emit('update:show', false)
    // void router.push({
    //   path: '/download',
    // })
    // })
  },
  methods: {
    handleClick(quality) {
      console.log(quality)
      console.log('click', this.list)
      // updateSetting({ 'download.savePath': '/Users/huangyd/Desktop/mu' })
      console.log(appSetting)
      console.log(appSetting['download.savePath'])
      void createDownloadTasks(this.list.filter(item => item.source != 'local'), quality)
      this.handleClose()
      this.$emit('confirm')
    },
    downfromcacheservice() {
      console.log('serveraddr:')
      console.log(appSetting['network.proxy.serveraddr'])
      if (appSetting['network.proxy.serveraddr'] == '') {
        alert('请先配置缓存服务器')
        return
      }
      axios.post(appSetting['network.proxy.serveraddr'] + '/index.php', { list: this.list, quality: '128k' })
        .then((response) => {
          this.remoteData = response.data
          console.log(response)
          const arr = response.data
          let len = arr.length
          let dowload_num = 0
          let wait_dowload_num = 0
          for (let i = 0; i < len; i++) {
            if (arr[i].url) {
              wait_dowload_num++
              axios.post(appSetting['network.proxy.serveraddr'] + '/index.php?s=home/index/getmp3base64&url=' + arr[i].url, { url: arr[i].url }).then((responseContent) => {
                console.log(responseContent)
                if (responseContent.data.base64_content) {
                  fs.writeFile(appSetting['download.savePath'] + '/' + arr[i].name, responseContent.data.base64_content, 'base64', (err) => {
                    if (err) {
                      console.error('Error saving the file:', err)
                    } else {
                      dowload_num++
                      console.log('File saved successfully! wait_dowload_num:' + wait_dowload_num)
                      console.log('File saved successfully! num:' + dowload_num)
                      if (dowload_num == wait_dowload_num) {
                        console.log('缓存服务器已经加载完毕:' + dowload_num)
                        alert('缓存服务器已经加载完毕 已加载个数:' + dowload_num)
                      }
                    }
                  })
                }
              }).catch((error) => {
                console.error('获取远程数据时出错:', error)
                this.dataLoaded = true // 加载失败后仍然标记数据已加载
              })
            }
          }
          if (dowload_num == 0 && dowload_num == wait_dowload_num) {
            console.log('缓存服务器已经加载完毕:' + dowload_num)
            alert('缓存服务器已经加载完毕已加载个数0')
          }
          this.dataLoaded = true
        })
        .catch((error) => {
          alert('缓存服务器加载失败')
          console.error('获取远程数据时出错:', error)
          this.dataLoaded = true // 加载失败后仍然标记数据已加载
        })
    },
    handleClose() {
      this.$emit('update:show', false)
    },
  },
}

</script>


<style lang="less" module>
@import '@renderer/assets/styles/layout.less';

.main {
  padding: 15px;
  max-width: 400px;
  min-width: 200px;
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  h2 {
    font-size: 13px;
    color: var(--color-font);
    line-height: 1.3;
    text-align: center;
    margin-bottom: 15px;
  }
}

.btn {
  display: block;
  margin-bottom: 15px;
  &:last-child {
    margin-bottom: 0;
  }
}

</style>
