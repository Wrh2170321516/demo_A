<template>
  <scroll-view class="main" scroll-y="true" scroll-with-animation="true">
    <!-- 收货地址 -->
    <view class="cart_add">
      <view class="cart_add_h1">{{
        user.address ? "收货地址" : "请添加地址"
      }}</view>
      <view class="user_info" v-if="user.address" @click="handleChooseAddress">
        <view>
          {{
            user.address.provinceName +
            user.address.cityName +
            user.address.countyName +
            user.address.detailInfo
          }}
          <view>{{ user.address.userName }} {{ user.address.telNumber }}</view>
        </view>
        <text> <text> > </text> </text>
      </view>
      <!-- 获取收货地址 -->
      <button
        class="cart_add_address"
        type="primary"
        plain
        v-if="!user.address"
        @click="handleChooseAddress"
      >
        + 获取收货地址
        <!-- {{ user.address.name ? "更改地址" : "" }} -->
      </button>
    </view>

    <!-- 订单列表 -->
    <!-- <view class="cart_list">
      <view class="cart_list_h1">{{
        AData.shopping_cart.length
          ? `共${AData.shopping_cart.length}件商品`
          : "你还没有选购任何商品"
      }}</view>
      <view class="cart_price"
        >总金额: ￥{{ AData.total_price.toFixed(2) }}</view
      >
      <view
        class="cart_list_item"
        v-for="(item, index) in AData.shopping_cart"
        :key="index"
      >
        <view class="cart_list_item_info">
          <view class="cart_list_item_info_name">{{ item.name }}</view>
          <button @click="delete_shopping_cart(index)" type="warn" size="mini">
            删除
          </button>
          <view class="cart_list_item_info_value">
            <view class="cart_list_item_info_price">￥{{ item.price }}</view>
            <view v-for="item1 in item.data" :key="item1.id">{{
              item1.value
            }}</view>
          </view>
        </view>
      </view>
    </view> -->
    <view class="cart">
      <view class="cart_list_h1">{{
        AData.shopping_cart.length
          ? `共${AData.shopping_cart.length}件商品`
          : "你还没有选购任何商品"
      }}</view>
      <view class="cart_price">
        总金额: ￥{{ AData.total_price.toFixed(2) }}
      </view>
      <scroll-view
        class="cart_list"
        scroll-y="true"
        scroll-with-animation="true"
      >
        <view
          class="cart_list_item"
          v-for="(item, index) in AData.shopping_cart"
          :key="index"
        >
          <view class="cart_list_item_info">
            <view class="cart_list_item_info_name">{{ item.name }}</view>
            <button
              @click="delete_shopping_cartA(index)"
              type="warn"
              size="mini"
            >
              删除
            </button>
            <view class="cart_list_item_info_value">
              <view class="cart_list_item_info_price">￥{{ item.price }}</view>
              <view v-for="item1 in item.data" :key="item1.id">{{
                item1.value
              }}</view>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>
    <button @click="ordersConfirm">关闭</button>
  </scroll-view>
</template>

<script>
export default {
  props: ["AData"],
  data() {
    return {
      user: { address: null },
    };
  },
  methods: {
    // 获取收货地址
    handleChooseAddress() {
      // 1.获取 权限状态
      wx.getSetting({
        success: (result) => {
          // console.log(result);
          // 2. 获取权限状态
          // 主要发现一些 属性名很怪异的时候 都要使用 [] 形式来获取属性
          const scopeAddres = result.authSetting["scope.address"];
          // 判断用户的权限状态
          if (scopeAddres === true || scopeAddres === undefined) {
            wx.chooseAddress({
              success: (result1) => {
                console.log(result1);
                //获取数据
                this.user.address = result1;
                // console.log(this.user);
              },
            });
          } else {
            // 3. 用户 以前拒绝过授予权限  先引导用户打开权限页面
            wx.openSetting({
              success: (result2) => {
                // 4.可以调用  获取收获地址代码
                wx.chooseAddress({
                  success: (result3) => {
                    // console.log(result3);
                    //获取数据
                    this.user.address = result3;
                    // console.log(this.user);
                  },
                });
              },
            });
          }
        },
      });
    },
    // 关闭订单页面
    ordersConfirm() {
      uni.$emit("orders-confirm", false);
    },
    // 删除购物车商品
    delete_shopping_cartA(index) {
      uni.$emit("delete-shopping-cart", index);
    },
  },
};
</script>

<style lang="scss">
.main {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 999;
  background-color: rgb(239, 239, 239);
  .cart_add {
    padding: 20rpx;
    background-color: #fff;
    margin-bottom: 20rpx;
    .cart_add_h1 {
      font-size: 40rpx;
      font-weight: 800;
      color: #333;
    }
    .user_info {
      display: flex;
      margin-top: 20rpx;
      padding: 20rpx;
      background-color: #fff;
      border-radius: 10rpx;
      box-shadow: 0 0 10rpx #ccc;
      view {
        flex: 8;
      }
      text {
        font-size: 67rpx;
        color: #ccc;
        position: relative;
        flex: 1;
        text {
          position: absolute;
          right: 5px;
          top: 50%;
          -webkit-transform: translate(0, -50%);
          transform: translate(0, -50%);
        }
      }
    }
  }
}
.cart {
  // width: 75%;
  // max-height: 60vh;
  // position: fixed;
  // top: 0;
  padding: 20rpx;
  background-color: rgb(250, 246, 246);
  margin-bottom: 20rpx;
  .cart_list_h1 {
    font-size: 40rpx;
    font-weight: 800;
    color: #333;
  }
  .cart_price {
    margin-top: 20rpx;
    font-size: 40rpx;
    font-weight: 800;
    color: red;
  }
  .cart_list {
    // width: 75%;
    max-height: 600rpx;
    // position: fixed;
    // top: 0;
    // padding: 20rpx;
    // background-color: rgb(250, 246, 246);
    // margin-bottom: 20rpx;

    .cart_list_item {
      padding: 20rpx;
      background-color: rgb(241, 237, 237);
      border-radius: 10rpx;
      box-shadow: 0 0 10rpx #ccc;

      .cart_list_item_info {
        position: relative;
        display: flex;
        justify-content: space-between;
        button {
          position: absolute;
          left: 0;
          bottom: 0;
          height: 40rpx;
          line-height: 40rpx;
          padding: 0 15rpx;
          border: 1px solid #ccc;
          margin-right: 20rpx;
          &:active {
            background-color: #ccc;
          }
        }
        .cart_list_item_info_name {
          font-size: 30rpx;
          font-weight: 800;
          color: #333;
        }
        .cart_list_item_info_value {
          text-align: right;
          font-size: 30rpx;
          font-weight: 800;
          color: #333;
          .cart_list_item_info_price {
            color: red;
          }
          .cart_list_item_info_price_item {
            font-size: 30rpx;
            font-weight: 800;
            color: #333;
          }
        }
      }
    }
  }
}
</style>