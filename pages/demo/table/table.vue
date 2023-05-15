<template>
  <view>
    <view class="uni-header">
      <view class="uni-group hide-on-phone">
        <view class="uni-title">{{ $t("demo.table.title") }}</view>
      </view>
      <view class="uni-group">
        <input
          class="uni-search"
          type="text"
          v-model="searchVal"
          @confirm="search"
          :placeholder="$t('common.placeholder.query')"
        />
        <button class="uni-button" type="default" size="mini" @click="search">
          {{ $t("common.button.search") }}
        </button>
        <button
          class="uni-button"
          type="primary"
          size="mini"
          @click="addAddress"
        >
          {{ $t("common.button.add") }}
        </button>
        <button class="uni-button" type="warn" size="mini" @click="delTable()">
          {{ $t("common.button.batchDelete") }}
        </button>
      </view>
    </view>
    <view class="uni-container">
      <uni-table
        :loading="loading"
        border
        stripe
        type="selection"
		toggleRowSelection="selectedItems"
        :emptyText="$t('common.empty')"
        @selection-change="selectionChange"
      >
        <uni-tr>
          <uni-th width="150" align="center">日期</uni-th>
          <uni-th width="150" align="center">姓名</uni-th>
          <uni-th align="center">地址</uni-th>
          <uni-th width="204" align="center">设置</uni-th>
        </uni-tr>
        <uni-tr v-for="(item, index) in tableData" :key="index">
          <uni-td>{{ item.date }}</uni-td>
          <uni-td>
            <view class="name">{{ item.name }}</view>
          </uni-td>
          <uni-td>{{ item.address }}</uni-td>
          <uni-td>
            <view class="uni-group">
              <button class="uni-button" size="mini" type="primary">
                {{ $t("common.button.edit") }}
              </button>
              <button
                class="uni-button"
                size="mini"
                type="warn"
                @click="oneDel(index)"
              >
                {{ $t("common.button.delete") }}
              </button>
            </view>
          </uni-td>
        </uni-tr>
      </uni-table>
      <view class="uni-pagination-box">
        <uni-pagination
          show-icon
          :page-size="pageSize"
          :current="pageCurrent"
          :total="total"
          @change="change"
        />
      </view>
    </view>
    <!-- #ifndef H5 -->
    <fix-window />
    <!-- #endif -->
  </view>
</template>

<script>
import tableData from "./tableData.js";
export default {
  data() {
    return {
      searchVal: "",
      // 全部数据
      allData: [],
      // 表格单页数据
      tableData: [],
      // 每页数据量
      pageSize: 10,
      // 当前页
      pageCurrent: 1,
      // 数据总量
      total: 0,
      loading: false,
    };
  },
  onLoad() {
    this.selectedIndexs = [];
    if (!this.allData.length) this.allData = tableData;
    this.getData(1);
  },
  methods: {
    // 多选处理
    selectedItems() {
      return this.selectedIndexs.map((i) => this.tableData[i]);
    },
    // 多选
    selectionChange(e) {
      console.log(e.detail.index);
      this.selectedIndexs = e.detail.index;
    },
    addAddress() {},
    // 删除单个
    oneDel(index) {
      let { pageCurrent, pageSize, allData } = this;
      allData.splice(index + (pageCurrent - 1) * pageSize, 1);
      if (allData.length < pageCurrent * pageSize)
        pageCurrent = Math.ceil(allData.length / pageSize);
      this.getData(pageCurrent);
    },
    //批量删除
    delTable() {
      let { pageCurrent, selectedIndexs, pageSize } = this;
      if (!selectedIndexs.length) return;
      this.allData = this.allData.filter((item, i) => {
        if (selectedIndexs.indexOf(i - (pageCurrent - 1) * pageSize) > -1) {
          return false;
        }
        return true;
      });
      if (this.allData.length < pageCurrent * pageSize)
        pageCurrent = Math.ceil(this.allData.length / pageSize);
      this.getData(pageCurrent);
      this.selectedIndexs=[];
    },
    // 分页触发
    change(e) {
      this.getData(e.current);
    },
    // 搜索
    search() {
      this.getData(1, this.searchVal);
    },
    // 获取数据
    getData(pageCurrent, value = "") {
      this.loading = true;
      this.pageCurrent = pageCurrent;
      this.request({
        pageSize: this.pageSize,
        pageCurrent: pageCurrent,
        value: value,
        success: (res) => {
          //   console.log(this.allData.length, "dddddddddddddd");
          this.tableData = res.data;
          this.total = res.total;
          this.loading = false;
        },
      });
    },
    // 伪request请求
    request(options) {
      const { pageSize, pageCurrent, success, value } = options;

      let { allData } = this;
      let total = allData.length;
      let data = allData.filter((item, index) => {
        const idx = index - (pageCurrent - 1) * pageSize;
        return idx < pageSize && idx >= 0;
      });
      if (value) {
        data = [];
        allData.forEach((item) => {
          if (item.name.indexOf(value) !== -1) {
            data.push(item);
          }
        });
        total = data.length;
      }

      setTimeout(() => {
        typeof success === "function" &&
          success({
            data: data,
            total: total,
          });
      }, 500);
    },
  },
};
</script>

<style>
/* #ifndef H5 */
page {
  padding-top: 85px;
}

/* #endif */
</style>