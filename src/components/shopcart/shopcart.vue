<template>
	<div>
	  <div class="shopcart">
	  	<div class='content' @click="toggleList">
	  		<div class="content-left">
	  			<div class="logo-wrapper">
	  				<div class="logo" :class="{'heightLight':totalCount>0}">
	  					<span class="icon-shopping_cart" :class="{'heightLight':totalCount>0}"></span>
	  				</div>
	  				<div class="num" v-show="totalCount>0">
	  					{{totalCount}}
	  				</div>
	  			</div>
	  			<div class="price" :class="{'heightLight':totalPrice>0}">￥{{totalPrice}}元</div>
	  			<div class="desc">另需配送费￥{{deliveryPrice}}元</div>
	  		</div>
	  		<div class="content-right" @click.stop.prevent="pay">
	  			<div class="pay" :class='payClass'>
	  				{{payDesc}}
	  			</div>
	  		</div>
	  	</div>
	  	<div class="ball-container">
	  		<div v-for="ball in balls">
						<!-- 过度钩子函数 -->
						<!--<transition name="drop" v-on:before-enter="beforeDrop" v-on:enter="dropping" v-on:after-enter="afterDrop">-->
				<transition v-on:before-enter="beforeDrop" v-on:enter="dropping" v-on:after-enter="afterDrop">
					<div class="ball" v-show="ball.show">
						<div class="inner inner-hook"></div>
					</div>
				</transition>
			</div>
	  	</div>
	  	<transition name='fold'>
		  	<div class="shopcart-list" v-show="listShow">
		  		<div class="list-header">
		  			<h1 class="title">购物车</h1>
		  			<span class="empty" @click='empty'>清空</span>
		  		</div>
		  		<div class="list-content" ref='listContent'>
		  			<ul>
		  				<li class="food" v-for='food in selectFoods'>
		  					<span class="name">{{food.name}}</span>
		  					<div class="price">
		  						<span>${{food.price*food.count}}</span>
		  					</div>
		  					<div class="cartcontrol-wrapper">
		  						<cartcontrol :food="food"></cartcontrol>
		  					</div>
		  				</li>
		  			</ul>
		  		</div>
		  	</div>
	  	</transition>
	  </div>
	  <transition name='fade'>
	  	<div class="list-mask" v-show='listShow' @click='hideList'></div>
	  </transition>
	</div>
</template>

<script>
import BScroll from 'better-scroll'
import cartcontrol from "../../components/cartcontrol/cartcontrol"
	export default{
		props:{
			selectFoods:{
				type:Array,
				default(){
					return []
				}
			},
			deliveryPrice:{
				type:Number,
				default:0
			},
			minPrice:{
				type:Number,
				default:0
			}
		},
		computed:{
			totalPrice(){
				let total=0;
				this.selectFoods.forEach((food)=>{
					total+=food.price*food.count;
				});
				return total;
			},
			totalCount(){
				let count=0;
				this.selectFoods.forEach((food)=>{
					count+=food.count;
				})
				return count;
			},
			payDesc(){
				if(this.totalPrice===0){
					return `￥${this.minPrice}元起送`
				}else if(this.totalPrice<this.minPrice){
					let diff=this.minPrice-this.totalPrice
					return `还差￥${diff}元起送`
				}else{
					return '去结算'
				}
			},
			payClass(){
				if(this.totalPrice<this.minPrice){
					return 'not-enough'
				}else{
					return 'enough'
				}
			},
			listShow(){
				if(!this.totalCount){
					this.fold=true;
					return false;
				}
				let show=!this.fold;
				if(show){
					this.$nextTick(()=>{
						if(!this.scroll){
								this.scroll=new BScroll(this.$refs.listContent,{
									click:true
								})
						}else{
							this.scroll.refresh()
						}
					})
				}
				return show;
			}
		},
		data(){
			return{
				balls:[
					{show:false},
					{show:false},
					{show:false},
					{show:false},
					{show:false}
				],
				dropBalls:[],
				fold:true
			}
		},
		methods:{
			drop(el){
				for(let i=0; i<this.balls.length; i++){
					let ball = this.balls[i];
					if(!ball.show){
						// 只有改变元素显示隐藏状态才会触发 过度钩子函数
						ball.show = true;
						ball.el = el;
						console.log(ball);
						this.dropBalls.push(ball);
						return;
					}
				}
			},
			beforeDrop(el) {
				let count = this.balls.length;
				while(count--){
					let ball = this.balls[count];
					if(ball.show){
					    //元素相对于视口的距离
						let rect = ball.el.getBoundingClientRect();

						// x,y 为初始点 与 目标点的差值
						let x = rect.left - 32;
						let y = -(window.innerHeight - rect.top -22);
						el.style.display = '';

						// el （初始位置为 0，0，0）和购物车icon在一起，将小球（el）  放到加号位置去
						//纵向动画
						el.style.webkitTransform = `translate3d(0,${y}px,0)`;
						el.style.transform = `translate3d(0,${y}px,0)`;

						// el.style.webkitTransform = `translate3d(0,${x}px,0)`;
						// el.style.transform = `translate3d(0,${x}px,0)`;

						//横向动画  inner-hook，  仅仅定义类 dom选择器，不做样式
						let inner = el.getElementsByClassName('inner-hook')[0];
						inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
						inner.style.transform = `translate3d(${x}px,0,0)`;
					}
				}
			},
			dropping(el,done) {
			    //手动触发浏览器重绘，便于translate3d，--rf 变量不会使用
				let rf = el.offsetHeight;

				this.$nextTick(() => {
				    //小球样式位置，置于购物车按钮位置处
					el.style.webkitTransform = 'translate3d(0,0,0)';
					el.style.transform = 'translate3d(0,0,0)';

					let inner = el.getElementsByClassName('inner-hook')[0];
					inner.style.webkitTransform = 'translate3d(0,0,0)';
					inner.style.transform = 'translate3d(0,0,0)';

					el.addEventListener("transitionend",done);
				})
			},
			afterDrop(el) {
			    //此轮动画结束后，将此次的 ball 取出 ，ball状态重置，，el display:none
				let ball = this.dropBalls.shift();
				if(ball){
					ball.show = false;
					el.style.display = "none";
				}
			},
			toggleList(){
				if(!this.totalCount){
					return;
				}
				this.fold=!this.fold;
			},
			empty(){
				this.selectFoods.forEach((food)=>{
					food.count=0;
				})
			},
			hideList(){
				this.fold=true;
			},
			pay(){
				if(this.totalPrice<this.minPrice){
					return
				}
				window.alert(`支付${this.totalPrice}元`)
			}
		},
		components:{
		    cartcontrol
		}
	}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang='less'>
