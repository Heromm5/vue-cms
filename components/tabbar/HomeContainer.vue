<template>
    <div>

        <!-- 轮播图区域 -->
        <swiper :lunbotu-list="lunbotuList" :isfull="true"></swiper>

        <!-- 九宫格到六宫格的改造过程 -->
        <ul class="mui-table-view mui-grid-view mui-grid-9">
            <li class="mui-table-view-cell mui-media mui-col-xs-4 mui-col-sm-3">
                <router-link to="/home/newslist">
                <img src="../../src/images/menu1.png">
                <div class="mui-media-body">新闻资讯</div></router-link></li>
            <li class="mui-table-view-cell mui-media mui-col-xs-4 mui-col-sm-3">
                <router-link to="/home/photolist">
                <img src="../../src/images/menu2.png">
                <div class="mui-media-body">图片分享</div></router-link></li>
            <li class="mui-table-view-cell mui-media mui-col-xs-4 mui-col-sm-3">
                <router-link to="/home/goodslist">
                <img src="../../src/images/menu3.png">
                <div class="mui-media-body">商品购买</div></router-link></li>
            <li class="mui-table-view-cell mui-media mui-col-xs-4 mui-col-sm-3">
                <router-link to="#">
                <img src="../../src/images/menu4.png">
                <div class="mui-media-body">留言反馈</div></router-link></li>
            <li class="mui-table-view-cell mui-media mui-col-xs-4 mui-col-sm-3">
                <router-link to="#">
                <img src="../../src/images/menu5.png">
                <div class="mui-media-body">视频专区</div></router-link></li>
            <li class="mui-table-view-cell mui-media mui-col-xs-4 mui-col-sm-3">
                <router-link to="#">
                <img src="../../src/images/menu6.png">
                <div class="mui-media-body">联系我们</div></router-link></li>
        </ul>

    </div>
</template>

<script>
    import {Toast} from 'mint-ui'
    import swiper from '../subcomponents/swiper.vue'

    export default {
      data() {
        return {
            lunbotuList: [] // 保存轮播图的数组
        }
      },
      created() {
        this.getLunbotu()
      },
      methods: {
        getLunbotu() {
          // 获取轮播图数据的方法
          this.$http.get("api/getlunbo").then(result => {
            // console.log(result.body)
            if (result.body.status === 0){
              // 成功了
              this.lunbotuList = result.body.message
              // Toast('加载轮播图成功')
            } else {
              // 失败的
              Toast('加载轮播图失败')
            }
          })
        }
      },
      components: {
        swiper
      }
    }
</script>

<style lang="scss" scoped>


    .mui-grid-view.mui-grid-9 {
        background-color: white;
    }

    .mui-table-view mui-grid-view mui-grid-9 {
        border: none;
    }

    .mui-media-body {
        font-size: 13px;
    }

    .mui-grid-view.mui-grid-9 .mui-table-view-cell {
        border: 0;
    }
</style>