<template>
    <div class="main-content">
        <v-header :seller="seller"></v-header>

        <div class="tab border-1px">
            <div class="tab-item">
                <!-- 使用 router-link 组件来导航. -->
                <!-- 通过传入 `to` 属性指定链接. -->
                <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
                <router-link to="/goods">商品</router-link>
            </div>
            <div class="tab-item">
                <router-link to="/ratings">评价</router-link>
            </div>
            <div class="tab-item">
                <router-link to="/seller">商家</router-link>
            </div>
        </div>
        <!-- 路由出口 -->
        <!-- 路由匹配到的组件将渲染在这里 -->
        <router-view :seller="seller"></router-view>
    </div>
</template>

<script type="text/ecmascript-6">
    import header from './components/header/header.vue';

    export default {
        data () {
            return {
                seller: {},
                goods: []
            };
        },
        created () {
            this.$http.get('/api/seller').then((response) => {
                response = response.body;
                if (response.errno === 0) {
                    this.seller = response.data;
                    console.log(this.seller);
                }
            });
        },
        components: {
            'v-header': header
        }
    };
</script>

<style lang="stylus" rel="stylesheet/stylus">
    @import "./common/stylus/mixin.styl"

    .main-content
        .tab
            display: flex
            width: 100%
            height: 40px
            line-height: 40px
            // border-bottom: 1px solid rgba(7,17,27,0.1)
            border-1px(rgba(7,17,27,0.1))
            .tab-item
                flex: 1
                text-align: center
                & > a
                    display: block
                    font-size: 14px
                    color: rgb(77,85,93)
                    &.active
                        color: rgb(240,20,20)

</style>
