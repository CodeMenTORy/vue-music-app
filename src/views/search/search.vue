<template>
<div class="search">
  <div class="search-box">
    <search-box ref="searchBox" @query="onQueryChange"></search-box>
  </div>
  <div class="shortcut-wrapper" v-show="!query" ref="shortcutWrapper">
    <scroll :refreshDelay="refreshDelay" class="shortcut" ref="shortcut" :data="shortcut">
      <div>
        <div class="hot-key">
          <h1 class="title">热门搜索</h1>
          <ul>
            <li class="item" v-for="(item, index) in hotKey" :key="index" @click="addQuery(item.k)">
              <span>{{ item.k }}</span>
            </li>
          </ul>
        </div>
        <div class="search-history" v-show="searchHistory.length">
          <h1 class="title">
            <span class="text">搜索历史</span>
            <span class="clear" @click="showConfirm">
              <i class="icon-clear"></i>
            </span>
          </h1>
          <search-list :searches="searchHistory" @select="addQuery" @delete="deleteOne"></search-list>
        </div>
      </div>
    </scroll>
  </div>
  <div class="search-result" v-show="query" ref="searchResult">
    <suggest :query="query" @listScroll="blurInput" @select="saveSearch" ref="suggest"></suggest>
  </div>
  <confirm text="是否清空所有搜索历史" confirmBtnText="清空" ref="confirm" @confirm="clearSearchHistory"></confirm>
</div>
</template>

<script>
import {
  getHotKey
} from 'api/search';
import {
  mapActions
} from 'vuex';
import {
  playlistMixin,
  searchMixin
} from 'common/js/mixin';
import Scroll from 'components/common/scroll/scroll';

import SearchBox from 'components/content/search-box/searchBox';
import Suggest from 'components/content/suggest/suggest';
import SearchList from 'components/content/search-list/searchList';
import Confirm from 'components/common/confirm/confirm';

export default {
  name: 'search',
  mixins: [playlistMixin, searchMixin],
  components: {
    SearchBox,
    Suggest,
    Scroll,
    SearchList,
    Confirm,
  },
  data() {
    return {
      hotKey: [],
      query: '',
    };
  },
  computed: {
    shortcut() {
      return this.hotKey.concat(this.searchHistory);
    },
  },
  created() {
    this._getHotKey();
  },
  methods: {
    // 获取热频词
    _getHotKey() {
      getHotKey().then((res) => {
        this.hotKey = res.data.hotkey.slice(0, 10);
      });
    },
    // 删除搜索历史
    deleteOne(item) {
      this.deleteSearchHistory(item);
    },
    // 展示确认框
    showConfirm() {
      this.$refs.confirm.show();
    },
    handlePlaylist(playlist) {
      const bottom = playlist.length > 0 ? '60px' : '';
      this.$refs.shortcutWrapper.style.bottom = bottom;
      this.$refs.shortcut.refresh();
      this.$refs.searchResult.style.bottom = bottom;
      this.$refs.suggest.refresh();
    },
    ...mapActions(['clearSearchHistory']),
  },
  watch: {
    // 当增加搜索历史，刷新滚动
    query(newQuery) {
      if (!newQuery) {
        setTimeout(() => {
          this.$refs.shortcut.refresh();
        }, 20);
      }
    },
  },
};
</script>

<style lang="stylus">
@import '~common/stylus/variable';
@import '~common/stylus/mixin';

.search {
  .search-box-wrapper {
    margin: 20px;
  }

  .shortcut-wrapper {
    position: fixed;
    top: 178px;
    bottom: 0;
    width: 100%;

    .shortcut {
      height: 100%;
      overflow: hidden;

      .hot-key {
        margin: 0 20px 20px 20px;

        .title {
          margin-bottom: 20px;
          font-size: $font-size-medium;
          color: $color-text-l;
        }

        .item {
          display: inline-block;
          padding: 5px 10px;
          margin: 0 20px 10px 0;
          border-radius: 6px;
          background: $color-highlight-background;
          font-size: $font-size-medium;
          color: $color-text-d;
        }
      }

      .search-history {
        position: relative;
        margin: 0 20px;

        .title {
          display: flex;
          align-items: center;
          height: 40px;
          font-size: $font-size-medium;
          color: $color-text-l;

          .text {
            flex: 1;
          }

          .clear {
            extend-click();

            .icon-clear {
              font-size: $font-size-medium;
              color: $color-text-d;
            }
          }
        }
      }
    }
  }

  .search-result {
    position: fixed;
    width: 100%;
    top: 178px;
    bottom: 0;
  }
}
</style>
