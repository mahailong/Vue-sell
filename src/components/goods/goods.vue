<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li v-for="(item, index) in goods" 
          :class="{'current':currentIndex===index}"
          class="menu-item"
          @click="selectMenu(index,$event)">
          <span class="text border-1px">
            <span 
              v-show="item.type>0" 
              class="icon" 
              :class="classMap[item.type]">
            </span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodWrapper">
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFun(food, $event)"
              v-for="food in item.foods" 
              class="food-item border-1px">
              <div class="icon">
                <img :src="food.icon" alt="">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span><span>
                  好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span>
                  <span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartctrl-wrapper">
                  <cartctrl :food="food"></cartctrl>
                </div>
              </div>
            </li>
          </ul>
        </li>  
      </ul>
    </div>
    <shopcart ref="shopcart"
      :select-foods="selectFoods" 
      :delivery-price="seller.deliveryPrice"
      :min-price="seller.minPrice"></shopcart>
    <food ref="food" :food="selectFood"></food>
  </div>
</template>

<script>
  import BScroll from 'better-scroll'
  import shopcart from '@/components/shopcart/shopcart'
  import cartctrl from '@/components/cartctrl/cartctrl'
  import food from '@/components/food/food'
  import eventHub from '@/common/js/eventHub'

  const ERR_OK = 0

  export default {
    props: {
      seller: {
        type: Object
      }
    },
    data () {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectFood: {}
      }
    },
    components: {
      shopcart,
      cartctrl,
      food
    },
    computed: {
      currentIndex () {
        for (let i = 0; i < this.listHeight.length; i++) {
          let currHeight = this.listHeight[i]
          let nextHeight = this.listHeight[i + 1]
          if (!nextHeight || (this.scrollY >= currHeight && this.scrollY < nextHeight)) {
            return i
          }
        }
        return 0
      },
      selectFoods () {
        let foods = []
        this.goods.forEach((good) => {
          foods = foods.concat(good.foods.filter(food => food.count))
        })
        return foods
      }
    },
    created () {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
      this.$http.get('/api/goods?id=' + this.seller.id).then((response) => {
        response = response.body
        if (response.errno === ERR_OK) {
          this.goods = response.data
          this.$nextTick(() => {
            this._initScroll()
            this._calculateHeight()
          })
        }
      })
      eventHub.$on('cart-add', this._drop)
    },
    beforeDestroy: function () {
      eventHub.$off('cart-add', this._drop)
    },
    methods: {
      _drop (target) {
        // 体验优化，异步执行下落动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target)
        })
      },
      _initScroll () {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        })
        this.foodScroll = new BScroll(this.$refs.foodWrapper, {
          probeType: 3,
          click: true
        })
        this.foodScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y))
        })
      },
      _calculateHeight () {
        let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook')
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      },
      selectMenu (index, event) {
        if (event._constructed) {
          let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook')
          let el = foodList[index]
          this.foodScroll.scrollToElement(el, 300)
        }
      },
      selectFun (food, event) {
        if (event._constructed) {
          this.selectFood = food
          this.$refs.food.toggle()
        }
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin"
  .goods
    display:flex
    position: absolute
    top: 174px
    width: 100%
    bottom: 46px
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
          margin-top: -1px
          z-index: 10
          background: #fff
          font-weight: 700
          .text
            border-none()
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            bg-image('assets/decrease_3')
          &.discount
            bg-image('assets/discount_3')
          &.guarantee
            bg-image('assets/guarantee_3')
          &.invoice
            bg-image('assets/invoice_3')
          &.special
            bg-image('assets/special_3')
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          font-size: 12px
          border-1px(rgba(7,17,27,0.1))
    .foods-wrapper
      flex:1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        border-left: 2px solid #d9dde1
        font-size: 12px
        color: rgb(147,153,159)
        background: #f3f5f7
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7,17,27,0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          margin-right: 10px
          img
            width: 57px
            height: 57px
        .content
          flex: 1
          .name
            margin: 2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color: rgb(7,17,27)
          .desc, .extra
            line-height: 10px
            font-size: 10px
            color: rgb(147,153,159)
          .desc
            margin-bottom: 8px
            line-height: 12px
          .extra
            .count
              margin-right: 12px
          .price
            font-weight: 700
            line-height:24px
            .now
              margin-right: 8px
              font-size: 14px
              color: rgb(240,20,20)
            .old
              text-decoration: line-through
              font-size: 10px
              color: rgb(147,153,159)  
          .cartctrl-wrapper
            position: absolute
            right: 0
            bottom: 12px   
</style>
