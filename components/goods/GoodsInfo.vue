<template>
    <div class="goodsinfo-container">

        <transition @before-enter="beforeEnter"
                    @enter="enter" @after-enter="afterEnter">
            <div class="ball" v-show="ballFlag" ref="ball"></div>
        </transition>

        <!-- 商品轮播图区域 -->
        <div class="mui-card">
            <div class="mui-card-content">
                <div class="mui-card-content-inner">
                    <swiper :lunbotuList="lunbotu" :isfull="false"></swiper>
                </div>
            </div>
        </div>

        <!-- 商品购买区域 -->
        <div class="mui-card">
            <div class="mui-card-header">{{goodsinfo.title}}</div>
            <div class="mui-card-content">
                <div class="mui-card-content-inner">
                    <p class="price">
                        市场价：
                        <del>￥{{goodsinfo.market_price}}</del>&nbsp;&nbsp;<span class="now_price">￥{{goodsinfo.sell_price}}</span>
                    </p>
                    <p>
                        购买数量：
                        <numbox @getcount="getSelectedCount" :max="goodsinfo.stock_quantity"></numbox>
                    </p>
                    <p>
                        <mt-button type="primary" size="small">立即购买</mt-button>
                        <mt-button type="danger" size="small" @click="addToShopCar">加入购物车</mt-button>
                        <!-- 分析：如何实现加入购物车时候，获取选择的数量
                             1.经过分析发现：按钮属于 goodsinfo 页面，数字属于 numbox 组件
                             2.由于涉及到了父子组件的嵌套了，所以无法直接在 goodsinfo 页面中获取到选中的商品的数量
                             3.怎么解决这个问题：涉及到了子组件向父组件传值（事件调用机制）
                             4.事件调用的本质：父向子传递方法，子调用这个方法，同时把数据当作参数传递给这个方法
                        -->
                    </p>
                </div>
            </div>
        </div>

        <!-- 商品参数区域 -->
        <div class="mui-card">
            <div class="mui-card-header">商品参数</div>
            <div class="mui-card-content">
                <div class="mui-card-content-inner">
                    <p>商品货号：{{goodsinfo.goods_no}}</p>
                    <p>库存情况：{{goodsinfo.stock_quantity}}</p>
                    <p>上架时间：{{goodsinfo.add_time | dateFormat}}</p>
                </div>
            </div>
            <div class="mui-card-footer">
                <mt-button type="primary" size="large" plain @click="goDesc(id)">图文介绍</mt-button>
                <mt-button type="danger" size="large" plain @click="goComment(id)">商品评论</mt-button>
            </div>
        </div>
    </div>
</template>

<script>
  // 导入轮播图组件
  import swiper from '../subcomponents/swiper.vue'
  // 导入数字选择框组件
  import numbox from '../subcomponents/goodsinfo_numbox.vue'

  export default {
    data() {
      return {
        id: this.$route.params.id,  // 将路由参数对象中的 id 挂载到 data，方便后期调用
        lunbotu: [], // 轮播图的数据
        goodsinfo: {}, // 获取到的商品信息
        ballFlag: false,  // 控制小球隐藏和显示的标识符
        selectedCount: 1    // 保存用户选中的商品数量，默认认为用户买1个
      }
    },
    created() {
      this.getLunbotu()
      this.getGoodsInfo()
    },
    methods: {
      getLunbotu() {
        this.$http.get('api/getthumimages/' + this.id).then(result => {
          if (result.body.status === 0) {
            result.body.message.forEach(item => {
              item.img = item.src
            })
            this.lunbotu = result.body.message
          }
        })
      },
      getGoodsInfo() {
        // 获取商品的信息
        this.$http.get('api/goods/getinfo/' + this.id).then(result => {
          if (result.body.status === 0) {
            this.goodsinfo = result.body.message[0]
          }
        })
      },
      goDesc(id) {
        this.$router.push({name: 'goodsdesc', params: {id}})
      },
      goComment(id) {
        this.$router.push({name: 'goodscomment', params: {id}})
      },
      addToShopCar() {
        // 添加到购物车
        this.ballFlag = !this.ballFlag
        // {id:商品id, count:要购买的数量, price:商品的单价, selected:false}
        // 拼接出一个要保存到 store 中 car 数组里的商品信息对象
        var goodsinfo = {
          id: this.id,
          count: this.selectedCount,
          price: this.goodsinfo.sell_price,
          selected: true
        }

        this.$store.commit("addToCar", goodsinfo)
      },
      beforeEnter(el) {
        el.style.transform = "translate(0, 0)"
      },
      enter(el, done) {
        el.offsetWidth

        /* 小球动画优化思路：
        *  1.先分析导致动画不准确的本质原因：我们把小球最终位移到的位置，已经局限在了某一分辨率下的滚动条未滚动的情况下
        *  2.只要分辨率和测试的时候不一样，或者滚动条有一定的滚动距离之后，问题就出现了
        *  3.因此，我们经过分析得到结论：不能把位置的横纵坐标直接写死了，而是应该根据不同情况，动态计算这个坐标值
        *  4.经过分析，得出解题思路：先得到徽标的横纵坐标，在得到小球的横纵坐标，然后再让 y 值求差，x 值求差，得到的结果，就是横纵坐标要位移的距离
        *  5.如何获取徽标和小球的位置       domObject.getBoundingClientRect()*/

        // 获取小球在页面中的位置
        const ballPosition = this.$refs.ball.getBoundingClientRect()
        // 获取徽标在页面中中的位置     refs 只能获取当前组件
        const badgePosition = document.getElementById("badge").getBoundingClientRect()

        const xDist = badgePosition.left - ballPosition.left
        const yDist = badgePosition.top - ballPosition.top


        el.style.transform = `translate(${xDist}px, ${yDist}px)`
        el.style.transition = "all 1s cubic-bezier(.4, -0.3, 1, .68)"
        done()
      },
      afterEnter(el) {
        this.ballFlag = !this.ballFlag
      },
      getSelectedCount(count) {
        // 当子组件把选中的数量传递给父组件的时候，把选中的值保存到 data 上
        this.selectedCount = count
        console.log("数量：" + this.selectedCount)
      }
    },
    components: {
      swiper,
      numbox
    }

  }
</script>

<style lang="scss" scoped>
    .goodsinfo-container {
        background-color: #eee;
        overflow: hidden;

        .now_price {
            color: red;
            font-size: 18px;
            font-weight: bold;
        }

        .mui-card-footer {
            display: block;

            button {
                margin: 20px 0;
            }
        }

        .ball {
            width: 15px;
            height: 15px;
            border-radius: 50%;
            background-color: red;
            position: absolute;
            z-index: 99;
            top: 491px;
            left: 152px;
        }
    }


</style>