<template>
  <div id="shop_details" v-if="detailData!==0">
    <el-row class="header-section">
      <el-col :span="24">
        <el-col :span="12">
          <div class="banner" v-if="detailData.productInfo.showWay===1">
            <el-carousel height="250px">
              <el-carousel-item v-for="(item,index) in detailData.productInfo.showUrls" :key="index">
                <img style="width: 100%;height: 250px" :src="item" alt="">
              </el-carousel-item>
            </el-carousel>
          </div>
          <div v-if="detailData.productInfo.showWay===2" style="text-align: center">
            <!--<video controls style="height: 250px;margin: auto;" :src="detailData.productInfo.showUrls[0]">-->
            <!--</video>-->
            <video-player class="video-player vjs-custom-skin"
                          ref="videoPlayer"
                          :playsinline="true"
                          :options="playerOptions"
            ></video-player>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="banner-right">
            <div class="pro_box_top_rjs">
              <table border="1" cellpadding="0" cellspacing="0" width="400">
                <tbody>
                <tr>
                  <td class="title" valign="top" width="127">
                    <p>订单号</p>
                  </td>
                  <td class="text">
                    <p>{{detailData.orderInfo.orderId}}</p>
                  </td>
                </tr>
                <tr>
                  <td class="title" valign="top" width="127">
                    <p>商品名称</p>
                  </td>
                  <td class="text">
                    <p>{{detailData.productInfo.productName}}</p>
                  </td>
                </tr>
                <tr>
                  <td class="title" valign="top" width="127">
                    <p>成交价格</p>
                  </td>
                  <td class="text" style="color: darkred;">
                    <p>{{detailData.orderInfo.orderMoney}}元</p>
                  </td>
                </tr>
                <tr>
                  <td class="title" valign="top" width="127">
                    <p>运费</p>
                  </td>
                  <td class="text">
                    <p>{{detailData.productInfo.transferMoney || 0}}元</p>
                  </td>
                </tr>
                <!--<tr>-->
                <!--<td class="title" valign="top" width="127">-->
                <!--<p>实际金额</p>-->
                <!--</td>-->
                <!--<td class="text">-->
                <!--<p>-->
                <!--123-->
                <!--</p>-->
                <!--</td>-->
                <!--</tr>-->
                <tr>
                  <td class="title" valign="top" width="127">
                    <p>下单账号</p>
                  </td>
                  <td class="text">
                    <p>
                      {{detailData.orderInfo.orderUserId}}
                    </p>
                  </td>
                </tr>
                <tr>
                  <td class="title" valign="top" width="127">
                    <p>商品来源</p>
                  </td>
                  <td class="text">
                    <p>
                      {{detailData.orderInfo.shopId}}
                    </p>
                  </td>
                </tr>
                </tbody>
              </table>
            </div>
          </div>
        </el-col>
      </el-col>
    </el-row>
    <div class="select-address">
      <select-address :showHeader=false @postAddressData="postAddressData"></select-address>
    </div>
    <div class="flex flex-end" style="margin-top: 22px;font-size: 16px;">
      <div style="margin-right: 12px;line-height: 38px">发票类型</div>
      <el-select v-model="invoiceInfo" placeholder="无需发票">
        <el-option
          label="无需发票"
          value="无需发票">
        </el-option>
        <el-option
          label="明细（纸质）个人"
          value="明细（纸质）个人">
        </el-option>
        <el-option
          label="明细（电子）个人"
          value="明细（电子）个人">
        </el-option>
      </el-select>
    </div>
    <div class="foot flex">
      <div>
        实付款：￥<span class="text">{{detailData.orderInfo.orderMoney + detailData.productInfo.transferMoney || 0}}</span>
      </div>
      <el-button type="primary" @click="addCollection">提交订单</el-button>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import PingLun from '../../common/pinglun/pinglun.vue'
  import PingLunList from '../../common/pinglun/list.vue'
  import JingPaiChuJia from '../../common/jingpai/jingpai.vue'
  import JingPaiChuJiaList from '../../common/jingpai/jingPaiList.vue'
  import ScrollSole from '../../common/scroll/scrollSole.vue'
  import CountDown from '../../common/countDown/countDown.vue'
  import selectAddress from '../../common/Address/selectAddress.vue'

  export default {
    data () {
      return {
        activeName: '0',        //tab默认选中
        commentVisi: false,     //评论dialog
        commentList: [],        //评论列表
        commentPageNo: 1,        //评论当前页码
        commentCount: 10,        //评论总数
        AuctionDidPrice: 0,     //拍卖价格
        priceListPageNo: 1,     //出价当前页码
        priceListCount: 10,     //出价当前页码
        priceList: [],          //出价列表
        invoiceInfo: '无需发票',  //发票类型
        detailData: 0,          //单品详情
        address: '',
        telphone: '',
        callBackUrl: localStorage.getItem('url'),
        playerOptions: {
          playbackRates: [0.7, 1.0, 1.5, 2.0], //播放速度
          autoplay: false, //如果true,浏览器准备好时开始回放。
          muted: false, // 默认情况下将会消除任何音频。
          loop: false, // 导致视频一结束就重新开始。
          preload: 'auto', // 建议浏览器在<video>加载元素后是否应该开始下载视频数据。auto浏览器选择最佳行为,立即开始加载视频（如果浏览器支持）
          language: 'zh-CN',
          aspectRatio: '16:9', // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
          fluid: true, // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
          sources: [{
            type: '',
            src: '' //url地址
          }],
          poster: '', //你的封面地址
          // width: document.documentElement.clientWidth,
          notSupportedMessage: '此视频暂无法播放，请稍后再试', //允许覆盖Video.js无法播放媒体源时显示的默认信息。
          controlBar: {
            timeDivider: true,
            durationDisplay: true,
            remainingTimeDisplay: false,
            fullscreenToggle: true  //全屏按钮
          }
        }
      }
    },
    methods: {
      getDetailProduct () {
        const url = 'paimai/front/detail_order'
        const ret = r => {
          console.log('单品详情----------------')
          console.log(r)
          if (r.busCode === 200) {
            this.$set(this, 'detailData', r.data)
            if (r.data.productInfo.showWay === 2) {
              this.playerOptions.sources[0].src = r.data.productInfo.showUrls[0]
            }
          } else {
            this.$message.error(r.data)
          }
          console.log(this.detailData)
        }
        console.log(this.$route.query.orderId)
        this.ajax(url, 'get', {orderId: this.$route.query.orderId}, ret, 30000, false)
      },
      addCollection () {
        const data = {
          address: this.address,
          invoiceInfo: this.invoiceInfo,
          orderId: this.$route.query.orderId,
          telphone: this.telphone
        }
        const url = 'paimai/front/submit_order'
        const ret = r => {
          if (r.busCode === 200) {
            this.payment(
              r.data.orderId,
              r.data.orderMoney,
              this.detailData.productInfo.productName,
              null,
              r.data.receiveId,
              window.location.href.split(this.$route.path)[0] + '/payment_success?orderId=' + data.orderId
            )
          }
        }
        this.ajax(url, 'post', data, ret)
      },
      postAddressData (data) {
        this.address = data.region + ' ' + data.detailAddress
        this.telphone = data.telephone
      }
    },
    components: {
      PingLun,
      PingLunList,
      JingPaiChuJia,
      ScrollSole,
      CountDown,
      JingPaiChuJiaList,
      selectAddress
    },
    created () {
      this.getDetailProduct()
      console.log()
    },
    mounted () {
      console.log(1)
      console.log()
    }
  }
