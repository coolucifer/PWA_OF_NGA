<!-- 帖子列表 -->
<template>
  <page-frame class="post-list">
    <page-header slot="header" :title="pageTitle" :menuList="menuList" @menuClick="menuClickHandler"></page-header>
    <mu-load-more @refresh="refresh" :refreshing="refreshing" :loading="loading" @load="loadMore">
      <!-- <cube-scroll :data="list" :options="scrollOptions" @pulling-down="refresh" @pulling-up="loadMore"> -->
        <list-item v-for="item in list" :key="item.tid" :item="item" @click.native="postClickHandler(item)"></list-item>
      <!-- </cube-scroll> -->
    </mu-load-more>
    <transition-router-view></transition-router-view>
  </page-frame>
</template>

<script>
import PageFrame from '@/components/PageFrame.vue';
import PageHeader from '@/components/PageHeader.vue';
import ListItem from './ListItem.vue';
import { mapGetters, mapMutations } from 'vuex';

export default {
  data() {
    return {
      menuList: [
        { name: '提醒' },
        { name: '消息' },
        { name: '历史浏览' },
        { name: '搜索' },
        { name: '字号' },
      ],
      refreshing: false,
      loading: false,
      list: [],
      pageTitle: '',
      currentPage: 1,
    };
  },
  components: {
    PageFrame,
    PageHeader,
    ListItem,
  },
  computed: {
    ...mapGetters(['currentFid']),
    scrollOptions() {
      const { pullDownRefresh, pullUpLoad } = this;
      return {
        pullDownRefresh,
        pullUpLoad,
        scrollbar: {
          fade: true,
          interactive: false, // 1.8.0 新增
        },
      };
    },
    pullDownRefresh() {
      return {
        threshold: 50,
        stop: 50,
        stopTime: 500,
        txt: '刷新成功',
      };
    },
    pullUpLoad() {
      return {
        threshold: 0,
        txt: {
          more: '加载成功',
          noMore: '已经到底啦',
        },
      };
    },
  },
  watch: {
    $route() {
      this.refresh();
    },
    currentFid(newVal) {
      if (newVal) {
        this.refresh();
      }
    },
  },
  created() {
    this.updateFid(this.$route.params.fid);
    // this.refresh();
  },
  mounted() {
  },
  methods: {
    ...mapMutations('current', ['updateFid', 'updateTid']),
    getList() {
      const {
        currentFid,
        $http,
        $urls,
        currentPage,
      } = this;
      const params = {
        // sign: 'bd2f80c1cbbe85f7ba6f3beac0cbae13',
        // t: new Date().getTime(),
        // 进入页面的时候已经把params.fid赋值给currentFid
        fid: currentFid,
        // app_id: 1010,
        page: currentPage,
        // stid: 0,
      };
      return new Promise((resolve, reject) => {
        $http.post($urls.getList, params)
          .then(data => {
            this.pageTitle = data.forumname;
            if (currentPage === 1) {
              this.list = data.result.data;
            } else {
              // 要进行帖子去重
              const totalList = this.list.concat(data.result.data);
              const tidArr = this.list.map(item => item.tid).concat(data.result.data.map(item => item.tid));
              const tidSet = new Set(tidArr);
              const filteredList = [...tidSet].map(tid => totalList.find(item => item.tid === tid));
              this.list = filteredList;
            }
            if (data.result.data.length) ++this.currentPage;
            resolve();
          })
          .catch(err => {
            console.log('err: ', err);
            reject(err);
          });
      });
    },
    async refresh() {
      this.refreshing = true;
      this.currentPage = 1;
      await this.getList();
      this.refreshing = false;
    },
    async loadMore() {
      this.loading = true;
      await this.getList();
      this.loading = false;
    },
    postClickHandler({ tid, is_forum: isForum }) {
      console.log('itemClick: ', tid, isForum);
      // 判断是否是子版面
      if (isForum) {
        // 子版面使用原接口会被重定向
        this.$router.push({
          path: `/blocks/${tid}`,
        });
        return;
      }
      this.$router.push({
        path: `/post/${tid}`,
      });
    },
    menuClickHandler(menu) {
      console.log('menuClick: ', menu);
    },
  },
};
</script>

<style lang="scss">
.post-list {
  background-color: $color-primary;
  .mu-load-more {
    min-height: 100%;
  }
}
</style>
