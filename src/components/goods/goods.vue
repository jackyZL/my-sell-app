<template xmlns:v-el="http://www.w3.org/1999/xhtml">
    <div class="goods">
        <!--使用v-el生成vue可以直接使用的对象，注意这里一定要使用中划线，不要使用驼峰。那么在js中就可以直接使用：this.$els.menuWrapper -->
        <div class="menu-wrapper" v-el:menu-wrapper>
            <ul>
                <li v-for="item in goods" class="menu-item" :class="{'current':currentIndex===$index}" @click="selectMenu($index, $event)">
                    <span class="text border-1px">
                        <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
                    </span>
                </li>
            </ul>
        </div>
        <div class="foods-wrapper" v-el:foods-wrapper>
            <ul>
                <li v-for="item in goods" class="food-list food-list-hook">
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
                                <div class="cartcontrol-wrapper">
                                    <cartcontrol :food="food"></cartcontrol> <!--food传递到子组件cartcontrol中去。在子组件中去给food添加count属性-->
                                </div>
                            </div>

                        </li>
                    </ul>
                </li>
            </ul>
        </div>
        <!--父组件给子组件传递值，需要注意，一定要使用中划线, v-ref父组件引用子组件-->
        <shopcart v-ref:shopcart :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" :select-foods="selectFoods"></shopcart>
    </div>
</template>

<script type="text/ecmascript-6">

    import BScroll from 'better-scroll'
    import shopcart from 'components/shopcart/shopcart'
    import cartcontrol from 'components/cartcontrol/cartcontrol'

    const ERR_OK = 0;
    export default{
        props: {
            seller: {
                type: Object
            }
        },
        components:{
            shopcart,
            cartcontrol
        },
        data(){
            return {
                goods: [],
                listHeight: [],  //存放每个商品分类区间的高度
                scrollY: 0
            }
        },
        created(){

            this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
            this.$http.get("/api/goods").then((response) => {
                response = response.body;
                if (response.errno === ERR_OK) {  // 请求状态
                    this.goods = response.data;  // 数据保存在goods下

                    // 注意，这里为什么使用vue自带的这个函数$nextTick？？？， 因为滚动组件需要计算高度，vue执行填充数据之后(异步)，再计算高度
                    this.$nextTick(() => {  //注意：dom重新渲染之后的回调，保证dom渲染了之后才计算
                        this._initScroll();
                        this._calculateHight();
                    })
                }
            });
        },
        computed: {
            currentIndex (){ // 利用vue的计算属性，当scrollY变化的时候。computed会重新去做计算
                for (let i = 0; i < this.listHeight.length; i++) {
                    let height1 = this.listHeight[i];
                    let height2 = this.listHeight[i + 1]; // 遍历到最后一个，超出数组范围，是一个undefine
                    if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
                        return i;
                    }
                }
                return 0;
            },
            selectFoods(){ // 遍历goods中的good,然后遍历good中的food，而food中的count，是在cartcontrol.vue组件中添加的
                let foods = [];
                this.goods.forEach((good)=>{
                    good.foods.forEach((food)=>{
                        if(food.count){ // foods
                            foods.push(food);  //food中的count变化就会通知selectFoods重新计算, 而shopcart购物车中的界面，也是依赖selectFoods变化。从而形成联动
                        }
                    })
                });
                return foods;
            }
        },
        methods: {
            selectMenu(index,event){ // 在pc上和手机上，有兼容性问题。pc上会响应两次click事件。需要下面的判断
                if(!event._constructed){ // better-scroll派发的事件，_constructed为true，浏览器自带的event事件，_constructed为false
                    return;
                }
                let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');
                let el = foodList[index]; // 拿到需要滚动到的右侧元素
                this.foodScroll.scrollToElement(el, 300); // 调用better-scroll的api，滚动到指定元素
            },
            _initScroll(){
                this.menuScroll = new BScroll(this.$els.menuWrapper, {
                    click:true // 因为better-scroll在监听scroll事件的时候，会阻止冒泡。 click为true，会派发一次点击事件
                });
                this.foodScroll = new BScroll(this.$els.foodsWrapper, {
                    probeType: 3,  //api需要传入类型，监听滚动事件
                    click:true // 因为better-scroll在监听scroll事件的时候，会阻止冒泡。 click为true，会派发一次点击事件
                });

                this.foodScroll.on('scroll', (pos) => { // 监听滚动事件
                    this.scrollY = Math.abs(Math.round(pos.y));
                });
            },
            _calculateHight(){
                let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');
                let height = 0;
                this.listHeight.push(height);
                for (let i = 0; i < foodList.length; i++) {
                    let item = foodList[i];
                    height += item.clientHeight;
                    this.listHeight.push(height);
                }

            },
            _drop(target){ // 调用子组件shopcart中的drop方法
                // 体验优化，异步执行下落动画
                this.$nextTick(()=>{
                    this.$refs.shopcart.drop(target); // 通过$refs访问子组件，前提是在子组件间使用了： v-ref:shopcart
                })
            }
        },
        events:{  // 事件监听
            'cart.add'(target){ // 接收shopcart子组件dispatch传递过来的'cart.add'事件
                this._drop(target);
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
                &.current
                    position relative
                    background #fff
                    font-weight 700
                    z-index 10
                    margin-top -1px
                    .text
                        border-none()
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

                    .cartcontrol-wrapper
                        position absolute
                        right 0
                        bottom 12px
</style>