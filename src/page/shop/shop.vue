 <template>
    <div>
        <section v-if="!showLoading" class="shop_container main_container">
            <!-- <nav class="goback" @click="goback">
                <svg width="4rem" height="100%" xmlns="http://www.w3.org/2000/svg" version="1.1">
                    <polyline points="12,18 4,9 12,0" style="fill:none;stroke:rgb(255,255,255);stroke-width:3"/>
                </svg>
            </nav> -->
            
            <head-top head-title="TRY & BUY" signinUp="user"></head-top>
            
            <section class="change_show_type" ref="chooseType">
                <div>
                    <span :class='{activity_show: changeShowType =="food"}' @click="changeShowType='food'">Tops</span>
                </div>
                <div>
                    <span :class='{activity_show: changeShowType =="rating"}' @click="changeShowType='rating'">KeySmart</span>
                </div>
                <div>
                    <span :class='{activity_show: changeShowType =="Wallet"}' @click="changeShowType='Wallet'">Wallet</span>
                </div>
            </section>
            
            <transition name="fade-choose">

                <section v-show="changeShowType =='food'" class="food_container">
                   
                    <section class="menu_container">

                        <section class="menu_right" ref="menuFoodList">
                            <section class="banner_container">
                                <img src='../../images/banner.png'>
                            </section>
                            <section class="category_container swiper-inner">
                                <swiper :options="swiperOption" ref="mySwiper">
                                <!-- slides -->
                                <swiper-slide>
                                    <div class="swiper-item" style="background-image:url('/static/img/category_1.png')"></div>
                                </swiper-slide>
                                <swiper-slide>
                                    <div class="swiper-item" style="background-image:url('/static/img/category_2.png')"></div>
                                </swiper-slide>
                                <swiper-slide>
                                    <div class="swiper-item" style="background-image:url('/static/img/category_1.png')"></div>
                                </swiper-slide>
                                <swiper-slide>
                                    <div class="swiper-item" style="background-image:url('/static/img/category_2.png')"></div>
                                </swiper-slide>
                                <!-- Optional controls -->
                                <div class="swiper-pagination"  slot="pagination"></div>
                                <!-- <div class="swiper-button-prev" slot="button-prev"></div>
                                <div class="swiper-button-next" slot="button-next"></div>
                                <div class="swiper-scrollbar"   slot="scrollbar"></div> -->
                              </swiper>
                            </section>
                            <ul  class="menu_detail_list">
                                <li v-for="(item,index) in imartgo.products" :key="index">

                                    <div class="product_detail_link">
                                        <section class="product_img">
                                            <img :src="item.mainPicUrl">
                                        </section>
                                        <section class="product_description">
                                            <vue-numeric :precision="2" currency="$" separator="," v-model="item.price" readOnlyClass="product_price"></vue-numeric>
                                            <vue-numeric :precision="2" currency="$" separator="," v-model="item.rrpPrice" readOnlyClass="product_rrpPrice"></vue-numeric>
                                        </section>
                                    </div>

                                </li>
                            </ul>
                        </section>
                    </section>
                </section>
            </transition>
            <transition name="fade-choose">
                <section class="rating_container" id="ratingContainer" v-show="changeShowType =='rating'">
                    <section v-load-more="loaderMoreRating" type="2">
                        
                    </section>
                </section>
            </transition>
        </section>

        <foot-guide></foot-guide>

       <loading v-show="showLoading || loadRatings"></loading>
       <!-- <section class="animation_opactiy shop_back_svg_container" v-if="showLoading">
           <img src="../../images/shop_back_svg.svg">
       </section> -->
       <transition name="router-slid" mode="out-in">
            <router-view></router-view>
        </transition>
    </div>
</template>

