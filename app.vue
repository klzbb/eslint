<!--
 * @Descripttion: 牛人汇首页
 * @version: 1.0.0
 * @Author:
 * @Date: 2019-07-15 18:29:07
 * @LastEditors: konglingzhan
 * @LastEditTime: 2020-08-10 10:44:27
 -->
<template>
  <div class="home">
    <div class="pr">
      <van-swipe
        class="home_banner"
        :autoplay="3000"
        indicator-color="white"
        :stop-propagation="false"
      >
        <van-swipe-item
          v-for="(item,index) in bannerList"
          :key="index"
          @click="bannerClick(item,index)"
        >
          <img
            style="width: 100%;display: block;"
            :src="item.file_type+item.image_data"
            alt
          >
        </van-swipe-item>
      </van-swipe>

      <div class="home_tabs flex">
        <div
          v-for="(item,index) in tabs"
          :key="index"
          class="home_tabs_item flex"
          @click="tabsClick(item)"
        >
          <div>
            <img
              :src="item.icon"
              class="home_tabs_item_icon"
              alt=""
            >
          </div>
          <div class="home_tabs_item_text">
            <div class="title pr">
              <span>{{ item.desc_name }}</span>
              <img
                v-if="item.isHot"
                src="static/images/icon_hot.png"
                class="pa icon_hot"
                alt=""
              >
            </div>
            <div class="desc">
              {{ item.list_detail }}
            </div>
          </div>
        </div>
      </div>
      <div class="home_message">
        <div class="home_message_left">
          <img
            class="home_message_left_img"
            src="static/images/nrkb.png"
            alt
          >
        </div>
        <van-swipe
          class="home_message_swipe"
          :show-indicators="false"
          :autoplay="3000"
          vertical
        >
          <van-swipe-item
            v-for="(item,index) in messageList"
            :key="index"
          >
            {{ item.message_text }}
          </van-swipe-item>
          <van-swipe-item v-if="messageList.length === 0">
            暂无牛人快报消息
          </van-swipe-item>
        </van-swipe>
      </div>

      <div
        v-if="isShowDsgs"
        class="home_dsgs"
      >
        <panel-header
          title="大赛高手"

          :need-right-btn="dsgsList.length === 3"
          @rightClick="dsgsMore(dsgsActivityNo)"
        />
        <template v-for="(item,index) in dsgsList">
          <row-item-a
            :key="index"
            :item="item"
            :field-eng-name="profit_all"
            @itemClick="profitDetail"
            @subscri="subs"
          />
        </template>
        <empty
          v-if="dsgsList.length === 0"
          :need-define-text="true"
        >
          <div class="home_rank_empty">
            暂无数据
          </div>
        </empty>
      </div>
      <div
        v-if="!competitionIsEmpty"
        class="home_competition"
      >
        <panel-header
          title="累计收益高手"
          :need-right-btn="competitionList.length === 3"
          @rightClick="rank('profit_all')"
        />
        <template v-for="(item,index) in competitionList">
          <row-item-a
            :key="index"
            :item="item"
            :field-eng-name="profit_all"
            @itemClick="profitDetail"
            @subscri="totalSubscri"
          />
        </template>
        <empty
          v-if="competitionList.length === 0"
          :need-define-text="true"
        >
          <div class="home_rank_empty">
            暂无数据
          </div>
        </empty>
      </div>
      <div class="home_rank">
        <panel-header
          title="月收益高手"
          :need-right-btn="rankList.length === 3"
          @rightClick="rank('profit_near_month')"
        />
        <template v-for="(item,index) in rankList">
          <row-item-a
            :key="index"
            :item="item"
            :field-eng-name="profit_near_month"
            @itemClick="profitDetail"
            @subscri="monthSubscri"
          />
        </template>
        <empty
          v-if="rankIsEmpty || rankList.length === 0"
          :need-define-text="true"
        >
          <div class="home_rank_empty">
            暂无数据
          </div>
        </empty>
      </div>
      <div class="home_like">
        <panel-header
          title="猜你喜欢"
          :need-right-btn="false"
        />
        <like-list
          :list="likeList"
          @subscri="likeSubscri"
          @itemClick="profitDetail"
        />
      </div>
      <Dialog
        :is-show="$store.getters.dialogInfo.isDialogShow"
        title="提示"
        content="更多功能即将上线，敬请期待"
        text-position="center"
      />
    </div>
  </div>
