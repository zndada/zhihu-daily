<template>
  <!-- 接收唯一的一个prop:id,也就是文章的id，如果id变化了，就说明切换了文章，需要请求新的文章内容 -->
  <div class="daily-article">
    <div class="daily-article-title">{{ data.title }}</div>
    <div class="daily-article-content" v-html="data.body"></div>
    <div class="daily-comments" v-show="comments.length">
      <span>评论（{{ comments.length }}）</span>
      <div class="daily-comment" v-for="comment in comments" :key="comment.id">
        <div class="daily-comment-avatar">
          <img :src="comment.avatar">
        </div>
        <div class="daily-comment-content">
          <div class="daily-comment-name">
            {{ comment.author}}
          </div>
          <div class="daily-comment-time" v-time="comment.time"></div>
          <div class="daily-comment-text">{{ comment.content }}</div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import Time from '../directives/time.js'
import $ from '../libs/util.js'
export default {
  props: {
    id: {
      type: Number,
      default: 0
    }
  },
  directives: {
    Time
  },
  data () {
    return {
      data: {},
      comments: []
    }
  },
  methods: {
    getArticle () {
      // 获取具体文章内容
      $.ajax.get('news/' + this.id).then(res => {
        // 将文章中的图片地址替换为代理的地址
        res.body = res.body.replace(/src="http/g, 'src="' + $.imgPath + 'http')
        res.body = res.body.replace(/src="https/g, 'src="' + $.imgPath + 'https')
        this.data = res
        // 返回文章顶端
        window.scrollTo(0, 0)
        this.getComments()
      })
    },
    getComments () {
      // 获取评论
      this.comments = []
      $.ajax.get('story/' + this.id + '/short-comments').then(res => {
        this.comments = res.comments.map(comment => {
          // 将头像的图片地址转为代理地址
          comment.avatar = $.imgPath + comment.avatar
          return comment
        })
      })
    }
  },
  watch: {
    id (val) {
      if (val) {
        this.getArticle()
      }
    }
  }
}
</script>
