<template>
  <div class="alert-gift" v-show="showGift">
    <div class="shade" @click="onHide"></div>
    <div class="content-wrapper">
      <div
        class="content-box"
        :style="{backgroundImage:'url('+gift.bg+')',backgroundSize:'contain'}"
      >
        <div
          class="cover"
          :style="{backgroundImage:'url('+gift.cover+')',backgroundSize:'contain'}"
        ></div>
        <div class="text">{{gift.text}}</div>
      </div>
      <img :src="gift.btn" alt class="btn" @click="eventSendOut(gift.type)">
    </div>
  </div>
</template>
<script>
let user_id = 2314724;
import wx from "weixin-js-sdk";
export default {
  data() {
    return {};
  },
  props: {
    showGift: {
      type: Boolean,
      default: false
    },
    gift: {
      type: Object,
      default: {}
    }
  },
  methods: {
    onHide() {
      this.$emit("onHide");
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
    eventSendOut(type) {
      if (type == 1) {
        this.onHide();
      }
      if (type == 2) {
        this.onIntoGame();
      }
      if (type == 3) {
        this.onGoShare();
      }
    }
  }
};
</script>
<style lang="less" scoped>
.alert-gift {
  position: fixed;
  z-index: 201;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  .shade {
    position: absolute;
    background: rgba(0, 0, 0, 0.65);
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    z-index: 202;
  }
  .content-wrapper {
    position: absolute;
    width: 100%;
    height: auto;
    top: 1rem;
    left: 0;
    z-index: 203;
    .content-box {
      width: 3.65rem;
      height: 3.65rem;
      margin: 0 auto;
      overflow: hidden;
      .cover {
        width: 0.8rem;
        height: 0.8rem;
        overflow: hidden;
        margin: 38% auto 0 auto;
      }
      .text {
        width: 1.8rem;
        overflow: hidden;
        margin: 0 auto;
        margin-top: 7.5%;
        font-size: 0.16rem;
        text-align: center;
        color: white;
      }
    }
    .btn {
      width: 50%;
      display: block;
      margin: 0 auto;
      cursor: pointer;
    }
  }
}
</style>

