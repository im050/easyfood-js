<template>
	<view class="container">
		<view class="page-body">
			<scroll-view class="nav-left" scroll-y :style="'height:'+height+'px'">
				<view @click="scrollToGoods(index)" class="nav-left-item" :key="index" :class="item.active ?'active':''" v-for="(item,index) in categories">
					{{item.name}}
				</view>
			</scroll-view>
			<scroll-view :scroll-into-view="goodsViewId" class="nav-right" scroll-y @scroll="scroll" :style="'height:'+height+'px'"
			 scroll-with-animation>
				<view class="goods-box" :class="categories[index].active ? 'fixed' : ''" :id="'cat-' + index" :key="index" v-for="(item,index) in categories">
					<h1 class="cat-title" :class="categories[index].active ? 'fixed' : ''">{{ item.name }}<span class="desc">{{ item.desc }}</span></h1>
					<view>
						<view class="nav-right-item" :key="goodsIndex" v-for="(good, goodsIndex) in item.goods">
							<image class="goods-pic" :src="good.pic" />
							<view class="goods-info">
								<view class="goods-name">{{ good.name }}</view>
								<view class="goods-desc">{{ good.desc }}</view>
								<view class="goods-price"><span>￥</span>{{ good.realPrice }}</view>
							</view>
							<view class="stock-panel">
								<view class="reduct-cart" v-if="cart[index][goodsIndex] > 0" @click="reductCart(index, goodsIndex)">
									<i class="iconfont">&#xe6c4;</i>
								</view>
								<view class="cart-number" v-if="cart[index][goodsIndex] > 0">
									{{ cart[index][goodsIndex] }}
								</view>
								<view class="add-cart" @click="addCart(index, goodsIndex)">
									<i class="iconfont">&#xe600;</i>
								</view>
							</view>
						</view>
					</view>
				</view>
			</scroll-view>
			<!-- 底部购物栏 -->
			<view class="cart" :class="isTopCart ? 'top-layer' : ''">
				<!-- 购物车按钮 -->
				<view class="iconfont cart-icon" @click="showCart" :class="[cartIconAnimation]">
					<span>&#xe601;</span>
					<view class="cart-badge" v-if="cartTotalCount > 0">
						<span>{{ cartTotalCount }}</span>
					</view>
				</view>
				<!-- 总价信息 -->
				<view class="price-info" :class="cartTotalCount > 0 ? 'active' : ''">
					<template v-if="cartTotalCount <= 0">未选购商品</template>
					<template v-else>￥{{ totalPrice }}</template>
				</view>
				<!-- 下单按钮 -->
				<view class="buy" :class="cartTotalCount > 0 ? 'active' : ''">{{ buyText }}</view>
			</view>
		</view>
		<!-- 购物车popup -->
		<popup-layer ref="popupCart" @closeCallBack="closeCartCb" :offset="cartNavHeight" direction="top" v-model="showMoreChoice">
			<view class="cart-list">
				<view class="cart-panel">
					<span class="cart-panel-text">已选商品</span>
					<span class="cart-clear" @click="clearCart"><i class="iconfont">&#xe604;</i>清空</span>
				</view>
				<view class="cart-goods-box">
					<view class="cart-goods" v-for="(goods, index) in cartGoods" :key="index">
						<view class="cart-goods-info">
							<em>{{ getGoods(goods.catIndex, goods.goodsIndex).name }}</em>
							<p>
								{{ goods.attrDesc }}
							</p>
						</view>
						<view class="cart-goods-price">
							<em><span>￥</span>{{ goods.price }}</em>
						</view>
						<view class="stock-panel">
							<view class="reduct-cart" @click="reductCartWithGoods(index)" v-if="goods.count > 0">
								<i class="iconfont">&#xe6c4;</i>
							</view>
							<view class="cart-number" v-if="goods.count > 0">
								{{ goods.count }}
							</view>
							<view class="add-cart" @click="addCartWithGoods(index)">
								<i class="iconfont">&#xe600;</i>
							</view>
						</view>
					</view>
				</view>
			</view>
		</popup-layer>
		<!-- 商品更多选项popup -->
		<popup-layer ref="popupRef" direction="top" v-model="showMoreChoice">
			<view class="more-choice">
				<view class="more-choice-box">
					<view class="box-goods-info">
						<image class="goods-pic" :src="categories[current.catIndex].goods[current.goodsIndex].pic" />
						<view class="goods-info">
							<view class="goods-name">{{ categories[current.catIndex].goods[current.goodsIndex].name }}</view>
							<view class="goods-desc">{{ categories[current.catIndex].goods[current.goodsIndex].desc }}</view>
							<view class="goods-price"><span>￥</span>{{ calcGoodsPrice }}</view>
						</view>
					</view>
					<view class="attr" v-for="(attr, index) in categories[current.catIndex].goods[current.goodsIndex].attrs" :key="index">
						<view class="attr-group">{{ attr.name }}</view>
						<view class="sub-attr-box">
							<span class="sub-attr" @click="choiceAttr(index, attrIndex)" :class="current.attrs[index] && current.attrs[index].attrs[attrIndex].select ? 'active' : ''"
							 v-for="(subAttr, attrIndex) in attr.attrs" :key="attrIndex">
								{{ subAttr.name }}
							</span>
						</view>
					</view>
					<button type="primary" class="bottom" @tap="choiceDone()">选好了</button>
				</view>
			</view>
		</popup-layer>
	</view>
