<template>
  <div class="dynamic">
    <div class="title">
      <span class="act">动态</span>
    </div>
    <div class="lists">
      <van-list
          v-model="loadingMore"
          :finished="finished"
          :loading-text="$t('public.loading')"
          :finished-text="$t('public.noMore')"
          @load="handleCurrentChange"
        >
        <div class="list flexs" v-for="(item, index) in lists" :key="index">
          <img class="headImg" :src="accInfo.avatar || item.headImg" :onerror="errorCoinImg">
          <div class="mainData">
            <div class="name">{{accInfo.nick || item.fromx}}</div>
            <div class="num tip dinReg">捐赠数量: {{ item.quantity }}</div>
            <div class="content">{{ item.memo }}</div>
            <div class="time tip">{{handleToLocalTime(item.dealTime)}}</div>
          </div>
        </div>
      </van-list>
    </div>
  </div>
</template>

<script>
import Bus from '@/utils/bus';
import moment from 'moment';

import {getDateDiff, getCoin, toLocalTime} from '@/utils/public'
import {get_acc_fund_lists} from '@/utils/api'
export default {
  name: 'dynamic',
  data() {
    return {
      errorCoinImg: 'this.src="https://cdn.jsdelivr.net/gh/defis-net/material/icon/pig.png"',
      id: '',
      loadingMore: false,
      finished: false,
      page: 1,
      pagesize: 20,
      lists: [],
      accInfo: {
        avatar: "https://cdn.jsdelivr.net/gh/defis-net/material/coin/tagtokenmain-tag.png",
        cover: "https://cdn.jsdelivr.net/gh/defis-net/material/accBanner/banner0.png",
        desc: "",
        nick: "",
        sex: 2,
      },
    }
  },
  mounted() {
    Bus.$on('busForAccInfo', (val) => {
      this.accInfo = val;
      console.log(this.accInfo)
    });
    this.id = this.$route.params.id;
  },
  methods: {
    handleCurrentChange() {
      this.handleGetLists()
    },
    handleToLocalTime(time) {
      let t = moment(`${time}`).valueOf()
      const oDate = getDateDiff(t)
      return oDate
    },
    async handleGetLists() {
      const params = {
        page: this.page,
        limit: this.pagesize,
        user: this.id
      }
      const {status, result} = await get_acc_fund_lists(params)
      if (!status) {
        return
      }
      const rows = result.data || [];
      rows.forEach(v => {
        this.$set(v, 'headImg', getCoin(v.account, v.symbol))
        const replyNum = (v.reply_count || 0)
        this.$set(v, 'replyNum', replyNum)
        const t = toLocalTime(v.create_time).replace(/-/g, '/');
        const times = Date.parse(t) + 3600 * 8 * 1000;
        this.$set(v, 'dealTime', toLocalTime(times))
        // const likeNum = v.like_count * 1000;
        // this.$set(v, 'likeNum', likeNum.toFixed(0))
      })
      if (this.page === 1) {
        this.lists = rows;
      } else {
        this.lists.push(...rows)
      }
      this.page += 1;
      this.loadingMore = false;
      if (this.lists.length >= result.total || !rows.length) {
        this.finished = true;
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.dynamic{
  font-size: 28px;
  text-align: left;
  background: #FFF;
  position: relative;
  margin-top: -30px;
  // border-radius: 30px;
  overflow: hidden;
  padding: 28px;
  .title{
    font-size: 32px;
    text-align: left;
    margin-bottom: 10px;
    &>span{
      padding-left: 20px;
    }
    .act{
      color: $color-black;
      position: relative;
      &::before{
        content: '';
        position: absolute;
        width: 8px;
        height: 30px;
        background:#02C698;
        border-radius:4px;
        left: 0%;
        top: 50%;
        transform: translateY(-45%);
      }
    }
    .right{
      margin-right: 0
    }
  }
  .list{
    padding: 20px 0px;
    border-bottom: 1px solid rgba(220, 220, 220, .3);
    .headImg{
      width: 80px;
      height: 80px;
      min-width: 80px;
      border-radius: 40px;
      overflow: hidden;
      margin-right: 15px;
    }
    .name{
      font-size: 30px;
      font-weight: 500;
    }
    .num{
      text-align: left;
      font-size: 24px;
      margin-top: 6px;
      font-weight: normal;
    }
    .content{
      margin: 8px 0;
      overflow: hidden;
      word-break: break-all;
      white-space: pre-wrap;
    }
    .time{
      margin-top: 4px;
      font-size: 20px;
    }
  }
}
</style>
