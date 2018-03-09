<template>
  <div id="app">
    <el-row :gutter="10">
      <el-col :sm="24" :md="{span: 12, offset: 6}">
        <div class="grid-content">
          <!-- 基金输入栏信息 -->
          <el-card v-loading="loading" class="box-card card-wrap fund-card" :body-style="{padding: '7.5px'}">
            <div slot="header">
              <span>基金</span>
            </div>
            <div>
              <el-row :gutter="5">
                <el-col :span="12">
                  <el-input v-model="fundCode" :maxlength="6" placeholder="代码"></el-input>
                </el-col>
                <el-col :span="12">
                  <el-input v-model="fundName" placeholder="名称"></el-input>
                </el-col>
              </el-row>
              <el-row :gutter="5">
                <el-col :span="23">
                  <div class="read-content-wrap">
                    <b>当前价格：</b>
                    <span>{{fundCurrentPrice}}</span>
                    <b></b>
                  </div>
                </el-col>
              </el-row>
            </div>
          </el-card>
          <!-- 存量输入栏信息 -->
          <el-card class="box-card card-wrap stock-card" :body-style="{padding: '7.5px'}">
            <div slot="header">
              <span>存量</span>
            </div>
            <div>
              <el-row :gutter="5">
                <el-col :span="12">
                  <el-input v-model="fundCost" placeholder="成本"></el-input>
                </el-col>
                <el-col :span="12">
                  <el-input v-model="fundPosition" placeholder="持仓"></el-input>
                </el-col>
              </el-row>
              <el-row :gutter="5" class="read-content-wrap">
                  <el-col :span="12">
                    <b>市值：</b>
                    <span>{{mvalue}}</span>
                  </el-col>
                  <el-col :span="12">
                    <b>收益：</b>
                    <span>{{profit}}</span>
                    <i v-bind:class="isRiseClass">{{percentage}}</i>
                  </el-col>
              </el-row>
            </div>
          </el-card>
          <!-- 定投 -->
          <el-card class="box-card card-wrap" :body-style="{padding: '7.5px'}">
            <div slot="header">
              <span>定投</span>
            </div>
            <div>
              <el-row :gutter="5">
                <el-col :span="24">
                  <el-input v-model="fundTarget" placeholder="目标市值">
                    <template slot="prepend" v-if="fundTarget" >目标市值</template>
                  </el-input>
                </el-col>
              </el-row>
              <el-row :gutter="5">
                <el-col :span="23">
                  <div class="read-content-wrap">
                    <b>本次应投：</b>
                    <span>{{investment}}</span>
                    <b>份</b>
                  </div>
                </el-col>
              </el-row>
            </div>
          </el-card>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import _ from 'lodash';
import axios from 'axios';
export default {
  name: 'app',
  data () {
    return {
      loading: false,
      fundCode: "",  // 基金代码
      fundName: "",  // 基金名称
      fundCurrentPrice: "",  // 基金当前金额
      fundCost: "",  // 成本价
      fundPosition: "",  // 持仓
      fundTarget: "",  // 目标市值
    }
  },
  computed: {
   /** 市值 */
    mvalue: function() {
      let mvalue = this.fundCost * this.fundPosition;
      return mvalue.toFixed(2);
    },
    /** 收益 */
    profit: function() {
      let profitValue = (this.fundCurrentPrice - this.fundCost) * this.fundPosition;
      return profitValue.toFixed(2);
    },
    /** 投入收益百分比 */
    percentage: function() {
      if(this.fundCurrentPrice && this.fundCost) {
        let currentPercentage = ((this.fundCurrentPrice / this.fundCost) - 1) * 100;
        currentPercentage = currentPercentage.toFixed(2);
        return currentPercentage + "%";
      }else {
        return null;
      }
    },
    /** 是否涨 */
    isRiseClass: function() {
      return {
        'rise': (this.fundCost < this.fundCurrentPrice) && this.fundCost,
        'fall': this.fundCost > this.fundCurrentPrice
      }
    },
    /** 应该投入的份额 */
    investment: function() {
      if(this.fundTarget && this.fundCurrentPrice && this.fundPosition) {
        let _inves = (this.fundTarget - this.fundCurrentPrice * this.fundPosition) / this.fundCurrentPrice;
        let inves = Math.round(_inves / 100) * 100;
          return inves;
        }else{
          return null;
        }
      }
  },
  watch: {
    fundCode: function () {
      // 只有当编号为6位时，才执行
      if(this.fundCode.length === 6) {
        this.loading = true;
        axios.get(`http://api.thisjs.com/fund/${this.fundCode}`)
            .then(res => {
              const data = res.data;
              if(data.success === true) {
                this.fundName = data.name;
                this.fundCurrentPrice = data["dwjz"];
              }
              this.loading = false;
            })
        setTimeout(() => {
          this.loading = false;
        }, 5000)
      }else{
        this.fundName = "";
        this.fundCurrentPrice = "";
      }
    }
  }
}
</script>

<style lang="scss">
  .card-wrap{
    &:nth-of-type(n+2) {
      margin-top: 10px;
    }
    .el-card__header {
      padding: 5px 11px;
      background-color: #E5E9F2;
    }
  }
  .read-content-wrap {
    margin-top: 10px;
    b{
      color: #5e6d82;
      font-weight: normal;
    }
    span {
      display: inline-block;
      padding: 0 7.5px;
      border-bottom: 1px solid #bfcbd9;
    }
    i{
      font-style: normal;
    }
  }
  .rise {
    color: red;
    &:after{
      content: '↑'
    }
  }
  .fall {
    color: green;
    &:after{
      content: '↓'
    }
  }
</style>
