<template>
  <view>
    <view class="goods-list">
      <view v-for="(goods, i) in goodsList" :key="i" @click="gotoDetail(goods)">
        ，<my-goods :goods="goods"></my-goods>
      </view>
    </view>
  </view>
</template>

<script>
	export default {
		data() {
			return {
				// 请求参数对象
        queryObj: {
          // 查询关键词
          query: '',
          // 商品分类Id
          cid: '',
          // 页码值
          pagenum: 1,
          // 每页显示多少条数据
          pagesize: 10
        },
        goodsList: [],     //商品列表数组
        total: 0,
        //节流阀
        isLoading: false
        
			};
		},
    onLoad(options) {
      // 将页面参数转存到 this.queryObj 对象中
      this.queryObj.query = options.query || ''
      this.queryObj.cid = options.cid || ''
      
      this.getGoodsList()
    },
   methods: {
     // 获取商品列表数据的方法
     async getGoodsList(cb) {
       //打开节流阀
       this.isLoading = true
       // 发起请求
       const { data: res } = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
       //关闭节流阀
       this.isLoading = false
       cb && cb()
       if (res.meta.status !== 200) return uni.$showMsg()
       // 为数据赋值
       this.goodsList = [...this.goodsList, ...res.message.goods]
       this.total = res.message.total
     },
     //跳转到商品详情页
     gotoDetail(goods){
       uni.navigateTo({
         url: '/subpkg/goods-detail/goods-detail?goods_id=' + goods.goods_id
       })
     }
   },
   //上拉触底
   onReachBottom() {
     if(this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg("数据加载完毕！")
     
     if(this.isLoading) return
     
     //让页码值自增+1
     this.queryObj.pagenum ++
     this.getGoodsList()
   },
    //下拉刷新
   onPullDownRefresh() {
     //重置关键数据
     this.queryObj.pagenum = 1
     this.total = 0
     this.isLoading = false
     this.goodsList = []
     
     //重新发起数据请求
     this.getGoodsList(() => {
       uni.stopPullDownRefresh()
     })
     
   }
	}
</script>

<style lang="scss">

</style>
