<template>
  <view class="content1">
    <view class="h1"> {{ food_spu_tags.name }} </view>
    <view class="goods">
      <view
        class="goods_item"
        v-for="item in food_spu_tags.spus"
        :key="item.id"
      >
        <image
          mode="aspectFill"
          src="http://p0.meituan.net/deal/a2a739fe32546826505d3a834b6c870b178238.jpg"
        ></image>
        <view class="name">
          <view class="name_">
            {{ item.name }}
          </view>
          <view class="price">价格: ￥{{ item.price }}</view>
        </view>
        <button
          class="mini-btn"
          @click="sizeConfirm(item)"
          type="warn"
          size="mini"
        >
          选规格
        </button>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  props: ["food_spu_tags"],
  //1.声明自定义事件：组件的自定义事件
  emits: ["specification-selection"],
  data() {
    return {};
  },
  methods: {
    // 选择规格
    sizeConfirm(data) {
      uni.$emit("specification-selection", data);
    },
  },
};
</script>

<style scoped lang="scss">
.content1 {
  font-weight: 800;
  overflow: hidden;
  .h1 {
    font-size: 29rpx;
    font-weight: 600;
    margin: 0 0 30rpx 0;
    background-color: #fff;
  }

  .goods_item {
    position: relative;
    display: flex;
    align-items: center;
    height: 200rpx;
    margin-bottom: 35rpx;
    image {
      height: 200rpx;
      width: 200rpx;
    }
    .name {
      display: flex;
      flex-direction: column;
      // justify-content: space-between;
      height: 100%;
      // width: 100%;
      .name_ {
        font-size: 30rpx;
        font-weight: 600;
        margin: 0 0 10rpx 0;
      }
      .price {
        color: red;
      }
    }

    .mini-btn {
      position: absolute;
      right: 20rpx;
      bottom: 15rpx;
      padding: 0;
      margin: 0;
      width: 110rpx;
      height: 45rpx;
      line-height: 40rpx;
      font-weight: normal;
    }
  }
}
</style>
