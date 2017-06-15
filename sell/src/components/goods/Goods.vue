<template>
  <div>
    <div class="goods">
  	  <div class="menu-wrapper" ref="menuWrapper">
  	  <ul>
  	  	<li v-for="(item,index) in goods" class="menu-item" :class="{'current':currentIndex===index}" @click="selectMenu(index,$event)">
  	  	<span class="text border-1px">
  	  	  <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
  	  	</span>
  	  	</li>
  	  </ul>
  	  </div>
  	  <div class="foods-wrapper" ref="foodsWrapper">
  	    <ul>
  	      <li v-for="item in goods" class="food-list food-list-hook">
  	        <h1 class="title">{{item.name}}</h1>
  	        <ul>
  	          <li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px">
  	            <div class="icon">
  	          	  <img width="57" height="57" :src="food.icon">
  	            </div>
  	            <div class="content">
  	          	  <h2 class="name">{{food.name}}</h2>
  	          	  <p class="desc">{{food.description}}</p>
  	          	  <div class="extra">
  	          	    <span>月售{{food.sellCount}}份</span><span>好评率{{food.rating}}</span>
  	          	  </div>
  	          	  <div class="price">
  	          	    <span class="now">￥{{food.price}}</span><span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
  	          	  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food"></cartcontrol>
                  </div>
  	            </div>
  	          </li>
  	        </ul>
  	      </li>
  	    </ul>
  	  </div>
      <div>
        <shopcart ref="shopcart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
      </div>
    </div>
    <food :food="selectedFood" ref="food"></food>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'
  import shopcart from '@/components/shopcart/ShopCart'
  import cartcontrol from '@/components/cartcontrol/CartControl'
  import food from '@/components/food/Food'

  const ERR_OK = 0
  export default {
    components: {
      shopcart,
      cartcontrol,
      food
    },
    props: {
      seller: {
        type: Object
      }
    },
    created () {
      this.classMap = ['decrease', 'discount', 'guarantee', 'invoice', 'special']
      this.$http.get('/api/goods').then((response) => {
        response = response.body
        if (response.errno === ERR_OK) {
          this.goods = response.data
          this.$nextTick(() => {
            this._calculateHeight()
            this._initScroll()
          })
        }
      })
      this.$root.eventHub.$on('cartadd', (target) => {
        this._drop(target)
      })
      this.$root.eventHub.$on('addFood', (target) => {
        this._drop(target)
      })
    },
    data () {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      }
    },
    methods: {
      _drop (target) {
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target)
        })
      },
      _initScroll () {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        })
        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true,
          probeType: 3
        })
        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y))
        })
      },
      _calculateHeight () {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      },
      selectMenu (index, event) {
        if (!event._constructed) {
          return
        }
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
        let el = foodList[index]
        this.foodsScroll.scrollToElement(el, 300)
      },
      selectFood (food, event) {
        if (!event._constructed) {
          return
        }
        this.selectedFood = food
        this.$refs.food.show()
      }
    },
    computed: {
      currentIndex () {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i]
          let height2 = this.listHeight[i + 1]
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i
          }
        }
        return 0
      },
      selectFoods () {
        let foods = []
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food)
            }
          })
        })
        return foods
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin"
  .goods
    position: absolute
    top: 174px
    bottom: 46px
    width: 100%
    display: flex
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      .menu-item
        display: table
        height: 54px
        width: 56px
        line-height: 14px
        padding: 0 12px
        &.current
          position: relative
          z-index: 10
          margin: -1px 0
          background: #fff     
          .text
            border-none-all()
            font-weight: 700
        .icon
          vertical-align: top
          display: inline-block
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
             bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          font-size: 12px
          display: table-cell
          vertical-align: middle
          width: 56px
          border-1px(rgba(7, 17, 27, 0.1))
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        color: rgb(147, 153, 159)
        background: #f3f5f7
        font-size: 12px
        line-height: 26px
        height: 26px
        border-left: 2px solid #d9dde1
      .food-item
        display: flex
        padding: 18px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          felx: 0 0 57px
          margin-right: 10px
        .content
          flex: 1
          .cartcontrol-wrapper
            position: absolute
            right: 12px
            bottom: 12px
          .name
            margin: 2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color: rgb(7,17,27)
          .desc, .extra
            margin-bottom: 8px
            font-size: 10px
            color: rgb(147,153,159)
            span
              margin-right: 8px
          .desc
            line-height: 12px
          .price
            font-weight: 700
            line-height: 24px
            .now
              font-size: 10px/14px
              color: rgb(240, 20, 20)
              margin-right: 8px
            .old
              text-decoration: line-through
              color: rgb(147,153,159)



</style>
