<template>
	<ul class="list">
		<li class="item" v-for="item in musicList" :key="item.artistName" @click="() => {updatePlayingList(item)}">
			<img :src="item.artistCover" :alt="item.artistName" />
			<p>{{ item.artistName }}</p>
		</li>
	</ul>
</template>

<script>
	export default {
		name: 'Artists',
		props: ['musicList'],
		emits: ['updatePlayingList'],
		data() {
			return {
			}
		},
		onLoad() {
		},
		methods: {
			updatePlayingList(item) {
				this.$emit('updatePlayingList', item.children || [])
			}
		}
	}
</script>

<style lang="less" scoped>
	.list {
		z-index: 1;
		margin-bottom: 60px;
		.item {
			display: inline-block;
			padding: 10px;
			width: 27%;
			height: 140px;
			color: #fff;
			img {
				width: calc(100%);
			}
			p {
				height: 30px;
				display: -webkit-box;//对象作为弹性伸缩盒子模型显示 
				-webkit-box-orient: vertical;//设置或检索伸缩盒对象的子元素的排列方式 
				-webkit-line-clamp: 2;//溢出省略的界限
				overflow:hidden;
				font-size: 13px;
				line-height: 15px;
			}
		}
	}
</style>
