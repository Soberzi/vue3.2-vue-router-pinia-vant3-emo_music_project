<script setup>
import FooterMusic from '@/components/item/FooterMusic.vue'
import { FooterMusicStore } from '@/store/FooterMusic.js'
const store = FooterMusicStore()

</script>

<template>
  <router-view />
  <FooterMusic v-show="store.isFooterMusic"></FooterMusic>
</template>

<style lang="less">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.icon {
  width: .5rem;
  height: .5rem;
}
a{
  color: black;
}
</style>