</template>

<script>
	let categoryObj = {
		name: "",
		active: "",
		id: "",
		goods: [],
		desc: "描述啊这个是"
	}
	let goodObj = {
		name: "",
		price: "",
		unit: "杯",
		pic: "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1559107295874&di=5d76a4ca1d1c0b44f84d9c80f0ad4be1&imgtype=0&src=http%3A%2F%2Fimages.3158.cn%2Fdata%2Fattachment%2Ftougao%2Farticle%2F2019%2F04%2F27%2F7693a49192b52b83e5a88d13042a186c.jpg",
		realPrice: "99.00",
		desc: "这个是描述啊",
		attrs: [{
			name: "规格",
			multiple: false,
			required: true,
			attrs: [{
				id: 4,
				name: "中杯",
				markup: 0.00,
				select: false
			}, {
				id: 5,
				name: "大杯",
				markup: 3.00,
				select: false
			}]
		}, {
			name: "口味",
			multiple: false,
			required: true,
			attrs: [{
				id: 1,
				name: "去冰",
				markup: 0,
				select: false
			}, {
				id: 2,
				name: "少冰",
				markup: 0.00,
				select: false
			}, {
				id: 3,
				name: "正常冰",
				markup: 0.00,
				select: false
			}]
		}, {
			name: "加料（可多选）",
			multiple: true,
			required: false,
			attrs: [{
				id: 1,
				name: "珍珠",
				markup: 2,
				select: false
			}, {
				id: 2,
				name: "椰果",
				markup: 3.00,
				select: false
			}, {
				id: 3,
				name: "奶盖",
				markup: 5.00,
				select: false
			}]
		}]
	}

	import popupLayer from '@/components/popup-layer/popup-layer.vue';
	import {
		cloneObject
	} from "@/common/utils.js";

	export default {
		components: {
			popupLayer
		},
		data() {
			return {
				goodsViewId: 'cat-0',
				goodsScrollPos: 0,
				categories: [],
				lastActiveIndex: 0,
				height: 0,
				goodsHeight: [],
				initFinish: false,
				buyText: "选好了",
				cartIconAnimation: "",
				totalPrice: 0,
				cart: [], //用来存放商品索引，统计商品数量
				cartGoods: [], //存放选购的商品信息
				cartTotalCount: 0, //冗余商品总数
				showMoreChoice: false,
				current: {
					isDone: false,
					totalPrice: 0,
					catIndex: 0,
					goodsIndex: 0,
					attrs: []
				},
				isTopCart: false
			}
		},
		computed: {
			calcGoodsPrice() {
				let goodsPrice = this.calcGoodsPriceMethod();
				return goodsPrice;
			},
			cartNavHeight() {
				return uni.upx2px(90);
			}
		},
		methods: {
			//计算价格
			calcGoodsPriceMethod() {
				let goods = this.getCurrentGoods();
				let goodsPrice = goods.realPrice * 1;
				for (let i in this.current.attrs) {
					for (let j in this.current.attrs[i].attrs) {
						let attr = this.current.attrs[i].attrs[j];
						if (attr.select) {
							goodsPrice += attr.markup * 1;
						}
					}
				}
				return goodsPrice;
			},
			//显示商品的更多选项
			activeMoreChoice() {
				this.$refs.popupRef.show();
			},
			//获取当前正在购买的商品
			getCurrentGoods() {
				return this.getGoods(this.current.catIndex, this.current.goodsIndex);
			},
			//根据索引获取商品
			getGoods(catIndex, goodsIndex) {
				return this.categories[catIndex].goods[goodsIndex];
			},
			//选择商品属性
			choiceAttr(attrIndex, subAttrIndex) {
				if (this.current.attrs[attrIndex].multiple) {
					if (this.current.attrs[attrIndex].attrs[subAttrIndex].select == true) {
						this.current.attrs[attrIndex].attrs[subAttrIndex].select = false;
					} else {
						this.current.attrs[attrIndex].attrs[subAttrIndex].select = true;
					}
				} else {
					for (let i in this.current.attrs[attrIndex].attrs) {
						this.current.attrs[attrIndex].attrs[i].select = false;
					}
					this.current.attrs[attrIndex].attrs[subAttrIndex].select = true;
				}
			},
			initFirstChoice() {
				for (let i in this.current.attrs) {
					if (this.current.attrs[i].required) {
						for (let j in this.current.attrs[i].attrs) {
							this.current.attrs[i].attrs[j].select = false;
						}
						this.current.attrs[i].attrs[0].select = true;
					}
				}
			},
			showCart() {
				if (this.cartTotalCount <= 0) {
					return;
				}
				this.isTopCart = true;
				this.$refs.popupCart.show();
			},
			closeCartCb() {
				setTimeout(() => {
					this.isTopCart = false;
				}, 500);
			},
			addCart(catIndex, goodsIndex) {
				this.current.catIndex = catIndex;
				this.current.goodsIndex = goodsIndex;
				this.current.attrs = JSON.parse(JSON.stringify(this.categories[catIndex].goods[goodsIndex].attrs));
				if (this.hasAttrs(catIndex, goodsIndex)) {
					this.initFirstChoice();
					this.activeMoreChoice();
					return;
				}
				this.pushCurrentToCart();
			},
			pushCurrentToCart() {
				this.cartIconAnimation = "shake active";
				let _this = this;
				setTimeout(function() {
					_this.cartIconAnimation = "active";
				}, 500);
				this.cart[this.current.catIndex][this.current.goodsIndex]++;
				this.cartTotalCount++;
				let realPrice = this.calcGoodsPriceMethod();
				let realGoods = cloneObject(this.current);
				realGoods.price = realPrice;
				realGoods.count = 1;
				realGoods.attrDesc = this.extraAttrDesc(realGoods);
				let existsIndex = this.getIndexInCart(realGoods);
				if (existsIndex != undefined) {
					this.cartGoods[existsIndex].count++;
					this.cartGoods[existsIndex].price += realPrice;
				} else {
					this.cartGoods.push(realGoods);
				}
				this.totalPrice += realPrice;
			},
			extraAttrDesc(realGoods) {
				let desc = '';
				for (let i in realGoods.attrs) {
					for (let j in realGoods.attrs[i].attrs) {
						let attr = realGoods.attrs[i].attrs[j];
						if (attr.select) {
							desc += `${attr.name} / `;
						}
					}
				}
				if (desc == '') {
					return this.categories[realGoods.catIndex].goods[realGoods.goodsIndex].unit;
				} else {
					return desc.substr(0, desc.length - 2);
				}
			},
			//检查是否已经存在购物车
			getIndexInCart(realGoods) {
				for (let i in this.cartGoods) {
					let goods = this.cartGoods[i];
					if (goods.catIndex == realGoods.catIndex && goods.goodsIndex == realGoods.goodsIndex) {
						if (JSON.stringify(goods.attrs) == JSON.stringify(realGoods.attrs)) {
							return i;
						}
					}
				}
				return undefined;
			},
			/**
			 * 多选
			 */
			choiceDone() {
				this.pushCurrentToCart();
				this.close();
			},
			hasAttrs(catIndex, goodsIndex) {
				let goods = this.categories[catIndex].goods[goodsIndex];
				if (goods.attrs.length > 0) {
					return true;
				}
				return false;
			},
			reductCartNumber(catIndex, goodsIndex) {
				this.cart[catIndex][goodsIndex]--;
				this.cartTotalCount--;
				if (this.cartTotalCount <= 0) {
					this.cartIconAnimation = "";
				}
			},
			reductCart(catIndex, goodsIndex) {
				let res = this.checkDifferentExists(catIndex, goodsIndex);
				if (res.isExists) {
					uni.showModal({
						title: "提示",
						content: "存在不同规格的商品，请进入购物车中移除。",
						showCancel: false,
						success: (res) => {
							if (res.confirm) {
								this.showCart();
							}
						}
					});
					return;
				}
				this.reductCartWithGoods(res.firstIndex);
			},
			//检查是否同商品不有同规格的存在
			checkDifferentExists(catIndex, goodsIndex) {
				let flag = 0;
				let res = {
					firstIndex: -1,
					isExists: false
				}
				for (let i in this.cartGoods) {
					let goods = this.cartGoods[i];
					if (goods.catIndex == catIndex && goods.goodsIndex == goodsIndex) {
						if (flag == 0) {
							res.firstIndex = i;
						}
						flag++;
					}
				}
				res.isExists = flag > 1;
				return res;
			},
			reductCartWithGoods(cartGoodsIndex) {
				let goods = this.cartGoods[cartGoodsIndex];
				let price = goods.price / goods.count;
				this.cartGoods[cartGoodsIndex].count--;
				this.cartGoods[cartGoodsIndex].price -= price;
				this.totalPrice -= price;
				this.reductCartNumber(goods.catIndex, goods.goodsIndex);
				if (this.cartGoods[cartGoodsIndex].count <= 0) {
					this.cartGoods.splice(cartGoodsIndex, 1);
				}
				if (this.cartTotalCount <= 0) {
					this.$refs.popupCart.close();
				}
			},
			addCartWithGoods(cartGoodsIndex) {
				let goods = this.cartGoods[cartGoodsIndex];
				this.current.catIndex = goods.catIndex;
				this.current.goodsIndex = goods.goodsIndex;
				this.current.attrs = cloneObject(goods.attrs);
				this.pushCurrentToCart();
			},
			//清空购物车
			clearCart() {
				this.initCartContainer();
				this.cartGoods = [];
				this.totalPrice = 0;
				this.cartTotalCount = 0;
				this.$refs.popupCart.close();
				this.cartIconAnimation = "";
			},
			scroll(e) {
				if (!this.initFinish) {
					return;
				}

				let scrollTop = e.detail.scrollTop;
				for (let i in this.goodsHeight) {
					if (scrollTop < this.goodsHeight[i]) {
						if (this.categories[i].active == false) {
							this.setActive(i);
						}
						break;
					}
				}
			},

			scrollToGoods(index) {
				this.goodsViewId = "cat-" + index;
				this.setActive(index);
			},

			setActive(index) {
				this.categories[this.lastActiveIndex].active = false;
				this.categories[index].active = true;
				this.lastActiveIndex = index;
			},

			getCategory() {
				for (let j = 0; j < 5; j++) {
					let good = JSON.parse(JSON.stringify(goodObj));
					good.name = "黑糖琉璃茶";
					if (j > 2) {
						good.attrs = [];
					}
					categoryObj.goods.push(good)
				}
				let names = ["琉璃水晶", "牛乳茶系列", "鲸沙系列", "桃胶火山琼系列", "满分水果茶"]
				for (let i in names) {
					let cat = JSON.parse(JSON.stringify(categoryObj));
					cat.name = names[i];
					cat.active = false;
					this.categories.push(cat);
				}
				this.initCartContainer();
			},
			initCartContainer() {
				for (let i in this.categories) {
					console.log("i:" + i);
					this.cart[i] = [];
					for (let j in this.categories[i].goods) {
						this.cart[i][j] = 0;
					}
				}
			},
			initGoodsBoxHeight() {
				let titleHeight = uni.upx2px(64);
				let paddingHeight = uni.upx2px(20) * 2;
				let goodsPaddingHeight = uni.upx2px(10) * 2;
				let goodsRowHeight = uni.upx2px(170);
				let goodsHeight = 0;
				this.goodsHeight = [];
				for (let i in this.categories) {
					let goodsCount = this.categories[i].goods.length;
					goodsHeight += goodsPaddingHeight * goodsCount + goodsRowHeight * goodsCount + titleHeight + paddingHeight;
					this.goodsHeight.push(goodsHeight);
				}
			},
			close() {
				this.$refs.popupRef.close();
			}
		},
		onLoad: function() {
			this.getCategory();
			this.initGoodsBoxHeight();
			this.setActive(0);
			this.height = uni.getSystemInfoSync().windowHeight - uni.upx2px(90);
			this.initFinish = true;
		}
	}
