<template>
  <view>
    <uni-nav-menu
      :active="value"
      activeKey="value"
      :activeTextColor="activeTextColor"
      :uniqueOpened="uniqueOpened"
      @select="onSelect"
    >
      <uni-menu-sidebar :data="userMenu"></uni-menu-sidebar>
      <uni-menu-sidebar :data="staticMenu"></uni-menu-sidebar>
    </uni-nav-menu>
  </view>
</template>

<script>
import { mapState, mapActions } from "vuex";
import { buildMenus } from "./util.js";
import data from "./data";
export default {
  data() {
    return {
      menus: [],
      // userMenu: [],
      famliy: [],
    };
  },
  mixins: [uniCloud.mixinDatacom],
  props: {
    // 当前激活菜单的 url
    value: {
      type: String,
      default: "",
    },
    // 当前激活菜单的文字颜色
    activeTextColor: {
      type: String,
      default: "#42B983",
    },
    // 是否只保持一个子菜单的展开
    uniqueOpened: {
      type: Boolean,
      default: false,
    },
    staticMenu: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  watch: {
    // localdata: {
    //   handler(newval) {
    //     if (this.hasLocalData(newval)) {
    //       this.userMenu = newval;
    //     }
    //   },
    //   immediate: true,
    // },
    // TODO 暂时无需监听，需要看后面会出现什么问题
    menus: {
      immediate: true,
      handler(newVal, oldVal) {
        const item = this.menus.find((m) => m.value === this.$route.path);
        // 设置面包屑
        if (item) {
          this.getMenuAncestor(item.menu_id, newVal);
          item && this.setRoutes && this.setRoutes(this.famliy);
        }
      },
    },
    $route: {
      immediate: false,
      handler(val, old) {
        if (val.fullPath !== old.fullPath) {
          this.famliy = [];
          const menu = this.menus.find((m) => m.value === val.path);
          const menu_id = menu && menu.menu_id;
          this.getMenuAncestor(menu_id, this.menus);
          this.setRoutes && this.setRoutes(this.famliy);
        }
      },
    },
  },
  created() {
    console.log(data);
    if (this.hasLocalData(this.localdata)) return;
    // this.load()
  },
  computed: {
    ...mapState("user", ["userInfo"]),
    userMenu() {
      // console.log(this.menus, 1111);
      return this.getUserMenu((this.menus.length && this.menus) || data);
    },
  },
  methods: {
    ...mapActions({
      setRoutes: "app/setRoutes",
    }),
    getUserMenu(menuList) {
      // console.log(this.state.userInfo)
      //   if(userInfo) console.log(userInfo);
      //   const { permission, role } = {
      //     username: "111111",
      //     permission: ["editor"],
      //     role: ["admin"],
      //   };
      let { permission, role } = uniCloud.getCurrentUserInfo();
      permission = ["editor"];
      role = ["admin"];
      // 删除无权限访问的菜单
      // if (!role.includes("admin")) {
      //   this.permissionFilter(this.markChildNode(menuList), permission);
      //   console.log(menuList, "dddddddddddd");
      // }
      menuList = this.markChildNode(menuList);
      // 删除无权限访问的菜单
      if (!role.includes("admin")) {
        menuList = menuList.filter((item) => {
          if (item.isLeafNode) {
            if (item.permission && item.permission.length) {
              return item.permission.some(
                (item) => permission.indexOf(item) > -1
              );
            }
            return false;
          }
          return true;
        });
      }
      console.log(buildMenus(menuList), "111111");
      return buildMenus(menuList);
    },
    // 权限过滤
    permissionFilter(menuList, permission) {
      menuList = menuList.filter((item, i) => {
        if (item.isLeafNode) {
          if (item.children && item.children.length) {
            item.children = this.permissionFilter(
              this.markChildNode(item.children),
              permission
            );
            return item.children;
          }
          if (item.permission && item.permission.length) {
            return item.permission.some(
              (item) => permission.indexOf(item) > -1
            );
          }
          return true;
        }
        return true;
      });
      return menuList;
    },
    markChildNode(list) {
      // 标记叶子节点
      list.map((item) => {
        if (
          !list.some((subMenuItem) => subMenuItem.parent_id === item.menu_id)
        ) {
          item.isLeafNode = true;
        }
      });
      return list;
    },
    onSelect(menu) {
      this.famliy = [];
      this.getMenuAncestor(menu.menu_id, this.menus || data);
      this.emit(menu);
    },
    emit(menu) {
      this.$emit("select", menu, this.famliy);
      this.$emit("input", menu.value);
    },
    hasLocalData(value) {
      return Array.isArray(value) && value.length > 0;
    },
    load() {
      if (this.mixinDatacomLoading == true) {
        return;
      }
      this.mixinDatacomLoading = true;
      this.mixinDatacomGet()
        .then((res) => {
          this.mixinDatacomLoading = false;
          const { data, count } = res.result;
          this.menus = data;
          this.userMenu = this.getUserMenu(this.menus);
        })
        .catch((err) => {
          this.mixinDatacomLoading = false;
          this.mixinDatacomErrorMessage = err;
        });
    },
    getMenuAncestor(menuId, menus) {
      menus.forEach((item) => {
        if (item.menu_id === menuId) {
          const route = {
            name: item.text,
          };
          const path = item.value;
          if (path) {
            route.to = {
              path,
            };
          }
          this.famliy.unshift(route);
          const parent_id = item.parent_id;
          if (parent_id) {
            this.getMenuAncestor(parent_id, menus);
          }
        }
      });
      //   return famliy
    },
  },
};
</script>

<style>
</style>
