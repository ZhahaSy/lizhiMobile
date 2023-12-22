<template>
	<view class="playerWrapper">
		<img :class="['cover', isPlaying? 'isPlaying' : 'isPause']" :src="currentMusic.cover" alt="专辑图片" />
		<text class="iconfont icon-player-fast-back pre" @click="handlePre"></text>
		<text @click="handlePlay" :class="['iconfont', !isPlaying ? 'icon-player-play' : 'icon-player-pause-01']"></text>
		<text class="iconfont icon-player-fast-forward next" @click="handleNext"></text>
		<text class="currentMusicName">{{currentMusic.name}}</text>
		<text class="iconfont icon-player-list-play listBtn" @click="open"></text>
		<uni-popup ref="popup" type="bottom">
			<view class="musicListWrapper">
				<h3>播放列表</h3>
				<ul class="list">
					<li v-for="(item,index) in musicList" :key="item.url"  :class="{
						active: index === currentMusicIndex
					}" @click="() => {handleChangeMusic(index)}" class="listItem">{{item.name}}-{{item.artist}}</li>
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
				audioContext: undefined,
			};
		},
		mounted() {
			this.audioContext = uni.createInnerAudioContext()
			this.audioContext.volume = 1
			this.audioContext.onEnded(this.handleNext)
		},
		computed: {
				currentMusic() {
					return this.musicList[this.currentMusicIndex] || {}
				}
		},
		watch:{
			musicList(val) {
					this.currentMusicIndex = 0
			},
			currentMusic(val) {
				if (!(val && this.audioContext)) return
				this.audioContext.src = val.url;
				this.isPlaying = true;
				this.audioContext.play()
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
	align-items: center;
	z-index: 2021;
	position: fixed;
	bottom: 0;
	width: calc(100% - 40px);
	height: 60px;
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
	.cover {
		width: 30px;
		height: 30px;
		border-radius: 15px;
		animation: rotate 10s linear infinite;
		animation-fill-mode: forwards;
		 @keyframes rotate {
		        0% {
		            transform: rotate(0);
		        }
		
		        25% {
		            transform: rotate(90deg);
		        }
		
		        50% {
		            transform: rotate(180deg);
		        }
		
		        75% {
		            transform: rotate(270deg);
		        }
		
		        100% {
		            transform: rotate(360deg);
		        }
		    }
		animation-fill-mode: forwards;
		&.isPlaying {
			animation-play-state: running;
		}
		&.isPause {
			animation-play-state: paused;
		}
	}
	
}
.musicListWrapper {
	height: 60vh;
	background-color: #fff;
	color: #333;
	padding: 10px;
	border-radius: 10px 10px 0 0;
	h3 {
		background-color: #fff;
	}
	.list {
		height: calc(60vh - 40px);
		overflow: scroll;
		padding-top: 10px;
		.listItem {
			padding: 10px;
			font-size: 14px;
			font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
			color: #a2a2a2;
			&.active {
				color: #111;
			}
		}
	}
}
</style>
