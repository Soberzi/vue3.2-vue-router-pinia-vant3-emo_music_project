# emo_music移动端
vue3.2 + vue-router + pinia + vant
基于vue-cli创建
# 安装pinia
npm install pinia -S
## 创建store
import { defineStore } from 'pinia'
export const useStore = defineStore('main', {
  state: () => ({
  }),
  getters: {
  },
  actions: {
  }
})

# rem移动适配
## 1.在public存放静态资源新建js文件夹,在js文件夹创建rem.js实现移动适配布局:
function remSize () {
  /* 获取设备宽度 */
  let deviceWith = document.documentElement.clientWidth || window.innerWidth
  /* 设计稿宽度 */
  if (deviceWith >= 750) {
    deviceWith = 750
  }
  if (deviceWith <= 320) {
    deviceWith = 320
  }
  /* 设置rem，
  750px--> 1rem=100px
  375px--> 1rem=50px
  */
  document.documentElement.style.fontSize = (deviceWith / 7.5) + 'px'
  /* 设置字体大小 */
  document.querySelector('body').style.fontSize = 0.3 + 'rem'
}
remSize()
/* 当窗口发生变化 */
window.onresize = function () {
  remSize()
}
## 2.在index.html引入
 <div id="app"></div>
    <!-- 引入rem.js，<%= BASE_URL %>这个为基础路径 -->
    <script src="<%= BASE_URL %>js/rem.js"></script>


# 阿里图标引入
## 1.在官网添加项目后复制symbol代码在index.html引入
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <!-- 引入阿里图标 -->
    <script src="//at.alicdn.com/t/font_3415587_mpuudaajazg.js"></script>
## 2.使用svg
在官网https://www.iconfont.cn/manage/index?manage_type=myprojects&projectId=3415587
查看使用帮助，这里用到的是symbol
    <svg class="icon" aria-hidden="true">
      <use xlink:href="#icon-liebiao2"></use>
    </svg>
注意：类名要加#号
## 3.初始化样式，设置图标大小
symbol是设置width和height，而font class是设置fontsize
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.icon {
  width: .4rem;
  height: .4rem;
}
## 4.可以用px to rem 插件进行px和rem转换
需要对插件进行配置，基准font-size这里设置为50
那就是1rem=50px


# 完成首页页面
在component下创建home文件夹

## 头部导航组件TopNav.vue
在home文件夹创建TopNav.vue
<template>
  <div class="topNav">
    <div class="topLeft">
      <svg class="icon" aria-hidden="true">
        <use xlink:href="#icon-liebiao2"></use>
      </svg>
    </div>
    <div class="topContent">
      <span>我的</span>
      <span class="active">发现</span>
      <span>云村</span>
      <span>视频</span>
    </div>
    <div class="topRight">
      <svg class="icon" aria-hidden="true">
        <use xlink:href="#icon-sousuo"></use>
      </svg>
    </div>
  </div>
</template>

<style lang='less' scoped>
.topNav {
  width: 100%;
  height: 1rem;
  padding: 0.2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;

  .topContent {
    width: 65%;
    height: 100%;
    display: flex;
    justify-content: space-around;
    font-size: .36rem;

    .active {
      font-weight: 900;
    }
  }
}
</style>

## 使用vant组件库
官网：https://vant-contrib.gitee.io/vant/#/zh-CN
### 1.安装vant3组件
npm i vant
### 2.引入vant(自动按需引入，安装插件)
参照官网步骤
npm i babel-plugin-import -D
在.babelrc 或 babel.config.js 中添加配置
### 3.测试按钮（在main.js进行注册）
import { Button } from 'vant';
app.use(Button);

## 轮播图组件SwiperTop.vue
### 1.对引入vant组件库进行集中管理(封装)
在src目录创建plugins文件夹
在plugins文件夹创建index.js：
import { Swipe, SwipeItem, Button } from 'vant'
/* 放入数组中 */
const plugins = [
  Swipe, SwipeItem, Button
]

/* 循环将每一个插件注册到app上，main.js直接调用这个方法即可 */
export default function getVant (app) {
  plugins.forEach((item) => {
    return app.use(item)
  })
}

### 2.在main.js引入调用方法即可
import getVant from './plugins'
const app = createApp(App)
getVant(app)
### 3.使用SwiperTop组件
vant官网复制懒加载，调整样式

