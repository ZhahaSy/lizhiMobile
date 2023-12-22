<template>
	<view class="playerWrapper">
		<text class="iconfont icon-player-fast-back pre" @click="handlePre"></text>
		<text @click="handlePlay" :class="['iconfont', !isPlaying ? 'icon-player-play' : 'icon-player-pause-01']"></text>
		<text class="iconfont icon-player-fast-forward next" @click="handleNext"></text>
		<text class="currentMusicName">{{currentMusic.name}}</text>
		<text class="iconfont icon-player-list-play listBtn" @click="open"></text>
		<uni-popup ref="popup" type="bottom">
			<view class="musicListWrapper">
				<h3>播放列表</h3>
				<ul class="list">
					<li v-for="(item,index) in musicList" :key="item.url" @click="() => {handleChangeMusic(index)}" class="listItem">{{item.name}}-{{item.artist}}</li>
				</ul>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		name: 'musicPlayer',
		props: {
			musicList: Array,
		},
		data() {
			return {
				isPlaying: false,
				currentMusicIndex: 0,
				audioContext: uni.createInnerAudioContext(),
			};
		},
		onLoad() {
				this.audioContext.volume = 1
				this.audioContext.loop = true
		},
		computed: {
				currentMusic() {
					return this.musicList[this.currentMusicIndex] || {}
				}
		},
		watch:{
			currentMusic(val) {
				if (!(val && this.audioContext)) return
				this.audioContext.src = val.url;
				this.isPlaying = true
			},
			isPlaying(val) {
				val ? this.audioContext.play() : this.audioContext.pause()
			}
		},
		methods: {
			handlePlay() {
				if (this.currentMusic.url) {
					this.isPlaying = !this.isPlaying
				}
			},
			handlePre() {
				if (this.currentMusicIndex > 0) {
					this.isPlaying = false
					this.currentMusicIndex -= 1
				} else {
					this.audioContext.stop()
					this.isPlaying = false
					this.currentMusicIndex = this.musicList.length - 1
				}
			},
			handleNext() {
				if (this.currentMusicIndex < this.musicList.length) {
					this.audioContext.stop()
					this.isPlaying = false
					this.currentMusicIndex += 1
				} else {
					this.audioContext.stop()
					this.isPlaying = false
					this.currentMusicIndex = 1
				}
			},
			open(){
				// 通过组件定义的ref调用uni-popup方法 ,如果传入参数 ，type 属性将失效 ，仅支持 ['top','left','bottom','right','center']
				this.$refs.popup.open()
			},
			close(){
				// 通过组件定义的ref调用uni-popup方法 ,如果传入参数 ，type 属性将失效 ，仅支持 ['top','left','bottom','right','center']
				this.$refs.popup.close()
			},
			handleChangeMusic(index) {
				this.audioContext.stop()
				this.isPlaying = false
				this.currentMusicIndex = index;
				this.close()
			}
		}
	}
</script>

<style lang="less" scoped>
.playerWrapper {
	display: flex;
	z-index: 2021;
	position: fixed;
	bottom: 0;
	width: calc(100% - 40px);
	height: 40px;
	background-color: #111;
	color: #fff;
	padding: 0 20px 0;
	text {
		margin: 0 10px;
		line-height: 40px;
		font-size: 20px;
	}
	.listBtn {
		font-size: 22px;
		float: right;
	}
	.currentMusicName {
		display: inline-block;
		flex: 1;
		overflow: hidden;
		text-overflow: ellipsis;
		font-size: 16px
	}
	
}
.musicListWrapper {
	height: 60vh;
	background-color: #fff;
	color: #333;
	padding: 10px;
	border-radius: 10px 10px 0 0;
	.list {
		margin-top: 10px;
		.listItem {
			padding: 6px;
			width: 100%;
			font-size: 13px;
		}
	}
}
</style>
