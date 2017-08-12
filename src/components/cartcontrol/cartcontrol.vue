<template>
    <div class="cartcontrol">
        <div class="cart-decrease icon-remove_circle_outline" v-show="food.count>0" @click="decreaseCart($event)"></div>
        <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
        <div class="cart-add icon-add_circle" @click="addCart($event)"></div>
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
                    Vue.set(this.food, 'count', 1);
                } else {
                    this.food.count++;
                }
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
        .cart-decrease, .cart-add
            display inline-block
            padding 6px  //通过padding，增加可点击区域
            line-height 24px
            font-size 24px
            color rgb(0, 160, 220)
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


</style>