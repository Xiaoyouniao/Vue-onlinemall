<template>
  <div>
    <nav-header></nav-header>
    <nav-bread><span>Goods</span></nav-bread>
    <div class="accessory-result-page accessory-page">
      <div class="container">
        <div class="filter-nav">
          <span class="sortby">Sort by:</span>
          <a href="javascript:void(0)" class="default cur">Default</a>
          <a href="javascript:void(0)" class="price" :class="{'sort-up':sortFlag}" @click="sortPrice">Price <svg class="icon icon-arrow-short"><use xlink:href="../../static/svg.svg#icon-arrow-short"></use></svg></a>
          <a href="javascript:void(0)" class="filterby stopPop" @click="filterByShow">Filter by</a>
        </div>
        <div class="accessory-result">
          <!-- filter -->
          <div class="filter stopPop" id="filter" :class="{ 'filterby-show':filterBy }">
            <dl class="filter-price">
              <dt>Price:</dt>
              <dd><a href="javascript:void(0)" :class="{ 'cur': priceChecked==='all' }" @click="setPriceFilter('all')">All</a></dd>
              <dd v-for="(price,index) in priceFilter" :key="index">
                <a href="javascript:void(0)" :class="{'cur': priceChecked==index}" @click="setPriceFilter(index)">{{price.startPrice}} - {{price.endPrice}}</a>
              </dd>
            </dl>
          </div>

          <!-- search result accessories list -->
          <div class="accessory-list-wrap">
            <div class="accessory-list col-4">
              <ul>
                <li v-for="item in goodsLists" :key="item._id">
                  <div class="pic">
                    <a href="#"><img v-lazy="'../../static/img/'+item.productImage"></a>
                  </div>
                  <div class="main">
                    <div class="name">{{item.productName}}</div>
                    <div class="price">{{item.salePrice}}</div>
                    <div class="btn-area">
                      <a href="javascript:;" @click="addToCart(item.productId)" class="btn btn--m">加入购物车</a>
                    </div>
                  </div>
                </li>
              </ul>
              <div class="view-more-normal"
                   v-infinite-scroll="loadMore"
                   infinite-scroll-disabled="busy"
                   infinite-scroll-distance="20">
                <img src="./../assets/loading-spinning-bubbles.svg" v-show="loading">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="md-overlay" v-show="overLayFlag" @click.stop="closePop"></div>
    <nav-footer></nav-footer>
    <modal :mdShow="mdShowFlag" @close="closeModal">
      <h3 slot="message" v-html="mdMsg"></h3>
      <div slot="btnGroup">
        <a class="btn btn--m" @click="btnA" v-text="btna"></a>
        <a class="btn btn--m" @click="btnB" v-text="btnb" v-if="showBtnB"></a>
      </div>
    </modal>
  </div>
</template>

<script>
import NavHeader from '@/components/NavHeader.vue'
import NavFooter from '@/components/NavFooter.vue'
import NavBread from '@/components/NavBread.vue'
import Modal from '@/components/Modal.vue'
import axios from 'axios'

export default {
  name: 'GoodsList',
  data(){
    return{
      scrollTimer: '', // 为vue-infinite-scroll插件定义的定时器
      page: 1, // 定义访问的数据库页码
      pageSize: 8, // 定义访问数据库中每页数据条数
      sortFlag: true, // 
      goodsLists: [],
      overLayFlag: false,
      filterBy: false,
      priceChecked: 'all',
      priceFilter: [{
            startPrice: 0.00,
            endPrice: 100.00
          },{
            startPrice: 100.00,
            endPrice: 500.00
          },{
            startPrice: 500.00,
            endPrice: 1000.00
          },{
            startPrice: 1000.00,
            endPrice: 5000.00
          }],
      busy: true,
      loading: false,
      mdShow: false,
      mdShowCart: false,
      mdShowFlag: false, //控制模态框显示与否的变量
      btna:'',
      btnb: '',
      showBtnB:false,
      mdMsg: '' // 模态框中显示的内容（v-html绑定）
      
    }
    
  },
  components:{
    NavHeader,
    NavFooter,
    NavBread,
    Modal
  },
  mounted(){
    this.getGoodLists()
  },
  methods: {
    btnA(){ // 根据不同情况显示不同的按钮A点击事件
      if(this.btna == '确认'){
        this.closeModal()
      }
    },
    btnB(){ // 根据不同情况显示不同的按钮A点击事件

    },
    filterByShow(){ // 屏幕变窄后点击按钮出现遮罩和隐藏的价格列表
      this.filterBy = true
      this.overLayFlag = true
    },
    closePop(){ // 屏幕变窄时关闭遮罩，隐藏价格列表
      this.filterBy = false
      this.overLayFlag = false
    },
    setPriceFilter(option) { // 点击价格区间后关闭遮罩，改变价格列表样式，向服务器发起数据请求
      this.priceChecked = option
      this.closePop()
      this.getGoodLists()
    },
    getGoodLists(flag){ // 根据选项获取指定商品数据
      var param = {
        page: this.page,
        pageSize: this.pageSize,
        sort: this.sortFlag?1:-1,
        priceLevel: this.priceChecked
      }
      
      axios.get('/goods/list', {params: param}).then(res=>{
        
        if(res.data.status == 0){
          if(flag){
            this.loading = true
            this.goodsLists = this.goodsLists.concat(res.data.result.list)
            if(res.data.result.count==0) {
              this.busy = true
              this.loading = false
              clearInterval(this.scrollTimer)
            }else{
              this.busy = false
            }
          } else {
            this.goodsLists = res.data.result.list
            this.busy = false
          }
        }else{
          console.log('2')
          this.goodsLists = []
        }
      }).catch(err=>{
        console.log(err)
      })
    },
    sortPrice(){ // 商品价格排序，关闭遮罩
      this.sortFlag = !this.sortFlag
      this.page = 1
      this.getGoodLists()
    },
    loadMore() { // vue-infinit-scroll插件函数
      this.busy = true;
 
      this.scrollTimer=setTimeout(() => {
        this.page ++
        this.getGoodLists(true)
        this.busy = false;
      }, 1000); 
    },
    closeModal(){ // 关闭模态框
      this.mdShowFlag = false
      this.mdMsg = ''
    },
    addToCart(productId) { // 将产品添加至购物车
    /**
     * 1. 检验登录状态，拦截将商品加入购物车的请求
     */
      if(document.cookie) { 
        var param = { productId:productId }
        axios.post('goods/addCart', param).then(res=>{
          if (res.data.status == 0){
            this.mdShowFlag = true
            this.mdMsg = '<h3>加入购物车成功！</h3>'
            this.btna = '确认'
          }
        })
      } else {
        this.mdShowFlag = true
        this.mdMsg = '<h3>您尚未登录，请登录后再选购商品！</h3>'
        this.btna = '确认'
      }
      
    }
  }
}
</script>