</template>
<script>
import PanelHeader from "@/components/panel-header/index.vue";
import Empty from "@/components/empty/index.vue";
import Dialog from "@/components/base-popup/index.vue";
import {
  getRecommendInfoHighLinesList,
  getAllBoardList,
  getBoardRule,
  getAdvertisementOnLine,
  getNrhBulletin,
  F900008,
  getActivityBoardList,
  getActivityBoardResult,
  getActivityIndexVoByActivityNo
} from "@/service/nrh";
import RowItemA from '@/components/row-item/row-item-a.vue'
import LikeList from '@/components/like-list/index.vue'
import { list_mixins } from '@/mixins'
import { p001 } from "@/utils/publicUtil.js"
export default {
  name: "Home",
  components: {
    RowItemA,
    LikeList,
    PanelHeader,
    Empty,
    Dialog
  },
  filters: {
    vJoinNumber: function (val) {
      // 新手练习场
      if (val) {
        return val + "人";
      } else {
        return "";
      }
    }
  },
  mixins: [list_mixins],
  data () {
    return {
      isShowDsgs: false,
      dsgsActivityNo: '',
      indexMonthRuleNo: '',
      indexAllRuleNo: '',
      likeList: [],
      profit_near_month: '',
      profit_all: '',
      tabs: [],
      isPracticeEmpty: false,
      date: "",
      isCompitionClick: false,
      hotTitle: "市价/买入量",
      show: true,
      active: 0,
      rankIsEmpty: false,
      competitionIsEmpty: true,
      hotIsEmpty: false,
      bannerList: [],
      hotList: [],
      competitionList: [],
      rankList: [],
      dsgsList: [],
      messageList: [],
      practiceInfo: {},
      isLoading: false,
      isFinishLoad: false,
      maxPage: null,
      list: [],
      pageSize: 10,
      field_eng_name: "profit_all", // 默认累计收益率
      isEmpty: false,
      isSwipe: true
    };
  },
  created () {
    // ios qq 打开分享页面时，接口异常报错
    const hrefQueryObj = p001();
    const sys = this.$store.getters.getLocalType;
    if ("share" in hrefQueryObj) {
      // 非掌中投环境(掌中投分享到ios qq 打开时接口异常)
      if (sys.type === "ios" && sys.browser.isQQ) {
        return false;
      } else {
        this.init();
      }
    } else {
      this.init();
    }
  },
  methods: {
    // 大赛高手订阅
    subs (item, type) {
      this.G004(item, this, (data) => {
        // attention_status 0-订阅 1-已订阅
        const { nrh_account, attention_status } = data;
        let index = this.dsgsList.findIndex(item => item.nrh_account === nrh_account);
        if (index !== -1) {
          this.dsgsList[index].attention_status = attention_status === '0' ? '1' : '0';
        }
      })
    },
    // 大赛高手
    async getDsgsList () {
      let resultA = await F900008({ configNo: "146" });
      if (resultA && resultA.data.code === 0) {
        const { int_config } = resultA.data.data; // 活动编号（后台系统参数配置）
        this.dsgsActivityNo = int_config;
        let result = await getActivityIndexVoByActivityNo({ activity_no: int_config });
        if (result && result.data.code === 0) {
          const { status } = result.data.data;
          if (status === '1') {
            // 活动进行中
            this.isShowDsgs = true;
            let resultB = await getActivityBoardList({ activity_no: int_config })
            if (resultB && resultB.data.code === 0) {
              let { index_no } = resultB.data.data.filter(item => item.field_eng_name === 'profit_all')[0];
              let param = {
                page_num: 1,
                activity_no: int_config,
                index_no
              }
              let resultC = await getActivityBoardResult(param);
              if (resultC && resultC.data.code === 0) {
                const { boardResultList } = resultC.data.data;
                if (boardResultList && boardResultList.length > 0) {
                  this.dsgsList = boardResultList.splice(0, 3);
                }
              }
            }
          } else {
            this.isShowDsgs = false;
          }
        }
      }
    },
    dsgsMore (activity_no) {
      this.$router.push({
        name: 'ActivityAdapter',
        query: {
          activity_no
        }
      })
    },
    tabsClick (item) {
      const { rule_no, desc_name } = item;
      this.$onTdEvent('NE01002001_发现页榜单入口', desc_name)
      if (desc_name === '炒股大赛') {
        this.$router.replace({
          name: 'Activity'
        })
      } else if (desc_name === '门派巅峰战') {
        this.$router.push({
          name: 'ActivityAdapter',
          query: {
            activity_no: this.dsgsActivityNo,
            active: 'mp_rank'
          }
        })
      } else {
        this.$router.push({
          name: 'AllRank',
          query: { rule_no }
        })
      }
    },
    // 个人收益详情
    profitDetail (item, type) {
      if (type === '2') {
        this.$onTdEvent('NE04001004_点击用户头像昵称等查看组合详情', '猜你喜欢_用户头像昵称');
      } else {
        this.$onTdEvent('NE04001004_点击用户头像昵称等查看组合详情', '发现页_用户头像昵称');
      }
      const { nrh_account, fund_account } = item;
      const { appLoginStatus } = this.$store.getters;
      const query = this.$Base64Encode({ nrh_account, fund_account });
      if (appLoginStatus === '2') {
        // 已登录资金账号
        const { fund_account: my_fund_account } = this.$store.getters.getUserInfo
        if (my_fund_account === fund_account) {
          this.$router.push({
            name: 'ProfitDetail',
            query: { query: query }
          })
        } else {
          this.$router.push({
            name: 'ProfitDetailOther',
            query: { query: query }
          })
        }
      } else if (appLoginStatus === '0') {
        this.$router.push({
          name: 'ProfitDetailOther',
          query: { query: query }
        })
      }
    },
    // 猜你喜欢订阅
    likeSubscri (item) {
      this.G004(item, this, (data) => {
        // attention_status 0-订阅 1-已订阅
        const { nrh_account, attention_status } = data;
        let index = this.likeList.findIndex(item => item.nrh_account === nrh_account);
        if (index !== -1) {
          this.likeList[index].attention_status = attention_status === '0' ? '1' : '0';
          let label = attention_status === '0' ? '订阅' : '取消订阅';
          this.$onTdEvent('NE01005002_推荐订阅/取消订阅', label)
        }
      })
    },
    // 累计收益高手订阅
    totalSubscri (item) {
      this.G004(item, this, (data) => {
        // attention_status 0-订阅 1-已订阅
        const { nrh_account, attention_status } = data;
        let index = this.competitionList.findIndex(item => item.nrh_account === nrh_account);
        if (index !== -1) {
          this.competitionList[index].attention_status = attention_status === '0' ? '1' : '0';
          let label = attention_status === '0' ? '订阅' : '取消订阅';
          this.$onTdEvent('NE01003003_累计榜单订阅/取消订阅', label)
        }
      })
    },
    // 月收益订阅
    monthSubscri (item) {
      this.G004(item, this, (data) => {
        // attention_status 0-订阅 1-已订阅
        const { nrh_account, attention_status } = data;
        let index = this.rankList.findIndex(item => item.nrh_account === nrh_account);
        if (index !== -1) {
          this.rankList[index].attention_status = attention_status === '0' ? '1' : '0';
          let label = attention_status === '0' ? '订阅' : '取消订阅';
          this.$onTdEvent('NE01004003_月榜单订阅/取消订阅', label)
        }
      })
    },
    async init (isRefresh = false) {
      this.$AppLoading.show();
      Promise.all([
        this.getAdvertisementOnLine(),
        this.getNrhBulletin(),
        this.longLine(),
        this.getRecommendInfoHighLinesList(),
        this.getDsgsList()
      ]).then(
        res => {
          this.$AppLoading.hide();
        }
      ).catch(e => {
        this.$AppLoading.hide();
        if (e.message.indexOf("timeout of") !== -1) {
          window._this = this;
          this.$store.commit("navBar/SET_IS_NET_WORK_ERROR", "1");
        } else {
          this.$toast({
            message: e.message
          })
        }
      })
    },
    // tabs,累计收益高手,月收益高手
    async longLine (item) {
      const res = await getAllBoardList({ scope: 'INDEX' });
      if (res && res.data.code === 0) {
        const { data: list } = res.data;
        // tabs
        this.tabs = [];
        list.forEach(item => {
          if (item.desc_name === '人气高手') {
            item.icon = 'static/images/moods.png';
            this.tabs.push(item);
          } else if (item.desc_name === '周收益高手') {
            this.tabs.push({
              icon: 'static/images/tab_icon_2.png',
              desc_name: '门派巅峰战',
              list_detail: '股林最强门派PK'
            });
          } else if (item.desc_name === '日收益高手') {
            item.icon = 'static/images/win-rate.png';
            this.tabs.push(item);
          }
        })

        let obj = {
          icon: 'static/images/icon_cgds.png',
          desc_name: '炒股大赛',
          list_detail: '最高赢万元大奖',
          isHot: '1'
        }
        this.tabs.push(obj)

        const indexAll = list.findIndex(item => item.desc_name === '累计收益高手');
        const indexMonth = list.findIndex(item => item.desc_name === '月收益高手');
        // 累计收益高手
        if (indexAll !== -1) {
          let rule_no = this.indexAllRuleNo = list[indexAll].rule_no;
          const params = {
            rule_no: rule_no,
            page_num: '1'
          }
          const res = await getBoardRule(params)
          if (res && res.data.code === 0) {
            const { boardResultList, order_by } = res.data.data;
            this.profit_all = order_by;
            if (boardResultList && boardResultList.length > 0) {
              this.competitionIsEmpty = false;
            } else {
              this.competitionIsEmpty = true;
            }
            this.competitionList = boardResultList.splice(0, 3)
          }
        }
        // 月计收益高手
        if (indexMonth !== -1) {
          let rule_no = this.indexMonthRuleNo = list[indexMonth].rule_no;
          const params = {
            rule_no: rule_no,
            page_num: '1'
          }
          const res = await getBoardRule(params)
          if (res && res.data.code === 0) {
            const { boardResultList, order_by } = res.data.data;
            this.profit_near_month = order_by;
            if (boardResultList && boardResultList.length > 0) {
              this.rankIsEmpty = false;
            } else {
              this.rankIsEmpty = true;
            }
            this.rankList = boardResultList.splice(0, 3)
          }
        }
      }
    },
    // 轮播图
    async getAdvertisementOnLine () {
      const res = await getAdvertisementOnLine();
      if (res && res.data.code === 0) {
        this.bannerList = res.data.data;
        return res.data.data;
      }
    },
    // 牛人快报
    async getNrhBulletin () {
      const res = await getNrhBulletin();
      if (res && res.data.code === 0) {
        this.messageList = res.data.data || [];
        return res;
      }
    },
    // 猜你喜欢
    async getRecommendInfoHighLinesList () {
      const res = await getRecommendInfoHighLinesList();
      if (res && res.data.code === 0) {
        this.likeList = res.data.data;
      }
    },
    async refresh () {
      await this.init(true);
      this.isLoading = false;
    },
    // banner click
    bannerClick (item, index) {
      if (this.isSwipe) {
        const { activity_no } = item;
        let params = item.ad_no + '_广告' + (index + 1);
        let param = {};
        param[params] = params;
        this.$onTdEvent('NE01001001_首页广告', '首页广告', param);
        if (item.activity_no === item.ad_no) { // 广告
          if (item.ad_image_path) {
            window.location.href = item.ad_image_path
          }
        } else {
          this.$router.push({
            name: 'ActivityAdapter',
            query: {
              activity_no
            }
          })
        }
      } else {
        this.isSwipe = true;
      }
    },
    // 全量榜单
    rank (label) {
      let rule_no;
      if (label === 'profit_all') {
        this.$onTdEvent('NE01003001_累计榜单查看更多', '累计榜单查看更多');
        rule_no = this.indexAllRuleNo;
      } else if (label === 'profit_near_month') {
        this.$onTdEvent('NE01004001_月榜单查看更多', '月榜单查看更多');
        rule_no = this.indexMonthRuleNo;
      }
      this.$router.push({
        name: 'AllRank',
        query: {
          rule_no: rule_no
        }
      })
    },
    know () {
      this.$store.commit("dialog/SET_DIALOG_SHOW", false);
    }

  }
};
</script>
<style lang="scss">
.home {
  background-color: #f0f0f0;
  &_tabs {
    flex-wrap: wrap;
    align-items: space-between;
    box-sizing: border-box;
    height: 160px;
    padding: 10px 15px;
    background-color: #fff;
    border-bottom: 1px solid #f0f0f0;
    .icon_hot {
      top: -6px;
      right: -2px;
      width: 25px;
      height: 17px;
    }
    &_item {
      align-items: center;
      width: 172.5px;
      &_icon {
        width: 50px;
        height: 50px;
        margin-right: 10px;
      }
      &_text {
        .title {
          margin-bottom: 4px;
          color: #222;
          font-size: 15px;
        }
        .desc {
          color: #888;
          font-size: 12px;
        }
      }
    }
  }
  &_banner {
    height: 138px;
  }
  &_message {
    display: flex;
    flex-direction: row;
    align-items: center;
    box-sizing: border-box;
    height: 50px;
    background-color: #fff;
    .van-swipe-item {
      display: flex;
      align-items: center;
      height: 50px;
      color: #333;
      font-size: 12px;
    }
    &_swipe {
      flex: 1;
      height: 50px;
    }
    &_left {
      position: relative;
      margin-right: 16px;
      margin-left: 15px;
      &_img {
        width: 68px;
        height: 17px;
      }
    }
  }
  &_dsgs {
    margin-top: 10px;
  }
  &_competition {
    margin-top: 10px;
  }
  &_rank {
    margin-top: 10px;
    &_empty {
      color: #888;
      font-size: 14px;
      span {
        color: #3b7be6;
      }
    }
    &_top {
      justify-content: space-between;
      align-items: center;
      height: 44px;
      padding: 0 15px;
      color: #888;
      font-size: 12px;
      background-color: #fbfbfb;
    }
  }
  &_like {
    margin-top: 10px;
  }
  .box-hot {
    background: #fff;
  }
  .game_end {
    color: gray;
    transform: translateZ(0);
            filter: grayscale(100%);
            filter: gray;

    -webkit-filter: grayscale(100%);
       -moz-filter: grayscale(100%);
        -ms-filter: grayscale(100%);
         -o-filter: grayscale(100%);

    .mnds_item_right {
      color: gray;
      .mnds_item_right_title,
      .mnds_item_right_award,
      .mnds_item_right_joiner,
      .mnds_item_right_times {
        color: gray;
      }
    }
  }
}

.competition_list {
  &_dialog {
    padding: 25px 20px 0;
    &_text {
      color: #222;
      font-size: 16px;
      line-height: 23px;
    }
  }
  &_know {
    position: relative;
    height: 50px;
    margin-top: 40px;
    color: #3b7be6;
    font-size: 16px;
    line-height: 50px;
    text-align: center;
  }
  &_know:before {
    content: "";
    position: absolute;
    top: 0;
    left: -20px;
    width: 290px;
    height: 1px;
    background-color: #f0f0f0;
    transform: scaleY(.5);
  }
}
</style>

