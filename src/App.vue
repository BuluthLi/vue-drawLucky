// 1.初始版本是css版本，但由于失误操作了dom,出现bug,采用了js控制转盘rotate,
// 2.频繁改变transform的rotate，触发浏览器的重绘次数过多，会造成白屏闪烁，故修改成现在版本
// 3.不应该操作dom,(let target = document.getElementsByClassName("draw-lucky-content")[0];);只用对dom的style属性赋值，故此对js上动态改style的rotate属性；
// 4.猜想css实现时使用动态改style属性，而不去操作dom,就不会出现css实现方式中的首次抽奖转盘转动角度偏小的bug,此猜想未验证，相关博客：https://blog.csdn.net/qq_42833001/article/details/85015872
<template>
  <div id="app">
    <!-- 世界喊话 -->
    <div class="world">
      <div class="content">
        <div class="world-text" :style="{left:left+'px'}" v-html="totalStr"></div>
      </div>
    </div>
    <header>
      <img src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6bddc68065.jpg" alt>
    </header>
    <div class="draw-lucky-box">
      <!-- 顶部介绍 -->
      <!-- <div class="chance-num">剩余抽奖机会：{{chanceNum}}次</div> -->
      <!-- 转盘 -->
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6bddc5dd71.jpg"
        alt
        class="draw-lucky-bg"
      >
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656818164d.png"
        alt
        class="draw-lucky-content"
        :style="{transform:'translateX(-50%) rotate(' + startRotate + 'deg)'}"
      >
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656712c788.png"
        alt
        class="main-btn"
        @click="onDraw"
      >
      <!-- 底部介绍 -->
      <div class="draw-lucky-text">{{totalInfo}}</div>
    </div>
    <div class="button-box">
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656713bd15.png"
        class="rewards"
        alt
        @click="onShowRewards"
      >
      <img
        v-if="chanceNum==0"
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656715acac.png"
        class="startgame"
        alt
        @click="onIntoGame"
      >
    </div>
    <div class="rule-box">
      <img src=" https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6bddc47794.jpg" alt>
    </div>
    <div class="goods-box">
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6568218e2b.png"
        class="title-img"
        alt
      >
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656817fc68.png"
        alt
        class="good-item"
        @click="onIntoDetail(2269)"
      >
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656815bd9c.png"
        alt
        class="good-item"
        @click="onIntoDetail(53)"
      >
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc65681c8942.png"
        alt
        class="good-item"
        @click="onIntoDetail(928)"
      >
      <img
        src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656817fe37.png"
        alt
        class="good-item"
        @click="onIntoDetail(0)"
      >
    </div>
    <img
      src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc65671810c1.gif"
      alt
      class="float-bird"
      @click="onGoShare"
    >
    <img
      src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656811a87d.png"
      alt
      class="go-index"
      @click="onIntoIndex"
    >
    <Rewards
      :showRewards="showRewards"
      v-on:onHide="onHide"
      :personalRewardsList="personalRewardsList"
    ></Rewards>
    <Gift :showGift="showGift" v-on:onHide="onHide" :gift="gift"></Gift>
  </div>
</template>

<script>
import wx from "weixin-js-sdk";
import Rewards from "./components/rewards/Rewards";
import Gift from "./components/alertgift/AlertGift";

