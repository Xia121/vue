<template>
	<div class="goods">
		<div class="menu-wrapper" ref="menu-wrapper">
			<ul>
				<li v-for="(item,index) in goods" class="menu-item border-1px" :class="{'current':currentIndex===index}" @click="selectMenu(index, $event)">
					<span class="text">
						<span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
						{{item.name}}
					</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" ref="foods-wrapper">
			<ul>
				<li v-for="item in goods" class="foods-list foods-list-hook">
					<h1 class="title">{{item.name}}</h1>
					<ul>
						<li @click="selectFood(food, $event)" v-for="food in item.foods" class="food-item border-1px">
							<div class="icon">
								<img width="57" height="57" :src="food.icon">
							</div>
							<div class="content">
								<h2 class="name">{{food.name}}</h2>
								<p class="desc">{{food.description}}</p>
								<div class="extra">
									<span>月售{{food.sellCount}}份</span>
									<span>好评率{{food.rating}}%</span>
								</div>
								<div class="price">
									<span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice"> ￥{{food.oldPrice}}</span>
								</div>
								<div class="cartcontrol-wrapper">
									<cartcontrol @add="addFood" :food="food"></cartcontrol>
								</div>
							</div>
						</li>
					</ul>
				</li>
			</ul>
		</div>
		<shopcart ref="shopcart" :selectFoods="selectFoods" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice"></shopcart>

		<food @add="addFood" :food="selectedFood" ref="food"></food>
	</div>

	
</template>

<script typr="text/ecmascript-6">
	import BScroll from 'better-scroll'
	import shopcart from '../shopcart/shopcart'
	import cartcontrol from '../cartcontrol/cartcontrol'
	import food from '../food/food'

	const ERR_OK = 0;
	const debug = process.env.NODE_ENV !== 'production';

	export default{
		props: {
			seller: {
				type: Object
			}
		},

		data () {
			return {
				goods: [],
				listHeight: [],
				scrollY: 0,
				selectedFood: {}
			}
		},

		created () {
			this.classMap = ['decrease','discount','guarantee','invoice','special'];

			const url = debug ? '/api/goods' : 'http://ustbhuangyi.com/sell/api/goods';

			this.$http.get(url).then( (response) => {

				response = response.body;

				if(response.errno == ERR_OK){
					this.goods = response.data;

					this.$nextTick( () => {
						this._initScroll();
						this._calculateHeight();
					});	
				}
			})
		},

		methods: {
			selectFood (food, event) {
				if(!event._constructed){
					return;
				}
				//console.log(food)

				this.selectedFood = food;
				this.$refs.food.show();
			},

			selectMenu (index, event) {
				if(!event._constructed){
					return;
				}

				let foodList = this.$refs['foods-wrapper'].getElementsByClassName('foods-list-hook');

				let ref = foodList[index];

				this.foodsScroll.scrollToElement(ref, 300)
			},

			addFood (target) {
				this._drop(target)
			},

			_drop (target) {
				// 体验优化，异步执行
				this.$nextTick(() => {
					this.$refs['shopcart'].drop(target)
				})
			},

			_initScroll () {
				this.menuScroll = new BScroll(this.$refs['menu-wrapper'], {
					click: true
				})

				this.foodsScroll = new BScroll(this.$refs['foods-wrapper'], {
					probeType: 3,
					click: true
				});

				this.foodsScroll.on('scroll', (pos) => {
					this.scrollY =Math.abs(Math.round(pos.y));
				})
			},

			_calculateHeight () {
				let foodList = this.$refs['foods-wrapper'].getElementsByClassName('foods-list-hook');

				let height = 0;

				this.listHeight.push(height);

				for(let i=0; i<foodList.length; i++) {
					let item = foodList[i];
					height += item.clientHeight;
					this.listHeight.push(height);
				}

			}
		},

		computed: {
			currentIndex() {

				for(let i=0; i<this.listHeight.length; i++) {
					let height1 = this.listHeight[i];
					let height2 = this.listHeight[i+1];
					if(!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
						return i;
					}
				}

				return 0;
			},

			selectFoods() {
				let foods = [];
				this.goods.forEach((good) => {
					good.foods.forEach((food) => {
						if(food.count) {
							foods.push(food);
						}
					});
				});
				//console.log(this.goods[0].foods)
				//console.log(foods)
				return foods
			}
		},

		components: {
			shopcart,
			cartcontrol,
			food
		}
	}
</script>

<style lang="stylus" rel="stylesheet/stylus">
	@import "../../common/stylus/mixin.styl"
	.goods
		position: absolute
		width: 100%
		top: 174px
		bottom: 46px
		display: flex
		overflow: hidden
		.menu-wrapper
			flex: 0 0 80px
			width: 80px
			background: #f3f5f7	
			.menu-item
				display: flex
				align-items: center
				height: 54px
				width: 56px
				line-height: 14px
				padding: 0 12px
				&.current
					position: relative 
					margin-top: -1px
					z-index: 10
					background: #fff;
					font-weight: 700;
					.text
						border-no()
				border-1px(rgba(7, 17, 27, 0.1))
				.icon
					display: inline-block
					width: 12px
					height: 12px
					margin-right: 2px
					vertical-align: middle
					background-size: 12px 12px
					background-repeat: no-repeat
					&.decrease
						bg-image('decrease_3')		
					&.discount
						bg-image('discount_3')
					&.guarantee
						bg-image('guarantee_3')
					&.invoice
						bg-image('invoice_3')
					&.special
						bg-image('special_3')
				.text 
					font-size: 12px
					display: inline-block
					width: 56px
					vertical-align: middle
		.foods-wrapper
			flex: 1
			.title
				padding-left: 14px
				height: 26px
				line-height: 26px
				border-left: 2px solid rgba(7, 17, 27, 0.1)
				font-size: 12px
				color: rgb(147, 153, 159)
				background: #f3f5f7
			.food-item
				display: flex
				margin: 18px
				padding: 18px 0 0 0
				border-1px(rgba(7, 17, 27, 0.1))
				.icon
					flex: 0 0 57px
					margin-right: 10px
				.content
					flex: 1	
					.name
						margin: 2px 0 8px 0
						height: 14px
						line-height: 14px
						font-size: 14px
						color: rgb(7, 17, 27)
					.desc, .extra
						line-height: 10px
						font-size: 12px
						color: rgb(147, 153, 159)
					.desc
						margin-bottom: 8px
					.extra
						.count
							margin-right: 12px
					.price
						font-weight: 700
						line-height: 24px
						.now
							margin-right: 18px
							font-size: 14px
							color: rgb(240, 20, 20)
						.old
							text-decoration: line-through
							font-size: 10px
							color: rgb(147, 153, 159)
					.cartcontrol-wrapper
						position: absolute
						right: 0
						bottom: -12px
				&:first-child
					border-none()
					padding-top: 0
							
</style>