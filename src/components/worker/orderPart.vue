<template>
	<div id="box">
		<!--顶部-->
		<Header title="订单" back="hidden"></Header>
		<div>
			<div>
				<ul class="navBox">
					<li v-for="(item, index) in orderStatus" :key="index" @click="changeMenu(index)">
						<a class="routerLink" :class="{active: item.active}">{{item.text}}</a>
					</li>
				</ul>
				<div style="height: 0.8rem;"></div>
			</div>
		</div>
		<div class="page-content">
			<Pagination :render="render" :autoFill="false" :param="pagination" :autoload="false" :need-token="true" uri="/serviceOrder/listForWorker" ref="pagination">
				<div class="order-content" v-show="pagination.content.length>0">
					<div class="contBox" v-for="(item,index) in pagination.content" :key="index">
						<!--姓名 电话-->
						<a v-if="isWeixin != 'yes'" @click="callPhone(item.userPhone)" class="peopleName">
							<span>{{item.userName}}</span>
							<span>{{item.userPhone}}</span>
							<div></div>
						</a>
						
						<a v-if="isWeixin == 'yes'" :href="'tel:' + item.userPhone" class="peopleName">
							<span>{{item.userName}}</span>
							<span>{{item.userPhone}}</span>
							<div></div>
						</a>
						<!--订单号-->
						<div style="padding: 0.15rem 0;">
							<div class="cont">
								<span class="contLeft">订单号：</span>
								<span class="contRight">{{item.orderNo}}</span>
							</div>
							<!--服务时间-->
							<div class="cont">
								<span class="contLeft">服务时间：</span>
								<span class="contRight">{{item.serviceTime}}</span>
							</div>
							<!--服务地址-->
							<div class="cont">
								<span class="contLeft">服务地址：</span>
								<span class="contRight position">{{item.serviceAddress.address}}</span>
								<a class="map-lnik" @click="goAmap(item.serviceAddress)">
									<span>去这里</span>
								</a>
							</div>
						</div>
						
						<!--服务详情-->
						<div class="thingList">
							<ul class="thingCont">
								<li class="things">
									<a>
										<img :src="item.serviceImage" />
										<div class="thingName">
											<div v-text="item.serviceTitle"></div>
											<span>&yen;{{item.unitPrice}}</span>
										</div>
										<span class="counts" v-if="item.priceType == 1">x{{item.serviceMount}}</span>
									</a>
								</li>
							</ul>
						</div>
						<!--合计-->
						<div class="totalMoney">
							<span class="totalRight">&yen; <i>{{item.totalPrice}}</i></span>
							<span class="tolalLeft">合计:</span>
						</div>
						<div class="send" v-show="item.operation.length>0">
							<div v-for="(obj, key) in item.operation" :class="obj.operationType == 6 ? 'noOperate' : ''" @click="operateOrder(obj,key, item.orderNo, item.operation)">{{obj.event}}</div>
						</div>
					</div>
					<!--<div class="kong"></div>-->
				</div>
			    <div class="none-data-tip" v-show="pagination.content.length == 0 && pagination.loadEnd">暂无订单</div>
					
	        </Pagination>

		</div>
			
		<confirm-modal :show="show" 
			@cancel_modal="cancel_modal" 
			@confirm_modal="confirm_modal" 
			@closeModal="show = false" 
			:has_input="true"
			message="">
		</confirm-modal>
		
		<workerPart actived="first"></workerPart>
	</div>
</template>

