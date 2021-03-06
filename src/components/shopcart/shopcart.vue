<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'heighLight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'heighLight':totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          </div>
          <div class="price" :class="{'heighLight':totalCount>0}">￥{{totalPrice}}</div>
          <div class="desc">另需配送费￥{{delivery}}元</div>
        </div>
        <div class="content-right" :class="payClass" @click.stop.prevent="payCart">{{payDesc}}</div>
      </div>

      <div class="ball-wrapper">
        <div v-for="(ball,index) in balls" :key="index+'1'">
          <transition
            name="drop"
            @before-enter="beforeDrop"
            @enter="dropping"
            @after-enter="afterDrop"
          >
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <transition name="fold">
        <div class="list-wrapper" v-show="listShow">
          <div class="header">
            <div class="shopCart">购物车</div>
            <div class="empty" @click="empty">清空</div>
          </div>
          <div class="list-content" ref="list">
            <ul>
              <li class="content" v-for="(food,index) in selectFoods" :key="index">
                <span class="name">{{food.name}}</span>
                <span class="price">
                  ￥
                  <span>{{food.price}}</span>
                </span>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <div class="mask" v-show="listShow" @click="hideList"></div>
  </div>
</template>

<script type="text/ecmascript-6">
import BScroll from "better-scroll";
import cartcontrol from "../../components/cartcontrol/cartcontrol";
export default {
  components: {
    cartcontrol
  },
  data() {
    return {
      balls: [
        { show: false },
        { show: false },
        { show: false },
        { show: false },
        { show: false }
      ],
      dropBalls: [],
      fold: true
    };
  },
  props: {
    delivery: {
      type: Number
    },
    minPrice: {
      type: Number
    },
    selectFoods: {
      type: Array,
      default() {
        return [];
      }
    }
  },
  methods: {
    payCart() {
      if (this.totalPrice > this.minPrice) {
        window.alert(`总共${this.totalPrice}元`);
      }
    },
    hideList() {
      this.fold = true;
    },
    empty() {
      this.selectFoods.forEach(food => {
        food.count = 0;
      });
    },
    toggleList() {
      if (!this.totalCount) {
        return;
      }
      this.fold = !this.fold;
    },
    drop(target) {
      this.balls.forEach(ball => {
        if (!ball.show) {
          ball.show = true;
          ball.el = target;
          this.dropBalls.push(ball);
        }
      });
    },
    beforeDrop(el) {
      let count = this.balls.length;
      while (count--) {
        let ball = this.balls[count];
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect();
          let x = rect.left - 32;
          let y = -(window.innerHeight - rect.top - 22);
          el.style.display = "";
          el.style.webkitTransform = `translate3d(0,${y}px,0)`;
          el.style.transform = `translate3d(0,${y}px,0)`;
          let inner = el.getElementsByClassName("inner-hook")[0];
          inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
          inner.style.transform = `translate3d(${x}px,0,0)`;
        }
      }
    },
    dropping(el, done) {
      /* eslint-disable no-unused-vars */
      let rf = el.offsetHeight;
      this.$nextTick(() => {
        el.style.webkitTransform = "translate3d(0,0,0)";
        el.style.transform = "translate3d(0,0,0)";
        let inner = el.getElementsByClassName("inner-hook")[0];
        inner.style.webkitTransform = "translate3d(0,0,0)";
        inner.style.transform = "translate3d(0,0,0)";
        el.addEventListener("transitionend", done);
      });
    },
    afterDrop(el) {
      let ball = this.dropBalls.shift();
      if (ball) {
        ball.show = false;
        el.style.display = "none";
      }
    }
  },

  computed: {
    listShow: {
      get() {
        if (!this.totalCount) {
          console.log(1);
          return false;
        }
        var show = !this.fold;
        if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              console.log(this.$refs.list);
              this.scroll = new BScroll(this.$refs.list, {
                click: true
              });
            } else {
              this.scroll.refresh();
            }
          });
        }
        return show;
      }
    },
    totalPrice() {
      var price = 0;
      this.selectFoods.forEach(food => {
        price += food.price * food.count;
      });
      return price;
    },
    totalCount() {
      var count = 0;
      this.selectFoods.forEach(food => {
        count += food.count;
      });
      return count;
    },
    payDesc() {
      if (this.totalPrice === 0) {
        return `￥${this.minPrice}起送`;
      } else if (this.totalPrice < this.minPrice) {
        var diff = this.minPrice - this.totalPrice;
        return `还差￥${diff}起送`;
      } else {
        return "去结算";
      }
    },
    payClass() {
      if (this.totalPrice < this.minPrice) {
        return "not-enough";
      } else {
        return "enough";
      }
    }
  }
};
</script>

