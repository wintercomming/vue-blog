<template>
  <div class="list">
    <div class="list__top-title">
      <i class="fa fa-tags" aria-hidden="true"></i>&nbsp;标签
    </div>
    <ul class="list__tag">
      <el-tag
        closable
        v-for="(tag, index) in tagList"
        :key="tag.id"
        @click="toggleSelectFn(tag.id)"
        @close="deleteTagFn(tag.id)"
        :hit="true"
        class="list__newTag"
        :type="randomType(index)"
      >
        {{ tag.name }}
      </el-tag>
    </ul>
    <ul class="list__article">
      <li @click="createArticle" class="list__article__button"><i class="fa fa-plus" aria-hidden="true"></i>&nbsp;新建文章</li>
      <li v-for="(article, index) in articleList" :key="article.id" @click="switchArticle(index)" class="list__article__item" :class="{'list__article__item--active': currentArticle.index == index}">
        <h1 class="list__article__item__title"  :class="{'list__article__item__title--active': currentArticle.index == index}">{{ article.title | cutTitle}}</h1>
        <p class="list__article__item__publish" v-if="article.publish">已发布</p>
        <!-- <div class="list__article__item__info">
          <i class="fa fa-tag" aria-hidden="true"></i>
          <span v-for="tag in article.tags" :key="tag.id"> {{ tag.name }}</span>
          <p class="list__article__item__createTime"><i class="fa fa-calendar" aria-hidden="true"></i>&nbsp; {{article.lastEditTime}}</p>
        </div> -->
      </li>
      <pagination :curPage='curPage' :allPage='allPage' @changePage='changePage'></pagination>
    </ul>
  </div>
</template>


<script>
import Pagination from 'publicComponents/Pagination.vue';
import {
  mapGetters,
  mapActions,
  mapMutations,
} from 'vuex';
export default {
  name: 'list',
  computed: {
    ...mapGetters([
      'articleList',
      'tagList',
      'currentArticle',
      'allPage',
      'curPage',
      'selectTagArr',
    ]),
  },
  components: {
    Pagination,
  },
  data() {
    return {};
  },
  filters: {
    cutTitle(value) {
      if (value.length > 24) {
        return value.substring(0, 24) + '...';
      } else {
        return value;
      }
    },
  },
  methods: {
    ...mapActions([
      'getAllArticles',
      'getAllTags',
      'getCurrentArticle',
      'deleteTag',
    ]),
    ...mapMutations({
      setAllPage: 'SET_ALL_PAGE',
      setCurPage: 'SET_CUR_PAGE',
      toggleSelect: 'TOGGLE_SELECT_TAG',
    }),
    randomType(index) {
      const types = ['', 'success', 'info', 'warning', 'danger'];
      return types[index % 5];
    },
    toggleSelectFn(id) {
      this.toggleSelect(id);
    },
    switchArticle(index) {
      if (!this.currentArticle.save) {
        this.$message.error('请先保存当前文章');
        return;
      }
      this.getCurrentArticle(index);
    },
    createArticle() {
      this.getCurrentArticle(-1);
    },
    deleteArticle() {
      this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        this.$store.dispatch('deleteArticle', {
          id: this.currentArticle._id,
          index: this.currentArticle.index,
        }).then((res) => {
          if (res.data.success) {
            this.$message({
              message: '删除成功',
              type: 'success',
            });
          }
        }).catch((err) => {
          console.log(err);
          this.$message.error(err.response.data.error);
        });
      }).catch(() => {
        // this.$message({
        //   type: 'info',
        //   message: '已取消删除'
        // });
      });
    },
    deleteTagFn(id) {
      this.$confirm('此操作将永久删除该标签, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        this.deleteTag({
          id,
        }).then((res) => {
          if (res.data.success) {
            this.$message({
              message: '删除成功',
              type: 'success',
            });
            // this.getAllArticles({
            //   tag: this.selectTagArr
            // })
          }
        }).catch((err) => {
          console.log(err);
          this.$message.error(err.response.data.error);
        });
      }).catch(() => {
        // this.$message({
        //   type: 'info',
        //   message: '已取消删除'
        // });
      });
    },
    changePage(cur) {
      this.getAllArticles({
        page: cur,
        tag: this.selectTagArr,
      }).then(res => {
        this.getCurrentArticle(0);
      });
    },
  },
  mounted() {
    this.getAllArticles({ limit: 10 }).then(res => {
      console.log('allPage:', this.allPage);
      console.log('curPage:', this.curPage);
    });
    this.getAllTags();
  },
  watch: {
    selectTagArr(val, oldVal) {
      console.log(val);
      console.log('change selectTagArr');
      this.getAllArticles({
        tag: val,
      });
    },
  },
};
</script>


<style lang="stylus" scoped>
@import '../assets/stylus/_settings.styl'
.list
  padding 15px
  &__top-title
    width 100%
    font-size 18px
    padding 10px
    color $blue
    span
      padding-left 15px
  &__newTag
    margin 2px 2px
  &__tag
    height 140px
    overflow auto
    display flex
    flex-direction row
    flex-wrap wrap
    list-style none
    align-content flex-start
  &__tag__item
    //flex-grow 1
    flex-shrink 1
    background-color $blue
    color white
    border-radius 5px
    text-align center
    margin 5px
    padding 7px
    cursor pointer
    height 36px
  &__tag__item--active
    background-color $orange
  &__article
    margin-top 5px
    list-style none
  &__article__item__title
    font-size 16px
  &__article__item__title--active
    color red
  &__article__button
    padding 10px
    font-size 18px
    color $blue
    cursor pointer
  &__article__item
    position relative
    width 100%
    height 30px
    // background-color $grey-bg
    // padding 15px
    margin-bottom 5px
    cursor pointer
  &__article__item--active
    // border-left 10px solid $blue
    color: $red
  &__article__item__info
    position absolute
    bottom 5px
    right 15px
    text-align right
  &__article__item__abstract
    width 100%
    max-height 50px
    word-wrap: break-word;
    word-break all
  &__article__item__publish
    position absolute
    right 0px
    top 0px
    font-size 13px
    color $grey-publish
</style>
