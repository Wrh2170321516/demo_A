<template>
  <view class="content">
    <!-- <view class="User_info"> 益职院专送 </view> -->
    <usermsg v-if="pageScrollToValue"></usermsg>
    <view class="menu">
      <view class="menu_top">
        <text>下单</text>
        <view class="menu_top_">
          <view class="menu_top_left">免费配送</view>
          <navigator url="#" class="menu_top_center">我的订单</navigator>
        </view>
      </view>

      <view class="menu_bottom">
        <view class="menu_bottom_center">
          <!-- 二级菜单左 -->
          <text v-if="!pageScrollToValue">{{ menu_title }}</text>
          <scroll-view :scroll-y="false" class="menu_bottom_left_">
            <view
              v-for="(item, index) in food_spu_tags"
              :key="index"
              class="menu_bottom_left_item"
              :class="
                menu_bottom_left_click === index ? 'menu_bottom_left_click' : ''
              "
              @click="menu_bottom_left_item(index)"
              :bindtap="leftClickfn"
              >{{ item.name }}</view
            >
          </scroll-view>
          <!-- 三级菜单右 -->
          <scroll-view
            class="menu_bottom_right_"
            scroll-y="true"
            scroll-with-animation="true"
            @scrolltoupper="menu_bottom_right_upper"
            @scrolltolower="menu_bottom_right_lower"
            @scroll="menu_bottom_right_scroll"
            :scroll-top="menu_scroll_top"
          >
            <!-- <view id="R_demo1">菜单1</view> -->
            <goods
              class="scroll-item rightblock"
              v-for="item in food_spu_tags"
              :key="item.tag"
              :food_spu_tags="item"
            ></goods>
          </scroll-view>
          <view class="menu_bottom_bottom">
            <!-- 购物车清单 -->
            <view class="cart" v-if="shopping_cart_visible">
              <view class="cart_list_h1">{{
                shopping_cart.length
                  ? `共${shopping_cart.length}件商品`
                  : "你还没有选购任何商品"
              }}</view>
              <view class="cart_price">
                总金额: ￥{{ orders_data.total_price.toFixed(2) }}
              </view>
              <scroll-view
                class="cart_list"
                scroll-y="true"
                scroll-with-animation="true"
              >
                <view
                  class="cart_list_item"
                  v-for="(item, index) in shopping_cart"
                  :key="index"
                >
                  <view class="cart_list_item_info">
                    <view class="cart_list_item_info_name">{{
                      item.name
                    }}</view>
                    <button
                      @click="delete_shopping_cart(index)"
                      type="warn"
                      size="mini"
                    >
                      删除
                    </button>
                    <view class="cart_list_item_info_value">
                      <view class="cart_list_item_info_price"
                        >￥{{ item.price }}</view
                      >
                      <view v-for="item1 in item.data" :key="item1.id">{{
                        item1.value
                      }}</view>
                    </view>
                  </view>
                </view>
              </scroll-view>
              <button
                type="warn"
                size="mini"
                @click="shopping_cart_visible = !shopping_cart_visible"
              >
                关闭购物车
              </button>
            </view>

            <!-- 展开购物车 -->
            <button
              type="warn"
              size="mini"
              @click="shopping_cart_visible = !shopping_cart_visible"
              v-if="!shopping_cart_visible"
            >
              {{ shopping_cart_visible ? "关闭购物车" : "展开购物车" }}
            </button>
            <!-- 总金额 -->
            <view class="cart_list_item_h2">
              总金额：￥{{ orders_data.total_price.toFixed(2) }}
            </view>
            <button @click="settled" type="warn" size="mini">确定下单</button>
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
        <!-- 下单板块 -->
        <orders v-if="orders_data.visible" :AData="orders_data"></orders>
      </view>
    </view>
    <!-- <router-view></router-view> -->
  </view>
</template>

