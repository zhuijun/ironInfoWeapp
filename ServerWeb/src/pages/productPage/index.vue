<template>
  <div class="product-container">
    <div class="product-list">
      <el-row :gutter="20">
        <el-col :span="6" v-for="v in irons" :key="v.id">
          <el-card class="product-item" :body-style="{ padding: '0px' }">
            <img :src="v.photo" class="image">
            <div style="padding: 14px;">
              <span class="name">{{v.name}}</span>
              <div class="bottom">
                <time class="price">￥ {{v.new_price}}</time>
                <el-button type="warning" icon="el-icon-shopping-bag-1" circle @click="openBuy(v)"></el-button>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>
    <el-dialog
      title="购买"
      :visible.sync="buyDialog"
      width="60%">
      <template v-if="buyStep === 1">
        <table class="table" border="0">
          <thead>
            <tr>
              <th>名称</th>
              <th>单价(元/吨)</th>
              <th>重量</th>
              <th>总价</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{buyData.subject}}</td>
              <td>{{buyData.price}}</td>
              <td>
                <el-input-number
                  size="mini"
                  controls-position="right"
                  v-model="buyData.number"
                  @change="handleChange"
                  :min="1">
                </el-input-number>吨
              </td>
              <td>{{buyData.totalAmount}} 元</td>
            </tr>
          </tbody>
        </table>
        <div class="bottom">
          <div>总计: {{buyData.totalAmount}} 元</div>
          <div>应付: {{buyData.totalAmount}} 元</div>
        </div>
        <el-divider>收货信息</el-divider>
        <el-form label-position="left" :model="buyData" :rules="rules" label-width="80px" ref="form" >
          <el-form-item label="收货地址" prop="address">
            <el-input size="medium" v-model="buyData.address"></el-input>
          </el-form-item>
          <el-form-item label="收货电话" prop="mobile">
            <el-input-number size="medium" v-model="buyData.mobile" :controls="false"></el-input-number>
          </el-form-item>
          <el-form-item label="备注">
            <el-input size="medium" v-model="buyData.body" type="textarea" rows="5"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button size="mini" @click="buyDialog = false">取 消</el-button>
          <el-button size="mini" type="primary" @click="buy">购 买</el-button>
        </span>
      </template>
      <template v-else-if="buyStep === 2">
        <div class="fingerprint-spinner">
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
        </div>
        <div class="loading-text">[沙箱环境]用户名: yyakxq3055@sandbox.com, 密码: 111111</div>
        <el-button class="finish-btn" type="primary">已完成付款</el-button>
      </template>
      <template v-else-if="3">
        <div class="result success" v-if="result">
          <div class="el-icon-circle-check"></div>
          <div>付款成功</div>
        </div>
        <div class="result warn" v-else>
          <div class="el-icon-circle-close"></div>
          <div>付款进行中</div>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { getAllIron } from '@/api/iron.api.js'
