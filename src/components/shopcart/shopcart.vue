<template xmlns:v-el="http://www.w3.org/1999/xhtml">
    <div class="shopcart">
        <div class="content" @click="toggleList">
            <div class="content-left">
                <div class="logo-wrapper">
                    <div class="logo" :class="{'highLight':totalCount>0}">
                        <i class="icon-shopping_cart" :class="{'highLight':totalCount>0}"></i>
                    </div>
                    <div class="num" v-show="totalCount>0">{{totalCount}}</div>
                </div>
                <div class="price" :class="{'highLight':totalPrice>0}">￥{{totalPrice}}元</div>
                <div class="desc">另需配速费￥{{deliveryPrice}}元</div>
            </div>
            <div class="content-right" @click.stop.prevent="pay">
                <div class="pay" :class="payClass">{{payDesc}}</div>
            </div>
            c
        </div>
        <div class="ball-container">
            <div transition="drop" class="ball" v-for="ball in balls" v-show="ball.show"><!--外层元素做纵向动画-->
                <div class="inner inner-hook"></div><!--内层元素做横向动画-->
            </div>
        </div>
        <div class="shopcart-list" v-show="listShow" transition="fold">
            <div class="list-header">
                <h1 class="title">购物车</h1>
                <span class="empty" @click="empty">清空</span>
            </div>
            <div class="list-content" v-el:list-content>
                <ul>
                    <li class="food" v-for="food in selectFoods">
                        <span class="name">{{food.name}}</span>
                        <div class="price">
                            <span>￥{{food.price * food.count}}</span>
                        </div>
                        <div class="cartcontrol-wrapper">
                            <cartcontrol :food="food"></cartcontrol>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
    </div>
    <div class="list-mask" v-show="listShow" transition="fade" @click="hideList"></div>
</template>