<script type="text/javascript">
import { Toast } from 'mint-ui'
	export default {
		data(){
			return {
				isWeixin: this.$storage.get('isWeixinTerm'),
				show: false,
				currOperate: {},
                currOrderNo: undefined,
				
				orderStatus: [{
					text: '待服务',
					active: false,
					flag: 1
				},{
					text: '服务中',
					active: false,
					flag: 2
				},{
					text: '已完成',
					active: false,
					flag: 3
				},{
					text: '已取消',
					active: false,
					flag: 4
				}],
				pagination: {
                    content: [],
                    page: 1, 
					pageSize: 6,
					param: {
						params: {
							flag: 1
						}
					},
					loadEnd: false
                },
                
                order_status: 0
			}
		},
		mounted() {
//			this.changeMenu(this.order_status)
		},
		beforeRouteEnter (to, from, next) {
	    	if(/amap/g.test(from.fullPath)) {
	    		next()
	    	}else {
	    		next(vm=>{
	    			vm.show = false
					vm.currOperate = {}
	                vm.currOrderNo = undefined
	    			vm.changeMenu(vm.order_status)
			        
	        	})
	    	}	
		},
		methods: {
			// 切换菜单
			changeMenu(index) {
				this.order_status = index
				this.pagination = {
                    content: [],
                    page: 1, 
					pageSize: 6,
					param: {
						params: {
							flag: this.orderStatus[index].flag
						}
					}
				}
				for (let i in this.orderStatus) {
					this.orderStatus[i].active = false;
					if (i == index) {
						this.orderStatus[i].active = true;
					}
				}
				this.$refs.pagination.refresh()
			},
			render(res) {
                res.result.list.forEach((item) => {
                	if(item.operation == null) {
                		item.operation = []
                	}
                	this.pagination.content.push(item)
                })
            },
            cancel_modal() {
            	this.show = false
            },
            confirm_modal(serviceMoney) {
            	this.$api.updateOrderStatus({
					flag: this.currOperate.flag,
					orderNo: this.currOrderNo,
					type: this.currOperate.type,
					price: serviceMoney
				}, (res) => {
					Toast({
					  message: res.err_msg,
					  position: 'middle',
					  iconClass: 'toast-icon icon-success',
					  duration: 1000
					})
	                setTimeout(() => {
						this.changeMenu(this.order_status)
					},500)
				})
            },
            operateOrder(obj,key, orderNo, operation) {
            	if(obj.operationType == 3) {
            		this.currOperate = obj
            		this.currOrderNo = orderNo
            		this.show = true
            	}else if(obj.operationType == 6) {
            		console.log('不操作')
            	}else {
            		this.$api.updateOrderStatus({
						flag: obj.flag,
						orderNo: orderNo,
						type: obj.type
					}, (res) => {
						Toast({
						  message: res.err_msg,
						  position: 'middle',
						  iconClass: 'toast-icon icon-success',
						  duration: 1000
						})
		                setTimeout(() => {
							this.changeMenu(this.order_status)
						},500)
					})
            	}
            },
            callPhone(phone) {
            	this.$bridge.callPhone(phone)
            },
            goAmap(address) {
            	this.$storage.set('currAddr', address)
            	this.$router.push('/amap')
            }
		}
	}
</script>

