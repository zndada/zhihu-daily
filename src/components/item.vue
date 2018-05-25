<template>
  <a class="daily-item">
    <div class="daily-img" v-if="data.image">
      <img :src="imgPath + data.images[0]">
    </div>
    <div class="daily-title" :class="{noImg: !data.image}">{{type}}{{data.title}}</div>
  </a>
</template>

<script>
import $ from '../libs/util.js'
export default {
  props: {
    data: {
      type: Object
    }
  },
  data () {
    return {
      imgPath: $.imgPath,
      msg:'陈佳锌是傻子'
    }
  }
}
</script>