// let user_id = 2401452;
let user_id = 2314724;
let draw = false;
let timeDely = 3;
let s = null;
let ss = null;
export default {
  name: "App",
  data() {
    return {
      startRotate: 0,
      rewardsList: [
        { id: 119, name: "乳胶枕", rotate: 0 },
        { id: 121, name: "袋子", rotate: 288 },
        { id: 122, name: "优惠券", rotate: 216 },
        { id: 117, name: "谢谢参与", rotate: 144 },
        { id: 124, name: "棉被", rotate: 72 }
      ],
      totalStr: "",
      left: 0,
      blank: `<span class="blank-span" style="display: inline-block;
        width: 100px;
        height: 19px;"></span>`,
      chanceNum: 3,
      showRewards: false,
      personalRewardsList: [],
      totalInfo: "点击立即抽奖赢大礼额~",
      showGift: false,
      // type为1继续抽奖，type为2去玩游戏，type为3去分享活动
      gift: {
        cover: "",
        text: "",
        btn:
          "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656729ad13.png",
        bg:
          "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6568180e17.png",
        type: 1
      },
      beat: 0,
      share: 0,
      wordsList: ""
    };
  },
  created() {
    draw = false;
    let params = new FormData();
    params.append("func", "get-user-can-lucky-number");
    params.append("user_id", user_id);
    // 请求用户的使用次数
    this.$http
      .post(process.env.API_HOST + "index.php?s=/Campain/Api/action", params)
      .then(res => {
        console.log(res.data);
        if (res.data.status == 1) {
          this.totalInfo = res.data.remind_info;
          this.chanceNum = res.data.remain_times;
          this.beat = res.data.is_beat_boss;
          this.share = res.data.is_share;
          // this.chanceNum = 0;
          if (this.chanceNum == 0) {
            this.noChance(1);
          }
        }
      });
  },
  mounted() {
    let params = new FormData();
    params.append("func", "get-game-user-prize");
    // 请求用户的世界喊话
    this.$http
      .post(process.env.API_HOST + "index.php?s=/Campain/Api/action", params)
      .then(res => {
        this.wordsList = res.data;
        let singleWidth = this.averageWidth(res.data);
        this.worldWordsInit(singleWidth);
      });
  },
  destroyed() {
    draw = false;
    clearInterval(s);
  },
  methods: {
    onDraw() {
      let that = this;
      if (draw) return;
      if (this.chanceNum == 0) {
        this.noChance(2);
        return;
      }
      draw = true;
      // gift内容重置
      that.giftReset();
      let params = new FormData();
      params.append("func", "get-lucky-draw-for-game");
      params.append("user_id", user_id);
      that.$http
        .post(process.env.API_HOST + "index.php?s=/Campain/Api/action", params)
        .then(res => {
          if (res.data.status == 1) {
            console.log("抽中了" + res.data.message[0].name);
            that.totalInfo = res.data.remind_info;
            that.chanceNum = res.data.remain_times;
            // 变量不存在竟然不报错（人都傻了）
            // if (that.chanceNum == 0) {
            //   console.log("zhixing");
            //   that.gift.type = 2;
            //   that.gift.btn =
            //     "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc65672bc8a6.png";
            // }
            for (let i = 0; i < that.rewardsList.length; i++) {
              console.log("waibu");
              if (res.data.message[0].id == that.rewardsList[i].id) {
                console.log("进来了");
                let promise = new Promise(function(resolve, reject) {
                  that.doWheel(i);
                  setTimeout(() => {
                    resolve(res.data.message[0]);
                  }, (timeDely + 0.5) * 1000);
                });
                promise.then(result => {
                  console.log(result);
                  that.showGift = true;
                  if (result.id != that.rewardsList[3].id) {
                    that.gift = {
                      ...that.gift,
                      ...{
                        bg:
                          "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6567297dc0.png",
                        cover: result.cover,
                        text: result.name
                      }
                    };
                  }

                  console.log(that.gift);
                });
                return;
              }
            }
          }
        })
        .catch(err => {
          draw = false;
        });
    },
    doWheel(index) {
      // let target = document.getElementsByClassName("draw-lucky-content")[0];
      let count = Math.floor(this.startRotate / 360) + 1;
      let degree = this.rewardsList[index].rotate + 360 * 3 + count * 360;
      console.log(degree);
      // let step = 6;
      // let interval = (step * timeDely * 1000) / (degree - this.startRotate);
      let interval = 20;
      let step = (interval * (degree - this.startRotate)) / timeDely / 1000;
      ss = setInterval(res => {
        // target.style.transform =
        //   "translateX(-50%) rotate(" + this.startRotate + "deg)";
        this.startRotate += step;
        if (this.startRotate >= degree) {
          clearInterval(ss);
        }
      }, interval);
    },
    // 计算文字宽度
    averageWidth(data) {
      let arr = [];
      data.every(item => {
        return arr.push(item.length);
      });
      let max = Math.max.apply(null, arr);
      let min = Math.min.apply(null, arr);

      let dom = document.querySelector("#app");
      let fontSize = window.getComputedStyle(dom).fontSize.replace(/px/, "");
      return (Math.floor(max + min) / 2) * fontSize;
    },
    worldWordsInit(width) {
      let that = this;
      for (let i = 0; i < that.wordsList.length; i++) {
        that.totalStr += that.wordsList[i] + that.blank;
      }
      // let target = document.querySelector(".world-text");
      // target.innerHTML = totalStr;

      s = setInterval(() => {
        that.left -= 1;
        if (Math.abs(that.left) > width * that.wordsList.length) {
          that.left = 0;
        }
      }, 30);
    },
    onShowRewards() {
      this.showRewards = true;
      let params = new FormData();
      params.append("func", "get-game-lucky-draw-user-prize");
      params.append("user_id", user_id);
      this.$http
        .post(process.env.API_HOST + "index.php?s=/Campain/Api/action", params)
        .then(res => {
          this.personalRewardsList = res.data;
          // this.personalRewardsList = [];
        });
    },
    onHide() {
      this.showRewards = false;
      this.showGift = false;
      // 查看关闭后才能再次请求
      draw = false;
    },
    onIntoGame() {
      console.log("开始玩游戏");
      window.location.href =
        "https://wds-test.yfway.com/minigame/stage4/?user_id=" + user_id;
    },
    onGoShare() {
      console.log("去分享页面");
      wx.miniProgram.navigateTo({
        url: "/pages/campain/campain-qrcode/campain-qrcode"
      });
    },
    onIntoIndex() {
      wx.miniProgram.switchTab({
        url: "/pages/index/index"
      });
    },
    giftReset() {
      this.gift = {
        cover: "",
        text: "",
        btn:
          "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc656729ad13.png",
        bg:
          "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6568180e17.png",
        type: 1
      };
    },
    // type为1是正常初次加载的流程机会为零了，type为2是点击抽奖的情况
    noChance(type) {
      if (type == 1 && this.beat == 1 && this.share == 1) return;
      this.gift =
        this.beat == 0
          ? {
              bg:
                "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc65672b5da4.png",
              btn:
                "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc65672bc8a6.png",
              type: 2
            }
          : this.share == 0
          ? {
              bg:
                "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6567278711.png",
              btn:
                "https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc65672677da.png",
              type: 3
            }
          : this.gift;
      this.showGift = true;
    },
    onIntoDetail(id) {
      if (id != 0) {
        wx.miniProgram.navigateTo({
          url: "/pages/indexdetail/indexdetail?id=" + id
        });
      } else {
        wx.miniProgram.switchTab({
          url: "/pages/index/index"
        });
      }
    }
  },
  components: {
    Rewards,
    Gift
  }
};
</script>