<script>
    import {mapState, mapMutations} from 'vuex'
    import {msiteAdress, shopDetails, foodMenu, getRatingList, ratingScores, ratingTags,iMartGo} from 'src/service/getData'
    import loading from 'src/components/common/loading'
    import buyCart from 'src/components/common/buyCart'
    import ratingStar from 'src/components/common/ratingStar'
    import {loadMore, getImgPath} from 'src/components/common/mixin'
    import {imgBaseUrl} from 'src/config/env'
    import BScroll from 'better-scroll'
    import { swiper, swiperSlide } from 'vue-awesome-swiper'
    import VueNumeric from 'vue-numeric'

    import headTop from '../../components/header/head'
    import footGuide from '../../components/footer/footGuide'

    export default {
        data(){
            return{
                geohash: '', //geohash位置信息
                shopId: null, //商店id值
                showLoading: true, //显示加载动画
                changeShowType: 'food',//切换显示商品或者评价
                shopDetailData: null, //商铺详情
                showActivities: false, //是否显示活动详情
                menuList: [], //食品列表
                menuIndex: 0, //已选菜单索引值，默认为0
                menuIndexChange: true,//解决选中index时，scroll监听事件重复判断设置index的bug
                shopListTop: [], //商品列表的高度集合
                TitleDetailIndex: null, //点击展示列表头部详情
                categoryNum: [], //商品类型右上角已加入购物车的数量
                totalPrice: 0, //总共价格
                cartFoodList: [], //购物车商品列表
                showCartList: false,//显示购物车列表
                receiveInCart: false, //购物车组件下落的圆点是否到达目标位置
                ratingList: null, //评价列表
                ratingOffset: 0, //评价获取数据offset值
                ratingScoresData: null, //评价总体分数
                ratingTagsList: null, //评价分类列表
                ratingTageIndex: 0, //评价分类索引
                preventRepeatRequest: false,// 防止多次触发数据请求
                ratingTagName: '',//评论的类型
                loadRatings: false, //加载更多评论是显示加载组件
                foodScroll: null,  //食品列表scroll
                showSpecs: false,//控制显示食品规格
                specsIndex: 0, //当前选中的规格索引值
                choosedFoods: null, //当前选中视频数据
                showDeleteTip: false, //多规格商品点击减按钮，弹出提示框
                showMoveDot: [], //控制下落的小圆点显示隐藏
                windowHeight: null, //屏幕的高度
                elLeft: 0, //当前点击加按钮在网页中的绝对top值
                elBottom: 0, //当前点击加按钮在网页中的绝对left值
                ratingScroll: null, //评论页Scroll
                imgBaseUrl,

                swiperOption: {
                  pagination: '.swiper-pagination',
                  effect: 'coverflow',
                  grabCursor: true,
                  centeredSlides: true,
                  slidesPerView: 'auto',
                  coverflow: {
                      rotate: 50,
                      stretch: 0,
                      depth: 100,
                      modifier: 1,
                      slideShadows : true
                  },
                },
                imartgo:null
            }
        },
        created(){
            this.geohash = this.$route.query.geohash;
            this.shopId = this.$route.query.id || 3;
            this.INIT_BUYCART();
        },
        mounted(){
            this.initData();
            this.windowHeight = window.innerHeight;
        },
        beforeDestroy(){
            // this.foodScroll.removeEventListener('scroll', )
        },
        mixins: [loadMore, getImgPath],
        components: {
            loading,
            ratingStar,
            buyCart,
            headTop,
            footGuide,
            swiper,
            swiperSlide,
            VueNumeric
        },
        computed: {
            ...mapState([
                'latitude','longitude','cartList'
            ]),
            promotionInfo: function (){
                return this.shopDetailData.promotion_info || '欢迎光临，用餐高峰期请提前下单，谢谢。'
            },
            //配送费
            deliveryFee: function () {
                if (this.shopDetailData) {
                    return this.shopDetailData.float_delivery_fee;
                }else{
                    return null;
                }
            },
            //还差多少元起送，为负数时显示去结算按钮
            minimumOrderAmount: function () {
                if (this.shopDetailData) {
                    return this.shopDetailData.float_minimum_order_amount - this.totalPrice;
                }else{
                    return null;
                }
            },
            //当前商店购物信息
            shopCart: function (){
                return {...this.cartList[this.shopId]};
            },
            //购物车中总共商品的数量
            totalNum: function (){
                let num = 0;
                this.cartFoodList.forEach(item => {
                    num += item.num
                })
                return num
            },
        },
        methods: {
            ...mapMutations([
                'RECORD_ADDRESS','ADD_CART','REDUCE_CART','INIT_BUYCART','CLEAR_CART','RECORD_SHOPDETAIL'
            ]),
            //初始化时获取基本数据
            async initData(){
                // if (!this.latitude) {
                //     //获取位置信息
                //     let res = await msiteAdress(this.geohash);
                //     // 记录当前经度纬度进入vuex
                //     this.RECORD_ADDRESS(res);
                // }
                // //获取商铺信息
                // this.shopDetailData = await shopDetails(this.shopId, this.latitude, this.longitude);
                // //获取商铺食品列表
                // this.menuList = await foodMenu(this.shopId);
                // //评论列表
                // this.ratingList = await getRatingList(this.shopId, this.ratingOffset);
                // //商铺评论详情
                // this.ratingScoresData = await ratingScores(this.shopId);
                // //评论Tag列表
                // this.ratingTagsList = await ratingTags(this.shopId);
                // this.RECORD_SHOPDETAIL(this.shopDetailData)
                //隐藏加载动画
                this.imartgo = await iMartGo();
                this.hideLoading();
            },
            gotoAddress(path){
                this.$router.push(path);
            },
            //获取食品列表的高度，存入shopListTop
            getFoodListHeight(){
                // const listContainer = this.$refs.menuFoodList;
                // const listArr = Array.from(listContainer.children[0].children);
                // listArr.forEach((item, index) => {
                //     this.shopListTop[index] = item.offsetTop;
                // });
                // this.listenScroll(listContainer)
            },
            //当滑动食品列表时，监听其scrollTop值来设置对应的食品列表标题的样式
            listenScroll(element){
                // this.foodScroll = new BScroll(element, {
                //     probeType: 3,
                //     deceleration: 0.001,
                //     bounce: false,
                //     swipeTime: 2000,
                //     click: true,
                // });

                // const wrapperMenu = new BScroll('#wrapper_menu', {
                //     click: true,
                // });

                // const wrapMenuHeight = this.$refs.wrapperMenu.clientHeight;
                // this.foodScroll.on('scroll', (pos) => {
                //     if (!this.$refs.wrapperMenu) {
                //         return 
                //     }
                //     this.shopListTop.forEach((item, index) => {
                //         if (this.menuIndexChange && Math.abs(Math.round(pos.y)) >= item) {
                //             this.menuIndex = index;
                //             const menuList=this.$refs.wrapperMenu.querySelectorAll('.activity_menu');
                //             const el = menuList[0];
                //             wrapperMenu.scrollToElement(el, 800, 0, -(wrapMenuHeight/2 - 50));
                //         }
                //     })
                // })
            },
            //控制活动详情页的显示隐藏
            showActivitiesFun(){
                this.showActivities = !this.showActivities;
            },
            //点击左侧食品列表标题，相应列表移动到最顶层
            chooseMenu(index){
                this.menuIndex = index;
                //menuIndexChange解决运动时listenScroll依然监听的bug
                this.menuIndexChange = false;
                this.foodScroll.scrollTo(0, -this.shopListTop[index], 400);
                this.foodScroll.on('scrollEnd', () => {
                    this.menuIndexChange = true;
                })
            },
            showTitleDetail(index){
                if (this.TitleDetailIndex == index) {
                    this.TitleDetailIndex = null;
                }else{
                    this.TitleDetailIndex = index;
                }
            },
            //加入购物车，所需7个参数，商铺id，食品分类id，食品id，食品规格id，食品名字，食品价格，食品规格
            addToCart(category_id, item_id, food_id, name, price, specs){
                this.ADD_CART({shopid: this.shopId, category_id, item_id, food_id, name, price, specs});
            },
            //移出购物车，所需7个参数，商铺id，食品分类id，食品id，食品规格id，食品名字，食品价格，食品规格
            removeOutCart(category_id, item_id, food_id, name, price, specs){
                this.REDUCE_CART({shopid: this.shopId, category_id, item_id, food_id, name, price, specs});
            },
            /**
             * 初始化和shopCart变化时，重新获取购物车改变过的数据，赋值 categoryNum，totalPrice，cartFoodList，整个数据流是自上而下的形式，所有的购物车数据都交给vuex统一管理，包括购物车组件中自身的商品数量，使整个数据流更加清晰
             */
            initCategoryNum(){
                let newArr = [];
                let cartFoodNum = 0;
                this.totalPrice = 0;
                this.cartFoodList = [];
                this.menuList.forEach((item, index) => {
                    if (this.shopCart&&this.shopCart[item.foods[0].category_id]) {
                        let num = 0;
                        Object.keys(this.shopCart[item.foods[0].category_id]).forEach(itemid => {
                            Object.keys(this.shopCart[item.foods[0].category_id][itemid]).forEach(foodid => {
                                let foodItem = this.shopCart[item.foods[0].category_id][itemid][foodid];
                                num += foodItem.num;
                                if (item.type == 1) {
                                    this.totalPrice += foodItem.num*foodItem.price;
                                    if (foodItem.num > 0) {
                                        this.cartFoodList[cartFoodNum] = {};
                                        this.cartFoodList[cartFoodNum].category_id = item.foods[0].category_id;
                                        this.cartFoodList[cartFoodNum].item_id = itemid;
                                        this.cartFoodList[cartFoodNum].food_id = foodid;
                                        this.cartFoodList[cartFoodNum].num = foodItem.num;
                                        this.cartFoodList[cartFoodNum].price = foodItem.price;
                                        this.cartFoodList[cartFoodNum].name = foodItem.name;
                                        this.cartFoodList[cartFoodNum].specs = foodItem.specs;
                                        cartFoodNum ++;
                                    }
                                }
                            })
                        })
                        newArr[index] = num;
                    }else{
                        newArr[index] = 0;
                    }
                })
                this.totalPrice = this.totalPrice.toFixed(2);
                this.categoryNum = [...newArr];
            },
            //控制购物列表是否显示
            toggleCartList(){
                this.cartFoodList.length ? this.showCartList = !this.showCartList : true;
            },
            //清除购物车
            clearCart(){
                this.toggleCartList();
                this.CLEAR_CART(this.shopId);
            },
            //监听圆点是否进入购物车
            listenInCart(){
                if (!this.receiveInCart) {
                    this.receiveInCart = true;
                    this.$refs.cartContainer.addEventListener('animationend', () => {
                        this.receiveInCart = false;
                    })
                    this.$refs.cartContainer.addEventListener('webkitAnimationEnd', () => {
                        this.receiveInCart = false;
                    })
                }
            },
            //获取不同类型的评论列表
            async changeTgeIndex(index, name){
                this.ratingTageIndex = index;
                this.ratingOffset = 0;
                this.ratingTagName = name;
                let res = await getRatingList(this.shopId, this.ratingOffset, name);
                this.ratingList = [...res];
                this.$nextTick(() => {
                    this.ratingScroll.refresh();
                })
            },
            //加载更多评论
            async loaderMoreRating(){
                if (this.preventRepeatRequest) {
                    return
                }
                this.loadRatings = true;
                this.preventRepeatRequest = true;
                this.ratingOffset += 10;
                let ratingDate = await getRatingList(this.shopId, this.ratingOffset, this.ratingTagName);
                this.ratingList = [...this.ratingList,...ratingDate];
                this.loadRatings = false;
                if (ratingDate.length >= 10) {
                    this.preventRepeatRequest = false;
                }
            },
            //隐藏动画
            hideLoading(){
                this.showLoading = false;
            },
            //显示规格列表
            showChooseList(foods){
                if (foods) {
                    this.choosedFoods = foods;
                }
                this.showSpecs = !this.showSpecs;
                this.specsIndex = 0;
            },
            //记录当前所选规格的索引值
            chooseSpecs(index){
                this.specsIndex = index;
            },
            //多规格商品加入购物车
            addSpecs(category_id, item_id, food_id, name, price, specs, packing_fee, sku_id, stock){
                this.ADD_CART({shopid: this.shopId, category_id, item_id, food_id, name, price, specs, packing_fee, sku_id, stock});
                this.showChooseList();
            },
            //显示提示，无法减去商品
            showReduceTip(){
                this.showDeleteTip = true;
                clearTimeout(this.timer);
                this.timer = setTimeout(() => {
                    clearTimeout(this.timer);
                    this.showDeleteTip = false;
                }, 3000);
            },
            //显示下落圆球
            showMoveDotFun(showMoveDot, elLeft, elBottom){
                this.showMoveDot = [...this.showMoveDot, ...showMoveDot];
                this.elLeft = elLeft;
                this.elBottom = elBottom;
            },
            beforeEnter(el){
                el.style.transform = `translate3d(0,${37 + this.elBottom - this.windowHeight}px,0)`;
                el.children[0].style.transform = `translate3d(${this.elLeft - 30}px,0,0)`;
                el.children[0].style.opacity = 0;
            },
            afterEnter(el){
                el.style.transform = `translate3d(0,0,0)`;
                el.children[0].style.transform = `translate3d(0,0,0)`;
                el.style.transition = 'transform .55s cubic-bezier(0.3, -0.25, 0.7, -0.15)';
                el.children[0].style.transition = 'transform .55s linear';
                this.showMoveDot = this.showMoveDot.map(item => false);
                el.children[0].style.opacity = 1;
                el.children[0].addEventListener('transitionend', () => {
                    this.listenInCart();
                })
                el.children[0].addEventListener('webkitAnimationEnd', () => {
                    this.listenInCart();
                })
            },
            goback(){
                this.$router.go(-1);
            }
        },
        watch: {
            //showLoading变化时说明组件已经获取初始化数据，在下一帧nextTick进行后续操作
            showLoading: function (value){
                if (!value) {
                    this.$nextTick(() => {
                        this.getFoodListHeight();
                        this.initCategoryNum();
                    })
                }
            },
            shopCart: function (value){
                this.initCategoryNum();
            },
            //购物车列表发生变化，没有商铺时，隐藏
            cartFoodList: function (value){
                if(!value.length){
                    this.showCartList = false;
                }
            },
            //商品、评论切换状态
            changeShowType: function (value){
                if (value === 'rating') {
                    this.$nextTick(() => {
                        this.ratingScroll = new BScroll('#ratingContainer', {
                            probeType: 3,
                            deceleration: 0.003,
                            bounce: false,
                            swipeTime: 2000,
                            click: true,
                        });
                        this.ratingScroll.on('scroll', (pos) => {
                            if (Math.abs(Math.round(pos.y)) >=  Math.abs(Math.round(this.ratingScroll.maxScrollY))) {
                                this.loaderMoreRating();
                                this.ratingScroll.refresh();
                            }
                        })
                    })
                }
            }
        }
    }
