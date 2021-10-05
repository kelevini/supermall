<template>
<div id="detail">
<detail-nav-bar class="detail-nav"  @titleClick='titleClick' ref='nav' ></detail-nav-bar>
<scroll class="content" ref ="scroll" @scroll="contentScroll" :probe-type="3">
<detail-swiper :top-images="topImages"></detail-swiper>
<detail-base-info :goods="goods"></detail-base-info>
<detail-shop-info :shop="shop"></detail-shop-info>
<detail-goods-info :detail-info="detailInfo"></detail-goods-info>
<detail-param-info :param-info="paramInfo"   ref="param"></detail-param-info>
<detail-comment-info :comment-info="commentInfo" ref="comment"  ></detail-comment-info>
<goods-list :goods="recommends"  ref="recommend"></goods-list>
</scroll>
<detail-bottom-bar @addCart='addToCart'></detail-bottom-bar>
</div>
</template>

<script>

  import {getDetail ,Goods,Shop,GoodsParam,getRecommend} from '../../network/detail.js'
  import DetailNavBar from './childComps/DetailNavBar.vue'
  import DetailSwiper from './childComps/DetailSwiper.vue'
  import DetailBaseInfo from './childComps/DetailBaseInfo.vue'
  import DetailShopInfo from './childComps/DetailShopInfo.vue'
  import DetailGoodsInfo from './childComps/DetailGoodsInfo.vue'
  import Scroll from'components/common/scroll/Scroll.vue'
  import DetailParamInfo from './childComps/DetailParamInfo.vue'
  import DetailCommentInfo from './childComps/DetailCommentInfo.vue'
  import GoodsList from'components/content/goods/GoodsList.vue'
  import DetailBottomBar from './childComps/DetailBottomBar.vue'
   import {backTopMixin} from "@/common/mixin";
    import {BACKTOP_DISTANCE} from "@/common/const";
    import {debounce} from "@/common/utils.js";
  export default{
     name:"Detail",
    data() {
      return {
      iid:null,
      topImages:[],
      goods:{},
      shop:{},
      detailInfo:{},
      paramInfo:{},
      commentInfo:{},
      recommends:[],
      themeTopYs:[],
      getThemeTopY:null,
      currentIndex:0
    }
  },
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    Scroll,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    GoodsList,
    DetailBottomBar
  },
  created(){

    //保存传入得iid
    this.iid=this.$route.params.iid
    //根据iid请求详情数据
    getDetail(this.iid).then(res=>{
      //获取顶部轮播图数据
      // console.log(res)
      const data=res.data.result

      this.topImages=data.itemInfo.topImages
      // this.$router.go(0)

      //获取商品信息
      this.goods=new Goods(data.itemInfo,data.columns,data.shopInfo.services)
      //创建店铺信息得对象
       this.shop=new Shop(data.shopInfo)
       //保存商品得详情数据
       this.detailInfo=data.detailInfo;
       //获取参数的信息
       this.paramInfo =new GoodsParam(data.itemParams.info,data.itemParams.rule)
       //获取评论信息
        if(data.rate.cRate!=0){
          this.commentInfo=data.rate.list[0]
        }
    })

    // 3.请求推荐数据
    getRecommend().then(res=>{
     this.recommends=res.data.data.list
    })

    this.getThemeTopY= debounce(() =>{

        this.themeTopYs=[]
        this.themeTopYs.push(0)
        this.themeTopYs.push(this.$refs.param.$el.offsetTop)
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)



    })
  },

methods: {
  titleClick(index) {
    this.getThemeTopY()
   this.$refs.scroll.scrollTo(0,-this.themeTopYs[index],100)
    },

    contentScroll(position){
    const positionY= -position.y
    //console.log(positionY)
    let length =this.themeTopYs.length
    for (var i = 0; i < length; i++) {
      if(this.currentIndex!==i&&((i<length-1&&positionY>this.themeTopYs[i]&&positionY<this.themeTopYs[i+1])
      ||(i===length-1&&positionY>=this.themeTopYs[i]))){
        this.currentIndex=i;
        console.log(this.currentIndex)
        this.$refs.nav.currentIndex=this.currentIndex
      }
    }
    },
    addToCart(){
      console.log(1223)
       const product ={}
       product.image=this.topImages[0];
       product.title=this.goods.title;
       product.desc=this.goods.desc;
       product.price=this.goods.newprice;
       product.iid= this.iid;
    }

},



  }

</script>

<style scoped="scoped">
  #detail{
    position: relative;
    z-index: 9;
    background-color: #FFFFFF;
    height: 100vh;
  }
  .detail-nav{
    position: relative;
    z-index: 9;
    background-color: #FFFFFF;
  }

  .content{
    height: calc(100% - 44px - 49px);
  }

</style>
