<template>
  <div class="ratings" ref="ratings">
    <div class="ratings-content">
      <div class="overview">
        <div class="overview-left">
          <h1 class="score">{{ seller.score }}</h1>
          <div class="title">综合评分</div>
          <div class="rank">高于周边商家{{ seller.rankRate }}%</div>
        </div>
        <div class="overview-right">
          <div class="score-wrapper">
            <span class="title">服务态度</span>
            <v-star :size="36" :score="seller.serviceScore"></v-star>
            <span class="score">{{ seller.serviceScore }}</span>
          </div>
          <div class="score-wrapper">
            <span class="title">商品评分</span>
            <v-star :size="36" :score="seller.foodScore"></v-star>
            <span class="score">{{ seller.foodScore }}</span>
          </div>
          <div class="delivery-wrapper">
            <span class="title">送达时间</span>
            <span class="delivery">{{ seller.deliveryTime }}分钟</span>
          </div>
        </div>
      </div>
      <v-split></v-split>
      <v-ratingselect :select-type="selectType" :only-content="onlyContent" :desc="desc" :ratings="ratings"></v-ratingselect>
      <v-split></v-split>
      <div class="rating-wrapper">
        <ul>
          <li v-show="needShow(rating.rateType, rating.text)" v-for="rating in ratings" class="rating-item">
            <div class="avatar">
              <img width="28" height="28" :src="rating.avatar">
            </div>
            <div class="content">
              <h1 class="name">{{ rating.username }}</h1>
              <div class="star-wrapper">
                <v-star :size="24" :score="rating.score"></v-star>
                <span class="delivery" v-show="rating.deliveryTime">{{ rating.deliveryTime }}分钟送达</span>
              </div>
              <p class="text">{{ rating.text }}</p>
              <div class="recommend" v-show="rating.recommend || rating.recommend.length">
                <span class="icon-thumb_up"></span>
                <span class="item" v-for="item in rating.recommend">{{ item }}</span>
              </div>
              <div class="time">
                {{ rating.rateTime | formatDate }}
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script text="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import EventHub from '../../common/js/eventHub.js';
  import star from 'components/star/star';
  import ratingselect from 'components/ratingselect/ratingselect';
  import split from 'components/split/split';
  // export function formatDate（time， formatString）
  import {formatDate} from '../../common/js/date.js';

  /* eslint-disable no-unused-vars */
  const POSITIVE = 0;
  const NEGATIVE = 1;
  const ALL = 2;

  const ERR_OK = 0;

  export default {
    data () {
      return {
        // 商品详情初始化
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',      // 2
          positive: '推荐', // 0
          negative: '吐槽'  // 1
        },
        ratings: []
      };
    },
    methods: {
      // 过滤评论信息
      needShow (type, text) {
        if (this.onlyContent && !text) {
          return false;
        }
        if (this.selectType === ALL) {
          return true;
        } else {
          return type === this.selectType;
        }
      }
    },
    props: {
      seller: {
        type: Object
      }
    },
    components: {
      'v-star': star,
      'v-ratingselect': ratingselect,
      'v-split': split
    },
    created() {
      this.$http.get('/api/ratings').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          // 获取数据
          this.ratings = response.data;
          // 刷新DOM
          this.$nextTick(() => {
            // 设置滚动
            this.scroll = new BScroll(this.$refs.ratings, {
              click: true
            });
          });
        }
      });
    },
    filters: {
      formatDate(time) {
        let date = new Date(time);
        return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },
    updated() {
      // 获取子组件的selectType的更新
      EventHub.$on('ratingtype.select', selectType => {
        this.selectType = selectType;
        // 更改评价显示设置后更新DOM然后重新计算滚动
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      });
      EventHub.$on('content.toggle', onlyContent => {
        this.onlyContent = onlyContent;
        // 更改内容显示设置后更新DOM然后重新计算滚动
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      });
      // console.log(this.selectType + ' ' + this.onlyContent);
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import '../../common/stylus/mixin.styl'

  .ratings
    position: absolute
    top: 174px
    bottom: 0
    left: 0
    width: 100%
    height: 100hv // 不加也是一样的
    overflow: hidden
    .overview
      display:flex
      padding: 18px 0
      .overview-left
        padding: 6px 0
        flex: 0 0 137px
        width: 137px
        border-right: 1px solid rgba(7, 17, 27, .1)
        text-align: center
        // 考虑iphone5 屏幕宽度只有320 需用media query重设样式
        @media only screen and (max-width: 320px)
          flex: 0 0 120px
          width: 120px
        .score
          margin-bottom: 6px
          line-height:28px
          font-size: 24px
          color: rgb(255, 153, 0)
        .title
          margin-bottom: 8px
          line-height: 12px
          font-size: 12px
          color: rgb(7, 17, 27)
        .rank
          line-height: 10px
          font-size: 10px
          color: rgb(147, 153, 159)
      .overview-right
        flex: 1
        padding:6px 0 6px 24px
        @media only screen and (max-width: 320px)
          padding-left: 6px
        .score-wrapper
          margin-bottom: 8px
          font-size: 0
          .title
            display: inline-block
            vertical-align: top
            line-height: 18px
            font-size: 12px
            color: rgb(7, 17, 27)
          .star
            display: inline-block
            vertical-align: top
            margin: 0 12px
          .score
            display: inline-block
            vertical-align: top
            line-height: 18px
            font-size: 12px
            color: rgb(255, 153, 0)
        .delivery-wrapper
          font-size: 0
          .title
            line-height: 18px
            font-size: 12px
            color: rgb(7, 17, 27)
          .delivery
            font-size: 12px
            color: rgb(147, 153, 159)
            margin-left: 12px
    .rating-wrapper
      padding: 0 18px
      .rating-item
        display: flex
        padding: 18px 0
        border-1px(rgba(7, 17, 27, .1))
        .avatar
          flex: 0 0 28px
          width: 28px
          margin-right: 12px
          img
            border-radius: 50%
        .content
          position:relative
          flex: 1
          .name
            marrgin-bottom: 4px
            line-height: 12px
            font-size: 10px
            color: rgb(7, 17, 27)
          .star-wrapper
            margin-bottom: 6px
            font-size: 0
            .star
              display: inline-block
              margin-right: 6px
              vertical-align: top
            .delivery
              display: inline-block
              vertical-align: top
              line-height: 12px
              font-size: 10px
              color: rgb(147, 153, 159)
          .text
            margin-bottom: 8px
            line-height: 18px
            color: rgb(7, 17, 27)
            font-size: 12px
          .recommend
            line-height: 16px
            font-size: 0px
            .icon-thumb_up, .item
              display: inline-block
              margin: 0 8px 4px 0
              font-size: 9px
            .icon-thumb_up
              color: rgb(0, 160, 220)
            .item
              padding: 0 6px
              line-height: 16px
              border: 1px solid rgba(7, 17, 27, .1)
              border-radius: 1px
              color: rgb(147,153,159)
              background: #fff
            .time
              position: absolute
              top: 0
              right: 0
              line-height: 12px
              color: rgb(7, 17, 27)
</style>
