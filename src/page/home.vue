<template>
  <div class="daily">
    <div class="daily-menu">
      <div class="daily-menu-item" :class="{ on: type === 'recommend'}" @click="handleToRecommend">每日推荐</div>
      <div class="daily-menu-item" :class="{ on: type === 'daily'}" @click="showThemes = !showThemes">主题日报</div>
      <ul v-show="showThemes">
        <li v-for="item in themes" :key="item.id">
          <a :class="{ on: item.id === themeId && type === 'daily'}" @click="handleToTheme(item.id)">
            {{ item.name }}
          </a>
        </li>
      </ul>
    </div>
    <div class="daily-list" ref="list" @scroll="handleScroll">
      <template v-if="type === 'recommend'">
        <div v-for="list in recommendList" :key="list.date">
          <div class="daily-date">{{formatDay(list.date)}}</div>
          <!-- 绑定原生事件时要带事件修饰符.native，否则会认为监听的是来自Item组件的自定义事件click -->
          <Item v-for="item in list.stories" :data="item" :key="item.id" @click.native="handleClick(item.id)"></Item>
        </div>
      </template>
      <template v-if="type === 'daily'">
        <Item v-for="item in list" :data="item" :key="item.id" @click.native="handleClick(item.id)"></Item>
      </template>
    </div>
    <daily-article :id="articleId"></daily-article>
  </div>
</template>

<script>
import $ from '../libs/util.js'
import Item from '../components/item'
import dailyArticle from '../components/daily-article'
export default {
  data () {
    return {
      themes: [], // 主题日报的子类
      showThemes: false, // 判断是否展示主题日报的子类，默认不展示
      type: 'recommend', // 默认每日推荐
      recommendList: [], // 每日推荐文章列表的数据
      dailyTime: $.getTodayTime(),
      isLoading: false,
      list: [], // 主题日报的文章列表数据
      themeId: 0, // 主题的id
      articleId: 0 // 文章的id
    }
  },
  components: {
    Item,
    dailyArticle
  },
  methods: {
    getThemes () {
      // axios 发起get请求
      $.ajax.get('themes').then(res => {
        this.themes = res.others
      })
    },
    // 获取主题日报某类型主题的文章
    handleToTheme (id) {
      // 改变菜单分类
      this.type = 'daily'
      // 设置当前点击子类的主题日报id
      this.themeId = id
      // 清空中间栏的数据
      this.list = []
      $.ajax.get('theme/' + id).then(res => {
        // 过滤掉类型为1的文章，该类型下的文章为空
        this.list = res.stories.filter(item => item.type !== 1)
      })
    },
    // 切换子类需要重新获取一遍数据
    // 因此每次都需要清空列表并重新设置dailyTime
    handleToRecommend () {
      this.type = 'recommend'
      this.recommendList = []
      // 获取今天0点的时间戳
      this.dailyTime = $.getTodayTime()
      this.getRecommendList()
    },
    // 获取每日推荐文章列表的数据
    getRecommendList () {
      // 加载时设置为true，加载完成后置为false
      this.isLoading = true
      const prevDay = $.prevDay(this.dailyTime + 86400000)
      $.ajax.get('news/before/' + prevDay).then(res => {
        this.recommendList.push(res)
        this.isLoading = false
      })
    },
    formatDay (date) {
      let month = date.substr(4, 2)
      let day = date.substr(6, 2)
      if (month.substr(0, 1) === '0') month = month.substr(1, 1)
      if (day.substr(0, 1) === '0') day = day.substr(1, 1)
      return `${month}月${day}日`
    },
    handleClick (id) {
      this.articleId = id
    },
    handleScroll () {
      // 获取到DOM
      const $list = this.$refs.list
      // 在“主题日报”或正在加载推荐列表时停止操作
      if (this.type === 'daily' || this.isLoading) return
      // 已经滚动的距离加页面的高度等于整个内容域高度时，视为接触底部
      if ($list.scrollTop + document.body.clientHeight >= $list.scrollHeight) {
        // 时间相对减少一天
        this.dailyTime -= 86400000
        this.getRecommendList()
      }
    }
  },
  mounted () {
    // 初始化获取主题日报的子类
    this.getThemes()
    // 获取每日推荐列表
    this.getRecommendList()
    // // 获取到DOM
    // const $list = this.$refs.list
    // // 监听中栏的滚动事件
    // $list.addEventListener('scroll', () => {
    //   // 在“主题日报”或正在加载推荐列表时停止操作
    //   if (this.type === 'daily' || this.isLoading) return
    //   // 已经滚动的距离加页面的高度等于整个内容域高度时，视为接触底部
    //   if ($list.scrollTop + document.body.clientHeight >= $list.scrollHeight) {
    //     // 时间相对减少一天
    //     this.dailyTime -= 86400000
    //     this.getRecommendList()
    //   }
    // })
  }
}
</script>
