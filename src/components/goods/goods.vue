<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li class="menu-item" v-for="item,index in goods" :class="{'current':currentIndex===index}"
            @click="selectMenu(index,$event)">
          <span class="text">
            <span class="icon" v-show="item.type>0" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li class="food-item" v-for="food in item.foods">
              <img :src="food.icon" class="icon">
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="description" v-show="food.description">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  ￥<span class="now">{{food.price}}</span><span v-show="food.oldPrice"
                                                                class="old">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol  @add="addFood" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart ref="shopcart" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice" :selectFoods="selectFoods"></shopcart>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import shopcart from '../shopcart/shopcart';
  import cartcontrol from '../cartcontrol/cartcontrol';

  const ERR_OK = 0;

  export default{
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0
      };
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
      },
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        console.log(foods);
        return foods;
      }
    },
    created() {
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    },
    methods: {
      selectMenu(index, event) {
        if (!event._constructed) {
          return;
        }
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el, 300);
      },
      _initScroll() {
        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          probeType: 3,
          click: true
        });

        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
        });
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        });
      },
      _calculateHeight() {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      },
      addFood(target) {
        this._drop(target);
      },
      _drop(target) {
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target);
        });
      }
    },
    components: {
      shopcart: shopcart,
      cartcontrol: cartcontrol
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import '../../common/stylus/mixin.styl'

  .goods
    display flex
    position: absolute
    width: 100%
    top: 174px
    bottom: 46px
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background-color: #f3f5f7
      .menu-item
        display: table
        padding: 0 12px
        width: 56px
        height: 54px
        line-height: 14px
        &.current
          position: relative
          z-index: 10
          font-weight: 700
          background-color: #fff
          margin-top: -1px
          .text
            border-none()
        .text
          display: table-cell
          vertical-align: middle
          border-1px rgba(7, 17, 27, 0.1)
          font-size: 12px
          width: 56px
          line-height: 14px
          font-weight: 200
        .icon
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
    .foods-wrapper
      width: 100%
      .title
        width: 100%
        height: 26px
        background-color: #f3f5f7
        border-left: 4px solid #d9dde1
        padding-left: 14px
        line-height: 26px
        color: rgb(147, 153, 159)
        font-size: 12px
      .food-item
        margin: 18px
        padding-bottom: 18px
        display: flex
        border-1px rgba(7, 17, 27, 0.1)
        flex: 1
        &:last-child
          border-none()
        .icon
          width: 57px
          height: 57px
        .content
          display: flex
          flex-direction: column
          height: 72px
          margin-left: 10px
          font-size: 10px
          line-height: 10px
          color: rgb(147, 153, 159)
          .name
            font-size: 14px
            line-height: 14px
            color: rgb(7, 17, 27)
            margin: 2px 0 8px 0
          .description
            line-height: 12px
          .extra
            margin: 8px 0 0
            .count
              margin-right: 12px
          .price
            color: rgb(147, 153, 159)
            line-height: 24px
            font-size: 10px
            font-weight: 700
            .now
              color: rgb(240, 20, 20)
              font-size: 14px
            .old
              margin-left: 4px
              text-decoration: line-through
              vertical-align: top
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 10px;


</style>
