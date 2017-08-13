<template>
    <div class="food" v-show="showFlag" transition="move">
        <div class="food-content">
            <div class="image-header">
                <img :src="food.image">
                <div class="back" @click="hide">
                    <i class="icon-arrow_lift"></i>
                </div>
            </div>
            <div class="content">
                <h1 class="title">{{food.name}}</h1>
                <div class="detail">
                    <span class="sell-count">月售{{food.sellCount}}份</span>
                    <span class="rating">好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                    <span class="now">￥{{food.price}}</span>
                    <span class="old" v-show="food.oldPrice">{{food.oldPrice}}</span>
                </div>
            </div>
        </div>
    </div>
</template>

<script type="text/ecmascript-6">
    export default{
        props: {
            food: {
                type: Object
            }
        },
        data(){
            return {
                showFlag: false
            }
        },
        methods: {
            show(){
                this.showFlag = true;
            },
            hide(){
                this.showFlag = false;
            }
        }
    }
</script>

<style lang="stylus" rel="stylesheet/stylus">
    .food
        position fixed
        left 0
        top 0
        bottom 48px
        z-index 30
        width 100%
        background #fff
        &.move-transition
            transition all 0.2s linear
            transform translate3d(0, 0, 0)
        &.move-enter, &.move-leave
            transform: translate3d(100%, 0, 0)
        .image-header
            position relative
            width 100%
            height 0
            padding-top 100%  /*这里使用了黑科技： 在图片未加载进来的时候，使用padding（需要注意的是，padding在使用百分比的时候，其实是以父元素的宽度来参照的）
            来撑开高度，防止盒模型的变形*/
            img
                position absolute
                top 0
                left 0
                width 100%
                height 100%
            .back
                position: absolute
                top 10px
                left 0
                .icon-arrow_lift
                    display block
                    padding 10px /*小技巧，通过padding加大可点击区域*/
                    font-size 20px
                    color #fff
</style>