<style lang="stylus">
.shopcart {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  z-index: 50;
  height: 48px;

  .content {
    display: flex;
    background-color: #141d27;

    .content-left {
      flex: 1;

      .logo-wrapper {
        display: inline-block;
        vertical-align: top;
        position: relative;
        top: -10px;
        width: 56px;
        height: 56px;
        border-radius: 50%;
        padding: 6px;
        margin: 0 12px;
        background-color: #141d27;
        box-sizing: border-box;

        .logo {
          width: 100%;
          height: 100%;
          border-radius: 50%;
          text-align: center;
          background-color: #2b343c;

          &.heighLight {
            background-color: rgb(0, 160, 220);
          }

          i {
            font-size: 24px;
            line-height: 44px;
            color: #80858a;

            &.heighLight {
              color: #fff;
            }
          }
        }

        .num {
          position: absolute;
          top: 0;
          right: 0;
          width: 24px;
          height: 16px;
          background-color: rgb(240, 20, 20);
          color: #ffffff;
          line-height: 16px;
          border-radius: 16px;
          font-size: 9px;
          font-weight: 700;
          text-align: center;
          box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4);
        }
      }

      .price {
        display: inline-block;
        vertical-align: top;
        margin: 12px 0;
        padding-right: 12px;
        font-size: 16px;
        font-weight: 700;
        line-height: 24px;
        color: rgba(255, 255, 255, 0.4);
        border-right: 1px solid rgba(255, 255, 255, 0.1);

        &.heighLight {
          color: #ffffff;
        }
      }

      .desc {
        display: inline-block;
        vertical-align: top;
        font-size: 10px;
        margin: 12px 0 0 12px;
        line-height: 24px;
        color: rgba(255, 255, 255, 0.4);
      }
    }

    .content-right {
      flex: 0 0 105;
      width: 105px;
      padding: 0 8px;
      font-size: 12px;
      font-weight: 700;
      line-height: 48px;
      text-align: center;
      color: rgba(255, 255, 255, 0.4);

      &.not-enough {
        background: #2b333b;
      }

      &.enough {
        background-color: #00b43c;
        color: #fff;
      }
    }
  }

  .ball-wrapper {
    .ball {
      position: fixed;
      left: 32px;
      bottom: 22px;
      z-index: 200;
      transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41);

      .inner {
        width: 16px;
        height: 16px;
        border-radius: 50%;
        background: rgb(0, 160, 220);
        transition: all 0.4s linear;
      }
    }
  }

  .list-wrapper {
    position: absolute;
    left: 0;
    top: 0;
    z-index: -1;
    width: 100%;
    transform: translate3d(0, -100%, 0);

    &.fold-enter, &.fold-leave-to {
      transform: translate3d(0, 0, 0);
    }

    &.fold-enter-active, &.fold-leave-active {
      transition: all 0.5s;
    }

    .header {
      height: 40px;
      line-height: 40px;
      background-color: #f3f4f7;
      padding: 0 18px;
      border-bottom: 1px solid rgba(7, 17, 27, 0.1);

      .shopCart {
        font-size: 14px;
        font-weight: 200;
        color: rgb(7, 17, 27);
        float: left;
      }

      .empty {
        font-size: 12px;
        color: rgb(0, 160, 220);
        float: right;
      }
    }

    .list-content {
      width: 100%;
      height: 100%;
      max-height: 217px;
      overflow: hidden;

      .content {
        font-size: 0;
        position: relative;
        height: 48px;
        padding: 12px;
        box-sizing: border-box;
        background-color: #fff;

        .name {
          display: inline-block;
          vertical-align: top;
          font-size: 14px;
          line-height: 24px;
          color: rgb(7, 17, 27);
        }

        .price {
          display: inline-block;
          vertical-align: top;
          font-size: 0;
          position: absolute;
          right: 90px;
          font-size: 10px;
          line-height: 24px;
          color: rgb(240, 20, 20);

          span {
            font-size: 14px;
          }
        }

        .cartcontrol-wrapper {
          position: absolute;
          right: 0;
          bottom: 6px;
          margin-left: 12px;
        }
      }
    }
  }
}

.mask {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  z-index: 10;
  background-color: rgba(7, 17, 27, 0.6);
}
</style>
