<template>
  <div class="shopcart">
    <div class="content" @click="toggleList">
      <div class="content-left">
        <div class="logo-wrapper">
          <div class="logo" :class="{'highlight':totalCount>0}">
            <i class="icon-shopping_cart"></i>
          </div>
        </div>
        <div v-show="totalCount>0" class="num">{{totalCount}}</div>
        <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
        <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
      </div>
      <div class="content-right">
        <div class="pay" :class="{'enough':enough}">{{payDesc}}</div>
      </div>
      <div class="ball-container">
        <div v-for="ball in balls">
          <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <transition>
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food" v-for="food in selectFoods">
                <span class="name">{{food.name}}</span>
                <div class="cartcontrol-wrapper">
                  <span class="price">￥{{food.price * food.count}}</span>
                  <cartcontrol :food="food"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import cartcontrol from '../cartcontrol/cartcontrol.vue';

  export default{
    props: {
      selectFoods: {
        type: Array,

        default() {
          return [
            {
              price: 10,
              count: 2
            }
          ];
        }
      },
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        balls: [
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          }
        ],
        dropBalls: [],
        fold: true
      };
    },
    computed: {
      listShow() {
        if (!this.totalCount) {
          this.fold = true;
          return false;
        }
        let show = !this.fold;
        if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              this.scroll = new BScroll(this.$refs.listContent, {
                click: true
              });
            } else {
              this.scroll.refresh();
            }
          });
        }
        return show;
      },
      totalPrice() {
        let total = 0;
        this.selectFoods.forEach((food) => {
          total += food.price * food.count;
        });
        return total;
      },
      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },
      payDesc() {
        if (this.totalPrice === 0) {
          return '￥' + this.minPrice + '起送';
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice;
          return '还差￥' + diff + '起送';
        } else {
          return '去结算';
        }
      },
      enough() {
        if (this.totalPrice >= this.minPrice) {
          return true;
        }
      }
    },
    methods: {
      empty() {
        this.selectFoods.forEach((food) => {
          food.count = 0;
        });
      },
      toggleList() {
        if (!this.totalCount) {
          return;
        }
        this.fold = !this.fold;
      },
      drop(el) {
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            ball.show = true;
            ball.el = el;
            this.dropBalls.push(ball);
            return;
          }
        }
      },
      beforeDrop(el) {
        let count = this.balls.length;
        while (count--) {
          let ball = this.balls[count];
          if (ball.show) {
            let rect = ball.el.getBoundingClientRect();
            let x = rect.left - 32;
            let y = -(window.innerHeight - rect.top - 22);
            el.style.display = '';
            el.style.webkitTransform = `translate3d(0,${y}px,0)`;
            el.style.transform = `translate3d(0,${y}px,0)`;
            let inner = el.getElementsByClassName('inner-hook')[0];
            inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
            inner.style.transform = `translate3d(${x}px,0,0)`;
          }
        }
      },
      dropping(el, done) {
        /* eslint-disable no-unused-vars */
        let rf = el.offsetHeight;
        this.$nextTick(() => {
          el.style.webkitTransform = 'translate3d(0,0,0)';
          el.style.transform = 'translate3d(0,0,0)';
          let inner = el.getElementsByClassName('inner-hook')[0];
          inner.style.webkitTransform = 'translate3d(0,0,0)';
          inner.style.transform = 'translate3d(0,0,0)';
          el.addEventListener('transitionend', done);
        });
      },
      afterDrop(el) {
        let ball = this.dropBalls.shift();
        if (ball) {
          ball.show = false;
          el.style.display = 'none';
        }
      }
    },
    components: {
      cartcontrol: cartcontrol
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus" type="text/stylus">
  .shopcart
    position: fixed
    bottom: 0
    left: 0
    z-index: 50
    width: 100%
    height: 48px
    .content
      display flex
      background-color: #141d27
      font-size: 0
      .content-left
        flex: 1
        .logo-wrapper
          display: inline-block
          position: relative
          top: -10px
          margin: 0 12px
          padding: 6px
          width: 56px
          height: 56px
          box-sizing: border-box
          border-radius: 50%
          background-color: #141d27
          .logo
            width: 100%
            height: 100%
            border-radius: 50%
            text-align: center
            background-color: #2b343c
            .icon-shopping_cart
              line-height: 44px
              font-size: 24px
              color: #80858a
            &.highlight
              background-color: rgb(0, 160, 220)
              .icon-shopping_cart
                color: rgb(255, 255, 255)
        .num
          position: absolute
          top: -10px
          left: 40px
          text-align: center
          border-radius: 40%
          width: 24px
          color: rgb(255, 255, 255)
          font-size: 9px
          font-weight: 700
          line-height: 16px
          background-color: rgb(240, 20, 20)
          box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4)
        .price
          display: inline-block
          vertical-align: top
          font-size: 16px
          font-weight: 700
          line-height: 48px
          box-sizeing: border-box
          padding-right: 12px
          color: rgba(255, 255, 255, 0.4)
          border-right: 1px solid rgba(255, 255, 255, 0.1)
          &.highlight
            color: rgb(255, 255, 255)
        .desc
          display: inline-block
          vertical-align: top
          font-size: 10px
          line-height: 48px
          color: rgba(255, 255, 255, 0.4)
          padding: 0 12px
      .content-right
        flex: 0 0 105px
        background-color: rgb(43, 51, 59)
        text-align: center
        .pay
          font-size: 12px
          font-weight: 700
          line-height: 48px
          padding: 0 8px
          color: rgba(255, 255, 255, 0.4)
          &.enough
            background-color: #00b43c
            color: rgb(255, 255, 255)

    .ball-container
      .ball
        position: fixed
        left: 32px
        bottom: 22px
        z-index: 200
        transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
        .inner
          width: 16px
          height: 16px
          border-radius: 50%
          background: rgb(0, 160, 220)
          transition: all 0.4s linear
    .shopcart-list
      position: absolute
      left: 0
      top: 0
      z-index: -1
      width: 100%
      transform: translate3d(0, -100%, 0)
      &.fold-enter-active, &.fold-leave-active
        transition: all 0.5s
      &.fold-enter, &.fold-leave-active
        transform: translate3d(0, 0, 0)
      .list-header
        display: flex
        height: 40px
        padding: 0 18px
        line-height: 40px
        background-color: #f3f5f7
        border-bottom: 1px solid rgba(7, 17, 27, 0.1)
        .title
          flex: 1
          display: inline-block
          font-size: 14px
          font-weight: 200
          color: rgb(7, 17, 27)
        .empty
          font-size: 12px
          color: rgb(0, 160, 220)
      .list-content
        padding: 0 18px
        background-color: #fff
        .food
          box-sizing: border-box
          padding: 12px 0
          border-bottom: 1px solid rgba(7, 17, 27, 0.1)
          .name
            vertical-align middle
            font-size: 14px
            line-height: 24px
            font-color: rgb(7, 17, 27)
          .cartcontrol-wrapper
            float: right
            display: inline-block
            vertical-align: middle
            .price
              display: inline-block
              vertical-align: middle
              font-size: 14px
              line-height: 24px
              font-weight: 700
              color: rgb(240, 20, 20)
            .cartcontrol
              display: inline-block
              vertical-align: middle
</style>
