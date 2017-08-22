<template>

    <div>
        <v-header :seller="seller"></v-header>  <!--传入获取到的seller到子组件header-->
        <div class="tab border-1px">
            <div class="tab-item"><a v-link="{path:'/goods'}">商品</a></div>
            <div class="tab-item"><a v-link="{path:'/ratings'}">评论</a></div>
            <div class="tab-item"><a v-link="{path:'/seller'}">商家</a></div>
        </div>
        <!--keep-alive在切换vue-router的时候。保留各个视图的状态。各个router-link也不会去重新请求后台数据  -->
        <router-view :seller="seller" keep-alive></router-view>
    </div>
</template>

<script type="text/ecmascript-6">

    import {urlParse} from 'common/js/util'
    import header from 'components/header/header.vue'

    const ERR_OK = 0;

    export default {

        data(){
            return {
                seller:{
                    id:(()=>{
                        let queryParam = urlParse();
                        return queryParam.id;
                    })()
                }
            }
        },

        created(){
            this.$http.get('/api/seller?id='+this.seller.id).then((response) => {
                response = response.body;
                if(response.errno === ERR_OK){  // 请求状态
                    //this.seller = response.data;
                    this.seller = Object.assign({}, this.seller, response.data); //使用assign扩展对象属性，vue官网推荐的方法.
                }
            });
        },

        components: {
            'v-header': header
        }

    };

</script>

<style lang="stylus" rel="stylesheet/stylus">

    @import "common/stylus/mixin.styl"

    .tab
        display: flex
        width: 100%
        height: 40px
        line-height 40px
    //border-bottom 1px solid rgba(7,17,27,0.1)
        border-1px(rgba(7, 17, 27, 0.1))
        .tab-item
            flex: 1
            text-align: center
            & a
                display: block
                font-size: 14px
                color: rgb(77, 85, 93)
            & .active
                color: rgb(240, 20, 20)

</style>
