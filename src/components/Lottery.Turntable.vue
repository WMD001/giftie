<template>
  <div class="lottery-turntable-component full-screen">
    <LuckyGrid
      class="lucky-grid"
      ref="luckyGrid"
      width="300px"
      height="300px"
      :prizes="prizes"
      :blocks="blocks"
      :buttons="buttons"
      :default-style="defaultStyle"
      :active-style="activeStyle"
      @start="startCallBack"
      @end="endCallBack"
    />
  </div>
</template>

<script>
import {GLOBAL_KEYS, RESTART_COUNT, gifts, homeConfig} from '../gift.setting.js';
export default {
  name: 'LotteryTurntable',
  data() {
    this.defaultStyle = {
      fontColor: "#AFFFD7",
      fontSize: "10px",
      lineHeight: "13px",
      wordWrap: false,
    };
    this.activeStyle = {
      fontColor: "#fff",
    };
    this.blocks = [
      { padding: "10px", background: "rgba(0, 0, 0, 0)", borderRadius: "10px" },
    ];
    this.buttons = [
      {
        x: 1,
        y: 1,
        background: "rgba(0, 0, 0, 0)",
        fonts: [],
        imgs: [
          {
            src: require("../assets/images/button.png"),
            width: "100%",
            top: "0",
          },
        ],
      },
    ];
    return {
      prizes: [],
    };
  },

  methods: {
    getGiftList() {
      const prizes = [];
      let axis = [[0, 0], [1, 0], [2, 0], [2, 1], [2, 2], [1, 2], [0, 2], [0, 1],];
      let data = gifts.map(item => {
        return {
          fullName: item.name,
          name: item.alias,
          img: item.image,
          top: '70%',
        };
      });
      
      // let data = [];
      data.forEach((item, index) => {
        prizes.push({
          x: axis[index][0],
          y: axis[index][1],
          title: item.name,
          fullName: item.fullName,
          fonts: [{ text: item.name, top: item.top }],
          imgs: [
            {
              src: require('../assets/images/block-bg.png'),
              activeSrc: require('../assets/images/active-block-bg.png'),
              width: '100%',
              height: '100%',
            },
            {
              src: item.img,
              width: '70%',
              top: '3%',
            },
          ],
        });
      });
      this.prizes = prizes;
    },

    getTargetGiftIndex () {
      if (homeConfig.targetGiftIndex) {
        return homeConfig.targetGiftIndex;
      }
      if (homeConfig.targetGiftIndexList) {
        let playCount =  localStorage.getItem("playCount") || 0;
        return homeConfig.targetGiftIndexList[playCount];
      }
      return Math.random() * 8 >> 0;
    },

    startCallBack () {
      this.$refs.luckyGrid.play();
      let targetGiftIndex = this.getTargetGiftIndex();
      setTimeout(() => {
        this.$refs.luckyGrid.stop(targetGiftIndex)
      }, homeConfig.timeout);
    },
    endCallBack (prize) {
      const { imgs, fullName } = prize;
      const imageUrl = imgs[1].src;
      let restartCount = localStorage.getItem(GLOBAL_KEYS.RESTART_COUNT_KEY) || RESTART_COUNT;
      let playCount =  localStorage.getItem("playCount") || 0;
      playCount++;
      localStorage.setItem("playCount", playCount);
      localStorage.setItem(GLOBAL_KEYS.RESTART_COUNT_KEY, --restartCount + '');
      localStorage.setItem(GLOBAL_KEYS.EXIST_KEY, '1');
      localStorage.setItem(GLOBAL_KEYS.NAME_KEY, fullName);
      localStorage.setItem(GLOBAL_KEYS.IMAGE_KEY, imageUrl);
      setTimeout(() => {
        this.$emit('success');
      }, 1200);
    }
  },

  mounted() {
    this.getGiftList();
  }, 
};
</script>

<style scoped>
.lottery-turntable-component {
  background-color: #fad3e6;
  display: flex;
  justify-content: center;
  align-items: center;
}
.lucky-grid {
  width: 300px;
  height: 300px;
  background: rgb(245, 171, 184);
  position: relative;
  border-radius: 8px;
  overflow: hidden;
}
</style>