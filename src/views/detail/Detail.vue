<template>
  <div id="detail">

    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll class="content" ref="scroll" 
    :probe-type="3" 
    @scroll="contentScroll">
      <detail-swiper :top-images="topImages"/>
      <detail-base-info :goods="goods"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <detail-param-info ref="params" :param-info="paramInfo"/>
      <detail-comment-info ref="comment" :comment-info="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>
    </scroll>
    <detail-bottom-bar @addCart="addToCart"/>
    
    <back-top @click.native="backClick" v-show="isShowBacktop"/>

      <!-- <toast :message="message" :show="show"/> -->

  </div>
  
</template>

<script>
import DetailNavBar from './childComps/DetailNavBar'
import DetailSwiper from './childComps/DetailSwiper'
import DetailBaseInfo from './childComps/DetailBaseInfo'
import DetailShopInfo from './childComps/DetailShopInfo'
import DetailGoodsInfo from './childComps/DetailGoodsInfo'
import DetailParamInfo from './childComps/DetailParamInfo'
import DetailCommentInfo from './childComps/DetailCommentInfo'
import DetailBottomBar from './childComps/DetailBottomBar'



import Scroll from 'components/common/scroll/Scroll'
import GoodsList from 'components/content/goods/GoodsList'
// import Toast from 'components/common/toast/Toast'

import { getDetail,Goods,Shop,GoodsParam,getRecommend } from 'network/detail'
import {debounce} from 'common/utils'
import {itemListenerMixin,backTopMixin} from "common/mixin"

import { mapActions } from 'vuex'

export default {
 
    name:"Detail",
     components: { 
          DetailNavBar,
          DetailSwiper,
          DetailBaseInfo,
          DetailShopInfo,
          DetailGoodsInfo,
          DetailParamInfo,
          DetailCommentInfo,
          DetailBottomBar,
          Scroll,
          GoodsList,
          // Toast,
          
     },
     mixins: [itemListenerMixin,backTopMixin],
    data() {
        return {
            iid:null,
            topImages:[],
            goods: {},
            shop:{},
            detailInfo:{},
            paramInfo:{},
            commentInfo:{},
            recommends: [],
            themeTopYs:[],
            getThemeTopY: null,
            currentIndex:0,
            // message:'',
            // show: false
        }
    },
    created() {
        // 1.???????????????iid
        this.iid = this.$route.params.iid
        // 2.??????iid???????????????
        getDetail(this.iid).then(res =>{
            // console.log(res);
            
            const data = res.result;
            this.topImages = res.result.itemInfo.topImages
            // 2.??????????????????
            this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)
            // 3.???????????????????????????
            this.shop = new Shop(data.shopInfo)
            // 4.???????????????????????????
            this.detailInfo = data.detailInfo;
            // 5.?????????????????????
            this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule)
            // 6.?????????????????????
            if(data.rate.cRate !==0){
              this.commentInfo = data.rate.list[0]
            }

            // 1.??????????????????????????????
            // ?????????????????????this.$refs.params.$el??????????????????
            //   this.themeTopYs = []

            //   this.themeTopYs.push(0)
            //   this.themeTopYs.push(this.$refs.params.$el.offsetTop)
            //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
            //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)

            //   console.log(this.themeTopYs);

            // this.$nextTick(() =>{
              // 2.???????????????????????????
              // ?????????????????????????????????????????????
              // ?????????????????????????????????DOM????????????????????????
              // ?????????????????????????????????????????????????????????offsetTop??????????????????????????????
              // offsetTop?????????????????????????????????????????????

            //   this.themeTopYs = []

            //   this.themeTopYs.push(0)
            //   this.themeTopYs.push(this.$refs.params.$el.offsetTop)
            //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
            //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)

            //   console.log(this.themeTopYs);
            // })
        })
        // 3.??????????????????
        getRecommend().then(res => {
        // console.log(res);
        this.recommends = res.data.list
      });
      // 4.???getThemeTopY??????
        this.getThemeTopY = debounce(() =>{
          this.themeTopYs  = []
          this.themeTopYs.push(0)
          this.themeTopYs.push(this.$refs.params.$el.offsetTop)
          this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
          this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
          this.themeTopYs.push(Number.MAX_VALUE)
        },  100)
    },
   mounted(){

  },
  destroyed(){
    this.$bus.$off('itemImgLoad',this.itemImgListener)
  },
    methods: {
      ...mapActions(['addCart']),
      imageLoad(){
        this.newRefresh()
        // this.$refs.scroll.refresh()
        
        this.getThemeTopY()
      },
      titleClick(index) {
        // console.log(index);
        this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200)
      },
      contentScroll(position) {
        // 1.??????y???
        const positoinY = -position.y

        // 2.positionY???????????????????????????
        // [ 0, 11258, 11849, 12045???Number.MAX_VALUE]
        // console.log(Number.MAX_VALUE);
        
        // positoinY ??? 0 ??? 11258?????? ???index = 0
        // positoinY ??? 11258 ??? 11849?????? ???index = 1
        // positoinY ??? 11849 ??? 12045?????? ???index = 2
        // positoinY ??????9120 ???????????????????????????index = 3

        let length = this.themeTopYs.length
        for(let i = 0; i < this.themeTopYs.length; i++) {
          // if (positoinY > this.themeTopYs[parseInt(i)] && positoinY < this.themeTopYs[i+1]){
          //   console.log(i);
            
          // }
          if(this.currentIndex !== i && (positoinY >= this.themeTopYs[i] && positoinY < this.themeTopYs[i+1])){
            this.currentIndex = i;
            this.$refs.nav.currentIndex = this.currentIndex
          }
          // if (this.currentIndex !== i && ((i < length - 1 && positoinY >= this.themeTopYs[i] && positoinY < this.themeTopYs[i+1]) || (i === length -1 && positoinY > this.themeTopYs[i]))){
          //   this.currentIndex = i;
          //   this.$refs.nav.currentIndex = this.currentIndex
          // }
        }
       
        // 3.????????????????????????
        
      this.listenShoBackTop(position)
      },
      addToCart(){
        // 1.??????????????????????????????
        const product = {}
          product.image = this.topImages[0]
          product.title = this.goods.title
          product.desc = this.goods.desc
          product.price = this.goods.realPrice
          product.iid = this.iid
        // 2.??????????????????????????????
        // this.$store.commit('addCart',product)
        this.addCart(product).then(res =>{
          // this.show = true;
          // this.message = res;
          
          // setTimeout(() =>{
          //   this.show = false;
          //   this.message = ''
          // }, 1500)
          this.$toast.show()
          
        })

        // this.$store.dispatch('addCart',product).then(res =>{
        //   console.log(res);
        // })
      }
      
    }
} 
</script>

<style scoped>
  #detail {
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
  }
  .detail-nav{
    position: relative;
    z-index: 9;
    background-color: #fff;
  }
  .content{
    background-color: #fff;
    height: calc(100% - 44px - 49px);

  }
</style>