<template>
	<div id="box">
		<Header title="派单" back="hidden"></Header>
		<ul class="navBox" v-show="workerRole == 3">
			<li v-for="(item, index) in menus" @click="changeMenu(index)">
				<div class="routerLink" :class="{active: item.actived}">{{item.name}}</div>
			</li>
		</ul>
		<div class="page-content" :class="type == 2 ? 'pended' : ''" v-show="workerRole == 3">
			<div class="pended-status" v-show="type == 2 && workerRole == 3">
				<span v-for="(item, index) in pendedMenus" :class="{active: item.actived}" @click="changePended(index)">{{item.name}}</span>
			</div>
			<Pagination :render="render" :param="pagination" :autoload="false" :need-token="true" :uri="currApiUrl"  ref="pagination">
				
				<div class="list" v-show="pagination.content.length > 0">
					<div v-show="type == 1">
						<div class="contBox" v-for="(contBox,index) in pagination.content">
							<!--姓名 电话-->
							<a v-if="isWeixin != 'yes'" @click="callPhone(contBox.linkPhone)" class="contact">
								<span>{{contBox.linkName}}</span>
								<span>{{contBox.linkPhone}}</span>
								<div></div>
							</a>
							<a v-if="isWeixin == 'yes'" :href="'tel:' + contBox.linkPhone" class="contact">
								<span>{{contBox.linkName}}</span>
								<span>{{contBox.linkPhone}}</span>
								<div></div>
							</a>
							<!--订单号-->
							<div class="cont">
								<span class="contLeft">订单编号:</span>
								<span class="contRight">{{contBox.orderNo}}</span>
							</div>
							<!--服务时间-->
							<div class="cont">
								<span class="contLeft">服务时间:</span>
								<span class="contRight">{{contBox.serviceTime}}</span>
							</div>
							<!--服务地址-->
							<div class="cont" style="padding-bottom: 0.25rem;">
								<span class="contLeft">服务地址:</span>
								<span class="contRight position" style="line-height: 1.4;">{{contBox.serviceAddress.address}}</span>
								<a class="map-lnik" @click="goAmap(contBox.serviceAddress)">
									<span>去这里</span>
								</a>
							</div>
							
							<!--服务详情-->
							<div class="thingList">
								<ul class="thingCont">
									<li class="things">
										<a>
											<img :src="contBox.sImage" />
											<div class="thingName">
												<div v-text="contBox.sTitle"></div>
												<span>&yen;{{contBox.price}}</span>
											</div>
											<span class="counts" v-if="contBox.orderType == 1">x{{contBox.mount}}</span>
										</a>
									</li>
								</ul>
							</div>
							<!--合计-->
							<div class="totalMoney">
								<span class="totalRight">&yen;{{contBox.total}}</span>
								<span class="tolalLeft">合计:</span>
							</div>
							<!--派单-->
							<div class="send" v-show="type == 1">
								<div @click="orderMenu(contBox.orderNo)">派单</div>
							</div>
						</div>
					</div>
					<div v-show="type == 2">
						<div class="contBox" v-for="(contBox,index) in pagination.content">
							<!--姓名 电话-->
							<a v-if="isWeixin != 'yes'" @click="callPhone(contBox.userPhone)" class="contact">
								<span>{{contBox.userName}}</span>
								<span>{{contBox.userPhone}}</span>
								<div></div>
							</a>
							<a v-if="isWeixin == 'yes'" :href="'tel:' + contBox.userPhone" class="contact">
								<span>{{contBox.userName}}</span>
								<span>{{contBox.userPhone}}</span>
								<div></div>
							</a>
							<!--订单号-->
							<div class="cont">
								<span class="contLeft">订单编号:</span>
								<span class="contRight">{{contBox.orderNo}}</span>
							</div>
							<!--<div class="cont">
								<span class="contLeft">订单状态:</span>
								<span class="contRight">{{contBox.info}}</span>
							</div>-->
							<!--服务时间-->
							<div class="cont">
								<span class="contLeft">服务工人:</span>
								<span class="contRight">{{contBox.workerName}}&nbsp; {{contBox.workerPhone}}</span>
							</div>
							<div class="cont">
								<span class="contLeft">服务时间:</span>
								<span class="contRight">{{contBox.serviceTime}}</span>
							</div>
							<!--服务地址-->
							<div class="cont" style="padding-bottom: 0.25rem;">
								<span class="contLeft">服务地址:</span>
								<span class="contRight position" style="line-height: 1.4;">{{contBox.serviceAddress.address}}</span>
								<a class="map-lnik" @click="goAmap(contBox.serviceAddress)">
									<span>去这里</span>
								</a>
							</div>
							
							<!--服务详情-->
							<div class="thingList">
								<ul class="thingCont">
									<li class="things">
										<a>
											<img :src="contBox.serviceImage" />
											<div class="thingName">
												<div v-text="contBox.serviceTitle"></div>
												<span>&yen;{{contBox.unitPrice}}</span>
											</div>
											<span class="counts" v-if="contBox.orderType == 1">x{{contBox.serviceMount}}</span>
										</a>
									</li>
								</ul>
							</div>
							<!--合计-->
							<div class="totalMoney">
								<span class="totalRight">&yen;{{contBox.totalPrice}}</span>
								<span class="tolalLeft">合计:</span>
							</div>
							<!--派单-->
							<div class="send" v-show="contBox.operation.length>0">
								<div v-for="(obj, key) in contBox.operation"  @click="operateOrder(obj, contBox.orderNo)">{{obj.event}}</div>
							</div>
						</div>
					</div>	
						
				</div>
				<div class="none-data-tip" v-show="pagination.content.length == 0 && pagination.loadEnd">暂无派单</div>	
			</Pagination>
			
		</div>	
			
		<div class="none-data-tip" v-show="workerRole == 2">
			<img class="none-data-img" src="../../../static/43@2x.png" />
			<p>暂无派单功能</p>
		</div>

		<!--底部导航-->
		<workerPart actived="second"></workerPart>
	</div>
