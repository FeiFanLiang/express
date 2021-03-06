<template>
  <div class="map-wrap">
    <div class="map-header">
      <div
        v-if="isMyposts"
        class="left-icon"
        @click="$router.push({ name: 'person' })"
      >
        <van-icon name="arrow-left" />
      </div>
      <div class="input-wrap">
        <input v-model="title" placeholder="Search title" />
      </div>
      <div class="action">
        <van-icon name="search" @click="onSearch" style="margin-right:10px" />
        <van-icon
          v-if="!isMyposts"
          name="plus"
          @click="$router.push({ name: 'addActive' })"
        />
      </div>
    </div>
    <div v-if="isMyposts" class="map-header map-header-tab">
      <van-tabs v-model="tabActive" style="width:100%" @change="onTabChange">
        <van-tab title="My Release"></van-tab>
        <van-tab title="My Joins"></van-tab>
      </van-tabs>
    </div>
    <div class="map-content">
      <div ref="list" style="overflow-y: auto;height:100%;">
        <van-pull-refresh
          v-model="isLoading"
          @refresh="onRefresh"
          style="position: relative;"
          pulling-text="Pull down to refresh..."
          loosing-text="Release to refresh..."
          loading-text="loading..."
        >
          <van-list
            v-model="loading"
            error-text="Request failed, click reload"
            :finished-text="data.length === 0 ? '' : 'No more'"
            loading-text="loading..."
            :error.sync="error"
            :finished="finished"
            @load="onLoad"
          >
            <Item
              @click.native="
                $router.push({
                  name: 'activeInfo',
                  query: { _id: item._id, back: $route.name }
                })
              "
              @update="onSearch"
              v-for="item in data"
              :key="item._id"
              :item="item"
              :isMyposts="isMyposts"
            />
          </van-list>
        </van-pull-refresh>
      </div>
    </div>
  </div>
</template>

<script>
import Item from './Item'
export default {
  components: { Item },
  data() {
    return {
      data: [],
      loading: false,
      finished: false,
      pageNo: 0,
      isAjax: false,
      error: false,
      isLoading: false,
      title: '',
      tabActive: 0
    }
  },
  computed: {
    isMyposts() {
      return this.$route.name === 'myactive'
    },
    user() {
      return this.$store.getters.user
    }
  },
  methods: {
    onLoad() {
      if (this.isAjax) {
        return
      }
      this.isAjax = true
      this.pageNo++
      this.list(false, false)
    },
    async onRefresh() {
      this.isAjax = true
      try {
        this.pageNo = 1
        this.finished = false
        await this.list(true)
      } finally {
        setTimeout(() => {
          this.isLoading = false
          this.isAjax = false
        }, 300)
      }
    },
    async onSearch() {
      if (this.isAjax) {
        return
      }
      this.isAjax = true
      this.pageNo = 1
      this.finished = false
      try {
        await this.list(true)
      } finally {
        this.isAjax = false
      }
    },
    onTabChange() {
      this.onSearch()
    },
    async list(flag, isToast = true) {
      try {
        isToast &&
          this.$toast.loading({
            duration: 0
          })
        const { data } = await this.$http({
          url:
            this.tabActive === 0
              ? this.$api.active_list
              : this.$api.active_me_list,
          params: {
            pageNo: this.pageNo,
            title: this.title,
            isUser: this.isMyposts ? true : undefined,
            promoter: this.isMyposts ? this.user._id : undefined
          }
        })
        let list = data.rows || []
        if (flag) {
          this.data = list
          this.$nextTick(() => {
            this.$refs.list.scrollTop = 0
          })
        } else {
          this.data = this.data.concat(list)
        }
        this.pageNo = data.pageNo
        if (list.length !== data.size) {
          this.finished = true
        }
      } catch (e) {
        this.error = true
        this.pageNo--
      } finally {
        isToast && this.$toast.clear(true)
        this.isLoad = true
        this.$nextTick(() => {
          this.isAjax = false
          this.loading = false
        })
      }
    }
  }
}
</script>

<style lang="less" scoped>
@import '../home/header.less';

.map-wrap {
  background-color: #f0f3f6;
  .map-content {
  }
  .map-header {
    padding: 10px 15px;
    .left-icon {
      display: flex;
      align-items: center;
    }
    .input-wrap {
      height: 34px;
    }
    .action {
      display: flex;
      align-items: center;
    }
  }
}
.map-wrap .map-header {
  box-shadow: none;
}
.map-wrap .map-header-tab {
  border-bottom: 1px solid rgba(0, 0, 0, 0.15);
  padding-bottom: 0;
  /deep/ .van-tabs__wrap {
    &::after {
      display: none;
    }
  }
}
</style>
