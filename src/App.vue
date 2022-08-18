<script setup>
import { reactive, ref, watch, computed } from "vue";
import axios from "axios";
import gsap from "gsap";
const apiUrl =
	"https://awiclass.monoame.com/api/command.php?type=get&name=movies";

const data = reactive({
	movies: [],
	cart: [],
	isCartOpen: false,
});

const cardsContainer = ref(null);

axios.get(apiUrl).then((res) => {
	data.movies = res.data;
});

// methods
function wheelHandler(e) {
	let delta = e.deltaY ? e.deltaY * 2 : -e.deltaX * 2;
	if (delta > 200) delta = 200;
	if (delta < -200) delta = -200;
	if (
		cardsContainer.value.getBoundingClientRect().left >
		window.innerWidth / 2 - 100
	) {
		gsap.to(cardsContainer.value, {
			duration: 0.01,
			left: `+=${-10}px`,
		});
	} else if (
		cardsContainer.value.getBoundingClientRect().right <
		window.innerWidth / 2 + 100
	) {
		gsap.to(cardsContainer.value, {
			duration: 0.01,
			left: `+=${10}px`,
		});
	} else {
		gsap.to(cardsContainer.value, {
			duration: 0.3,
			left: `+=${delta}px`,
		});
	}
}
function addToCart(movie, e) {
	e.target.disabled = true;
	const img = new Image(50, 80);
	img.src = movie.cover;
	document.querySelector(".buyBox").append(img);
	gsap.from(".buyBox", {
		duration: 0.6,
		left: e.target.getBoundingClientRect().left,
		top: e.target.getBoundingClientRect().top,
		opacity: 1,
	});
	setTimeout(() => {
		data.cart.push(movie);
		img.remove();
	}, 600);
}
function controlHandler(e) {
	if (e.target !== document.querySelector(".control")) return;
	data.isCartOpen = !data.isCartOpen;
}
function cartHandler(i, name) {
	data.cart.splice(i, 1);
	const index = data.movies.map((movie) => movie.name).indexOf(name);
	document
		.querySelectorAll(".card")
		[index].querySelector("button").disabled = false;
}
// watch
watch(data.cart, () => {
	gsap.from(".fa-cart-shopping", 0.3, { scale: 0.5 });
});

// computed
const totalPrice = computed(() =>
	data.cart.map((movie) => movie.price).reduce((sum, price) => sum + price, 0)
);
</script>

<template>
	<h1 class="appTitle">BarZ Theater</h1>

	<div class="movieContainer" @wheel.prevent="wheelHandler">
		<ul
			class="cards"
			:class="{ isCartOpen: data.isCartOpen }"
			ref="cardsContainer"
		>
			<li class="card" v-for="movie in data.movies" :key="movie.name">
				<div class="left">
					<img :src="movie.cover" alt="cover image" />
				</div>
				<div class="right">
					<h2>{{ movie.name }}</h2>
					<h4>{{ movie.type }}</h4>
					<p>{{ movie.description }}</p>
					<div class="price">{{ movie.price }} $</div>
					<button class="add" @click="addToCart(movie, $event)">
						+ Add to My Cart
					</button>
				</div>
			</li>
		</ul>
	</div>

	<div class="buyBox"></div>

	<div class="fixed-control" @click="data.isCartOpen = !data.isCartOpen">
		<i class="fa-solid fa-cart-shopping"></i>
		<span>{{ data.cart.length }}</span>
	</div>

	<div
		class="control"
		:class="{ isCartOpen: data.isCartOpen }"
		@click="controlHandler"
	>
		<button class="close" @click="data.isCartOpen = !data.isCartOpen">x</button>
		<div class="panel">
			<h2 class="panel-title">My Movie Cart</h2>
			<ul>
				<li v-for="(movie, i) in data.cart">
					<img :src="movie.cover" alt="" />
					<h3>
						{{ movie.name }}
						<div class="price">{{ movie.price }}$</div>
						<button class="remove" @click="cartHandler(i, movie.name)">
							一
						</button>
					</h3>
				</li>
				<h3 v-if="!data.cart.length" class="empty">Your cart is empty...</h3>
			</ul>
			<hr />
			<h2 class="total" v-if="!!data.cart.length">Total: {{ totalPrice }} $</h2>
		</div>
	</div>
</template>

<style lang="scss" scoped>
@mixin size($width, $height: $width) {
	width: $width;
	height: $height;
}
@mixin shadow() {
	box-shadow: 0 5px 24px 5px rgb(0 0 0 / 0.24);
}
@mixin fixPosition($left, $top) {
	position: fixed;
	left: $left;
	top: $top;
}
.appTitle {
	@include fixPosition(2rem, 0.24rem);
	font-size: 2rem;
}
.movieContainer {
	flex: 1;
}
.cards {
	display: flex;
	justify-content: flex-start;
	align-items: center;
	height: 100%;
	margin: 0 20vw;
	transition: 0.3s, left 0.08s;
	position: relative;
	&.isCartOpen {
		transform: scale(0.8);
	}
}