</script>

<style lang="scss">
	$desc-color: rgb(199, 199, 199);
	$main-color: rgb(70, 160, 252);
	$orange-color: rgb(250, 90, 92);
	$tips-color: #666;

	.cart-panel {
		display: -webkit-flex;
		display: flex;
		-webkit-align-items: center;
		align-items: center;
		padding: 0 .4rem;
		padding: 0 4vw;
		border-bottom: .013333rem solid #ddd;
		border-bottom: .133333vw solid #ddd;
		background-color: #eceff1;
		color: #666;
		height: 1.066667rem;
		height: 10.666667vw;
	}

	.cart-panel-text {
		-webkit-flex: 1;
		flex: 1;
		display: -webkit-flex;
		display: flex;
		-webkit-align-items: center;
		align-items: center;
	}

	.cart-clear {
		-webkit-flex: none;
		flex: none;
		display: -webkit-flex;
		display: flex;
		-webkit-align-items: center;
		align-items: center;
		padding-left: .4rem;
		padding-left: 4vw;
		color: #666;
		text-decoration: none;
		font-size: .346667rem;
		line-height: .4rem;
		line-height: 4vw;
	}

	.cart-goods {
		display: -webkit-flex;
		display: flex;
		-webkit-align-items: center;
		align-items: center;
		padding: .2rem .333333rem .2rem 0;
		padding: 2vw 3.333333vw 2vw 0;
		min-height: 1.466667rem;
		min-height: 14.666667vw;
		margin-left: .333333rem;
		margin-left: 3.333333vw;
		position: relative;
		border-bottom: 2upx solid rgb(248, 248, 248);
		box-sizing: border-box;

		.stock-panel {
			right: 3.3333vw;
		}

		.cart-goods-info {
			width: 60%;

			em {
				font-weight: normal;
				font-size: 36upx;
				font-style: normal;
			}

			p {
				color: #999;
				font-size: 20upx;
			}
		}

		.cart-goods-price {
			em {
				color: #f15a54;
				font-size: 36upx;

				font-weight: bold;
				font-style: normal;
			}

			span {
				font-size: 24upx;
			}
		}
	}

	.more-choice {
		padding: 30upx;
		height: 146.666667vw;
		position: relative;

		.more-choice-box {
			width: 100%;
			height: 100%;
			position: relative;

			.attr-group {
				margin-bottom: 20upx;
			}

			.attr {
				color: $tips-color;
			}

			.sub-attr-box {
				margin-left: -1.6vw;

				.sub-attr {
					min-width: 27.733333vw;
					display: inline-block;
					color: rgb(92, 92, 92);
					background: rgb(245, 245, 245);
					font-size: 24upx;
					text-align: center;
					margin-left: 1.6vw;
					margin-right: 1.6vw;
					margin-bottom: 2.3vw;
					line-height: 8.533333vw;

					&.active {
						background: rgb(213, 234, 254);
						color: rgb(50, 152, 252);
					}
				}
			}



			.box-goods-info {
				height: 200upx;
				margin-bottom: 30upx;
			}

			image {
				width: 200upx;
				height: 200upx;
			}

			.goods-info {
				height: 200upx;

				.goods-name {
					font-size: 36upx;
				}

				.goods-price {
					font-size: 44upx;

					span {
						font-size: 32upx;
					}
				}
			}
		}

		.bottom {
			width: 100%;
			position: absolute;
			bottom: 0upx;
			box-sizing: border-box;
			background: rgb(45, 151, 252);
		}
	}

	.cart-list {
		padding-bottom: 30upx;
	}

	.cart-goods-box {
		max-height: 80vw;
		overflow: auto;

	}

	.cart {
		background: rgb(89, 89, 89);
		height: 90upx;
		position: fixed;
		bottom: 0;
		width: 100%;


		&.top-layer {
			z-index: 9999999;
		}

		.price-info {
			height: 90upx;
			line-height: 90upx;
			padding-left: 170upx;
			color: #8a8a8a;

			&.active {
				color: #fff;
				font-weight: bold;
				font-size: 32upx;
			}
		}

		.cart-icon {
			font-size: 52upx;
			line-height: 90upx;
			color: rgb(102, 102, 106);
			width: 90upx;
			height: 90upx;
			border-radius: 50%;
			position: absolute;
			text-align: center;
			bottom: 20upx;
			left: 40upx;
			background: rgb(54, 54, 54);
			border: 5px solid rgb(68, 68, 68);

			.cart-badge {
				width: 32upx;
				height: 32upx;
				position: absolute;
				z-index: 1;
				right: 0upx;
				top: -18upx;
				background: red;
				color: #fff;
				border-radius: 50%;
				text-align: center;
				line-height: 32upx;
				font-size: 28upx;

				span {
					-webkit-transform: scale(.75);
					transform: scale(.75);
					display: inline-block;
					width: 100%;
					height: 100%;
				}
			}


			&.shake {
				-webkit-animation: shake .5s ease-in-out;
				animation: shake .5s ease-in-out
			}

			&:before {
				content: "";
				width: 120upx;
				height: 120upx;
				border-radius: 50%;
				position: absolute;
				top: -15upx;
				left: -15upx;
				background: rgba(0, 0, 0, 0.1);
				z-index: -1;
			}

			&.active {
				color: #fff;
				background: $main-color;
			}
		}

		.buy {
			width: 30%;
			text-align: center;
			line-height: 90upx;
			color: #8a8a8a;
			background: rgb(83, 83, 86);
			position: absolute;
			right: 0;
			top: 0;

			&.active {
				background: $orange-color;
				color: #fff;
			}
		}
	}

	.goods-box {
		width: 100%;
		display: flex;
		padding: 20upx;
		box-sizing: border-box;
		flex-direction: column;
		position: relative;

		&.fixed {
			padding: 84upx 20upx 20upx 20upx;
		}
	}

	.cat-title {
		width: 100%;
		box-sizing: border-box;
		font-size: 32upx;
		line-height: 64upx;
		color: rgb(107, 107, 107);
		display: flex;
		position: relative;
		font-weight: normal;

		&.fixed {
			position: fixed;
			top: 0px;
			/* #ifdef H5 */
			top: 43px;
			/* #endif */
			padding-top: 22upx;
			background: #fff;
			z-index: 1;
		}
	}

	span.desc {
		color: $desc-color;
		font-size: 24upx;
		line-height: 64upx;
		margin-left: 20upx;
		font-weight: normal;
	}

	.page-body {
		display: flex;
	}

	.nav {
		display: flex;
		width: 100%;
	}

	.nav-left {
		width: 25%;
		background: rgba(248, 248, 248, 1);
	}

	.nav-left-item {
		padding: 36upx 30upx;
		font-size: 28upx;
		display: flex;
		align-items: center;
		justify-content: center;
		color: rgb(146, 146, 146);

		&.active {
			background: #ffffff;
			font-weight: bold;
			color: #000;
		}
	}

	.nav-right {
		width: 75%;
	}

	.nav-right-item {
		width: 100%;
		height: 195upx;
		flex-shrink: 0;
		box-sizing: border-box;
		text-align: left;
		padding: 10upx 0;
		font-size: 28upx;
		position: relative;

		.stock-panel {
			right: 0upx;
			bottom: 15upx;
		}
	}

	.stock-panel {
		display: flex;
		width: 154upx;
		height: 52upx;
		position: absolute;

		.add-cart,
		.reduct-cart,
		.cart-number {
			font-size: 52upx;
			color: $main-color;
			position: absolute;

			.iconfont {
				display: block;
				width: 52upx;
				height: 52upx;
				font-size: 52upx;
				line-height: 52upx;
			}
		}

		.add-cart {
			right: 0;
		}

		.reduct-cart {
			left: 0;
		}

		.cart-number {
			left: 50upx;
			width: 54upx;
			right: 54upx;
			text-align: center;
			color: #000;
			font-size: 32upx;
			font-weight: bold;
		}
	}



	.reduct-cart {
		right: 112upx;
	}

	.goods-info {
		float: left;
		height: 170upx;
		margin-left: 20upx;
		position: relative;

		.goods-name {
			font-size: 32upx;
			font-weight: bold;
			line-height: 100%;
		}

		.goods-price {
			position: absolute;
			font-weight: bold;
			bottom: 0;
			font-size: 32upx;
			color: rgb(241, 90, 84);
			line-height: 100%;

			span {
				font-size: 24upx;
			}
		}

		.goods-desc {
			font-size: 24upx;
			margin-top: 10upx;
			color: $desc-color;
		}

	}

	.nav-right-item image,
	.more-choice-box image {
		width: 170upx;
		height: 170upx;
		border-radius: 3px;
		float: left;
	}

	@-webkit-keyframes shake {
		0% {
			-webkit-transform: scale(1);
			transform: scale(1)
		}

		25% {
			-webkit-transform: scale(.8);
			transform: scale(.8)
		}

		50% {
			-webkit-transform: scale(1.1);
			transform: scale(1.1)
		}

		75% {
			-webkit-transform: scale(.9);
			transform: scale(.9)
		}

		to {
			-webkit-transform: scale(1);
			transform: scale(1)
		}
	}

	@keyframes shake {
		0% {
			-webkit-transform: scale(1);
			transform: scale(1)
		}

		25% {
			-webkit-transform: scale(.8);
			transform: scale(.8)
		}

		50% {
			-webkit-transform: scale(1.1);
			transform: scale(1.1)
		}

		75% {
			-webkit-transform: scale(.9);
			transform: scale(.9)
		}

		to {
			-webkit-transform: scale(1);
			transform: scale(1)
		}
	}
</style>
