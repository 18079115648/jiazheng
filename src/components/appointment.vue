<template>
	<div id="box">
		<!--顶部-->
		<Header title="确认订单"></Header>
		<!--客户信息-->
		<a class="clientMessage" href="#/addresses/select">
			<!--姓名 电话-->
			<div class="clientCont clear" v-if="hasDefaultAddr">
				<div class="contName fl">联系人:{{defaultAddr.name}}</div>
				<div class="contMobile fr">{{defaultAddr.phone}}</div>
			</div>
			<!--地址-->
			<div class="clientPosition" v-if="hasDefaultAddr">
				<a  style="display:inline-block; position: relative; padding-left: 0.5rem;">
					<img class="imgLeft fl" src="../../static/37@3x.png"/>
					<span class="fl">{{defaultAddr.address}} {{defaultAddr.detail}}</span>
					<img class="imgRight fr" src="../../static/34@3x.png"/>
				</a>
			</div>
			<router-link v-if="!hasDefaultAddr" to="/addresses/select" style="line-height: 1.8rem; font-size: 0.32rem; color: #2d92f4; padding: 0.5rem 1rem;">添加服务地址</router-link>
			<!--底部彩条-->
			<div class="imgBottom"></div>
			
		</a>
		<!--服务时间-->
		<div class="serveTime">
			<span class="fl">服务时间</span>
			<a @click="showTime = true">
				<img src="../../static/34@3x.png"/>
				<div class="fr" v-show="serveDataSelect.length<1">请选择服务时间</div>
				<div class="fr" style="color: #000;" v-show="serveDataSelect.length == 2">{{serveData}}</div>
			</a>
		</div>
		<!--选择服务数量-->
		<div class="serveCount">
			<div class="cont">
				<!--左边介绍-->
				<div class="serveLeft">
					<div class="serveBack fl">
						<img :src="appointmentData.picture"/>
					</div>
					<div class="serveTitle fl">
						<div>{{appointmentData.title}}</div>
						<!--<p v-html="appointmentData.content"></p>-->
						<span>&yen;{{appointmentData.price}}</span>
					</div>
				</div>
				<!--右边选择数量-->
				<div class="serveRight fr" v-if="appointmentData.type.value == 1 && appointmentData.priceType == 1">
					<div class="serveBtn">
						<div class="btnLeft fl" @click="reduce" >-</div>
						<span>{{score}}</span>
						<div class="btnRight fr" @click="add" >+</div>
					</div>
				</div>
			</div>
			<!--优惠券-->
			<div class="serveBottom" v-if="appointmentData.type.value == 1">
				<span class="fl">优惠券</span>
				<a @click="selectCoupon">
					<img class="fr" src="../../static/34@3x.png"/>
					<div class="fr" v-show="useCouponStatus">{{parseInt(currCoupon.price)}}元优惠券</div>
					<div class="fr" v-show="!useCouponStatus">使用优惠券</div>
				</a>
			</div>
		</div>
		<!--结账-->
		<div class="footPart" v-if="appointmentData.type.value == 1">
			<div class="partPrice">
				<div class="fl">服务金额</div>
				<span class="fr">&yen;{{(parseFloat(appointmentData.price) * score).toFixed(2)}}</span>
			</div>
			<div class="partPrice">
				<div class="fl">优惠金额</div>
				<span class="fr">- &yen;{{currCoupon.price}}</span>
			</div>
			<div class="pay">
				<div class="fl">实付金额</div>
				<span class="fr">&yen;{{(((parseFloat(appointmentData.price) * score) - currCoupon.price).toFixed(2)) > 0 ? ((parseFloat(appointmentData.price) * score) - currCoupon.price).toFixed(2) : '0.00'}}</span>
			</div>
		</div>
		<!--提交预约-->
		<div class="bottomBtn">
			<a @click="serviceAddOrder" v-show="!disabledBtn">提交预约</a>
			<a class="disable"  v-show="disabledBtn">提交预约</a>
		</div>
		<mt-popup v-model="showTime" class="service-time" position="bottom">
			<div class="showTime-content">
				<div class="showTime-header">
					<span @click="showTime = false">取消</span>
					<span @click="serveDataConfirm">确定</span>
				</div>
				<div class="showTime-body">
					<mt-picker 
						:slots="dateSlots" 
						:itemHeight="32" 
						:value-key="'name'"
						@change="onDataChange">
					</mt-picker>
				</div>
			</div>
		</mt-popup>
	</div>
</template>

