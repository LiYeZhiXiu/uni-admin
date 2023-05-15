<template>
  <view class="pointer">
    <template v-for="(item, index) in data">
      <template v-if="!item.children || !item.children.length">
        <uni-menu-item :index="item">
          <view :class="item.icon"></view>
          <text :class="{ title: item.icon }">{{
            item.text || item.name
          }}</text>
        </uni-menu-item>
      </template>
      <uni-sub-menu v-else :index="item">
        <template v-slot:title>
          <view :class="item.icon"></view>
          <text :class="{ title: item.icon }">{{
            item.text || item.name
          }}</text>
        </template>
        <uni-menu-sidebar
          class="item-bg"
          :data="item.children"
          :key="item._id"
        />
      </uni-sub-menu>
    </template>
  </view>
</template>

<script>
import { mapState, mapActions } from "vuex";
export default {
  name: "uniMenuSidebar",
  props: {
    data: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  data() {
    return {};
  },
  onload() {
    console.log(userInfo, "ddddd");
  },
  computed: {},
  methods: {},
};
</script>

<style lang="scss">
.title {
  margin-left: 5px;
}
</style>
