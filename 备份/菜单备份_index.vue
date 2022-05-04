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
    <!-- 选择规格 -->
    <view class="size_confirm_mask" v-if="size_confirmData.visible">
      <view class="size_confirm">
        <view class="size_confirm_name">{{ size_confirmData.name }}</view>
        <view class="size_item">
          <view
            class="dl_size"
            v-for="(item, index) in size_confirmData.attrs"
            :key="index"
          >
            <view class="dl_size_dt">{{ item.name }}</view>
            <view class="dl_size_dd">
              <view
                class="size_item_item"
                v-for="item1 in item.values"
                :key="item1.id"
              >
                <!-- <view class="size_item_item_name"></view> -->
                <view
                  class="size_item_item_btn"
                  @click="sizeConfirmSelect(item.name, item1, index)"
                  :class="{
                    size_item_item_btn_ok: item1.mutex_attrs === true,
                  }"
                  >{{ item1.value }}</view
                >
              </view>
            </view>
          </view>

          <view class="price">￥{{ size_confirmData.price }}</view>
          <button
            class="mini-btn"
            type="default"
            size="mini"
            @click="size_confirmData.visible = false"
          >
            取消
          </button>
          <button
            class="mini-btn"
            type="warn"
            size="mini"
            @click="sizeConfirmOK"
          >
            选购
          </button>
        </view>
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
    return {
      // 菜品规格选择
      size_confirmData: {
        visible: false,
        id: 999,
        name: "",
        price: 888,
        attrs: [
          {
            name: "辣度",
            values: [
              { id: 8345567257, value: "正常辣", mutex_attrs: null },
              // { "id": 8345567258, "value": "特辣", "mutex_attrs": null }
            ],
          },
        ],
      },
      // 零时选择栏
      TempSizeConfirmSelect: { data: [] },
      // 总购物车
      shopping_cart: [],
    };
  },
  methods: {
    // 选择规格
    sizeConfirm(data) {
      uni.$emit('specification-selection', data)
      this.TempSizeConfirmSelect = {
        // 菜品名称
        name: data.name,
        price: data.price,
        data: [],
      };
      this.size_confirmData = {
        visible: true,
        ...data,
      };
    },
    // 选择规格小项
    sizeConfirmSelect(name, data, indexA) {
      // data.mutex_attrs = true;
      // console.log(this.TempSizeConfirmSelect.data);
      // 判断是否存在
      let a = true;
      this.TempSizeConfirmSelect.data.forEach((item, index) => {
        if (item.name === name) {
          for (let index1 in this.size_confirmData.attrs[indexA].values) {
            // console.log(this.size_confirmData.attrs[indexA].values[index1]);
            this.size_confirmData.attrs[indexA].values[
              index1
            ].mutex_attrs = false;
          }
          this.TempSizeConfirmSelect.data.splice(index, 1);
          // a = false;
        } else if (item.id === data.id) {
          a = false;
        }
      });

      if (a) {
        data.mutex_attrs = true;
        // 数据打包
        let d = {
          // 菜品规格
          name: name,
          id: data.id,
          value: data.value,
          mutex_attrs: data.mutex_attrs,
        };
        this.TempSizeConfirmSelect.data.push(d);
      }
    },
    // 确认选择,加入购物车
    sizeConfirmOK() {
      this.shopping_cart.push(this.TempSizeConfirmSelect);
      // console.log(this.TempSizeConfirmSelect);
      console.log(this.shopping_cart);
      this.size_confirmData.visible = false;
    },
  },
};
</script>

<style scoped lang="scss">
.content1 {
  font-weight: 800;
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
  // 选择规格
  .size_confirm_mask {
    position: fixed;
    z-index: 999;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: $uni-bg-color-mask;
    .size_confirm {
      position: relative;
      top: 0;
      left: 0;
      width: 85%;
      height: 600rpx;
      margin: 50% auto;
      transform: translateY(-25%);
      background-color: $uni-bg-color-hover;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      .size_item {
        .dl_size {
          margin-top: 15px;
          margin-bottom: -10px;
          font-size: 12px;
          color: #2f2f2f;
          .dl_size_dt {
            height: 20px;
            line-height: 20px;
            margin-bottom: 5px;
          }
          .dl_size_dd {
            display: -webkit-flex;
            display: flex;
            -webkit-flex-wrap: wrap;
            flex-wrap: wrap;
            .size_item_item {
              .size_item_item_btn {
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
                height: 30px;
                min-width: 75px;
                max-width: 100%;
                font-size: 12px;
                color: #333333;
                position: relative;
                margin-right: 10px;
                box-sizing: border-box;
                line-height: 30px;
                text-align: center;
                padding: 0 10px;
                border: 1rpx solid #e5e5e5;
                -webkit-flex: 0 0 auto;
                flex: 0 0 auto;
                margin-bottom: 10px;
              }
              .size_item_item_btn_ok {
                color: #ffb000;
                border: 1px solid #ffb000;
                font-family: "PingFangSC-Medium", "Hiragino Sans GB", Arial,
                  Helvetica, "宋体", sans-serif;
                background-color: #fffbf1;
              }
            }
          }
        }
      }
    }
  }
}
</style>
