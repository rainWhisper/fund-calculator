<template>
  <div id="app">
    <el-row :gutter="10">
      <el-col :sm="24" :md="{span: 12, offset: 6}">
        <div class="grid-content">
          <!-- 基金输入栏信息 -->
          <el-card class="box-card card-wrap fund-card" :body-style="{padding: '7.5px'}">
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
                    <b>元</b>
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
              <el-row :gutter="5">
                <el-col :span="23">
                  <div class="read-content-wrap">
                    <b>收益：</b>
                    <span>{{profit}}</span>
                    <b>元</b>
                    <i>{{percentage}}</i>
                  </div>
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
                    <b>元</b>
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
import jsonp from 'jsonp';
export default {
  name: 'app',
  data () {
    return {
      fundCode: "",  // 基金代码
      fundName: "",  // 基金名称
      fundCurrentPrice: "",  // 基金当前金额
      fundCost: "",  // 成本价
      fundPosition: "",  // 持仓
      fundTarget: "",  // 目标市值
    }
  },
  computed: {
    /** 收益 */
    profit: function() {
      let profitValue = (this.fundCurrentPrice - this.fundCost) * this.fundPosition;
      return profitValue.toFixed(2);
    },
    /** 投入收益百分比 */
    percentage: function() {
      if(this.fundCurrentPrice && this.fundCost) {
        let currentPercentage = (this.fundCurrentPrice / this.fundCost) * 100;
        currentPercentage = currentPercentage.toFixed(2);
        return currentPercentage + "%";
      }else {
        return null;
      }
    },
    /** 应该投入的资金 */
    investment: function() {
      return null;
    }
  },
  watch: {
    fundCode: function () {
      // 只有当编号为6位时，才执行
      if(this.fundCode.length === 6) {
        // 查阅对应的名字
        jsonp(`http://quotes.money.163.com/stocksearch/json.do?type=FN&count=10&word=${this.fundCode}&t=${+new Date}`, (err, data) => {
          if(data.length > 0){
            this.fundName = data[0].name;
          }
        });
        /** 获取当前金额 */
        jsonp(`http://finance.sina.com.cn/fund/api/xh5Fund/nav/${this.fundCode}.js`, {
          name: "xh5Fund"
        }, (err, data) => {
          if(err) return console.log(err);
          /** 分解实时数据，获取最新数据 */
          this.fundCurrentPrice = data.data.split("#")[0].split(",")[1];
        })
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
</style>