</script>

<style lang="scss" scoped>
    @import 'src/style/mixin';
    @keyframes mymove{
       0%   { transform: scale(1) }
       25%  { transform: scale(.8) }
       50%  { transform: scale(1.1) }
       75%  { transform: scale(.9) }
       100% { transform: scale(1) }
    }
    @-moz-keyframes mymove{
       0%   { transform: scale(1) }
       25%  { transform: scale(.8) }
       50%  { transform: scale(1.1) }
       75%  { transform: scale(.9) }
       100% { transform: scale(1) }
    }
    @-webkit-keyframes mymove{
       0%   { transform: scale(1) }
       25%  { transform: scale(.8) }
       50%  { transform: scale(1.1) }
       75%  { transform: scale(.9) }
       100% { transform: scale(1) }
    }
    @-o-keyframes mymove{
       0%   { transform: scale(1) }
       25%  { transform: scale(.8) }
       50%  { transform: scale(1.1) }
       75%  { transform: scale(.9) }
       100% { transform: scale(1) }
    }
    .shop_back_svg_container{
        position: fixed;
        @include wh(100%, 100%);
        img{
            @include wh(100%, 100%);
        }
    }
    .shop_container{
        display: flex;
        flex-direction: column;
        position: absolute;
        right: 0;
        left: 0;
        height: 100%;
    }
    .goback{
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 2rem;
        z-index: 11;
        padding-top: 0.2rem;
        padding-left: 0.2rem;
    }
    .shop_detail_header{
        overflow: hidden;
        position: relative;
        .header_cover_img{
            width: 100%;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 9;
            filter: blur(10px);
        }
        .description_header{
            position: relative;
            z-index: 10;
            background-color: #152031;
            /*background:-moz-linear-gradient(bottom,#fc3c3f,#ff726f);
            background:-webkit-gradient(linear,0 0%,100% 100%,from(#fc3c3f),to(#ff726f));Old gradient for webkit  
            background:-webkit-linear-gradient(bottom,#fc3c3f,#ff726f);
            background:-o-linear-gradient(bottom,#fc3c3f,#ff726f);*/
            padding: 0.6rem 0.8rem 0.6rem 0.8rem;
            width: 100%;
            overflow: hidden;
            .description_top{
                display: flex;
                margin-top:0.6rem;
                .description_left{
                    margin-right: 0.3rem;
                    img{
                        @include wh(2.9rem, 2.9rem);
                        display: block;
                        border-radius: 0.15rem;
                    }
                }
                .description_right{
                    flex: 1;
                    .description_title{
                        @include sc(.7rem, #fff);
                        /*font-weight: bold;*/
                        width: 100%;
                        margin-top:0.5rem;
                        /*margin-bottom: 0.3rem;*/
                    }
                    .description_text{
                        @include sc(.5rem, #fff);
                        /*margin-bottom: 0.3rem;*/
                    }
                    .description_promotion{
                        @include sc(.5rem, #fff);
                        width: 11.5rem;
                    }
                }
                .description_arrow{
                    @include ct;
                    right: 0.3rem;
                    z-index: 11;
                }
                .shop_detail_vip{
                    display: inline-block;
                    color:#fff;
                    font-size: 12px;
                    line-height: 12px;
                    padding:10px;
                    padding-right:54px;
                    margin-top:20px;

                    background-color:#ff5a59;
                    @include bis('../../images/vip.jpg');
                    background-size: 34px auto;
                    background-position: 68px center;

                    border-radius: 6px;
                    border:1px solid rgba(0,0,0,0.5);
                    border-width:0 0 0.025rem 0;
                }
            }
            .description_footer{
                @include fj;
                margin-top: 0.5rem;
                padding-right: 1rem;
                p{
                    @include sc(.5rem, #fff);
                    span{
                        color: #fff;
                    }
                    .tip_icon{
                        padding: 0 .04rem;
                        border: 0.025rem solid #fff;
                        border-radius: 0.1rem;
                        font-size: .4rem;
                        display: inline-block;
                    }
                }
                .ellipsis{
                    width: 84%;
                }
                .footer_arrow{
                    @include wh(.45rem, .45rem);
                    position: absolute;
                    right: .3rem;
                }
            }

            &.empty{
                padding: 1rem 0.8rem 1rem 0.8rem;
                .description_top{
                    img{
                        @include wh(3.4rem, 3.4rem);
                    }
                    .shop_detail_vip{
                        display: none;
                    }
                    .description_title {
                        @include sc(.9rem, #fff);
                    }
                }
            }
        }
    }
    .activities_details{
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: #262626;
        z-index: 200;
        padding: 1.25rem;
        .activities_shoptitle{
            text-align: center;
            @include sc(.8rem, #fff);
        }
        .activities_ratingstar{
            display: flex;
            justify-content: center;
            transform: scale(2.2);
            margin-top: .7rem;
        }
        .activities_list{
            margin-top: 1.5rem;
            margin-bottom: 1rem;
            @include sc(.5rem, #fff);
            li{
                margin-bottom: .2rem;
                .activities_icon{
                    padding: 0 .02rem;
                    display: inline-block;
                    border: 0.025rem solid #fff;
                    border-radius: 0.1rem;
                }
                span{
                    color: #fff;
                    line-height: .6rem;
                }
            }
        }
        .activities_shopinfo{
            p{
                line-height: .7rem;
                @include sc(.5rem, #fff);
            }
        }
        .activities_title_style{
            text-align: center;
            margin-bottom: 1rem;
            span{
                @include sc(.5rem, #fff);
                border: 0.025rem solid #555;
                padding: .2rem .4rem;
                border-radius: 0.5rem;
            }

        }
        .close_activities{
            position: absolute;
            bottom: 1rem;
            @include cl;
        }
    }
    .emptyShop{
        @include bis('../../images/empty.png');
        background-color:#fff;
        background-size: 100% auto;
        display: flex;
        align-items: flex-end;
        justify-content: center;
        .start_button{
            @include wh(90%, auto);
            @include sc(.6rem, #fff);
            text-align: center;
            background-color:#fc3c3f;
            padding:.6rem 0;
            border-radius: .2rem;
        }
    }
    .food_container{
        display: flex;
        flex: 1;
        padding-bottom: 2rem;
        flex-direction: column;
        background-color: #fafbfc;
    }
    .menu_container{
        display: flex;
        flex: 1;
        overflow-y: hidden;
        position: relative;
        .menu_right{
            flex: 4;
            overflow-y: auto;
            background-color: #fff;
            
            .banner_container{
                img{
                    width: 100%;
                    margin: 0;
                    padding:0;
                }
            }
            .menu_detail_list{
                display: flex;
                flex-flow: row wrap;

                li{
                    width:49%;
                    margin-right:2%;
                    background-color: #fff;
                    line-height: 1rem;
                    
                    &:nth-of-type(even){
                        margin-right:0;
                    };

                    .product_detail_link{
                        margin-bottom:1rem;

                    }
                    .product_img{
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        width:100%;
                        height:180px;
                        overflow: hidden;

                        img{
                            width:auto;
                            height:100%;
                        }
                    }
                    .product_description{
                        display: flex;
                        .product_price{
                            margin-left:.5rem;
                            font-weight:bold;
                            display: inline;
                            @include sc(.6rem, #ff5f22);
                        }
                        .product_rrpPrice{
                            margin-left:.4rem;
                            padding-left:0;
                            /*font-weight:bold;*/
                            display: inline-block;
                            width:2.5rem;
                            text-decoration: line-through;
                            @include sc(.5rem, #b2b4c6);
                        }
                        .product_stock{

                        }
                    }
                }
            }
            .swiper-inner {
                width: 100%;
            }
            .swiper-slide {
                width: 280px;
                height: 200px;
                .swiper-item{
                    margin: 0 auto;
                    width:200px;
                    height:200px;
                    border:0;
                    @include bis('../../images/category_2.png');
                    background-position: center;
                    background-size: cover;
                }
            }
            .category_container{
                width: 100%;
                display: flex;
                flex-direction: row;
            }

        }
    }
    .buy_cart_container{
        position: absolute;
        background-color: #3d3d3f;
        bottom: 0;
        left: 0;
        z-index: 13;
        display: flex;
        @include wh(100%, 2rem);
        .cart_icon_num{
            flex: 1;
            .cart_icon_container{
                display: flex;
                background-color: #3d3d3f;
                position: absolute;
                padding: .4rem;
                border: 0.18rem solid #444;
                border-radius: 50%;
                left: .5rem;
                top: -.7rem;
                .cart_icon{
                    @include wh(1.2rem, 1.2rem);
                }
                .cart_list_length{
                    position: absolute;
                    top: -.25rem;
                    right: -.25rem;
                    background-color: #ff461d;
                    line-height: .7rem;
                    text-align: center;
                    border-radius: 50%;
                    border: 0.025rem solid #ff461d;
                    min-width: .7rem;
                    height: .7rem;
                    @include sc(.5rem, #fff);
                    font-family: Helvetica Neue,Tahoma,Arial;
                }
            }
            .move_in_cart{
                animation: mymove .5s ease-in-out;
            }
            .cart_icon_activity{
                 background-color: #3190e8;
            }
            .cart_num{
                @include ct;
                left: 3.5rem;

                div{
                    color: #fff;
                }
                div:nth-of-type(1){
                    font-size: .8rem;
                    font-weight: bold;
                    margin-bottom: .1rem;
                }
                div:nth-of-type(2){
                    font-size: .4rem;
                }
            }
        }
        .gotopay{
            position: absolute;
            right: 0;
            background-color: #535356;
            @include wh(5rem, 100%);
            text-align: center;
            display: flex;
            align-items: center;
            justify-content: center;
            .gotopay_button_style{
                @include sc(.7rem, #fff);
                font-weight: bold;
            }
        }
        .gotopay_acitvity{
            background-color: #4cd964;
        }
    }
    .cart_food_list{
        position: fixed;
        width: 100%;
        padding-bottom: 2rem;
        z-index: 12;
        bottom: 0;
        left: 0;
        background-color: #fff;
        header{
            @include fj;
            align-items: center;
            padding: .3rem .6rem;
            background-color: #eceff1;
            svg{
                @include wh(.6rem,.6rem);
                vertical-align: middle;
            }
            h4{
                @include sc(.7rem, #666);
            }
            .clear_cart{
                @include sc(.6rem, #666);
            }
        }
        .cart_food_details{
            background-color: #fff;
            max-height: 20rem;
            overflow-y: auto;
            .cart_food_li{
                @include fj;
                padding: .6rem .5rem;
                .cart_list_num{
                    width: 55%;
                    p:nth-of-type(1){
                        @include sc(.7rem, #666);
                        font-weight: bold;
                    }
                    p:nth-of-type(2){
                        @include sc(.4rem, #666);
                    }
                }
                .cart_list_price{
                    font-size: 0;
                    span:nth-of-type(1){
                        @include sc(.6rem, #f60);
                        font-family: Helvetica Neue,Tahoma;

                    }
                    span:nth-of-type(2){
                        @include sc(.7rem, #f60);
                        font-family: Helvetica Neue,Tahoma;
                        font-weight: bold;
                    }
                }
                .cart_list_control{
                    display: flex;
                    align-items: center;
                    span{
                        display: flex;
                        align-items: center;
                        justify-content: center;
                    }
                    svg{
                        @include wh(.9rem, .9rem);
                        fill: #3190e8;
                    }
                    .specs_reduce_icon{
                        fill: #999;
                    }
                    .cart_num{
                        @include sc(.65rem, #666);
                        min-width: 1rem;
                        text-align: center;
                        font-family: Helvetica Neue,Tahoma;
                    }
                }
            }
        }
    }
    .screen_cover{
        position: fixed;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        background-color: rgba(0,0,0,.3);
        z-index: 11;
    }
    .change_show_type{
        display: flex;
        background-color: #152031;
        /*background:-moz-linear-gradient(left,#fa5c5d,#fcaf9b);
        background:-webkit-gradient(linear,0 50%,100% 50%,from(#fa5c5d),to(#fcaf9b));
        background:-webkit-linear-gradient(left,#fa5c5d,#fcaf9b);
        background:-o-linear-gradient(left,#fa5c5d,#fcaf9b);*/ 
        padding: 0 0 .25rem;
        margin-top:1.95rem;
        div{
            padding:0 .8rem;
            text-align: center;
            span{
                @include sc(.65rem, #6D7C93);
                padding: .2rem .1rem;
                border-bottom: 0rem solid #2894FA;
            }
            .activity_show{
                color: #fff;
                border-width:0.12rem;
            }
        }
    }
    .rating_container{
        flex: 1;
        overflow-y: hidden;
        flex-direction: column;
        p, span, li, time{
            font-family: Helvetica Neue,Tahoma,Arial;
        }
        .rating_header{
            display: flex;
            background-color: #fff;
            padding: .8rem .5rem;
            margin-bottom: 0.5rem;
            .rating_header_left{
                flex: 3;
                text-align: center;
                p:nth-of-type(1){
                    @include sc(1.2rem, #f60);
                }
                p:nth-of-type(2){
                    @include sc(.65rem, #666);
                    margin-bottom: .1rem;
                }
                p:nth-of-type(3){
                    @include sc(.4rem, #999);
                }
            }
            .rating_header_right{
                flex: 4;
                p{
                    font-size: .65rem;
                    line-height: 1rem;
                    display: flex;
                    align-items: center;
                    justify-content: flex-start;
                    span:nth-of-type(1){
                        color: #666;
                        margin-right: .5rem;
                    }
                    .rating_num{
                        width: 3rem;
                        @include sc(.55rem, #f60);
                    }
                    .delivery_time{
                        @include sc(.4rem, #999);
                    }
                }
            }
        }
        .tag_list_ul{
            display: flex;
            flex-wrap: wrap;
            background-color: #fff;
            padding: .5rem;
            li{
                @include sc(.6rem, #6d7885);
                padding: .3rem .3rem;
                background-color: #ebf5ff;
                border-radius: 0.2rem;
                border: 1px;
                margin: 0 .4rem .2rem 0;
            }
            .unsatisfied{
                background-color: #f5f5f5;
                color: #aaa;
            }
            .tagActivity{
                background-color: #3190e8;
                color: #fff;
            }
        }
        .rating_list_ul{
            background-color: #fff;
            padding: 0 .5rem;
            .rating_list_li{
                border-top: 1px solid #f1f1f1;
                display: flex;
                padding: .6rem 0;
                .user_avatar{
                    @include wh(1.5rem, 1.5rem);
                    border: 0.025rem;
                    border-radius: 50%;
                    margin-right: .8rem;
                }
                .rating_list_details{
                    flex: 1;
                    header{
                        display: flex;
                        flex: 1;
                        justify-content: space-between;
                        margin-bottom: .3rem;
                        .username_star{
                            @include sc(.55rem, #666);
                            .username{
                                color: #666;
                                margin-bottom: .2rem;
                            }
                            .star_desc{
                                display: flex;
                                align-items: center;
                                .time_spent_desc{
                                    @include sc(.55rem, #666)
                                    margin-left: .15rem;
                                }
                            }
                        }
                        .rated_at{
                            @include sc(.4rem, #999);
                        }
                    }
                    .food_img_ul{
                        display: flex;
                        flex-wrap: wrap;
                        margin-bottom: .4rem;
                        li{
                            img{
                                @include wh(3rem, 3rem);
                                margin-right: .4rem;
                                display: block;
                            }
                        }
                    }
                    .food_name_ul{
                        display: flex;
                        flex-wrap: wrap;
                        li{
                            @include sc(.55rem, #999);
                            width: 2.2rem;
                            padding: .2rem;
                            border: 0.025rem solid #ebebeb;
                            border-radius: 0.15rem;
                            margin-right: .3rem;
                            margin-bottom: 4px;
                        }
                    }
                }
            }
        }
    }
    .specs_cover{
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: rgba(0,0,0,.4);
        z-index: 17;
    }
    .specs_list{
        position: fixed;
        top: 35%;
        left: 15%;
        width: 70%;
        background-color: #fff;
        z-index: 18;
        border: 1px;
        border-radius: 0.2rem;
        .specs_list_header{
            h4{
                @include sc(.7rem, #222);
                font-weight: normal;
                text-align: center;
                padding: .5rem;
            }
            .specs_cancel{
                position: absolute;
                right: .5rem;
                top: .5rem;
            }
        }
        .specs_details{
            padding: .5rem;
            .specs_details_title{
                @include sc(.6rem, #666);
            }
            ul{
                display: flex;
                flex-wrap: wrap;
                padding: .4rem 0;
                li{
                    font-size: .6rem;
                    padding: .3rem .5rem;
                    border: 0.025rem solid #ddd;
                    border-radius: .2rem;
                    margin-right: .5rem;
                    margin-bottom: .2rem;
                }
                .specs_activity{
                    border-color: #3199e8;
                    color: #3199e8;
                }
            }
        }
        .specs_footer{
            @include fj;
            align-items: center;
            background-color: #f9f9f9;
            padding: 0.5rem;
            border: 1px;
            border-bottom-left-radius: .2rem;
            border-bottom-right-radius: .2rem;
            .specs_price{
                span{
                    color: #ff6000;
                }
                span:nth-of-type(1){
                    font-size: .5rem;
                }
                span:nth-of-type(2){
                    font-size: .8rem;
                    font-weight: bold;
                    font-family: Helvetica Neue,Tahoma;
                }
            }
            .specs_addto_cart{
                @include wh(4rem, 1.3rem);
                background-color: #3199e8;
                border: 1px;
                border-radius: 0.15rem;
                @include sc(.6rem, #fff);
                text-align: center;
                line-height: 1.3rem;
            }
        }
    }
    .show_delete_tip{
        position: fixed;
        top: 50%;
        left: 15%;
        width: 70%;
        transform: translateY(-50%);
        background-color: rgba(0,0,0,.8);
        z-index: 18;
        @include sc(.65rem, #fff);
        text-align: center;
        padding: .5rem 0;
        border: 1px;
        border-radius: 0.25rem;
    }
    .move_dot{
        position: fixed;
        bottom: 30px;
        left: 30px;

        svg{
            @include wh(.9rem, .9rem);
            fill: #3190e8;
        }
    }
    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
    }
    .fade-enter, .fade-leave-active {
        opacity: 0;
    }
    .fade-choose-enter-active, .fade-choose-leave-active {
        transition: opacity .5s;
    }
    .fade-choose-leave, .fade-choose-leave-active {
        display: none;
    }
    .fade-choose-enter, .fade-choose-leave-active {
        opacity: 0;
    }
    .router-slid-enter-active, .router-slid-leave-active {
        transition: all .4s;
    }
    .router-slid-enter, .router-slid-leave-active {
        transform: translate3d(2rem, 0, 0);
        opacity: 0;
    }
    .toggle-cart-enter-active, .toggle-cart-leave-active {
        transition: all .3s ease-out;
    }
    .toggle-cart-enter, .toggle-cart-leave-active {
        transform: translateY(100%);
    }
    .search_container{
        background-color:#fdb7b7;
        
        @include wh(100%, 1.4rem);
        @include sc(.6rem, #fff);
        line-height: 1.4rem;
        text-align: center;
        font-weight: normal;
        margin:14px 0 0 0;
        border-radius: 4px;

    }
    .swiper-container-3d{
        .swiper-slide-shadow-left{
            
        }
        .swiper-slide-shadow-right{
            
        }
    }

</style>