<style lang='less' scoped>
#app {
  background: #4aa143;
  padding-bottom: 0.9rem;
  img {
    width: 100%;
    vertical-align: bottom;
  }
  .world {
    background: rgba(0, 0, 0, 0.3);
    width: 100%;
    height: 0.36rem;
    position: fixed;
    z-index: 87;
    top: 0;
    left: 0;
    color: white;
    .content {
      overflow: hidden;
      width: 92%;
      height: 0.24rem;
      position: absolute;
      left: 4%;
      top: 0.08rem;
      .world-text {
        white-space: nowrap;
        position: absolute;
        left: 0;
        top: 0;
      }
      .blank-span {
        display: inline-block;
        width: 50px;
        height: 19px;
      }
    }
  }
  header {
    img {
      width: 100%;
    }
  }
  .draw-lucky-box {
    width: 100%;
    height: auto;
    position: relative;
    z-index: 55;
    .chance-num {
      position: absolute;
      top: 12%;
      left: 50%;
      width: 2.2rem;
      height: 0.26rem;
      background: url("https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6568239717.png")
        no-repeat;
      background-size: contain;
      transform: translateX(-50%);
      z-index: 5;
      color: white;
      line-height: 0.26rem;
      font-size: 0.12rem;
      box-sizing: border-box;
      padding-left: 0.12rem;
    }
    .draw-lucky-bg {
      width: 100%;
    }
    .draw-lucky-content {
      width: 74%;
      position: absolute;
      top: 15.5%;
      left: 50%;
      transform: translateX(-50%);
      z-index: 4;
    }
    .main-btn {
      width: 18.6%;
      position: absolute;
      top: 33.5%;
      left: 50%;
      transform: translateX(-50%);
      z-index: 6;
      cursor: pointer;
    }
    .draw-lucky-text {
      position: absolute;
      bottom: 25.2%;
      left: 50%;
      max-width: 1.5rem;
      text-overflow: ellipsis;
      white-space: nowrap;
      overflow: hidden;
      text-align: center;
      transform: translateX(-50%);
      color: rgb(66, 66, 66);
    }
  }
  .button-box {
    background: #4aa143;
    margin-top: -0.4rem;
    z-index: 56;
    position: relative;
    .rewards,
    .startgame {
      display: block;
      width: 1.68rem;
      margin: 0 auto;
      cursor: pointer;
    }
    .startgame {
      margin-top: 0.1rem;
    }
  }
  .rule-box {
  }
  .goods-box {
    text-align: center;
    .title-img {
      display: block;
      width: 80%;
      margin: 0 auto;
    }
    .good-item {
      width: 1.31rem;
      height: 0.97rem;
      margin: 0.15rem 0.08rem 0 0.08rem;
      cursor: pointer;
    }
  }
  .go-index {
    display: block;
    position: fixed;
    bottom: 0;
    left: 0;
    z-index: 68;
    cursor: pointer;
  }
  .float-bird {
    position: fixed;
    bottom: 0.5rem;
    left: 0;
    z-index: 87;
    cursor: pointer;
    width: 0.8rem;
  }
}
</style>