<script type="text/javascript">
	import { Toast } from 'mint-ui'
	export default {
		data(){
			return {
				score:1,
				defaultAddr: {},
				hasDefaultAddr: true,
				showTime: false,
				dateSlots: [
		          {
		            flex: 1,
		            values: [],
		            className: 'slot1',
		            textAlign: 'center'
		          }, {
		            divider: true,
		            content: '-',
		            className: 'slot2'
		          }, {
		            flex: 1,
		            values: [],
		            className: 'slot3',
		            textAlign: 'center',
		            defaultIndex: 0
		          }
		        ],
		        serveDataChange: [],
		        serveDataSelect: [],
		        serveData: '',
		        
		        currCoupon: {
		        	price: 0
		        },
		        useCouponStatus: false,
		        
		        appointmentData: {
		        	type: {}
		        },
		        serviceId: undefined,
		        
		        dateObj: [],
		        
		        disabledBtn: false
				
			}
		},
		created() {
//			this.serviceId = this.$route.params.id
		},
		activated() {
			this.serviceId = this.$route.params.id
			this.disabledBtn = false
//			this.$api.serveConfirmOrder(null, (res) => {
//		    	this.defaultAddr = res.result.defaultAddr
//		    	this.hasDefaultAddr = this.$isEmptyObject(res.result.defaultAddr)
//		    })
		},
		beforeRouteEnter (to, from, next) {
	    	if(/orders/g.test(from.fullPath) || /paySubmit/g.test(from.fullPath) || /coupons/g.test(from.fullPath) || /addresses/g.test(from.fullPath)) {
	    		next(vm=>{
	    			vm.useCouponStatus = false
	    			vm.currCoupon = {
				        	price: 0
				        }
	    			if(vm.$storage.get('currCoupon')) {
						vm.currCoupon = vm.$storage.get('currCoupon')
						vm.useCouponStatus = true
					}
	    			vm.$api.serveConfirmOrder(null, (res) => {
				    	vm.defaultAddr = res.result.defaultAddr
				    	vm.hasDefaultAddr = vm.$isEmptyObject(res.result.defaultAddr)
				    })
	    		})
	    	}else {
	    		next(vm=>{
	    			vm.$storage.remove('currCoupon')
	    			vm.score = 1
	    			vm.showTime = false
	    			vm.dateSlots[0].values = []
	    			vm.dateSlots[2].values = []
	    			vm.serveDataChange =  []
			        vm.serveDataSelect = []
			        vm.serveData = ''
			        vm.useCouponStatus = false
					vm.appointmentData = {
			        	type: {}
			        }
					
					vm.currCoupon = {
				        	price: 0
				        }
			        vm.initData()
			        
	        	})
	    	}
	    	
	    	
		},
		methods:{
			initData() {
				this.appointmentData = this.$storage.get('appointmentData')
				this.$api.serveConfirmOrder(null, (res) => {
			    	this.defaultAddr = res.result.defaultAddr
			    	this.hasDefaultAddr = this.$isEmptyObject(res.result.defaultAddr)
//			    	this.dateObj[0] = new Array()
//			    	res.result.intervals.forEach((item) => {
//			    		item.name = item.interval
//			    		if(item.valid == 1) {
//			    			this.dateObj[0].push(item)
//			    		}
//			    	})
			    	res.result.nextTenDays.forEach((item, index) => {
			    		item.index = index
			    	})
			    	this.dateObj = res.result.nextTenDays
			    	this.dateSlots[0].values = res.result.nextTenDays
			    })
			},
			onDataChange(picker, values) {
				console.log(values)
				let self = this
				let index = 0
				if(values[0]) {
					index = values[0].index
					picker.setSlotValues(1, self.dateObj[index].intervals)
					this.serveDataChange = values
				}
				
				
			},
			serveDataConfirm() {
				this.serveDataSelect = this.serveDataChange.concat()
				console.log(this.serveDataSelect)
				this.serveData = this.serveDataSelect[0].name.slice(0,6) + ' ' + this.serveDataSelect[1].name
				this.showTime = false
			},
			add:function(){
				this.score++; 
			},
			reduce:function(){
				if(this.score==1){
					return false;
				}
				this.score--;
			},
			//选择优惠券
			selectCoupon() {
				this.$storage.set('currServeCoupon', {
					serviceId: this.serviceId,
					serviceNum: this.score
				})
				this.$router.push('/coupons/couponsLeft/use')
			},
			serviceAddOrder() {
				if(!this.hasDefaultAddr) {
					Toast({
	                    message: '请选择服务地址',
	                    position: 'bottom',
	                    duration: 1000
	                })
					return
				}
				if(this.serveDataSelect.length !== 2) {
					Toast({
	                    message: '请选择服务时间',
	                    position: 'bottom',
	                    duration: 2000
	                })
					return
				}
				this.disabledBtn = true
				this.$api.serveAddOrder({
					addressId: this.defaultAddr.id,
					contactName: this.defaultAddr.name,
					contactPhone: this.defaultAddr.phone,
					couponId: this.currCoupon.id,
					detailAddr: this.defaultAddr.address,
					serviceDate: this.serveDataSelect[0].timestamp,
					serviceId: this.serviceId,
					serviceMount: this.score,
					timeInterval: this.serveDataSelect[1].name,
					timeIntervalId: this.serveDataSelect[1].id
				}, (res) => {
					this.$storage.remove('currCoupon')
					this.$storage.remove('appointmentData')
					if(this.appointmentData.type.value == 1) {
						this.$router.replace('/paySubmit/' + res.result.orderSn)
					}else {
						Toast({
						  message: '预约成功',
						  position: 'middle',
						  iconClass: 'toast-icon icon-success',
						  duration: 800
						})  
						setTimeout(() => {
							this.$router.replace('/orders')
						},500)					
					}
				})
			}
		}
	}
