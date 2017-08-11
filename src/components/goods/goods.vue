<template>
    <div class="goods">
        <!--使用v-el生成vue可以直接使用的对象，注意这里一定要使用中划线，不要使用驼峰。那么在js中就可以直接使用：this.$els.menuWrapper -->
        <div class="menu-wrapper" v-el:menu-wrapper>
            <ul>
                <li v-for="item in goods" class="menu-item">
                    <span class="text border-1px">
                        <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
                    </span>
                </li>
            </ul>
        </div>
        <div class="foods-wrapper" v-el:foods-wrapper>
            <ul>
                <li v-for="item in goods" class="food-list">
                    <h1 class="title">{{item.name}}</h1>
                    <ul>
                        <li v-for="food in item.foods" class="food-item border-1px">
                            <div class="icon">
                                <img :src="food.icon" width="57px" height="57px">
                            </div>
                            <div class="content">
                                <h2 class="name">{{food.name}}</h2>
                                <p class="desc">{{food.description}}</p>
                                <div class="extra">
                                    <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                                </div>
                                <div class="price">
                                    <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                                </div>
                            </div>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</template>

<script type="text/ecmascript-6">

    import BScroll from 'better-scroll'

    const ERR_OK = 0;
    export default{
        props: {
            seller: {
                type: Object
            }
        },
        data(){
            return {
                goods: []
            }
        },
        created(){

            this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
            this.$http.get("/api/goods").then((response) => {
                response = response.body;
                if (response.errno === ERR_OK) {  // 请求状态
                    this.goods = response.data;  // 数据保存在goods下

                    // 注意，这里为什么使用vue自带的这个函数$nextTick？？？， 因为滚动组件需要计算高度，vue执行填充数据之后(异步)，再计算高度
                    this.$nextTick(()=>{
                        this._initScroll();
                    })
                }
            });
        },
        methods: {
            _initScroll(){
                this.menuScroll = new BScroll(this.$els.menuWrapper, {});
                this.foodScroll = new BScroll(this.$els.foodsWrapper, {});
            }
        }

    }
</script>

<style lang="stylus" rel="stylesheet/stylus">
    @import "../../common/stylus/mixin.styl"

    .goods
        position absolute
        display flex
        top 174px
        bottom 46px
        width 100%
        overflow hidden
        .menu-wrapper
            flex 0 0 80px
            width 80px /*不写width，在andriod浏览器上面还是会有问题*/
            background #f3f5f7
            .menu-item
                display table /*小技巧，不管一行还是两行都能垂直居中*/
                height 54px
                width 56px
                line-height 14px
                padding 0 12px
                .icon
                    display inline-block
                    vertical-align top
                    width 12px
                    height 12px
                    margin-right 2px
                    background-size 12px 12px
                    background-repeat no-repeat
                    &.decrease
                        bg-image("decrease_3")
                    &.discount
                        bg-image("discount_3")
                    &.guarantee
                        bg-image("guarantee_3")
                    &.invoice
                        bg-image("invoice_3")
                    &.special
                        bg-image("special_3")

                .text
                    font-size 12px
                    display table-cell
                    width 56px
                    vertical-align middle
                    border-1px(rgba(7, 17, 27, 0.6))
        .foods-wrapper
            flex 1
            .title
                padding-left 14px
                height 26px
                line-height 26px
                border-left 2px solid #d9dde1
                font-size 12px
                color rgb(147, 153, 159)
                background #f3f5f7
            .food-item
                display flex
                margin 18px /*上下margin的重叠效果，其实只有18px*/
                padding-bottom 18px
                border-1px(rgba(7, 17, 27, 0.6))
                &:last-child
                    border-none()
                    margin-bottom 0px
                .icon
                    flex 0 0 57px
                    margin-right 10px
                .content
                    flex 1
                    .name
                        margin 2px 0 8px 0
                        height 14px
                        line-height 14px
                        font-size 14px
                        color rgb(7, 17, 27)
                    .desc, .extra
                        line-height 10px
                        font-size 10px
                        color rgb(147, 153, 159)
                    .desc
                        line-height 12px
                        margin-bottom 8px
                    .extra
                        .count
                            margin-right 12px
                    .price
                        font-weight 700
                        line-height 24px
                        .now
                            margin-right 8px
                            font-size 14px
                            color rgb(240, 20, 20)
                        .old
                            text-decoration line-through
                            font-size 10px
                            color rgb(147, 153, 159)
</style>