<script>
import goods from "./components/menu_item.vue";
import user_msg from "./components/merchant_msg.vue";
import orders from "./components/orders.vue";
export default {
  components: {
    goods,
    usermsg: user_msg,
    orders,
  },
  data() {
    return {
      menu_bottom_left_click: 0,
      // 菜单滚动条位置
      menu_scroll_top: 0,
      //初始化数组
      initArr: [0],
      //初始化数字
      initNum: 0,
      // window距离顶部高度为零是否
      pageScrollToValue: true,
      // 三级标题
      menu_title: "大类",
      // 菜单数据
      // food_spu_tags: null,
      food_spu_tags: [
        {
          tag: "100",
          name: "热销",
          spus: [
            {
              id: 4689964571,
              name: "辣椒炒肉(含米饭)",
              price: 14,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8345567256, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 8345567257, value: "正常辣", mutex_attrs: null },
                    { id: 8345567258, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389293157,
              name: "螺蛳粉+煎蛋",
              price: 12.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7647282861, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 5701814932, value: "不辣", mutex_attrs: null },
                    { id: 5701814933, value: "微辣", mutex_attrs: null },
                    { id: 5701814934, value: "中辣", mutex_attrs: null },
                    { id: 5701814935, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389365131,
              name: "柳州螺蛳粉",
              price: 11.12,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7646889940, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 5701808354, value: "不辣", mutex_attrs: null },
                    { id: 5701808355, value: "微辣", mutex_attrs: null },
                    { id: 5701808356, value: "中辣", mutex_attrs: null },
                    { id: 5701808357, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389266814,
              name: "煎蛋",
              price: 1.88,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722916784, value: "1个", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 4692333157,
              name: "一碗香(含米饭）",
              price: 15.28,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8345298437, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 8345298438, value: "微辣", mutex_attrs: null },
                    { id: 8345298439, value: "中辣", mutex_attrs: null },
                    { id: 8345298440, value: "特辣", mutex_attrs: null },
                    { id: 8345298441, value: "正常辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4169204602,
              name: "热狗",
              price: 1.88,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722480737, value: "1个", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 4632137690,
              name: "朴豆角炒肉饭",
              price: 11.98,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8259287620, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4151532277,
              name: "油条",
              price: 1.98,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722477522, value: "1条", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 4965003880,
              name: "香干辣椒炒肉饭",
              price: 13.98,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8731537211, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4247884605,
              name: "牛丸",
              price: 2.38,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7740733748, value: "2粒", mutex_attrs: null }],
                },
              ],
            },
          ],
        },
        {
          tag: "act17",
          name: "折扣",
          spus: [
            {
              id: 3389365131,
              name: "柳州螺蛳粉",
              price: 11.12,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7646889940, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 5701808354, value: "不辣", mutex_attrs: null },
                    { id: 5701808355, value: "微辣", mutex_attrs: null },
                    { id: 5701808356, value: "中辣", mutex_attrs: null },
                    { id: 5701808357, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
          ],
        },
        {
          tag: "190052942",
          name: "镇店之宝",
          spus: [
            {
              id: 7348275580,
              name: "青椒肉片粉",
              price: 12.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 10850122384, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 10850122385, value: "正常辣", mutex_attrs: null },
                    { id: 10850122386, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389365131,
              name: "柳州螺蛳粉",
              price: 11.12,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7646889940, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 5701808354, value: "不辣", mutex_attrs: null },
                    { id: 5701808355, value: "微辣", mutex_attrs: null },
                    { id: 5701808356, value: "中辣", mutex_attrs: null },
                    { id: 5701808357, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 7338077894,
              name: "干捞螺蛳粉",
              price: 12.98,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 10849305557, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 10849305558, value: "不辣", mutex_attrs: null },
                    { id: 10849305559, value: "特辣", mutex_attrs: null },
                    { id: 10849305560, value: "微辣", mutex_attrs: null },
                    { id: 10849305561, value: "中辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 7348269523,
              name: "大片牛肉粉",
              price: 15.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 10849638499, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4598365733,
              name: "肉末炸酱粉",
              price: 9.9,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8172541696, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 8172541697, value: "微辣", mutex_attrs: null },
                    { id: 8172541698, value: "中辣", mutex_attrs: null },
                    { id: 8172541699, value: "不辣", mutex_attrs: null },
                    { id: 8172541700, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
          ],
        },
        {
          tag: "364384535",
          name: "擂辣拌饭",
          spus: [
            {
              id: 4632137690,
              name: "朴豆角炒肉饭",
              price: 11.98,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8259287620, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4965003880,
              name: "香干辣椒炒肉饭",
              price: 13.98,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8731537211, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 6048870703,
              name: "外婆菜炒肉饭",
              price: 14,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 9688117387, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 6056956179,
              name: "油豆腐炒肉饭",
              price: 13.28,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 9688385041, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 6947031615,
              name: "小炒拆骨肉（含饭）",
              price: 15,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 10491714212, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4044403460,
              name: "擂辣椒皮蛋拌饭",
              price: 15.28,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7646813185, value: "500克", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4692333157,
              name: "一碗香(含米饭）",
              price: 15.28,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8345298437, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 8345298438, value: "微辣", mutex_attrs: null },
                    { id: 8345298439, value: "中辣", mutex_attrs: null },
                    { id: 8345298440, value: "特辣", mutex_attrs: null },
                    { id: 8345298441, value: "正常辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4689964571,
              name: "辣椒炒肉(含米饭)",
              price: 14,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8345567256, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 8345567257, value: "正常辣", mutex_attrs: null },
                    { id: 8345567258, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4912337020,
              name: "烟笋腊肉（含饭）",
              price: 15.98,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8684669911, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
          ],
        },
        {
          tag: "190052943",
          name: "超值套餐",
          spus: [
            {
              id: 4597230396,
              name: "螺蛳粉+鸡爪",
              price: 13.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8178482964, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 8178970956, value: "微辣", mutex_attrs: null },
                    { id: 8178970957, value: "中辣", mutex_attrs: null },
                    { id: 8178970958, value: "不辣", mutex_attrs: null },
                    { id: 8178970959, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4598380935,
              name: "螺蛳粉+热狗",
              price: 13.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 8178576046, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 8178661821, value: "微辣", mutex_attrs: null },
                    { id: 8178661822, value: "中辣", mutex_attrs: null },
                    { id: 8178661823, value: "不辣", mutex_attrs: null },
                    { id: 8178661824, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389293157,
              name: "螺蛳粉+煎蛋",
              price: 12.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7647282861, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 5701814932, value: "不辣", mutex_attrs: null },
                    { id: 5701814933, value: "微辣", mutex_attrs: null },
                    { id: 5701814934, value: "中辣", mutex_attrs: null },
                    { id: 5701814935, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389293158,
              name: "螺蛳粉+卤蛋",
              price: 13.98,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7647406176, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 5701808364, value: "不辣", mutex_attrs: null },
                    { id: 5701808365, value: "微辣", mutex_attrs: null },
                    { id: 5701808366, value: "中辣", mutex_attrs: null },
                    { id: 5701808367, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389181438,
              name: "螺蛳粉+酸笋",
              price: 13.28,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7647257957, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 5701730918, value: "不辣", mutex_attrs: null },
                    { id: 5701730919, value: "微辣", mutex_attrs: null },
                    { id: 5701730920, value: "中辣", mutex_attrs: null },
                    { id: 5701730921, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389293161,
              name: "螺蛳粉+腐竹",
              price: 13.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7647260040, value: "1人份", mutex_attrs: null },
                  ],
                },
                {
                  name: "辣度",
                  values: [
                    { id: 5701814942, value: "不辣", mutex_attrs: null },
                    { id: 5701814943, value: "微辣", mutex_attrs: null },
                    { id: 5701814944, value: "中辣", mutex_attrs: null },
                    { id: 5701814945, value: "特辣", mutex_attrs: null },
                  ],
                },
              ],
            },
          ],
        },
        {
          tag: "190052944",
          name: "自选配菜",
          spus: [
            {
              id: 7351660758,
              name: "鹌鹑蛋",
              price: 2.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 10852254373, value: "3粒", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 7073160081,
              name: "白米饭",
              price: 2.5,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 10607358632, value: "1人份", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4388881358,
              name: "木耳",
              price: 2.68,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7758477915, value: "50克", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 4247884605,
              name: "牛丸",
              price: 2.38,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7740733748, value: "2粒", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 4169204602,
              name: "热狗",
              price: 1.88,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722480737, value: "1个", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 4181289264,
              name: "鱼丸",
              price: 2.68,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722875055, value: "2粒", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 4151532277,
              name: "油条",
              price: 1.98,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722477522, value: "1条", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 3389326872,
              name: "生菜",
              price: 2,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7758482695, value: "4片", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 3389405250,
              name: "腐竹（螺蛳粉专用，饭类不加）",
              price: 3,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7723000658, value: "20克", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389293162,
              name: "鸡爪",
              price: 2.68,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722915354, value: "1只", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 3389405251,
              name: "卤蛋",
              price: 2,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722915696, value: "1个", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 3389220551,
              name: "油豆腐",
              price: 2,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7723028093, value: "4粒", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 3389266814,
              name: "煎蛋",
              price: 1.88,
              attrs: [
                {
                  name: "份量",
                  values: [{ id: 7722916784, value: "1个", mutex_attrs: null }],
                },
              ],
            },
            {
              id: 3389181441,
              name: "酸笋",
              price: 3,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7723029701, value: "50克", mutex_attrs: null },
                  ],
                },
              ],
            },
          ],
        },
        {
          tag: "190052945",
          name: "饮料",
          spus: [
            {
              id: 3389220552,
              name: "矿泉水",
              price: 2.2,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7728555444, value: "500克", mutex_attrs: null },
                  ],
                },
                {
                  name: "温度",
                  values: [
                    { id: 5701801582, value: "常温", mutex_attrs: null },
                    { id: 5701801583, value: "冰", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389326879,
              name: "芬达",
              price: 3.2,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7728554446, value: "330克", mutex_attrs: null },
                  ],
                },
                {
                  name: "温度",
                  values: [
                    { id: 5701808382, value: "常温", mutex_attrs: null },
                    { id: 5701808383, value: "冰", mutex_attrs: null },
                  ],
                },
              ],
            },
            {
              id: 3389405256,
              name: "雪碧",
              price: 3.2,
              attrs: [
                {
                  name: "份量",
                  values: [
                    { id: 7728444830, value: "330克", mutex_attrs: null },
                  ],
                },
                {
                  name: "温度",
                  values: [
                    { id: 5701738724, value: "常温", mutex_attrs: null },
                    { id: 5701738725, value: "少冰", mutex_attrs: null },
                  ],
                },
              ],
            },
          ],
        },
      ],
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
      // 购物车是否显示
      shopping_cart_visible: false,
      // 用户信息
      user: null,
      // 购物车下单
      orders_data: {
        visible: false,
        // 当前购物车总金额
        total_price: 0,
        shopping_cart: [],
      },
      // 防抖
      debounceTimer: null,
    };
  },
  onLoad() {
    // 发送请求,获取菜单
    function getMenu() {
      wx.request({
        url: "http://127.0.0.1:5800/",
        success: (res) => {
          this.food_spu_tags = res.data;
        },
      });
    }
    // getMenu();

    // 获取菜品规格
    uni.$on("specification-selection", (data) => {
      this.SpecificationSelectionF(data);
    });
    // 绑定支付界面是否关闭
    uni.$on("orders-confirm", (data) => {
      this.orders_data.visible = data;
    });
    // 绑定支付界面删除购物车选项
    uni.$on("delete-shopping-cart", (index) => {
      this.delete_shopping_cart(index);
    });
  },
  // 页面首次渲染完毕时执行
  onShow() {
    // 获取高度
    function getRightblock() {
      const query = wx.createSelectorQuery();
      query.selectAll(".rightblock").boundingClientRect();
      query.selectViewport().scrollOffset();
      query.exec((res) => {
        // console.log(res[0]); //得到每个模块的高度
        res[0].map((e) => {
          this.initNum += e.height; //每个模块的高度给到初始值
          this.initArr.push(this.initNum); //添加到数组
        });
      });
    }
    // 挂载完毕执行
    setTimeout(() => {
      getRightblock.call(this);
    }, 500);
  },
  methods: {
    // 切换菜单左侧
    menu_bottom_left_item(index) {
      // console.log(index); + 15 * index
      this.menu_bottom_left_click = index;
      console.log(this.initArr[index]);
      this.menu_scroll_top = this.initArr[index];
    },
    menu_bottom_right_upper() {
      console.log("在顶部");
      this.pageScrollToValue = true;
    },
    menu_bottom_right_lower() {
      console.log("顶到最深处了");
    },
    //menu三级菜单滑动时
    menu_bottom_right_scroll(e) {
      if (this.pageScrollToValue && e.detail.scrollTop > this.initArr[1] / 5) {
        // 防抖函数100ms
        clearTimeout(this.debounceTimer);
        this.debounceTimer = setTimeout(() => {
          wx.pageScrollTo({
            scrollTop: 999, // 滚动到的位置（距离顶部 px）
            duration: 100, //滚动所需时间 如果不需要滚动过渡动画，设为0（ms）
          });
          setTimeout(() => {
            this.pageScrollToValue = false;
          }, 300);
        }, 200);
      }

      this.initArr.map((v, index) => {
        //当前滑动的高度大于数组里的项的时候，进行跳转到对应的商品区块
        if (e.detail.scrollTop + 3 >= v) {
          this.menu_bottom_left_click = index;
          // 改变标题
          this.menu_title = this.food_spu_tags[index].name;
        }
      });
      // console.log(e);
    },

    // 选择规格
    SpecificationSelectionF(data) {
      this.TempSizeConfirmSelect = {
        // 菜品名称
        id: data.id,
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
      let a = true; // 为false就停止添加
      this.TempSizeConfirmSelect.data.forEach((item, index) => {
        if (item.name === name) {
          // 把所有的mutex_attrs设置为false
          for (let index1 in this.size_confirmData.attrs[indexA].values) {
            // console.log(this.size_confirmData.attrs[indexA].values[index1]);
            this.size_confirmData.attrs[indexA].values[
              index1
            ].mutex_attrs = false;
          }
          this.TempSizeConfirmSelect.data[index].value = data.value;
          data.mutex_attrs = true;
          a = false;
        } else if (item.id === data.id) {
          a = false;
        }
      });
      // 添加到购物车
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
      this.orders_data.total_price = 0;
      this.shopping_cart.forEach((item) => {
        this.orders_data.total_price += item.price;
      });
      // console.log(this.TempSizeConfirmSelect);
      console.log(this.shopping_cart);
      this.size_confirmData.visible = false;
      // 清空菜品规格选项
      for (let indexA in this.size_confirmData.attrs) {
        for (let index1 in this.size_confirmData.attrs[indexA].values) {
          // console.log(this.size_confirmData.attrs[indexA].values[index1]);
          this.size_confirmData.attrs[indexA].values[
            index1
          ].mutex_attrs = false;
        }
      }
    },
    // 订单确认
    settled() {
      this.orders_data.shopping_cart = this.shopping_cart;
      this.orders_data.visible = true;
    },
    // 删除购物车商品
    delete_shopping_cart(index) {
      console.log(index);
      this.shopping_cart.splice(index, 1);
      this.orders_data.total_price = 0;
      this.shopping_cart.forEach((item) => {
        this.orders_data.total_price += item.price;
      });
    },
  },
};
</script>

