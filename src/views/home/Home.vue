<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control class='tab-control-fixed'
                 :title="['流行','新款','精选']"
                 @tabClick="tabClick"
                 ref="tabControl1" v-show="isTabControlFixed"/>
    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
      <recommend-view :recommends="recommends"/>
      <feature-view/>
      <tab-control class='tab-control'
                   :title="['流行','新款','精选']"
                   @tabClick="tabClick"
                   ref="tabControl2"/>
      <goods-list :goods="showGoods" />
    </scroll>
    <back-top @click.native="backClick" v-show="isBackTopShow"/>
  </div>


</template>

<script>
  import HomeSwiper from "./childComps/HomeSwiper";
  import RecommendView from "./childComps/RecommendView";
  import FeatureView from "./childComps/FeatureView";

  import NavBar from "@/components/common/navbar/NavBar";
  import TabControl from "@/components/content/tabControl/TabControl";
  import GoodsList from "@/components/content/goods/GoodsList";
  import Scroll from "@/components/common/scroll/Scroll";

  import {getHomeMultidata,getHomeGoods} from "@/network/home";
  // import {debounce} from "common/utils";
  import {itemListenerMixin, backTopMixin} from "@/common/mixin";

  export default {
    name: "Home",
    components: {
      HomeSwiper,
      RecommendView,
      FeatureView,
      NavBar,
      TabControl,
      GoodsList,
      Scroll,
    },
    data() {
      return {
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []},
        },
        currentType:'pop',
        tabOffsetTop: 0,
        isTabControlFixed: false,
        saveY: 0,
        homeItemImageLoad: null,
      }
    },
    mixins: [itemListenerMixin, backTopMixin],
    activated() {
      this.$refs.scroll.scrollTo(0,this.saveY);
      this.$refs.scroll.refresh()
    },
    deactivated() {
      // 保存Y值
      this.saveY = this.$refs.scroll.scrollY();

      // 取消全局事件监听
      this.$bus.$off('itemImageLoad', this.homeItemImageLoad)
    },
    created() {
      // 请求多个数据
      this.getHomeMultidata();//具体的方法逻辑写在methods里
      //请求商品数据
      this.getHomeGoods('pop');
      this.getHomeGoods('new');
      this.getHomeGoods('sell');
      // console.log(this.isTabControlFixed)
    },
    mounted() {

      // this.tabClick(0)
    },
    computed: {
      showGoods(){
        return this.goods[this.currentType].list;
      }
    },
    methods: {
      /**
       * 监听事件相关的方法
       **/
      tabClick(index) {
        switch (index) {
          case 0:
            this.currentType = 'pop';
            break;
          case 1:
            this.currentType = 'new';
            break;
          case 2:
            this.currentType = 'sell';
            break;
        }
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl2.currentIndex = index;
      },

      contentScroll(position) {
        // 判断BackTop是否显示
        this.listenerShowBackTop(position);
        // 决定tabControl是否停留
        this.isTabControlFixed = (-position.y) >= this.tabOffsetTop;

      },
      loadMore() {
        this.getHomeGoods(this.currentType);
      },
      swiperImageLoad() {
        this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop;
      },
      /**
       * 网络请求相关方法
       **/
      getHomeMultidata() {
        getHomeMultidata().then(res => {
          // console.log(res);
          this.banners = res.data.data.banner.list;
          this.recommends = res.data.data.recommend.list;
        })
      },
      getHomeGoods(type) {
        getHomeGoods(type,this.goods[type].page+1).then(res => {
          this.goods[type].list.push(...res.data.data.list);
          this.goods[type].page+=1;

          this.$refs.scroll.finishPullUp()
        })
      }
    }
  }
</script>

<style scoped>
  #home {
    /*padding-top: 44px;*/
    height: 100vh;/*viewport 高度*/
    position: relative;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: white;
    /*!*当使用原生滚动时才生效，在Better Scroll下不生效*!*/
    /*position: fixed;*/
    /*left: 0;*/
    /*right: 0;*/
    /*top: 0;*/
    /*z-index: 9;*/
  }

  .tab-control-fixed {
    position: relative;
    z-index: 9;
  }
  /*!*使tabcontrol到达距离top一定距离时停留不动*!*/
  /*!*在未使用Better Scroll时有用，使用了之后则失效了*!*/
  /*.tab-control {*/
  /*  position: sticky;*/
  /*  top: 44px;*/
  /*}*/

  /*.fixed {*/
  /*  position: fixed;*/
  /*  left:0;*/
  /*  right: 0;*/
  /*  top: 44px;*/
  /*}*/

  .content {
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
</style>
