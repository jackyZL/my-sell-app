<template>
    <div class="cartcontrol">
        <div class="cart-decrease" v-show="food.count>0" @click.stop.prevent="decreaseCart($event)" transition="move">
            <span class="inner icon-remove_circle_outline"></span>
        </div>
        <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
        <div class="cart-add icon-add_circle" @click.stop.prevent="addCart($event)"></div>
    </div>
</template>

<script type="text/ecmascript-6">

    import Vue from 'vue'

    export  default{
        props: {
            food: {
                type: Object
            }
        },
        methods: {
            addCart(event){
                if (!event._constructed) {   //解决pc端点击，触发两次的bug： better-scroll派发的事件，_constructed为true，浏览器自带的event事件，_constructed为false
                    return;
                }
                if (!this.food.count) {
                    // 注意：直接使用this.food.count = 1; food的变化，是不能被观测到的，也就是数据不能变化驱动视图改变的. 因为food本身没有count这个属性
                    Vue.set(this.food, 'count', 1); //给父组件传递过来的值，添加count属性。这在vue1.x中，会影响父组件传递过来的值
                } else {
                    this.food.count++;
                }

                this.$dispatch('cart.add',event.target); // 点击的时候派发一个事件（传递的参数为event.target），将dom对象传递出来，实现小球飞翔动画
            },
            decreaseCart(event){
                if (!event._constructed) {   //解决pc端点击，触发两次的bug： better-scroll派发的事件，_constructed为true，浏览器自带的event事件，_constructed为false
                    return;
                }
                if (this.food.count > 0) {
                    this.food.count--;
                }
            }
        }
    }
</script>

<style lang="stylus" rel="stylesheet/stylus">
    .cartcontrol
        font-size 0
        .cart-decrease
            display inline-block
            padding 6px  //通过padding，增加可点击区域
            transition all 0.4s linear
            &.move-transition
                opacity 1
                transform translate3d(0,0,0) //开启硬件加速，让动画更流畅
                .inner
                    display inline-block
                    line-height 24px
                    font-size 24px
                    color rgb(0, 160, 220)
                    transition all 0.4s linear
                    transform rotate(0)
            &.move-enter, &.move-leave
                opacity 0
                transform translate3d(24px, 0, 0) // 向右移动24px
                .inner
                    transform rotate(180deg)
        .cart-count
            display inline-block
            font-size 10px
            vertical-align top
            width 12px
            padding-top 6px
            line-height 24px
            text-align center
            color rgb(147,153,159)
        .cart-add
            display inline-block
            padding 6px  //通过padding，增加可点击区域
            line-height 24px
            font-size 24px
            color rgb(0, 160, 220)


</style>