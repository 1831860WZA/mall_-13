<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    <tab-control
        class="tab-control"
        :titles="['流行', '新款', '精选']"
        @tabClick="tabClick"
        ref="tabControl1"
        v-show="isTabFixed"/>
    <scroll
      class="content"
      ref="scroll"
      :probe-type="3"
      @scroll="contentScroll"
      :pull-up-load="true"
      @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad" />
      <recommend-view :recommends="recommends" />
      <feature-view />
      <tab-control
        class="tab-control"
        :titles="['流行', '新款', '精选']"
        @tabClick="tabClick"
        ref="tabControl2"/>
      <goods-list :goods="goods[currentType].list" />
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop" />
  </div>
</template>

<script>
import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";
import GoodsList from "components/content/goods/GoodsList";
import Scroll from "components/common/scroll/Scroll";
import BackTop from "components/content/backTop/BackTop";

import HomeSwiper from "views/home/childComps/HomeSwiper";
import RecommendView from "views/home/childComps/RecommendView";
import FeatureView from "views/home/childComps/FeatureView";

import { getHomeMultidata, getHomeGoods } from "network/home";
import { debounce } from 'common/utils'

export default {
  name: "Home",
  data() {
    return {
      // 轮播图数据
      banners: [],
      // 推荐模块数据
      recommends: [],
      // 商品列表数据
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] },
      },
      // 默认请求 'pop'(流行) 的相关数据
      currentType: "pop",
      // 是否显示backTop
      isShowBackTop: false,
      // tabControl距离顶部的距离
      tabOffsetTop: 0,
      // 默认不显示 tabControl1
      isTabFixed: false,
      saveY: 0
    };
  },
  components: {
    NavBar,
    HomeSwiper,
    RecommendView,
    FeatureView,
    TabControl,
    GoodsList,
    Scroll,
    BackTop,
  },
  methods: {
    getHomeMultidata() {
      getHomeMultidata().then((res) => {
        // console.log(res.data);
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },

    getHomeGoods(type) {
      //  第一次请求时 请求第1页的数据
      const page = this.goods[type].page + 1;
      getHomeGoods(type, page).then((res) => {
        // console.log(res);
        this.goods[type].list.push(...res.data.list);
        // 页数加1
        this.goods[type].page += 1;

        // 完成下拉加载更多
        this.$refs.scroll.finishPullUp();
      });
    },

    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
          break;
      }
      // console.log(index);
      this.$refs.tabControl1.currentIndex = index;
      this.$refs.tabControl2.currentIndex = index;
    },
    backClick() {
      // 回到顶部
      this.$refs.scroll.scrollTo(0, 0, 500);
    },
    contentScroll(position) {
      // console.log(position);
      // 在纵轴坐标大于 1000 时显示 backTop
      this.isShowBackTop = -position.y > 1000;
      // 判断tabControl的显示与隐藏
      this.isTabFixed = -position.y > this.tabOffsetTop;
    },
    loadMore() {
      // 加载更多 发送新的请求
      // console.log('----------');
      this.getHomeGoods(this.currentType);
    },
    swiperImageLoad() {
      // 通过$el拿到组件元素对象
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
      // console.log(this.tabOffsetTop);
    }
  },
  created() {
    // 1.请求多个数据
    this.getHomeMultidata();

    // 2.请求goods商品数据
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  mounted() {
    // 对商品列表图片的加载进行防抖
    const refresh = debounce(this.$refs.scroll.refresh, 50)
    
    // 监听item图片的加载
    this.$bus.$on("itemImageLoad", () => {
      refresh();
      // this.$refs.scroll.refresh();
    });
  },
  actived() {
    // 返回该路由时立刻跳转到上次记录的位置
    this.$refs.scroll.scrollTo(0, this.saveY, 0);
    this.$refs.scroll.refresh();
  },
  deactived() {
    // 记录路由跳转之前的纵轴位置
    this.saveY = this.$refs.scroll.getScrollY();
  }
};
</script>

<style scoped>
#home {
  height: 100vh;
  position: relative;
}

.home-nav {
  background-color: var(--color-tint);
  color: #fff;

  /* position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index: 9; */
}

.tab-control {
  position: sticky;
  top: 44px;
  z-index: 9;
}

.content {
  overflow: hidden;

  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
}
</style>