<style lang='scss' scoped>
.content {
  background-color: #666;
  height: 100vh;
  // .User_info {
  //   height: 300rpx;
  //   background-color: #5064bc;
  //   margin-bottom: 15rpx;
  // }

  .menu {
    overflow: hidden;
    background-color: #fff;
    height: 100%;
    border-radius: 10rpx;
    .menu_top {
      text-align: center;
      border-bottom: 1px solid $uni-border-color;
      padding-bottom: 10rpx;
      text {
        font-weight: 800;
        font-size: 32rpx;
        padding-bottom: 14rpx;
        border-bottom: 6rpx solid #000;
      }
      .menu_top_ {
        width: 100%;
        height: 30rpx;
        line-height: 30rpx;
        margin-top: 20rpx;
        .menu_top_left {
          float: left;
          margin-left: 28rpx;
          font-size: 28rpx;
          color: #666;
        }

        navigator.menu_top_center {
          float: right;
          margin-right: 28rpx;
          color: #60749a;
          font-size: 32rpx;
        }
      }
    }

    .menu_bottom {
      .menu_bottom_center {
        width: 100%;
        height: 100vh;
        position: relative;
        text {
          background: #fff;
          padding-left: 25rpx;
          font-size: 39rpx;
          font-weight: 800;
        }
        .menu_bottom_left_ {
          // background-color: #666;
          float: left;
          width: 25%;
          height: 90%;
          border-right: 1px solid $uni-border-color;
          padding-bottom: 100rpx;
          .menu_bottom_left_item {
            font-size: $uni-font-size-paragraph;
            color: $uni-text-color;
            margin: 60rpx 0;
            padding-left: 20rpx;
          }
          .menu_bottom_left_click {
            font-weight: 800;
            border-left: 6rpx solid red;
          }
        }
        .menu_bottom_right_ {
          float: right;
          width: 75%;
          height: 90%;
          // background-color: pink;
          padding-bottom: 100rpx;
          padding-left: 25rpx;
        }
      }

      .menu_bottom_bottom {
        position: fixed;
        display: flex;
        bottom: 0;
        width: 100%;
        height: 8%;
        background-color: rgb(177, 163, 163);
        .cart_list_item_h2 {
          line-height: 100rpx;
        }
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
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 640rpx;
        padding: 85rpx 35rpx 20rpx;
        background-color: $uni-bg-color-hover;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        .size_confirm_name {
          position: fixed;
          top: 30rpx;
          left: 50%;
          transform: translateX(-50%);
          font-size: $uni-font-size-title;
          color: $uni-text-color;
          font-weight: 800;
          margin-bottom: 50rpx;
        }
        .size_item {
          width: 100%;
          .dl_size {
            margin-top: 30rpx;
            margin-bottom: -20rpx;
            font-size: 24rpx;
            color: #2f2f2f;
            .dl_size_dt {
              height: 40rpx;
              line-height: 40rpx;
              margin-bottom: 10rpx;
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
                  height: 60rpx;
                  min-width: 150rpx;
                  max-width: 100%;
                  font-size: 24rpx;
                  color: #333333;
                  position: relative;
                  margin-right: 20rpx;
                  box-sizing: border-box;
                  line-height: 60rpx;
                  text-align: center;
                  padding: 0 20rpx;
                  border: 1rpx solid #e5e5e5;
                  -webkit-flex: 0 0 auto;
                  flex: 0 0 auto;
                  margin-bottom: 20rpx;
                }
                .size_item_item_btn_ok {
                  color: #ffb000;
                  border: 1rpx solid #ffb000;
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

  .cart {
    width: 75%;
    height: 92%;
    position: fixed;
    top: 0;
    padding: 20rpx;
    background-color: rgb(250, 246, 246);
    margin-bottom: 20rpx;
    button {
      position: absolute;
      bottom: 0;
      left: 0;
    }
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
      max-height: 75vh;
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

  .page-section-spacing {
    margin-top: 60rpx;
  }
  .scroll-view_H {
    white-space: nowrap;
  }

  .scroll-view-item_H {
    display: inline-block;
    width: 100%;
    // height: 900rpx;
  }
}
</style>