</template>

<script type="text/javascript">
import { Toast } from 'mint-ui'
	export default {
		data(){
			return {
				isWeixin: this.$storage.get('isWeixinTerm'),
				menus: [
					{
						name: '待派单',
						type: 1,
						actived: true
					}, 
					{
						name: '已派单',
						type: 2,
						actived: false
					}
				],
				pendedMenus: [
					{
						name: '待服务',
						flag: 1,
						actived: false
					}, 
					{
						name: '服务中',
						flag: 2,
						actived: false
					},
					{
						name: '已完成',
						flag: 3,
						actived: false
					}, 
					{
						name: '已取消',
						flag: 4,
						actived: false
					}
				],
				type: 10,
				pagination: {
					content: [],
					page: 1,
					pageSize: 6,
					param: {
						params: {
							type: 1
						}
					},
					loadEnd: false
					
				},
				
				pending_status: 0,
				
				workerRole: 3,
				
				currApiUrl: '/serviceOrder/pend',
				flag: 0
				
			}
		},
		created() {
//			this.pending_status = this.$route.params.id
		},
		mounted() {
//			if(this.$storage.get('currRole') == 2) {
//				this.workerRole = 2
//				return
//			}
//			this.changeMenu(this.pending_status)
		},
		beforeRouteEnter (to, from, next) {
	    	if(/amap/g.test(from.fullPath)) {
	    		next()
	    	}else {
	    		next(vm=>{
	    			if(vm.$storage.get('currRole') == 2) {
						vm.workerRole = 2
						return
					}
	    			vm.pending_status = vm.$route.params.id
	    			vm.workerRole = 3
	    			vm.flag = 1
	    			vm.changeMenu(vm.pending_status)
			        
	        	})
	    	}	
		},
		
		methods: {
			changeMenu(index) {
				if( index == (this.type - 1) ) {
					return
				}
				this.pending_status = index
				this.type = Number(index) + 1
				if(this.type == 1) {
					this.currApiUrl = '/serviceOrder/pend'
					this.pagination = {
	                    content: [],
	                    page: 1, 
						pageSize: 6,
						param: {
							params: {
								type: 1
							}
						}
					}
				}else {
					this.currApiUrl = 'serviceOrder/pended'
					if(this.flag == 0) {
						this.flag = 1
					}
					this.pagination = {
	                    content: [],
	                    page: 1, 
						pageSize: 6,
						param: {
							params: {
								flag: this.flag
							}
						}
					}
					for (var i in this.pendedMenus) {
						this.pendedMenus[i].actived = false
						if (i == (this.flag -1)) {
							this.pendedMenus[i].actived = true
						}
					}
				}
				for (var i in this.menus) {
					this.menus[i].actived = false
					if (i == index) {
						this.menus[i].actived = true
					}
				}

				

				this.$refs.pagination.refresh()
			},
			changePended(index) {
//				if( index == (this.flag - 1) ) {
//					return
//				}
				this.currApiUrl = 'serviceOrder/pended'
				this.flag = Number(index) + 1
				this.pagination = {
                    content: [],
                    page: 1, 
					pageSize: 6,
					param: {
						params: {
							flag: this.flag
						}
					}
				}
				for (var i in this.pendedMenus) {
					this.pendedMenus[i].actived = false
					if (i == (this.flag -1)) {
						this.pendedMenus[i].actived = true
					}
				}

				this.$refs.pagination.refresh()

			},
			
			//跳转工人页面
			orderMenu(id) {
				this.$router.push('/sendPeople/'+ id)
			},
			operateOrder(obj, orderNo) {
            	if(obj.operationType == 7) {
            		this.$router.push('/sendPeople/'+ orderNo)
            	}else if(obj.operationType == 1) {
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
							this.changePended(this.flag - 1)
						},500)
					})
            	}
            },
		

			render(response) {
				for(var i in response.result.list){
					if(response.result.list[i].operation == null) {
                		response.result.list[i].operation = []
                	}
					this.pagination.content.push(response.result.list[i])
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
	.page-content{
		position: absolute;
		width: 100%;
		left: 0;
		top: 1.76rem;
		bottom: 1rem;
		overflow-y: auto;
	}
	.page-content.pended{
		padding-top: 1rem;
	}
		
	.navBox{
		position: fixed;
		left: 0;
		top: 0.92rem;
		width: 7.5rem;
		height: 0.8rem;
		/*border-bottom: 2px solid #f2f2f2;*/
		background: #FFFFFF;
		display: flex;
		-webkit-box-orient: horizontal;
		display: -webkit-box;
		z-index: 50;
	}
	.navBox li{
		-webkit-box-flex: 1;
	}
	.routerLink{
		height: 100%;
		font-size: 0.26rem;
		color:#666;
		line-height:0.72rem ;
		display: inline-block;
		padding: 0 0.2rem;
		box-sizing: border-box;
	}
	.routerLink.active {
		color: #2173d6;
		border-bottom: 2px solid #2173d6;
	}
	.pended-status{
		position: fixed !important;
		left: 0;
		top: 1.76rem;
		width: 7.5rem;
		height: 0.8rem;
		z-index: 50;
		background: #fff;
		border-top: 0.2rem solid #F5F5F9;
		display: flex;
		align-items: center;
		justify-content: space-around;
	}
	.pended-status>span{
		background: #eee;
		height: 0.5rem;
		width: 1.5rem;
		line-height: 0.5rem;
		color: #999;
		border-radius: 0.08rem;
	}
	.pended-status>span.active{
		background: #258ef3;
		color: #fff;
	}
	.contBox {
		width: 7.5rem;
		background: #FFFFFF;
		display: inline-block;
		position: relative;
		margin-top: 0.2rem;
		overflow: hidden;
	}
	
	.contact {
		width: 7rem;
		height: 0.78rem;
		padding: 0 .25rem;
		border-bottom: 1px solid #f2f2f2;
		margin-bottom: .2rem;
		display: block;
	}
	.contact span {
		float: left;
		font-size: 0.26rem;
		color: #666;
		line-height: .78rem;
		margin-right: 0.15rem;
	}
	.contact div {
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
		padding: 0 .25rem;
		display: inline-block;
		position: relative;
	}
	
	.cont>span {
		float: left;
		margin-top: 0.12rem;
		margin-right: 0.15rem;
	}
	
	.contLeft {
		font-size: 0.26rem;
		width: 1.2rem;
		color: #CCCCCC;
		text-align: left;
		line-height: 0.26rem;
	}
	
	.contRight {
		font-size: 0.26rem;
		color: #666;
		line-height: 0.3rem;
	}
	
	.position {
		width: 4.7rem;
		text-align: left;
		min-height: 0.75rem;
	}
	/*右边去这里按钮*/
	
	.positionBtn {
		position: absolute;
		top: 2.5rem;
		right: 0.25rem;
		width: 0.75rem;
		height: 0.65rem;
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
		width: 7.06rem;
		padding: .22rem;
		background: #f5f5f9;
		display: inline-block;
	}
	
	.thingList li {
		width: 7rem;
		height: 1.6rem;
		position: relative;
	}
	
	.thingList a {
		width: 100%;
		height: 100%;
		display: block;
	}
	/*左边图片*/
	
	.thingList img {
		width: 1.6rem;
		height: 1.6rem;
		position: absolute;
		left: 0;
		top: 0;
	}
	/*右边内容*/
	
	.thingName {
		width: 5.5rem;
		height: 1.6rem;
		position: absolute;
		left: 1.82rem;
		top: 0;
		text-align: left;
	}
	/*商品名称*/
	
	.thingName div {
		width: 100%;
		height: 0.45rem;
		font-size: 0.26rem;
		line-height: 0.45rem;
		color: #666;
		padding-bottom: 0.3rem;
	}
	/*介绍*/
	
	.thingName p {
		width: 100%;
		height: 0.45rem;
		font-size: 0.24rem;
		line-height: 0.45rem;
		color: #cfcfcf;
		margin-top: .3rem;
	}
	/*价格*/
	
	.thingName span {
		width: 100%;
		height: 0.36rem;
		font-size: 0.26rem;
		line-height: 0.36rem;
		color: #ff5400;
	}
	/*数量*/
	
	.counts {
		position: absolute;
		right: 0.25rem;
		bottom: 0.1rem;
		font-size: 0.26rem;
		color: #666;
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
		font-size: 0.26rem;
		color: #666;
		line-height: 0.78rem;
	}
	
	.tolalLeft {
		font-size: 0.26rem;
		color: #666;
		line-height: 0.78rem;
	}
	
	.send {
		width: 7rem;
		height: 0.78rem;
		padding: 0 .25rem;
	}
	
	.send div {
		float: right;
		width: 1.4rem;
		height: 0.48rem;
		font-size: 0.26rem;
		color: #258ef3;
		line-height: 0.5rem;
		border-radius: 0.3rem;
		border: 1px solid #258ef3;
		margin-top: 0.14rem;
		margin-left: 0.2rem;
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
</style>