import { pay, getPayStatus } from '@/api/buy.api.js'
export default {
  name: 'ProductPage',
  data () {
    return {
      irons: [],
      currentPrice: 0,
      buyDialog: false,
      buyData: {
        outTradeNo: '',
        totalAmount: 0,
        subject: '',
        body: '',
        address: '',
        mobile:  '',
        number: 1
      },
      buyDetails: false,
      rules: {
        address: [
          { required: true, message: '请输入收货地址', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入收货电话', trigger: 'blur' }
        ]
      },
      buyStep: 1,
      result: false,
      checkInterval: null
    }
  },
  mounted () {
    this.getIrons()
  },
  methods: {
    getIrons () {
      getAllIron().then(res => {
        if (res) {
          this.irons = res
        }
      })
    },
    openBuy (iron) {
      this.currentPrice = iron.new_price
      this.buyData.totalAmount = iron.new_price
      this.buyData.subject = iron.name
      this.buyData.number = 1
      this.buyDialog = true
    },
    handleChange (v) {
      this.buyData.totalAmount = v * this.currentPrice
    },
    buy () {
      this.$refs['form'].validate(async (valid) => {
        if (valid) {
          this.buyData.outTradeNo = this.guid()
          const url = await pay(this.buyData)
          this.buyStep = 2
          window.open(url)
          this.checkPay()
        } else {
          return false
        }
      })
    },
    guid () {
      return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
        const r = Math.random() * 16 | 0
        let v = c === 'x' ? r : (r & 0x3 |0x8)
        return v.toString(16)
      })
    },
    checkPay () {
      this.checkInterval = setInterval(async () => {
        const result = await getPayStatus(this.buyData.outTradeNo)
        if (result.status) {
          clearInterval(this.checkInterval)
          this.result = true
          this.buyStep = 3
        }
      }, 1000)
    }
  }
}
</script>
<style lang="scss" scoped>
  .product-container {
    .product-list {
      .product-item {
        margin-bottom: 20px;
      }
      .bottom {
        margin-top: 13px;
        line-height: 12px;
        display: flex;
        justify-content: space-between;
        align-items: center;
      }

      .name {
        font-size: 18px;
        font-weight: bold;
      }
      .image {
        width: 100%;
        height: 200px;
        display: block;
      }
      .price {
        font-size: 18px;
        font-weight: bold;
        color: #ff0000;
      }
    }
    .table {
      width: 100%;
      border-bottom: 1px solid #dfdfdf;
      thead {
        background: #dfdfdf;
      }
      th {
        width: 200px;
      }
      th, td {
        padding: 10px;
        text-align: center;
      }
    }
    .bottom {
      margin-top: 20px;
      line-height: 2;
      text-align: right;
    }
  }
  .loading-text {
    padding-bottom: 20px;
    text-align: center;
    font-size: 18px;
    font-weight: bold;
  }
  .finish-btn {
    display: block;
    margin: auto;
  }
  .result {
    line-height: 2;
    text-align: center;
    font-size: 28px;
    font-weight: bold;
    .el-icon-circle-check,
    .el-icon-circle-close {
      font-size: 50px;
    }
    &.success {
      color: #67C23A;
    }
    &.fail {
      color: #F56C6C;
    }
    &.warn {
      color: #E6A23C;
    }
  }
  .fingerprint-spinner, .fingerprint-spinner * {
      box-sizing: border-box;
    }

    .fingerprint-spinner {
      margin: auto;
      height: 124px;
      width: 124px;
      padding: 2px;
      overflow: hidden;
      position: relative;
    }

    .fingerprint-spinner .spinner-ring {
      position: absolute;
      border-radius: 50%;
      border: 2px solid transparent;
      border-top-color: #ff1d5e;
      animation: fingerprint-spinner-animation 1500ms cubic-bezier(0.680, -0.750, 0.265, 1.750) infinite forwards;
      margin: auto;
      bottom: 0;
      left: 0;
      right: 0;
      top: 0;
    }

    .fingerprint-spinner .spinner-ring:nth-child(1) {
      height: calc(90px / 9 + 0 * 90px / 9);
      width: calc(90px / 9 + 0 * 90px / 9);
      animation-delay: calc(50ms * 1);
    }

    .fingerprint-spinner .spinner-ring:nth-child(2) {
      height: calc(90px / 9 + 1 * 90px / 9);
      width: calc(90px / 9 + 1 * 90px / 9);
      animation-delay: calc(50ms * 2);
    }

    .fingerprint-spinner .spinner-ring:nth-child(3) {
      height: calc(90px / 9 + 2 * 90px / 9);
      width: calc(90px / 9 + 2 * 90px / 9);
      animation-delay: calc(50ms * 3);
    }

    .fingerprint-spinner .spinner-ring:nth-child(4) {
      height: calc(90px / 9 + 3 * 90px / 9);
      width: calc(90px / 9 + 3 * 90px / 9);
      animation-delay: calc(50ms * 4);
    }

    .fingerprint-spinner .spinner-ring:nth-child(5) {
      height: calc(90px / 9 + 4 * 90px / 9);
      width: calc(90px / 9 + 4 * 90px / 9);
      animation-delay: calc(50ms * 5);
    }

    .fingerprint-spinner .spinner-ring:nth-child(6) {
      height: calc(90px / 9 + 5 * 90px / 9);
      width: calc(90px / 9 + 5 * 90px / 9);
      animation-delay: calc(50ms * 6);
    }

    .fingerprint-spinner .spinner-ring:nth-child(7) {
      height: calc(90px / 9 + 6 * 90px / 9);
      width: calc(90px / 9 + 6 * 90px / 9);
      animation-delay: calc(50ms * 7);
    }

    .fingerprint-spinner .spinner-ring:nth-child(8) {
      height: calc(90px / 9 + 7 * 90px / 9);
      width: calc(90px / 9 + 7 * 90px / 9);
      animation-delay: calc(50ms * 8);
    }

    .fingerprint-spinner .spinner-ring:nth-child(9) {
      height: calc(90px / 9 + 8 * 90px / 9);
      width: calc(90px / 9 + 8 * 90px / 9);
      animation-delay: calc(50ms * 9);
    }

    @keyframes fingerprint-spinner-animation {
      100% {
        transform: rotate( 360deg );
      }
    }
</style>