<script type="text/ecmascript-6">
    import cartcontrol from 'components/cartcontrol/cartcontrol'
    import BScroll from 'better-scroll'

    export default{

        props: {
            selectFoods: {
                type: Array,
                default () {  // 在vue中，如果type是一个Array，或者 Object，那么default就是一个函数
                    return [{
                        price: 10,
                        count: 4
                    }];
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
        components: {
            cartcontrol
        },
        data(){
            return {
                balls: [ //初始化5个小球
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
                fold: true  // 购物清单默认折叠
            }
        },
        computed: {
            totalPrice(){  // 总价
                let total = 0;
                this.selectFoods.forEach((food) => {
                    total += food.price * food.count;
                })
                return total;
            },
            totalCount(){ //商品总数量
                let count = 0;
                this.selectFoods.forEach((food) => {
                    count += food.count;
                })
                return count;
            },
            payDesc(){
                if (this.totalPrice === 0) {
                    return `￥${this.minPrice}元起送`;
                } else if (this.totalPrice < this.minPrice) {
                    let diff = this.minPrice - this.totalPrice;
                    return `还差￥${diff}元起送`;
                } else {
                    return "去结算";
                }
            },
            payClass(){
                if (this.totalPrice < this.minPrice) {
                    return "not-enough";
                } else {
                    return "enough"
                }
            },
            listShow(){
                if (!this.totalCount) {
                    this.fold = true;
                    return false;
                }
                let show = !this.fold;

                // 这里做better-scrolle初始化
                if (show) {
                    this.$nextTick(() => {
                        if (!this.scroll) { // 当scroll对象没有被实例化的时候，初识化
                            this.scroll = new BScroll(this.$els.listContent, {
                                click: true
                            });
                        } else { // 当scroll已经被实例化了，只需要调用better-scroll的refresh()去重新计算高度就可以了
                            this.scroll.refresh();
                        }

                    });
                }

                return show;

            }
        },
        methods: {
            drop(el){
                for (let i = 0; i < this.balls.length; i++) {
                    let ball = this.balls[i];
                    if (!ball.show) {  //从所有的balls中找到隐藏的小球。将其置为true
                        ball.show = true;
                        ball.el = el;
                        this.dropBalls.push(ball);
                        //console.log(el.getBoundingClientRect());
                        return;
                    }
                }

            },
            toggleList(){
                if (!this.totalCount) {
                    return;
                }
                this.fold = !this.fold;
            },

            empty(){
                this.selectFoods.forEach((food) => {
                    food.count = 0;
                });
            },
            hideList(){
                this.fold = true;
            },
            pay(){
                if(this.totalPrice < this.minPrice){
                    return;
                }
                window.alert(`支付${this.totalPrice}`);
            }
        },
        transitions: {
            drop: {
                beforeEnter(el){
                    let count = this.balls.length;
                    while (count--) {
                        let ball = this.balls[count];
                        if (ball.show) {
                            let rect = ball.el.getBoundingClientRect(); //获取el元素，相对于当前视口的位置
                            let x = rect.left - 32; // 32是指小球落入购物车的left
                            let y = -(window.innerHeight - rect.top - 22);
                            el.style.display = ''; //让小球显示出来，因为小球在v-show的时候，默认是不显示的，也就是display=none
                            el.style.webkitTransform = `translate3d(0,${y}px,0)`;
                            el.style.transform = `translate3d(0,${y}px,0)`;

                            let inner = el.getElementsByClassName('inner-hook')[0];
                            inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
                            inner.style.transform = `translate3d(${x}px,0,0)`;

                        }
                    }
                },
                enter(el){
                    /*主动触发浏览器重绘*/
                    let rf = el.offsetHeight;

                    this.$nextTick(() => {  // 把状态重置回来

                        el.style.display = '';
                        el.style.webkitTransform = "translate3d(0,0,0)";
                        el.style.transform = "translate3d(0,0,0)";

                        let inner = el.getElementsByClassName('inner-hook')[0];
                        inner.style.webkitTransform = "translate3d(0,0,0)";
                        inner.style.transform = "translate3d(0,0,0)";

                    });
                },
                afterEnter(el){
                    let ball = this.dropBalls.shift();
                    if (ball) {
                        ball.show = false;
                        el.style.display = 'none';
                    }
                }
            }
        }
    }
</script>

<style lang="stylus" rel="stylesheet/stylus">

    @import "../../common/stylus/mixin.styl"

    .shopcart
        position fixed
        left 0
        bottom 0
        z-index 50
        height 48px
        width 100%
        .content
            background #141d27
            display flex
            font-size 0px
            color rgba(255, 255, 255, 0.4)
            .content-left //左侧自适应
                flex 1
                .logo-wrapper
                    display inline-block
                    position relative
                    top -10px
                    margin 0 12px
                    padding 6px
                    width 56px
                    height 56px
                    box-sizing border-box /*宽高包括border+padding+width*/
                    vertical-align top
                    border-radius 50%
                    background #141d27
                    .num
                        position absolute
                        top 0
                        right 0
                        width 24px
                        height 16px
                        line-height 16px
                        text-align center
                        border-radius 16px
                        font-size 9px
                        font-weight 700
                        color #fff
                        background rgb(240, 20, 20)
                        box-shadow 0 4px 8px 0 rgba(0, 0, 0, 0.4)
                    .logo
                        width 100%
                        height 100%
                        border-radius 50%
                        text-align center
                        background #2b343c
                        &.highLight
                            background rgb(0, 160, 220)
                        .icon-shopping_cart
                            color #80858a
                            font-size 24px
                            line-height 44px
                            &.highLight
                                color #fff
                .price
                    display inline-block
                    vertical-align top
                    margin-top 12px
                    line-height 24px
                    padding-right 12px
                    box-sizing border-box
                    border-right 1px solid rgba(255, 255, 255, 0.1)
                    font-size 16px
                    font-weight 700
                    &.highLight
                        color #fff
                .desc
                    display inline-block
                    vertical-align top
                    line-height 24px
                    margin 12px 0 0 12px
                    font-size 10px
            .content-right // 右侧固定宽度
                flex 0 0 105px
                width 105px
                .pay
                    line-height 48px
                    height 48px
                    text-align center
                    font-size 12px
                    font-weight 700
                    background #2b333b
                    &.not-enough
                        background #2b333b
                    &.enough
                        background #00b43c
                        color #fff
        .ball-container
            .ball
                position fixed
                left 32px
                bottom 22px
                z-index 200
                &.drop-transition
                    transition all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
                    .inner
                        width 16px
                        height 16px
                        border-radius 50%
                        background rgb(0, 160, 220)
                        transition all 0.4s linear

        .shopcart-list
            position absolute
            top 0
            left 0
            z-index -1
            width 100%
            &.fold-transition
                transition all 0.5s
                transform translate3d(0, -100%, 0)
            /**小技巧：向上移动到自身高度的百分百**/
            &.fold-enter, &.fold-leave
                transform translate3d(0, 0, 0)
            .list-header
                height 40px
                line-height 40px
                padding 0 18px
                background #f3f5f7
                border-bottom 1px solid rgba(7, 17, 27, 0.1)
                .title
                    float left
                    font-size 14px
                    color rgb(7, 17, 27);
                .empty
                    float right
                    font-size 12px
                    color rgb(0, 160, 220)
            .list-content
                padding 0 18px
                max-height 217px
                overflow hidden
                background #fff
                .food
                    position relative
                    padding 12px 0
                    box-sizing border-box
                    border-1px(rgba(7, 17, 27, 0.1))
                    .name
                        line-height 24px
                        font-size 14px
                        color rgb(7, 17, 27)
                    .price
                        position absolute
                        right 90px
                        bottom 12px
                        line-height 24px
                        font-size 14px
                        font-weight 700
                        color rgb(240, 20, 20)
                    .cartcontrol-wrapper
                        position absolute
                        right 0%
                        bottom 6px

    .list-mask
        position fixed
        top 0
        left 0
        width 100%
        height 100%
        z-index 40
        backdrop-filter blur(10px)
        &.fade-transition
            transition all 0.4s
            opacity 1
            background rgba(7, 17, 27, 0.6)
        &.fade-enter, &.fade-leave
            background rgba(7, 17, 27, 0)


</style>