### 4.安装axios请求轮播图数据进行渲染
npm i axios -S
官方文档：http://www.axios-js.com/zh-cn/docs/

### 5.对axios进行封装
#### 创建utils文件夹，在其文件夹下创建request.js用于封装请求根路径：
import axios from 'axios'
const service = axios.create({
  baseURL: 'http://localhost:3000',
  timeout: 3000
})
export default service

#### 创建api文件夹，在其文件夹下创建homeApi.js模块对api接口进行封装：
import service from '@/utils/request.js'
/* 获取首页轮播图数据 */
/* 第一种写法
 export function getBanner () {
  return service({
    method: 'GET',
    url: '/banner?type=2'
  })
} */
// 第二种写法
export function getBanner () {
  return service.get('/banner?type=2')
}

## 图标列表组件IconList.vue
这里用到flex布局，修改主轴对齐方向
.iconList {
  width: 100%;
  height: 2rem;
  margin-top: .2rem;
  display: flex;
  justify-content: space-around;
  align-items: center;

  .iconItem {
    width: 25%;
    height: 100%;
    display: flex;
    /* 修改主轴对齐方向 */
    flex-direction: column;
    align-items: center;

    .icon {
      width: 1rem;
      height: 1rem;
    }
  }
}

## 发现好歌单组件MusicList.vue
1.先在homeApi.js封装获取好歌单列表的请求方法
2.用vue2语法调用请求拿到数据
3.用vant组件库轮播图自定义滑块大小
4.这里的播放量用到了函数进行处理：
 // 对播放量进行处理
    const changeCount = function (num) {
      if (num >= 100000000) {
        // toFixed(1)显示一位小数
        return (num / 100000000).toFixed(1) + '亿'
      } else if (num >= 10000) {
        return (num / 10000).toFixed(1) + '万'
      }
    }
5.文本超出显示省略号css写法：
 text-overflow: ellipsis;

## 点击歌单跳转路由
创建路由组件ItemMusic并在router进行配置路径
### 在点歌单进行routerlink包裹：(路由传参)
1.这里需要通过路由传参
<van-swipe-item v-for="item in musicList" :key="item.id">
  <router-link :to="{ path:'/itemMusic',query:{id:item.id} }">
    <img :src="item.picUrl">
    <span class="playCount">
      <svg class="icon" aria-hidden="true">
        <use xlink:href="#icon-a-24gl-play1"></use>
      </svg>
      {{ changeCount(item.playCount) }}
    </span>
    <span class="name">{{ item.name }}</span>
  </router-link>
</van-swipe-item>

# 歌单详情页
## 父组件路由组件ItemMusic.vue
### 1.需要先拿到路由传过来的参数id
import { useRoute } from 'vue-router'
import { onMounted } from 'vue'

// useRoute可以拿到路由的参数
onMounted(() => {
  /* 可以调用useRoute方法的query拿到id */
  const id = useRoute().query.id
  console.log(id)
})

### 2.在api封装歌单详情页请求的方法,itemApi.js:
import service from '@/utils/request.js'

/* 获取歌单详情页的数据 */
export function getMusicItem (data) {
  return service({
    method: 'GET',
    /* 这里用到模板字符串，将data参数传进来 */
    url: `/playlist/detail?id=${data}`
  })
}

### 3.获取数据
import { useRoute } from 'vue-router'
import { onMounted, reactive, toRefs } from 'vue'
import { getMusicItem } from '@/api/itemApi.js'

const state = reactive({
  playlist: {}
})

// useRoute可以拿到路由的参数
onMounted(async () => {
  /* 可以调用useRoute方法的query拿到id */
  const id = useRoute().query.id
  // console.log(id)
  const res = await getMusicItem(id)
  // console.log(res)
  state.playlist = res.data.playlist
})

const { playlist } = toRefs(state)

