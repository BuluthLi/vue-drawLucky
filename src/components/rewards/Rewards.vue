// 添加lang="html",可以避免vscode的莫名警告

<template lang="html">
  <div class="rewards" v-show="showRewards">
    <div class="shade" @click="onHide"></div>
    <div class="rewards-wrapper">
      <div class="rewards-list">
        <div class="blank-box" v-if="personalRewardsList.length==0">
          <img src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6568f3610a.png" alt class="blank-img">
          <p class="blank-text">大大，您没有中奖记录哦~</p>
        </div>
        <div class="normal-box" v-if="personalRewardsList.length>0">
          <div class="reward-item" v-for="(item,index) in personalRewardsList" :key="index">
            <div class="text">
              <p>{{item.rank_info}}</p>
              <p>奖励：{{item.nickname}}</p>
            </div>
            <div
              class="cover"
              :style="{'background':'url('+item.cover+') no-repeat center','background-size':'contain'}"
            ></div>
          </div>
        </div>
      </div>
      <img src="https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6568f38b45.png" alt class="close" @click="onHide">
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {};
  },
  props: {
    showRewards: {
      type: Boolean,
      default: false
    },
    personalRewardsList: {
      type: Array,
      default: []
    }
  },

  methods: {
    onHide() {
      this.$emit("onHide");
    }
  }
};
</script>
<style lang="less" scoped>
.rewards {
  position: fixed;
  z-index: 101;
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
    z-index: 102;
  }
  .rewards-wrapper {
    position: absolute;
    width: 100%;
    height: auto;
    top: 0;
    left: 0;
    z-index: 102;
    .rewards-list {
      width: 3.38rem;
      height: 3.7rem;
      background: url("https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6568f3729b.png")
        no-repeat;
      background-size: contain;
      margin: 0 auto;
      // 去除子元素的margin-top作用在父元素上
      overflow: hidden;
      margin-top: 1.57rem;
      position: relative;
      .blank-box {
        .blank-img {
          display: block;
          width: 0.72rem;
          height: 0.84rem;
          margin: 1.6rem auto 0 auto;
        }
        .blank-text {
          width: 1.45rem;
          height: 0.16rem;
          margin: 0.1rem auto 0 auto;
          font-size: 0.12rem;
          color: #828b6f;
        }
      }
      .normal-box {
        position: absolute;
        z-index: 111;
        width: 2.9rem;
        height: 2.15rem;
        bottom: 0.15rem;
        left: 50%;
        margin-left: -1.45rem;
        overflow-y: auto;
        .reward-item {
          width: 2.9rem;
          height: 0.5rem;
          background: url("https://wsaiosscdn.yfway.com/ds/Uploads/Picture/2019/04/29/5cc6568f3a889.png")
            no-repeat center;
          background-size: contain;
          margin-top: 0.05rem;
          padding-left: 0.5rem;
          box-sizing: border-box;
          display: flex;
          align-items: center;
          .text {
            width: 1.65rem;
            height: 0.4rem;
            font-size: 0.12rem;
            color: #828b6f;
            overflow: hidden;
            white-space: nowrap;
            display: flex;
            align-items: center;
          }
          .cover {
            width: 0.75rem;
            height: 0.4rem;
            overflow: hidden;
          }
        }
      }
    }
    .close {
      display: block;
      width: 0.31rem;
      height: 0.31rem;
      margin: 0 auto;
      margin-top: 0.25rem;
      cursor: pointer;
    }
  }
}
</style>