</script>

<style scoped>
	#box {
		width: 100%;
		height: 100%;
		padding: 0px;
		margin: 0px;
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
	.headCont span{
		color: #FFFFFF;
		width: 0.32rem;
		height: 0.32rem;
		background: url("../../static/return.png");
		background-size: 100% 100%;
	}
	.headCont p{
		height: 0.32rem;
		color: #FFFFFF;
		font-size: 0.32rem;
	}
	/*客户信息*/
	.clientMessage{
		width: 100%;
		background: #FFFFFF;
		display: inline-block;
		margin-bottom: 0.2rem;
		height: 2rem;
		position: relative;
	}
	/*客户姓名和电话部分*/
	.clientCont{
		width: 7.5rem;
		height: 0.25rem;
		font-size: 0.25rem;
		color: #222222;
		margin-top: 0.46rem;
		margin-bottom: 0.22rem;
	}
	.contName{
		width: 3rem;
		height: 0.25rem;
		margin-left: 0.75rem;
		text-align: left;
	}
	.contMobile{
		width: 2rem;
		height: 0.25rem;
		text-align: right;
		margin-right:0.6rem ;
	}
	/*地址   */
	.clientPosition{
		width:7rem ;
		display:inline-block;
		margin-left:0.24rem ;
		margin-bottom:0.3rem;
		position: relative;
	}
	.clientPosition span{
		width:6rem ;
		font-size:0.25rem ;
		line-height:0.3rem ;
		color:#333333 ;
		text-align: left;
		margin-top: 0.1rem;
	}
	.imgLeft{
		width:0.28rem ;
		height: 0.35rem;
		margin: 0.2rem 0.25rem  0 0;
		position: absolute;
		left: 0;
		top: 50%;
		margin-top: -0.17rem;
		
	}
	.imgRight{
		width: 0.13rem;
		height:0.2rem ;
		margin-top:0.12rem ;
	}
	/*底部彩条*/
	.imgBottom{
		width: 100%;
		height: 0.06rem;
		background: url("../../static/45@3x.png") no-repeat;
		background-size: 100% 100%;
		position: absolute;
		left: 0;
		bottom: 0;
	}
	/*选择时间*/
	.serveTime{
		width:7rem ;
		height:0.9rem ;
		padding: 0 .25rem;
		margin-bottom: 0.2rem;
		background: #FFFFFF;
		display: inline-block;
	}
	.serveTime a{
		display: inline-block;
		float: right;
	}
	.serveTime span{
		width:1.2rem ;
		height: 0.9rem;
		padding-left: 0.07rem;
		font-size: 0.24rem;
		line-height: 0.9rem;
		text-align: left;
		color: #4E4E4E;
		
	}
	.serveTime img{
		width:0.12rem;
		height:0.22rem;
		margin-top:0.34rem ;
		float: right;
	}
	.serveTime div{
		width: 4rem;
		height: 100%;
		text-align: right;
		font-size: 0.26rem;
		line-height: 0.9rem;
		color: #d6d6d6;
		margin-right: 0.28rem;
	}
	
	/*选择服务数量*/
	.serveCount{
		width: 7rem;
		display: inline-block;
		padding: 0 .25rem;
		background: #FFFFFF;
		overflow: hidden;
		margin-bottom: 0.2rem;
	}
	.cont{
		width: 100%;
		display: inline-block;
		padding-bottom: 0.2rem;
	}
	/*左边部分*/
	.serveLeft{
		display: inline-block;
		float: left;
	}
	/*服务图片*/
	.serveBack{
		width:1.4rem ;
		height:1.4rem ;
		margin-top: 0.2rem;
	}
	.serveBack img{
		width: 100%;
		height: 100%;
	}
	.serveTitle{
		width:3rem ;
		height: 1.4rem;
		margin: .2rem 0 0 .25rem;
		text-align: left;
		
	}
	.serveTitle div{
		width: 3rem;
		height: 0.45rem;
		font-size:0.26rem ;
		line-height:0.45rem ;
		color:#4e4e4e ;
		margin-bottom: 0.1rem;
		
	}
	.serveTitle p{
		width:3rem ;
		height:0.5rem ;
		font-size: 0.24rem;	
		line-height:0.5rem ;
		color: #adadad;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
	.serveTitle span{
		width: 3rem;
		height: 0.26rem;
		font-size: 0.26rem;
		color:#ff5400 ;
	}
	/*右边*/
	.serveRight{
		width: 2rem;
		height:1.4rem;
	}
	.serveBtn{
		width: 2rem;
		height: 0.6rem;
		margin-top: 1rem;
	}
	.serveBtn div{
		width: 0.6rem;
		height: 0.6rem;
		background: #efefef;
		font-size:0.25rem ;
		line-height: 0.6rem;
	}
	.btnLeft{
		color:#aaaaaa;
	}
	.btnRight{
		color:#666666;
	}
	.serveBtn span{
		width: 0.7rem;
		height: 0.6rem;
		background:#efefef ;
		display: inline-block;
		font-size: 0.3rem;
		color: #333333;
		line-height:0.6rem ;
	}
	/*优惠券*/
	.serveBottom{
		width: 7rem;
		border-top: 2px solid #f2f2f2;
		height: 0.88rem;
		display: block;
		background: #FFFFFF;
	}
	.serveBottom span{
		width:1rem;
		height:0.88rem;
		color:#222222 ;
		font-size: 0.26rem;
		text-align: left;
		line-height: 0.88rem;
	}
	.serveBottom a{
		width: 3.5rem;
		height: 0.88rem;
		display: block;
		float: right;
	}
	.serveBottom img{
		width: 0.12rem;
		height:0.22rem ;
		margin-top:0.33rem ;
	}
	.serveBottom div{
		width:2rem ;
		height: 0.88rem;
		text-align: right;
		color: #222222;
		font-size: 0.26rem;
		line-height: 0.88rem;
		margin-right: 0.2rem;
	}
	/*金额*/
	.footPart{
		width: 7rem;
		padding: 0 0.25rem;
		background: #FFFFFF;
		display: inline-block;
		
	}
	.partPrice{
		width: 7rem;
		height: 0.24rem;
		margin-top: 0.28rem;
		display: block;	
	}
	.partPrice div{
		width:1.5rem;
		height: 0.24rem;
		font-size: 0.24rem;
		text-align: left;
		color:#adadad ;
	}
	.partPrice span{
		width:1.5rem;
		height: 0.24rem;
		font-size: 0.24rem;
		text-align: right;
		color:#adadad ;
	}
	/*支付金额*/
	.pay{
		width: 7rem;
		height: 0.88rem;
		border-top: 2px solid #f2f2f2;
		margin-top: 0.28rem;
	}
	.pay div{
		width:1.5rem;
		height: 0.26rem;
		font-size: 0.26rem;
		text-align: left;
		color:#222222 ;
		line-height: 0.88rem;
	}
	.pay span{
		width:1.5rem;
		height: 0.26rem;
		font-size: 0.26rem;
		text-align: right;
		color:#ff8244 ;
		line-height: 0.88rem;
	}
	/*提交预约*/
	.bottomBtn{
		width: 6.9rem;
		height: 0.8rem;
		background: #2f94f4;
		border-radius: 0.5rem;
		margin: 0.7rem 0.3rem 0.28rem;
		overflow: hidden;
	}
	.bottomBtn a{
		display: block;
		width: 100%;
		height: 100%;
		font-size: 0.3rem;
		line-height: 0.8rem;
		color: #FFFFFF;
	}
	.bottomBtn a.disable{
		background: #ddd;
		color: #aaa;
	}

</style>