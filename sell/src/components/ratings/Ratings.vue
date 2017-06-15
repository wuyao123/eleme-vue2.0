<template>
  <div class="ratings" ref="ratings">
    <div class="rating-content">
      <div class="overview">
        <div class="overview-left">
          <h1 class="score">{{seller.score}}</h1>
          <div class="title">综合评分</div>
          <div class="rank">高于周边商家{{seller.rankRate}}%</div>
        </div>
        <div class="overview-right">
          <div class="score-wrapper">
            <span class="title">服务态度</span>
            <star :size="36" :score="seller.serviceScore"></star>
            <span class="score">{{seller.serviceScore}}</span>
          </div>
          <div class="score-wrapper">
            <span class="title">商品评分</span>
            <star :size="36" :score="seller.foodScore"></star>
            <span class="score">{{seller.foodScore}}</span>
          </div>
          <div class="delivery-wrapper">
          	<span class="title">送达时间</span>
          	<span class="delivery">{{seller.deliveryTime}}分钟</span>
          </div>
        </div>
      </div>
      <split></split>
      <ratingselect :select-type="selectType" :only-content="onlyContent" 
          :desc="desc" :ratings="ratings"></ratingselect>
      <div class="rating-wrapper">
        <ul>
          <li v-for="rating in ratings" v-show="needShow(rating.rateType,rating.text)" class="rating-item">
            <div class="avatar">
              <img :src="rating.avatar" width="28" height="28">
            </div>
            <div class="content">
              <h1 class="name">{{rating.username}}</h1>
              <div class="star-wrapper">
                <star :size="24" :score="rating.score"></star>
                <span class="delivery" v-show="rating.deliveryTime">{{rating.deliveryTime}}分钟送达</span>
              </div>
              <p class="text">{{rating.text}}</p>
              <div class="recommend" v-show="rating.recommend">
                <span class="icon-thumb_up"></span>
                <span class="item" v-for="item in rating.recommend">{{item}}</span>
              </div>
              <div class="time">
                {{rating.rateTime | formatDate}}
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import BScroll from 'better-scroll'
  import star from '@/components/star/star'
  import ratingselect from '@/components/ratingselect/ratingselect'
  import split from '@/components/split/split'
  import { formatDate } from '@/common/js/date'
  const ALL = 2
  const ERR_OK = 0
  export default {
    created () {
      this.$http.get('/api/ratings').then((response) => {
        response = response.body
        if (response.errno === ERR_OK) {
          this.ratings = response.data
          this.$nextTick(() => {
            this.scroll = new BScroll(this.$refs.ratings, {
              click: true
            })
          })
        }
      })
      this.$root.eventHub.$on('ratingTypeSelect', (type) => {
        this.selectType = type
        this.$nextTick(() => {
          this.scroll.refresh()
        })
      })
      this.$root.eventHub.$on('contensTtoggle', () => {
        this.onlyContent = !this.onlyContent
        this.$nextTick(() => {
          this.scroll.refresh()
        })
      })
    },
    filters: {
      formatDate (time) {
        let date = new Date(time)
        return formatDate(date, 'yyyy-MM-dd hh:mm')
      }
    },
    methods: {
      needShow (type, text) {
        if (this.onlyContent && !text) {
          return false
        }
        if (this.selectType === ALL) {
          return true
        } else {
          return type === this.selectType
        }
      }
    },
    data () {
      return {
        ratings: [],
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',
          positive: '满意',
          negative: '不满意'
        }
      }
    },
    props: {
      seller: {
        type: Object
      }
    },
    components: {
      star,
      ratingselect,
      split
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .ratings
    position:absolute
    top:174px
    bottom:0
    left:0
    width:100%
    overflow:hidden
    .overview
      display:flex
      padding:18px 0
      .overview-left
        flex:0 0 137px
        width:137px
        border-right:1px solid rgba(7,17,27,0.1)
        text-align:center
        padding:8px 0
        @media only screen and (max-width:320px)
          flex:0 0 120px
          width:120px
        .score
          line-height:28px
          font-size:24px
          color:rgb(255,153,0)
          margin-bottom:6px
        .title
          line-height:12px
          font-size:12px
          color:rgb(7,17,27)
          margin-bottom:8px
        .rank
          line-height:10px
          font-size:10px
          color:rgb(147,153,159)
      .overview-right
        flex:1
        padding:6px 0 6px 24px
        @media only screen and (max-width:320px)
          padding-left:6px  
        .score-wrapper
          margin-bottom:8px
          font-size:0
          .title, .star, .score
            line-height:18px
            display:inline-block
            vertical-align:top
            font-size:12px
          .title
            color:rgb(7,17,27)
          .star
            margin:0 12px
          .score
            color:rgb(255,153,0)
        .delivery-wrapper
          font-size:0
          .title
            line-height:18px
            font-size:12px
            color:rgb(7,17,27)
          .delivery
            font-size:12px
            color:rgb(147,153,159)
            margin-left:12px
    .rating-wrapper
      padding:0 18px
      .rating-item
        display:flex
        padding:18px 0
        border-bottom:1px solid rgba(7,17,27,0.1)
        .avatar
          flex:0 0 28px
          width:28px
          margin-right:12px
          img
            border-radius:50%
        .content
          flex:1
          position:relative
          .name
            line-height:12px
            font-size:10px
            color:rgb(7,17,27)
            margin-bottom:4px
          .star-wrapper
            margin-bottom:6px
            font-size:0
            .star
              display:inline-block
              vertical-align:top
              margin-right:6px
            .delivery
              display:inline-block
              vertical-align:top
              font-size:10px
              line-height:12px
              color:rgb(147,153,159)
              font-weight:200
          .text
            line-height:18px
            color:rgb(7,17,27)
            font-size:12px
            margin-bottom:8px
          .recommend
            font-size:0
            line-height:16px
            .icon-thumb_up, .item
              display:inline-block
              margin:0 8px 4px 0
            .icon-thumb_up
              font-size:12px
              color:rgb(0,160,220)
            .item
              padding:0 6px
              font-size:9px
              color:rgb(147,153,159)
              border:1px solid rgba(7,17,27,0.1)
              background:rgb(255,255,255)
          .time
            position:absolute
            top:0
            right:0
            font-size:10px
            color:rgb(147,153,159)
            line-height:12px
            font-weight:200
</style>
