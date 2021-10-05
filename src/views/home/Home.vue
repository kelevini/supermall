<template>
  <div id="home" >
      <NavBar class='home-nav'><div slot='center' > 购物街</div></NavBar>
      <tab-control  :titles="['流行','新款','精选']"
        @tabClick="tabClick" ref='tabControl1' class="tab-control" v-show="isTabFixed"></tab-control>
    <Scroll class="content"
    ref='scroll' :probe-type='3'
      @scroll="contentScroll"
      :pull-up-load="true"
       @pullingUp='loadMore'>
      <home-swiper :banners="banners"
       @swiperImageLoad='swiperImageLoad'
      ></home-swiper>
      <recommend-view :recommends='recommends'></recommend-view>
      <feature></feature>
      <tab-control  :titles="['流行','新款','精选']"
        @tabClick="tabClick" ref='tabControl2' ></tab-control>
      <goods-list :goods="showGoods" class="header"></goods-list>

    </Scroll>
    
      <back-top @click.native='backClick' v-show="isShowBackTop"></back-top>


  </div>
</template>

<script>
  import NavBar from'components/common/navbar/NavBar.vue'
  import HomeSwiper from'./childComps/HomeSwiper.vue'
  import RecommendView from './childComps/RecommendView'
  import Feature from'./childComps/FeatureView.vue'
  import tabControl from'components/content/tabControl/tabControl.vue'
  import GoodsList from'components/content/goods/GoodsList.vue'
  import Scroll from'components/common/scroll/Scroll.vue'
  import BackTop from'components/content/backTop/BackTop.vue'

  import {getHomeMultidata,getHomeGoods} from 'network/home.js'
  export default {
    name: "Home",

    computed:{
     showGoods(){
       return this.goods[this.currentType].list
     }
    },
    activated(){
    this.$refs.scroll.scrollTo(0,this.saveY,0)
    //this.$refs.scroll.refresh()
    },
    deactivated(){
      this.saveY=this.$refs.scroll.getScrollY
    },
    components:{
      NavBar,
      HomeSwiper,
      RecommendView,
      Feature,
      tabControl,
      GoodsList,
      Scroll,
      BackTop

    },

    data() {
      return {
       banners:[],
       recommends:[],
       goods:{
         'pop':{page:0,list:[]},
         'new':{page:0,list:[]},
         'sell':{page:0,list:[]},
       },
       currentType:'pop',
      tabOffsetTop:0,
      isShowBackTop:false,
      isTabFixed:false,
      saveY:0
      }
    },
    created() {
        this.getHomeMultidata()
        this. getHomeGoods('pop')
        this. getHomeGoods('new')
        this. getHomeGoods('sell')

    },
    mounted(){
      this.$bus.$on('itemImageLoad',()=>{
        this.$refs.scroll.refresh()
      })


    },
    methods:{
    /*
    事件监听得事件
    */
    tabClick(index){
     switch(index){
       case 0:
       this.currentType='pop'
       break
       case 1:
       this.currentType='new'
       break
       case 2:
       this.currentType='sell'
       break
     }
      this.$refs.tabControl1.currentIndex=index;
      this.$refs.tabControl2.currentIndex=index;
    },
      /*
       网络请求相关的方法
      */
       getHomeMultidata(){
         getHomeMultidata().then(res=>{
           this.banners=res.data.data.banner.list;
           this.recommends=res.data.data.recommend.list;
         })
       },
       getHomeGoods(type){
         const page =this.goods[type].page+1
         getHomeGoods(type,page).then(res=>{
           
            this.goods[type].list.push(...res.data.data.list)
            this.goods[type].page+=1
            //完成上拉加载更多
            this.$refs.scroll.finishPullUp()
         })
       },
       backClick(){
        this.$refs.scroll.scrollTo(0,0,500)
       },

      contentScroll(position){
       this.isShowBackTop=(-position.y)>1000

        this.isTabFixed=(-position.y)>this.tabOffsetTop
      },
      loadMore(){
       this.getHomeGoods(this.currentType)
      },
      swiperImageLoad(){
      this.tabOffsetTop=this.$refs.tabControl2.$el.offsetTop
      }

    },


  }
</script>

<style scoped>
  #home{
    height: 100vh;
    position: relative;
  }
.home-nav{
  background-color: pink;
  color: #FFFFFF;
 /* position: fixed;
  left: 0px;
  right: 0px;
  top: 0rem;
  z-index: 9; */
}
.header{
  margin-bottom:50px ;
}

.content{
  position: absolute;
  overflow: hidden;
  top: 44px;
  bottom: 49px;
  left: 0px;
  right: 0px;
}
.tab-control{
  position: relative;
  z-index: 9;
}
</style>
