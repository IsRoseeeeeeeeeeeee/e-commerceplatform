<script setup>
import { ref } from 'vue'
import { getCheckInfoService, addAddressService, submitOrderService } from '@/apis/order'
import { ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'
import { useCartStore } from '@/stores'
const router = useRouter()
const checkInfo = ref({}) // 订单对象
const curAddress = ref({}) // 地址对象
const getCheckInfo = async () => {
  const res = await getCheckInfoService()
  checkInfo.value = res.result
  curAddress.value = res.result.userAddresses.find((item) => item.isDefault === 0)
}
getCheckInfo()
const toggleFlag = ref(false) // 切换地址
const addFlag = ref(false) // 添加地址

// 切换地址
const activeAddress = ref({})
const switchAddress = (item) => {
  activeAddress.value = item
}
//确认切换地址
const confirmAddress = () => {
  if (activeAddress.value.id) {
    curAddress.value = activeAddress.value
    toggleFlag.value = false
  } else {
    ElMessage.warning('请选择地址')
  }
}
//添加地址信息
const addressInfo = [
  '姓名',
  '联系方式',
  '省份编码',
  '城市编码',
  '地区编码',
  '详细地址',
  '邮政编码',
  '地址标签',
  '是否为默认地址',
  '完整地址'
]
const items = ref(new Array(10).fill(''))

const handleAddAddress = async () => {
  const receiver = ref(items.value[0])
  const contact = ref(items.value[1])
  const provinceCode = ref(items.value[2])
  const cityCode = ref(items.value[3])
  const countyCode = ref(items.value[4])
  const address = ref(items.value[5])
  const postalCode = ref(items.value[6])
  const addressTags = ref(items.value[7])
  const isDefault = ref(items.value[8])
  const fullLocation = ref(items.value[9])
  if (
    receiver.value &&
    contact.value &&
    provinceCode.value &&
    cityCode.value &&
    countyCode.value &&
    address.value &&
    postalCode.value &&
    addressTags.value &&
    isDefault.value &&
    fullLocation.value
  ) {
    // 发送请求
    await addAddressService({
      receiver: receiver.value,
      contact: contact.value,
      provinceCode: provinceCode.value,
      cityCode: cityCode.value,
      countyCode: countyCode.value,
      address: address.value,
      postalCode: postalCode.value,
      addressTags: addressTags.value,
      isDefault: isDefault.value,
      fullLocation: fullLocation.value
    })
    ElMessage.success('添加地址成功')
    addFlag.value = false
    getCheckInfo()
  } else {
    ElMessage.warning('请填写完整信息')
  }
}

//提交订单
const cartStore = useCartStore()
const submitOrder = async () => {
  const res = await submitOrderService({
    deliveryTimeType: 1,
    payType: 1,
    payChannel: 1,
    buyerMessage: '',
    goods: checkInfo.value.goods.map((item) => ({ skuId: item.skuId, count: item.count })),
    addressId: curAddress.value.id
  })
  const id = res.result.id
  router.push(`/pay/${id}`)
  cartStore.getCartList()
}
</script>

<template>
  <div class="xtx-pay-checkout-page">
    <div class="container">
      <div class="wrapper">
        <!-- 收货地址 -->
        <h3 class="box-title">收货地址</h3>
        <div class="box-body">
          <div class="address">
            <div class="text">
              <div class="none" v-if="!curAddress">您需要先添加收货地址才可提交订单。</div>
              <ul v-else>
                <li>
                  <span>收<i />货<i />人：</span>{{ curAddress.receiver }}
                </li>
                <li><span>联系方式：</span>{{ curAddress.contact }}</li>
                <li><span>收货地址：</span>{{ curAddress.fullLocation }} {{ curAddress.address }}</li>
              </ul>
            </div>
            <div class="action">
              <el-button size="large" @click="toggleFlag = true">切换地址</el-button>
              <el-button size="large" @click="addFlag = true">添加地址</el-button>
            </div>
          </div>
        </div>
        <!-- 商品信息 -->
        <h3 class="box-title">商品信息</h3>
        <div class="box-body">
          <table class="goods">
            <thead>
              <tr>
                <th width="520">商品信息</th>
                <th width="170">单价</th>
                <th width="170">数量</th>
                <th width="170">小计</th>
                <th width="170">实付</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="i in checkInfo.goods" :key="i.id">
                <td>
                  <a href="javascript:;" class="info">
                    <img :src="i.picture" alt="" />
                    <div class="right">
                      <p>{{ i.name }}</p>
                      <p>{{ i.attrsText }}</p>
                    </div>
                  </a>
                </td>
                <td>&yen;{{ i.price }}</td>
                <td>{{ i.price }}</td>
                <td>&yen;{{ i.totalPrice }}</td>
                <td>&yen;{{ i.totalPayPrice }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <!-- 配送时间 -->
        <h3 class="box-title">配送时间</h3>
        <div class="box-body">
          <!-- 默认不限制送货时间 -->
          <a class="my-btn active" href="javascript:;">不限送货时间：周一至周日</a>
          <a class="my-btn" href="javascript:;">工作日送货：周一至周五</a>
          <a class="my-btn" href="javascript:;">双休日、假日送货：周六至周日</a>
        </div>
        <!-- 支付方式 -->
        <h3 class="box-title">支付方式</h3>
        <div class="box-body">
          <!-- 默认在线支付 -->
          <a class="my-btn active" href="javascript:;">在线支付</a>
          <a class="my-btn" href="javascript:;">货到付款</a>
          <span style="color: #999">货到付款需付5元手续费</span>
        </div>
        <!-- 金额明细 -->
        <h3 class="box-title">金额明细</h3>
        <div class="box-body">
          <div class="total">
            <dl>
              <dt>商品件数：</dt>
              <dd>{{ checkInfo.summary?.goodsCount }}件</dd>
            </dl>
            <dl>
              <dt>商品总价：</dt>
              <dd>¥{{ checkInfo.summary?.totalPrice.toFixed(2) }}</dd>
            </dl>
            <dl>
              <dt>运<i></i>费：</dt>
              <dd>¥{{ checkInfo.summary?.postFee.toFixed(2) }}</dd>
            </dl>
            <dl>
              <dt>应付总额：</dt>
              <dd class="price">{{ checkInfo.summary?.totalPayPrice.toFixed(2) }}</dd>
            </dl>
          </div>
        </div>
        <!-- 提交订单 -->
        <div class="submit">
          <el-button type="primary" size="large" @click="submitOrder">提交订单</el-button>
        </div>
      </div>
    </div>
  </div>
  <!-- 切换地址 -->
  <el-dialog v-model="toggleFlag" title="切换收货地址" width="30%" center>
    <div class="addressWrapper">
      <div
        class="text item"
        :class="{ active: activeAddress.id === item.id }"
        @click="switchAddress(item)"
        v-for="item in checkInfo.userAddresses"
        :key="item.id"
      >
        <ul>
          <li>
            <span>收<i />货<i />人：</span>{{ item.receiver }}
          </li>
          <li><span>联系方式：</span>{{ item.contact }}</li>
          <li><span>收货地址：</span>{{ item.fullLocation + item.address }}</li>
        </ul>
      </div>
    </div>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="toggleFlag = false">取消</el-button>
        <el-button type="primary" @click="confirmAddress">确定</el-button>
      </span>
    </template>
  </el-dialog>
  <!-- 添加地址 -->
  <el-dialog v-model="addFlag" title="添加收货地址" width="30%" center>
    <div class="addAddress">
      <div v-for="(item, index) in addressInfo" :key="item">
        <p class="addTitle">{{ item }}</p>
        <el-input
          :placeholder="`请输入${index !== 8 ? item : '是否为默认地址，0为是1为不是'}`"
          v-model="items[index]"
          class="inputContent"
        >
        </el-input>
      </div>
    </div>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="addFlag = false">取消</el-button>
        <el-button type="primary" @click="handleAddAddress"> 确定 </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<style scoped lang="scss">
.xtx-pay-checkout-page {
  margin-top: 20px;

  .wrapper {
    background: #fff;
    padding: 0 20px;

    .box-title {
      font-size: 16px;
      font-weight: normal;
      padding-left: 10px;
      line-height: 70px;
      border-bottom: 1px solid #f5f5f5;
    }

    .box-body {
      padding: 20px 0;
    }
  }
}

.address {
  border: 1px solid #f5f5f5;
  display: flex;
  align-items: center;

  .text {
    flex: 1;
    min-height: 90px;
    display: flex;
    align-items: center;

    .none {
      line-height: 90px;
      color: #999;
      text-align: center;
      width: 100%;
    }

    > ul {
      flex: 1;
      padding: 20px;

      li {
        line-height: 30px;

        span {
          color: #999;
          margin-right: 5px;

          > i {
            width: 0.5em;
            display: inline-block;
          }
        }
      }
    }

    > a {
      color: $xtxColor;
      width: 160px;
      text-align: center;
      height: 90px;
      line-height: 90px;
      border-right: 1px solid #f5f5f5;
    }
  }

  .action {
    width: 420px;
    text-align: center;

    .btn {
      width: 140px;
      height: 46px;
      line-height: 44px;
      font-size: 14px;

      &:first-child {
        margin-right: 10px;
      }
    }
  }
}

.goods {
  width: 100%;
  border-collapse: collapse;
  border-spacing: 0;

  .info {
    display: flex;
    text-align: left;

    img {
      width: 70px;
      height: 70px;
      margin-right: 20px;
    }

    .right {
      line-height: 24px;

      p {
        &:last-child {
          color: #999;
        }
      }
    }
  }

  tr {
    th {
      background: #f5f5f5;
      font-weight: normal;
    }

    td,
    th {
      text-align: center;
      padding: 20px;
      border-bottom: 1px solid #f5f5f5;

      &:first-child {
        border-left: 1px solid #f5f5f5;
      }

      &:last-child {
        border-right: 1px solid #f5f5f5;
      }
    }
  }
}

.my-btn {
  width: 228px;
  height: 50px;
  border: 1px solid #e4e4e4;
  text-align: center;
  line-height: 48px;
  margin-right: 25px;
  color: #666666;
  display: inline-block;

  &.active,
  &:hover {
    border-color: $xtxColor;
  }
}

.total {
  dl {
    display: flex;
    justify-content: flex-end;
    line-height: 50px;

    dt {
      i {
        display: inline-block;
        width: 2em;
      }
    }

    dd {
      width: 240px;
      text-align: right;
      padding-right: 70px;

      &.price {
        font-size: 20px;
        color: $priceColor;
      }
    }
  }
}

.submit {
  text-align: right;
  padding: 60px;
  border-top: 1px solid #f5f5f5;
}

.addressWrapper {
  max-height: 500px;
  overflow-y: auto;
}

.addAddress {
  .addTitle {
    margin: 10px 0;
  }
}

.text {
  flex: 1;
  min-height: 90px;
  display: flex;
  align-items: center;

  &.item {
    border: 1px solid #f5f5f5;
    margin-bottom: 10px;
    cursor: pointer;

    &.active,
    &:hover {
      border-color: $xtxColor;
      background: lighten($xtxColor, 50%);
    }

    > ul {
      padding: 10px;
      font-size: 14px;
      line-height: 30px;
    }
  }
}
</style>