## 详情页子组件（两个组件）
## 子组件itemMusicTop.vue组件，详情页头部
这里用到父传子，将父组件获取到的数据传给子组件（props传参）
### 1.props将数据传给子组件：
子组件：
import { defineProps } from 'vue'
const props = defineProps(['playlist'])
注意：为防止页面刷新，数据丢失，将数据保存到sessionStorage
### 2.将数据保存到本地sessionStorage
父组件：
// useRoute可以拿到路由的参数
onMounted(async () => {
  /* 可以调用useRoute方法的query拿到id */
  const id = useRoute().query.id
  // console.log(id)
  const res = await getMusicItem(id)
  // console.log(res)
  state.playlist = res.data.playlist
  /* 为防止页面刷新，数据丢失，将数据保存到sessionStorage */
  sessionStorage.setItem('itemDetail', JSON.stringify(state))
})
### 3.子组件进行使用props值
注意：当传过来的层级较深，子组件无法读取到，需要从本地进行读取，可以转存到一个数据进行存储使用
import { reactive } from 'vue'
export default {
  props: ['playlist'],
  setup (props) {
    // 通过props进行传值，判断如果数据拿不到，则从本地读取数据
    let creator = reactive({})

    if ((props.playlist.creator === '')) {
      creator = JSON.parse(sessionStorage.getItem('itemDetail')).playlist.creator
    }
    // 对播放量进行处理
    const changeCount = (num) => {
      if (num >= 100000000) {
        // toFixed(1)显示一位小数
        return (num / 100000000).toFixed(1) + '亿'
      } else if (num >= 10000) {
        return (num / 10000).toFixed(1) + '万'
      }
    }
    return {
      props,
      creator,
      changeCount
    }
  }


## 子组件itemMusicList.vue组件,歌单歌曲列表组件
同理用父传子，将父组件获取的数据传给子组件（props传参）
父组件：
<script setup>
import { useRoute } from 'vue-router'
import { onMounted, reactive } from 'vue'
import { getMusicItem, getMusicItemList } from '@/api/itemApi.js'
/* 引入子组件 */
import ItemMusicTop from '@/components/item/itemMusicTop.vue'
import ItemMusicList from '@/components/item/itemMusicList.vue'

const state = reactive({
  playlist: {}, // 歌单详情页数据
  itemList: []// 歌单的歌曲
})

// useRoute可以拿到路由的参数
onMounted(async () => {
  /* 可以调用useRoute方法的query拿到id */
  const id = useRoute().query.id
  // console.log(id)
  /* 获取歌单详情 */
  const res = await getMusicItem(id)
  // console.log(res)
  state.playlist = res.data.playlist
  /* 获取歌单歌曲 */
  const result = await getMusicItemList(id)
  // console.log(result)
  state.itemList = result.data.songs
  /* 为防止页面刷新，数据丢失，将数据保存到sessionStorage */
  sessionStorage.setItem('itemDetail', JSON.stringify(state))
})

</script>

<template>
  <ItemMusicTop :playlist="state.playlist"></ItemMusicTop>
  <ItemMusicList :itemList="state.itemList" :subscribedCount="state.playlist.subscribedCount"></ItemMusicList>
</template>
子组件：
import { defineProps } from 'vue'
const props = defineProps(['itemList', 'subscribedCount'])
console.log(props)

# 底部组件FooterMusic.vue（全局，数据共享使用pinia）
在App.vue引入该组件
## 1.在store定义初始数据
  state: () => {
    return {
      playlist: [{ // 播放列表
        al: {
          id: 127728498,
          name: '嘉宾',
          pic: 109951165994864000,
          picUrl: 'https://p2.music.126.net/LoVqPgkI7DwSNZk50gcODg==/109951165994863998.jpg',
          pic_str: '109951165994863998'
        },
        id: 1846489646
      }],
      playListIndex: 0, // 默认下标为0
      isbtnShow: true// 暂停按钮的显示
    }
  },
  getters: {

  },
  actions: {
    // 修改暂停按钮的显示
    updateIsbtnShow (value) {
      this.isbtnShow = value
    },
}
## 2.实现假数据播放和暂停音乐
在渲染数据时注意store的使用
  <div class="FooterMusic">
    <div class="fouterLeft">
      <img :src="store.playlist[store.
      playListIndex].al.picUrl">
      <div>
        <p>{{store.playlist[store.
      playListIndex].al.name}}</p>
        <span>横滑切换上下首</span>
      </div>
    </div>
    <div class="fouterRight">
      <svg class="icon" aria-hidden="true" v-if="store.isbtnShow" @click="play">
        <use xlink:href="#icon-24gl-playCircle"   ></use>
      </svg>
      <svg class="icon" aria-hidden="true" v-else  @click="play">
        <use xlink:href="#icon-zanting"></use>
      </svg>
      <svg class="icon" aria-hidden="true">
        <use xlink:href="#icon-liebiao1"></use>
      </svg>
    </div>
    <audio ref="audio" :src="`https://music.163.com/song/media/outer/url?id=${store.playlist[store.
      playListIndex].id}.mp3`"></audio>
这里用到元素属性获取，即vue2的this.$refs来调用audio的方法
<script setup>
import { FooterMusicStore } from '@/store/FooterMusic.js'
import { ref, onMounted } from 'vue'
const store = FooterMusicStore()

/* vue3中this.$ref的使用变化 */
const audio = ref(null)
onMounted(() => {
  console.log(audio)
})

const play = () => {
  /* 判断是否已暂停 */
  if (audio.value.paused) {
  /* 调用audio的播放功能方法 */
    audio.value.play()
    // 让其显示播放按钮
    store.updateIsbtnShow(false)
  } else {
    /* 调用暂停方法 */
    audio.value.pause()
    // 让其隐藏播放按钮
    store.updateIsbtnShow(true)
  }
}
</script>

## 3.实现下一首上一首和点击音乐播放真数据的功能
先在store定义修改数据方法
    // 修改歌曲信息
    updatePlayList (value) {
      this.playlist = value
      console.log(this.playlist)
    },
    // 修改歌曲数组索引精确到哪首歌去
    updateplayListIndex (value) {
      this.playListIndex = value
    }
### 歌单列表点击歌曲播放
在itemMusicList.vue即歌单列表歌曲组件进行传参
import { defineProps } from 'vue'
import { FooterMusicStore } from '@/store/FooterMusic.js'
const store = FooterMusicStore()
const props = defineProps(['itemList', 'subscribedCount'])
console.log(props)
/* 点击播放音乐将音乐列表数据和点击的歌曲索引传进去 */
const playMusic = (index) => {
  store.updatePlayList(props.itemList)
  store.updateplayListIndex(index)
}

### 实现点击哪首歌，歌的信息同步到底部组件中
<script setup>
import { FooterMusicStore } from '@/store/FooterMusic.js'
import { ref, onMounted, watch } from 'vue'
const store = FooterMusicStore()

/* vue3中this.$ref的使用变化 */
const audio = ref(null)
onMounted(() => {
  // console.log(audio)
})

const play = () => {
  /* 判断是否已暂停 */
  if (audio.value.paused) {
  /* 调用audio的播放功能方法 */
    audio.value.play()
    // 让其显示播放按钮
    store.updateIsbtnShow(false)
  } else {
    /* 调用暂停方法 */
    audio.value.pause()
    // 让其隐藏播放按钮
    store.updateIsbtnShow(true)
  }
}

/* 监听下标，如果下标改变就自动播放音乐 */
watch(() => store.playListIndex, () => {
  audio.value.autoplay = true
  if (audio.value.paused) {
    // 如果是暂停状态，需要改变图标显示
    store.updateIsbtnShow(false)
  }
})

/* 监听数据变化，解决点击第一首可以播放这个bug */
watch(() => store.playlist, () => {
  if (store.isbtnShow) {
    audio.value.autoplay = true
    if (audio.value.paused) {
    // 如果是暂停状态，需要改变图标显示
      store.updateIsbtnShow(false)
    }
  }
})
</script>

# 歌曲播放详情页
在底部组件使用vant组件库的弹出层，这样就可以覆盖底部全局组件：
    <van-popup v-model:show="store.detailShow" position="right" :style="{ height: '100%', width: '100%' }">
      内容</van-popup>>

## 将歌曲播放详情页需要的数据通过父组件进行传值
父组件：
 <MusicDetail :musicList="store.playlist[store.
      playListIndex]"></MusicDetail>
子组件：
<script setup>
import { defineProps } from 'vue'
const props = defineProps(['musicList'])
console.log(props)
</script>

## 使用vue3marquee实现歌名走马灯效果
https://www.npmjs.com/package/vue3-marquee
npm install vue3-marquee@latest --save

## 磁盘效果
去网易云官网查找network请求过来的图片资源进行下载
