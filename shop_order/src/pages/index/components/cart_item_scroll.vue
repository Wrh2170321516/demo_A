<template>
  <scroll-view class="cartS" scroll-y="true" scroll-with-animation="true" :style="styleA">
      <!-- {{ styleA.maxHeight }} -->
    <view
      class="cart_list_item"
      v-for="(item, index) in shopping_cart"
      :key="index"
    >
      <view class="cart_list_item_info">
        <view class="cart_list_item_info_name">{{ item.name }}</view>
        <button @click="delete_shopping_cartS(index)" type="warn" size="mini">
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
</template>

<script>
export default {
  props: ["shopping_cart","styleA"],
  data() {
    return {

    };
  },
  methods: {
    // 删除购物车商品
    delete_shopping_cartS(index) {
      uni.$emit("delete-shopping-cart", index);
    },
  },
};
</script>

<style lang="scss" scoped>
.cartS {
//   height: 100%;
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
</style>