</script>
<style lang="less">
  #shop_details {
    .el-carousel__item h3 {
      color: #475669;
      font-size: 14px;
      opacity: 0.75;
      line-height: 250px;
      margin: 0;
    }

    .el-carousel__item:nth-child(2n) {
      background-color: #99a9bf;
    }

    .el-carousel__item:nth-child(2n+1) {
      background-color: #d3dce6;
    }
  }
</style>
<style scoped lang="less">
  #shop_details {
    width: 900px;
    margin: auto;
    .header-section {
      border-bottom: 1px solid #eeeeee;
      .banner {
        padding: 30px;
      }
      .banner-right {
        margin-top: 30px;
        padding-left: 45px;
        .pro_box_top_rzi {
          padding-top: 12px;
          line-height: 26px;
          font-size: 18px;
          color: #AF0C00;
          padding-bottom: 10px;
        }
        table {
          line-height: 32px;
          font-size: 14px;
          .title {
            width: 95.4pt;
            border: solid windowtext 0.0pt;
            mso-border-alt: solid windowtext .5pt;
            padding: 0cm 5.4pt 2pt 5.4pt;
          }
          .text {
            width: 225.0pt;
            border: solid windowtext 0.0pt;
            border-left: none;
            mso-border-left-alt: solid windowtext .5pt;
            mso-border-alt: solid windowtext .5pt;
            padding: 0cm 5.4pt 2pt 5.4pt;
          }
        }
      }
    }
    .foot {
      justify-content: flex-end;
      padding-right: 12px;
      margin-top: 22px;
      .text {
        font-size: 22px;
        color: red;
        margin-right: 12px;
      }
    }
  }
</style>