<style scoped>
	#box {
		position: absolute;
		width: 100%;
		height: 100%;
		left: 0;
		top: 0;
		bottom: 0;
	}
	.page-content{
		position: absolute;
		width: 100%;
		left: 0;
		top: 1.76rem;
		bottom: 1rem;
		overflow-y: auto;
	}
	.headPart{
		width: 100%;
		height: 0.92rem;
		background: #2d91f4;
		overflow: hidden;
	}
	.headCont{
		width: 7rem;
		height: 0.32rem;
		margin: 0.25rem;
		/*border: 1px solid red;*/
	}
	.headCont a{
		display: block;
	}
	.headCont p{
		height: 0.32rem;
		color: #FFFFFF;
		font-size: 0.32rem;
	}
	/*顶部导航*/
	.navBox{
		position: fixed;
		left: 0;
		top: 0.92rem;
		width: 7.5rem;
		height: 0.78rem;
		/*border-bottom: 2px solid #f2f2f2;*/
		background: #FFFFFF;
		display: flex;
		z-index: 50;
	}
	.navBox li{
		flex: 1;
		position: relative;
		z-index: 140;
	}
	.routerLink{
		font-size: 0.28rem;
		color:#222222;
		line-height:0.78rem ;
		display: inline-block;
		padding: 0 0.2rem;
	}
	.routerLink.active {
		color: #2173d6;
		border-bottom: 2px solid #2173d6;
	}
	.contBox {
		width: 7.5rem;
		background: #FFFFFF;
		display: inline-block;
		position: relative;
		margin-top: 0.2rem;
		overflow: hidden;
	}
	/*姓名电话*/
	
	.peopleName {
		width: 7rem;
		height: 0.78rem;
		padding: 0 .25rem;
		border-bottom: 1px solid #f2f2f2;
		display: block;
	}
	
	.peopleName span {
		float: left;
		/*font-size: 0.24rem;*/
		color: #666;
		line-height: .78rem;
		margin-right: 0.15rem;
	}
	
	.peopleName div {
		width: 0.42rem;
		height: 0.42rem;
		background: url("../../../static/wokerPhone.png") no-repeat;
		float: right;
		background-size: 100% 100%;
		margin-top: 0.2rem;
	}
	/*订单内容*/
	
	.cont {
		width: 7rem;
		padding: 0.1rem .25rem;
		overflow: hidden;
		text-align: left;
		display: flex;
		position: relative;
	}
	
	.cont span {
		
	}
	
	.contLeft {
		color: #bbb;
		width: 1.5rem;
	}
	
	.contRight {
		color: #666;
		flex: 1;
	}
	
	.contRight.position {
		padding-right: 1rem;
		text-align: left;
		min-height: 0.76rem;
	}
	
	.positionBtn img {
		width: 0.2rem;
		height: 0.3rem;
		display: block;
		margin: 0 auto;
	}
	
	.positionBtn span {
		font-size: 0.18rem;
		color: #CCCCCC;
	}
	/*服务详情*/
	/*物品列表*/
	
	.thingList {
		width: 7rem;
		padding: 0.25rem;
		background: #f5f5f9;
		display: inline-block;
	}
	
	.thingList li {
		width: 7rem;
		position: relative;
	}
	
	.thingList a {
		width: 100%;
		height: 100%;
		display: block;
		display: flex;
	}
	/*左边图片*/
	
	.thingList img {
		width: 1.40rem;
		height: 1.40rem;
		margin-right: 0.3rem;
	}
	/*右边内容*/
	
	.thingName {
		flex: 1;
		height: 1.4rem;
		text-align: left;
	}
	/*商品名称*/
	
	.thingName div {
		width: 100%;
		height: 0.6rem;
		font-size: 0.26rem;
		line-height: 0.6rem;
		color: #222222;
	}
	/*介绍*/
	
	.thingName p {
		width: 100%;
		height: 0.45rem;
		font-size: 0.2rem;
		line-height: 0.45rem;
		color: #cfcfcf;
	}
	/*价格*/
	
	.thingName span {
		width: 100%;
		height: 0.36rem;
		font-size: 0.28rem;
		line-height: 0.36rem;
		color: #ff5400;
	}
	/*数量*/
	
	.counts {
		position: absolute;
		right: 0.25rem;
		bottom: 0.25rem;
		font-size: 0.28rem;
		color: #222222;
	}
	/*合计*/
	
	.totalMoney {
		width: 7rem;
		height: 0.78rem;
		padding: 0 0.25rem;
		border-bottom: 1px solid #f2f2f2;
	}
	
	.totalMoney span {
		float: right;
		margin-right: 0.16rem;
	}
	
	.totalRight {
		font-size: 0.28rem;
		color: #000000;
		line-height: 0.78rem;
	}
	.totalRight>i{
		font-size: 0.32rem;
		font-style: normal;
	}
	
	.tolalLeft {
		font-size: 0.28rem;
		color: #222222;
		line-height: 0.78rem;
	}
	.map-lnik{
		position: absolute;
		right: 0.25rem;
		top: 0.1rem;
		width: 0.8rem;
		height: 0.7rem;
		font-size: 0.2rem;
		text-align: center;
		color: #666;
		padding-top: 0.42rem;
		box-sizing: border-box;
	}
	.map-lnik:before{
		position: absolute;
		content: '';
		width: 0.34rem;
		height: 0.34rem;
		left: 0.2rem;
		top: 0;
		background: url(../../../static/worker01.png) no-repeat center;
		background-size: auto 100%; 
	}
	.send {
		width: 7rem;
		overflow: hidden;
		padding: 0.25rem;
	}
	
	.send div{
		float: right;
		height: 0.56rem;
		font-size: 0.24rem;
		color: #2173d6;
		line-height: 0.56rem;
		border-radius: 0.28rem;
		border: 1px solid #8ac4f9;
		padding: 0 0.2rem;
		margin-left: 0.2rem;
	}
	#qu{
		color: #222222;
		border: 1px solid #cccccc;
	}
</style>