.card {
	@include size(32rem, auto);
	margin: 60px;
	flex-shrink: 0;
	display: inline-flex;
	background-color: rgb(255 255 255 / 0.9);
	border-radius: 0.8rem;
	@include shadow();
	padding: 1.2rem 1rem 0;
	transition: transform 0.3s;
	&:hover {
		transform: translateY(-10px);
		.left img {
			transform: translateY(-10px);
		}
		.right {
			color: #555;
		}
	}

	.left {
		flex: 1;
		img {
			@include size(180px, 240px);
			@include shadow;
			border-radius: 8px;
			transition: transform 0.3s;
			position: relative;
			top: -50px;
			margin-bottom: -1rem;
		}
	}
	.right {
		flex: 2;
		padding: 10px 20px 0;
		color: #888;
		transition: color 0.3s;
		h2 {
			margin: 0;
			font-weight: bold;
			font-size: 24px;
		}
		h4 {
			margin: 10px 0;
			font-weight: normal;
			filter: brightness(0.81);
		}
		p {
			font-size: 0.81rem;
			line-height: 1.3;
			text-align: justify;
			opacity: 0.8;
			min-height: 5em;
		}
		.price {
			display: inline-block;
			margin-right: 20px;
		}
		button {
			padding: 0.4rem 0.8rem;
			background-color: #bbb;
			color: #eee;
			border-radius: 50px;
			cursor: pointer;
			transition: color 0.3s, background-color 0.3s, transform 0.24s;
			background-color: rgb(204, 122, 92);
			border: none;
			&:hover {
				color: #fff;
				background-color: rgb(201, 95, 81);
			}
			&:not(:disabled):active {
				transform: translateY(-0.24rem);
			}
			&:disabled {
				background-color: rgba(0, 0, 0, 0.3);
				color: #eee;
				cursor: not-allowed;
			}
		}
	}
}
.buyBox {
	@include size(50px, 80px);
	position: fixed;
	top: 30px;
	right: 30px;
	opacity: 0;
	img {
		opacity: 0;
	}
}
.fixed-control {
	position: fixed;
	right: 30px;
	top: 30px;
	color: #eee;
	z-index: 999;
	cursor: pointer;
	opacity: 0.8;
	transition: 0.3s;
	&:hover {
		opacity: 1;
		transform: scale(1.1);
	}
	&:active {
		transform: translateY(0.24rem);
	}
	i {
		font-size: 1.6rem;
		margin-right: 0.24rem;
	}
}
.control {
	@include size(100%);
	@include fixPosition(0, 0);
	display: grid;
	place-items: center;
	background-image: linear-gradient(
		10deg,
		#111 24%,
		hsl(0 0% 24% / 0.24) 81%,
		transparent 100%
	);
	padding: 5vw;
	opacity: 0;
	transition: 0.3s;
	visibility: hidden;
	&.isCartOpen {
		opacity: 1;
		visibility: visible;
	}
	.panel {
		width: 70%;
		border-radius: 0.8rem;
		padding: 2.4rem calc(2.4vw + 0.8rem);
		background-color: rgba(103, 126, 114, 0.24);
		h2.panel-title,
		h3.empty {
			text-align: center;
		}
		h2.panel-title {
			font-size: clamp(2rem, 2.4vw, 2.4rem);
		}
		h3.empty {
			filter: brightness(0.85);
			margin-block: 2.4rem;
		}
		ul {
			padding: 0;
			li {
				display: flex;
				gap: 1.2rem;
				padding: 0.8rem;
				margin-block: 0.8rem;
				cursor: pointer;
				border-radius: 5px;
				transition: 0.3s;
				&:hover {
					background-color: rgb(255 255 255 / 0.1);
					transform: translateY(-0.24rem);
					h3 {
						filter: brightness(1);
					}
				}
				h3 {
					font-size: 17px;
					font-weight: normal;
					display: inline-flex;
					justify-content: space-between;
					align-items: center;
					margin: 0;
					width: 100%;
					filter: brightness(0.81);
					.price {
						margin-left: auto;
						margin-right: 1.2rem;
					}
					.remove {
						border: none;
						padding: 0 0.6rem;
						border-radius: 0.24rem;
						font-size: 1.2rem;
						font-weight: bold;
						transition: 0.3s;
						&:hover {
							background-color: rgb(181, 57, 40);
							color: #eee;
						}
					}
				}
				img {
					@include size(50px, 70px);
				}
			}
		}
	}
	.total {
		text-align: right;
	}
	.close {
		position: fixed;
		right: min(8rem, 8%);
		top: min(20rem, 20%);
		border: none;
		border-radius: 5px;
		background-color: rgb(255 255 255 / 0.1);
		color: #eee;
		font-size: 2rem;
		cursor: pointer;
		transition: 0.3s;
		transform: scaleX(1.3);
		&:hover {
			background-color: rgb(255 255 255 / 0.3);
		}
		&:active {
			transform: scaleX(1.3) translateY(0.24rem);
		}
	}
}
</style>
