<template>
  <div>
    <div>
      <el-button @click="start">开始</el-button>
      <el-button @click="stop">停止</el-button>
    </div>
    <battleSetting style="float:left"></battleSetting>
    <lineChart style="float:left" :cdata="charData" :edata="enemyData"></lineChart>
  </div>
</template>
<script>
/* eslint-disable */
// 9.16  测试对战
import { atkAction, clone } from "@/components/calculate/calculate.js";
import lineChart from "@/components/draw/lineChart";
import battleSetting from "@/components/battleSetting";
export default {
  name: "",
  data() {
    return {
      cData: {},
      eData: {},
      initTime: 0,
      nowTime: 0,
      flowTime: 0,
      cState: {
        lastAckTime: 0,
        lastAckSpeed: 0
      },
      eState: {
        lastAckTime: 0,
        lastAckSpeed: 0
      },
      charData: [],
      enemyData: [],
      countTimeM: 0
    };
  },
  components: {
    lineChart: lineChart,
    battleSetting: battleSetting
  },
  methods: {
    initBattle() {
      this.cData = {};
      this.eData = {};
      this.initTime = 0;
      this.nowTime = 0;
      this.flowTime = 0;
      this.cState = {
        lastAckTime: 0,
        lastAckSpeed: 0
      };
      this.eState = {
        lastAckTime: 0,
        lastAckSpeed: 0
      };
      this.charData = [];
      this.enemyData = [];
      this.countTimeM = 0;
      //初始化战斗数据
      this.$store.dispatch(
        "setTemporaryED",
        clone(this.$store.getters.getEnemyData)
      );
      this.$store.dispatch(
        "setTemporaryCD",
        clone(this.$store.getters.getCharData)
      );
      this.getBattleData();
    },
    start() {
      window.clearTimeout(this.countTimeM);
      this.initBattle();
      this.initTime = new Date().valueOf();
      this.countTimeM = window.setInterval(this.test, 100);
    },
    stop() {
      window.clearTimeout(this.countTimeM);
    },
    test() {
      this.charData.push([this.flowTime, this.cData]);
      this.enemyData.push([this.flowTime, this.eData]);
      this.battle();
      //console.log(this.enemyData);
      if (this.flowTime >= 10000 + 10) {
        window.clearTimeout(this.countTimeM);
      }
    },
    battle() {
      this.getBattleData();
      //到达攻击时间
      this.countTime();
      if (
        this.cState.lastAckTime === 0 ||
        this.flowTime - this.cState.lastAckTime >=
          Math.floor(this.cState.lastAckSpeed * 1000) - 10
      ) {
        //攻击
        let result = atkAction(clone(this.cData), clone(this.eData));
        this.afterAckSavaData(true, false, result);
      }
      if (
        this.eState.lastAckTime === 0 ||
        this.flowTime - this.eState.lastAckTime >=
          Math.floor(this.eState.lastAckSpeed * 1000) - 10
      ) {
        //console.log(this.flowTime - this.eState.lastAckTime);
        //攻击
        let result = atkAction(clone(this.eData), clone(this.cData));
        this.afterAckSavaData(false, true, result);
      }
      //提交
      this.setBattleData();
    },
    //获取当前战斗数据
    getBattleData() {
      this.cData = clone(this.$store.getters.getTemporaryCD);
      this.eData = clone(this.$store.getters.getTemporaryED);
      //console.log(this.$store.getters.getTemporaryCD,this.$store.getters.getTemporaryED);
    },
    setBattleData() {
      this.$store.dispatch("setTemporaryED", clone(this.eData));
      this.$store.dispatch("setTemporaryCD", clone(this.cData));
      //console.log(this.$store.getters.getTemporaryCD,this.$store.getters.getTemporaryED);
    },
    // 攻击后保存数据
    afterAckSavaData(c, e, result) {
      if (c === true) {
        this.cData = clone(result.atkP);
        this.eData = clone(result.defP);
        this.cState.lastAckTime = this.flowTime;
        this.cState.lastAckSpeed = this.cData.baseAttackTime;
      }
      if (e === true) {
        this.eData = clone(result.atkP);
        this.cData = clone(result.defP);
        this.eState.lastAckTime = this.flowTime;
        this.eState.lastAckSpeed = this.eData.baseAttackTime;
      }
    },
    countTime() {
      this.nowTime = new Date().valueOf();
      this.flowTime = this.nowTime - this.initTime;
    },
    atkPlus() {
      this.$store.state.temporaryED.atk *= 2.5;
      //console.log(this.$store.state.temporaryED.atk);
    }
  },
  created() {
    //初始化战斗数据
    this.$store.dispatch(
      "setTemporaryED",
      clone(this.$store.getters.getCharData)
    );
    this.$store.dispatch(
      "setTemporaryCD",
      clone(this.$store.getters.getEnemyData)
    );
    this.initBattle();
  },
};
</script>
<style scoped>
* {
  margin: 0;
  padding: 0;
}
</style>