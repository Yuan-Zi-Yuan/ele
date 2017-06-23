<template>
  <div class="cartcontrol">
    <transition name="move">
      <div class="cart-decrease" v-show="food.count>0" @click.stop.prevent="decreaseCart">
        <i class="inner icon-remove_circle_outline"></i>
      </div>
    </transition>
    <span class="count" v-show="food.count>0">{{food.count}}</span>
    <i class="icon-add_circle" @click.stop.prevent="addCart"></i>
  </div>
</template>

<script type="text/ecmascript-6">
  import Vue from 'vue';

  export default{
    props: {
      food: {
        type: Object
      }
    },
    methods: {
      addCart(event) {
        if (!event._constructed) {
          return;
        }
        if (!this.food.count) {
          Vue.set(this.food, 'count', 1);
        } else {
          this.food.count++;
        }
        this.$emit('add', event.target);
      },
      decreaseCart() {
        if (this.food.count > 0) {
          this.food.count--;
        }
      }
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus" type="text/stylus">
  .cartcontrol
    font-size: 0
    .cart-decrease, .icon-add_circle
      display: inline-block
      font-size: 24px
      padding: 0 7px
      line-height: 24px
      color: rgb(0, 160, 220)
      opacity: 1
      .inner
        display: inline-block
        transition: all 0.44s linear
        transform: rotate(0)
      &.move-enter-active, &.move-leave-active
        transition: all 0.4s linear
      &.move-leave
        color: red
        font-size: 50px
      &.move-enter, &.move-leave-active
        opacity: 1
        transform: translate3d(24px, 0, 0)
        .inner
          transform: rotate(180deg)
    .count
      display: inline-block
      vertical-align top
      width: 10px
      font-size: 10px
      line-height: 24px
      text-align: center
</style>
