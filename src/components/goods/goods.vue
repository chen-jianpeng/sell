<template>
    <div class="goods">
        <div class="menu-wraper" ref="menuWrapper">
             <ul>
                <li v-for="(item,index) in goods" :key="item.name" class="menu-item" :class="{'current':currentIndex === index}" 
                @click="selectMenu(index,$event)">
                    <span class="text border-1px" >
                        <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
                    </span>
                </li>
            </ul>
        </div>
        <div class="food-wraper" ref="foodWrapper">
            <ul>
                <li v-for="item in goods" :key="item.name" class="food-list food-list-hook">
                    <h1 class="title">{{item.name}}</h1>
                    <ul>
                        <li v-for="food in item.foods" :key="food.name" class="food-item">
                            <div class="icon">
                                <img width="57" height="57" :src="food.icon">
                            </div>
                            <div class="content">
                                <h2 class="name">{{food.name}}</h2>
                                <p class="desc">{{food.description}}</p>
                                <div class="extra">
                                    <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                                </div>
                                <div class="price">
                                    <span class="new">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                                </div>
                                <div class="cartcontrol-wrap">
                                    <cartcontrol v-on:add-food="getEvent" :food="food"></cartcontrol>
                                </div>
                            </div>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
        <shopcart :selectFoods="selectFoods"  ref="shopcart" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    </div>
</template>

<script type="text/ecmascript-6">
    import BScroll from 'better-scroll';
    import shopcart from '../shopcart/shopcart';
    import cartcontrol from '../cartcontrol/cartcontrol';

    const ERR_OK = 0;

    export default {
        props: {
            seller: {
                type: Object
            }
        },
        data () {
            return {
                goods: [],
                classMap: [],
                listHeight: [],
                scrollY: 0,
                $index: 0
            };
        },
        computed: {
            currentIndex () {
                for (let i = 0; i < this.listHeight.length; i++) {
                    let height1 = this.listHeight[i];
                    let height2 = this.listHeight[i + 1];
                    if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
                        return i;
                    };
                };
                return 0;
            },
            selectFoods () {
                let foods = [];
                this.goods.forEach((good) => {
                    good.foods.forEach((food) => {
                        if (food.count) {
                            foods.push(food);
                        }
                    });
                });
                return foods;
            }
        },
        created () {
            this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
            this.$http.get('/api/goods').then((response) => {
                response = response.body;
                if (response.errno === ERR_OK) {
                    this.goods = response.data;
                    // 确保dom已加载完成
                    this.$nextTick(() => {
                        this._initScroll();
                        this._calculateHeight();
                    });
                }
            });
        },
        methods: {
            selectMenu (index, event) {
                /* 浏览器点击事件和BScroll的点击事件，在浏览器中会被执行两次，
                BScroll点击事件会有_constructed属性，因此可以用来予以区分，只相应BScroll的点击事件 */
                if (!event._constructed) {
                    return;
                }
                let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook');
                let el = foodList[index];
                this.foodScroll.scrollToElement(el);
            },
            _initScroll () {
                this.menuScroll = new BScroll(this.$refs.menuWrapper, {
                    click: true
                });
                this.foodScroll = new BScroll(this.$refs.foodWrapper, {
                    click: true,
                    probeType: 3
                });
                this.foodScroll.on('scroll', (pos) => {
                    this.scrollY = Math.abs(Math.round(pos.y));
                });
            },
            _calculateHeight () {
                let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook');
                let height = 0;
                this.listHeight.push(height);
                for (let i = 0; i < foodList.length; i++) {
                    let item = foodList[i];
                    height += item.clientHeight;
                    this.listHeight.push(height);
                };
            },
            getEvent: function (el) {
                // 体验优化,异步执行下落动画
                this.$nextTick(() => {
                    this.$refs.shopcart.drop(el);
                });
            }
        },
        components: {
            cartcontrol,
            shopcart
        }
    };
</script>

<style lang="stylus" rel="stylesheet/stylus">
    @import '../../common/stylus/mixin.styl';
    
    .goods
        display:flex
        position:absolute
        top:174px
        bottom:46px
        width:100%
        overflow:hidden
        .menu-wraper
            flex: 0 0 80px
            width: 80px
            background: #f3f5f7
            .menu-item
                display: table
                height: 54px
                width: 56px
                padding: 0 12px
                line-height: 14px
                &.current
                    position: relative
                    z-index: 10
                    margin-top: -1px
                    background: #fff
                    font-weight: 700
                    .text
                        border-none() 
                .text
                    display: table-cell
                    width: 56px
                    vertical-align: middle
                    border-1px(rgba(7, 17, 27, 0.1))
                    font-size: 12px
                    .icon
                        display: inline-block
                        vertical-align: top
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
        .food-wraper
            flex: 1
            .title
                padding-left:14px
                height: 26px
                line-height: 26px
                border-left: 2px solid #d9dde1
                color: rgb(147,153,159)
                font-size: 12px
                background: #f3f5f7
            .food-item
                display: flex
                margin: 10px
                padding-bottom: 10px
                border-1px(rgba(7,17,27,0.1))
                &:last-child
                    border-none()
                    margin-bottom: 0
                .icon
                    flex: 0 0 57px
                    margin-right: 10px
                .content
                    flex: 1
                    .name
                        margin-top: 2px
                        line-height: 14px
                        font-size:14px
                        color: rgb(7,17,27)
                    .desc,.extra
                        margin-top: 8px
                        line-height:10px
                        font-size: 10px
                        color: rgb(147,153,159)
                    .extra
                        .count
                            margin-right: 12px
                    .price
                        margin-top: 8px
                        line-height: 16px
                        font-weight: 700
                        vertical-align: middle
                        .new
                            margin-right: 8px
                            font-size: 16px
                            color: rgb(240,20,20)
                        .old
                            text-decoration: line-through
                            font-size: 12px
                            color: rgb(147,153,159)
                    .cartcontrol-wrap
                        position: absolute
                        right: 0
                        bottom: 1px
</style>