@import '../../common/less/mixin.less';
@import '../../common/less/base.less';
	.shopcart{
		position: fixed;
		left: 0;
		bottom: 0;
		z-index: 50;
		width: 100%;
		height: 48px;
		.content{
			display: flex;
			background:#141d27;
			font-size: 0;
			height:48px;
			color:rgba(255,255,255,0.4);
			.content-left{
				flex:1;
				.logo-wrapper{
					display: inline-block;
					position: relative;
					top:-10px;
					margin:0 12px;
					padding: 6px;
					width: 56px;
					height: 56px;
					box-sizing: border-box;
					vertical-align: top;
					border-radius: 50%;
					background: #141d27;
					.logo{
						width: 100%;
						height: 100%;
						border-radius: 50%;
						background: #2b343c;
						text-align: center;
						&.heightLight{
							background: rgb(0,160,220);
						}
						.icon-shopping_cart{
							font-size: 24px;
							color:#88858a;
							line-height: 44px;
							&.heightLight{
								color:#fff;
							}
						}
					}
					.num{
						position: absolute;
						top:0;
						right: 0;
						width: 24px;
						height: 16px;
						line-height: 16px;
						text-align: center;
						font-size: 9px;
						font-weight: 700;
						color:#fff;
						background: rgb(240,20,20);
						border-radius: 16px;
						box-shadow: 0 4px 8px 0 rgba(0,0,0,0.4);
					}
				}
				.price{
					display: inline-block;
					vertical-align: top;
					line-height: 24px;
					margin-top:12px;
					box-sizing: border-box;
					padding-right: 12px;
					border-right: 1px solid rgba(255,255,255,0.1);
					font-size: 16px;
					font-weight: 700;
					&.heightLight{
						color:#fff;
					}
				}
				.desc{
					display:inline-block;
					vertical-align:top;
					line-height:24px;
					padding-left:12px;
					font-size:10px;
					margin-top:12px;
				}
			}
			.content-right{
				flex:0 0 105px;
				width:105px;
				.pay{
					height: 48px;
					line-height: 48px;
					text-align: center;
					font-size: 12px;
					font-weight: 700;
					&.not-enough{
						background: #2b333b;
					}
					&.enough{
						background: #00b43c;
						color:#fff;
					}
				}	
			}
		}
		.ball-container{
			.ball{
				position:fixed;
				left:32px;
				bottom:22px;
				z-index:200;
				//y 轴 贝塞尔曲线
				transition:all 0.5s cubic-bezier(0.49, -0.29, 0.75, 0.41);
				.inner{
					width:16px;
					height:16px;
					border-radius:50%;
					background-color:rgb(0,160,220);
					//x 轴只需要线性缓动
					transition:all 0.5s linear;
				}
			}
		}
		.shopcart-list{
			position: absolute;
			top:0;
			left: 0;
			right: 0;
			z-index: -1;
			width: 100%;
			transform:translate3d(0,-100%,0);
			&.fold-enter-active,&.fold-leave-active{
				transition :all 0.4s linear;
			}
			&.fold-enter,&.fold-leave-active{
				transform:translate3d(0,0,0);
			}
			.list-header {
				height: 40px;
				line-height: 40px;
				padding: 0 18px;
				background: #f3f5f7;
				border-bottom: 1px solid rgba(7, 17, 27, 0.1);
				.title {
					float: left;
					font-size: 14px;
					color: rgb(7, 17, 27);
				}
				.empty {
					float: right;
					font-size: 12px;
					color: rgb(0, 160, 220);
				}
			}
			.list-content{
				padding:0 18px;
				max-height :217px;
				overflow :hidden;
				background :#fff;
				.food{
					position :relative;
					padding :12px 0;
					box-sizing :border-box;
					.border-1px(rgba(7,17,27,0.1));
					.name {
						line-height: 24px;
						font-size: 14px;
						color: rgb(7, 17, 27);
					}
					.price {
						position: absolute;
						right: 90px;
						bottom: 12px;
						line-height: 24px;
						font-size: 14px;
						font-weight: 700;
						color: rgb(240, 20, 20);
					}
					.cartcontrol-wrapper {
						position: absolute;
						right: 0;
						bottom: 6px;
					}
				}

			}
		}
	}
	.list-mask{
		position: fixed;
		top:0;
		left: 0;
		width:100%;
		height: 100%;
		z-index: 40;
		background-filter:blur(10px); 
		background: rgba(7,17,27,0.6);
		&.fade-enter-active,&.fade-leave{
			transition :all 0.5s;
		}
		&.fade-enter,&.fade-leave-active{
			opacity :0;
			background-color :rgba(7,17,27,0);
		}
	}
</style>

