<template>
	<view class="luch-audio__warp">
		<view class="luch-audio__cover" :class="{ 'luch-audio--cover-bg': !poster }">
			<image :src="poster" v-if="poster" class="luch-audio__cover-img"></image>
			<view class="luch-audio__play-btn" :class="{ 'luch-audio--btn-pause': videoIsPlay }" @click="handleBtnClick"></view>
		</view>
		<view class="luch-audio__audio-con">
			<view class="luch-audio__audio-info">
				<text class="luch-audio__audio-title luch-audio--text-eill">{{ name }}</text>
				<text class="luch-audio__audio-author luch-audio--text-eill">{{ author }}</text>
			</view>
			<view class="luch-audio__audio-time">{{ audioTimeUpdate }}</view>
		</view>
	</view>
</template>

<script>
/**
 * luch-audio
 * @module luch-audio
 * @Author lu-ch
 * @Date 2022-07-14
 * @Email webwork.s@qq.com
 * @description 音频播放组件，使用了createInnerAudioContext
 */
/**
 * Props itemsProps
 * @prop {Boolean} play - 是否播放，双向绑定，绑定时需使用.sync 如果为true 则播放，为false 则暂停
 * ... 其他api同文档 （https://uniapp.dcloud.io/api/media/audio-context?id=createinneraudiocontext）
 */
/**
 * 将秒转换为 分:秒
 * @param {Number} s - 秒数
 */
function sToHs(s) {
	//计算分钟
	//算法：将秒数除以60，然后下舍入，既得到分钟数
	let h;
	h = Math.floor(s / 60);
	//计算秒
	//算法：取得秒%60的余数，既得到秒数
	s = s % 60;
	//将变量转换为字符串
	h += '';
	s += '';
	//如果只有一位数，前面增加一个0
	h = h.length === 1 ? '0' + h : h;
	s = s.length === 1 ? '0' + s : s;
	return h + ':' + s;
}
export default {
	name: 'ComAudio',
	props: {
		// 是否播放或暂停
		play: {
			type: Boolean,
			default: false
		},
		// 播放src
		src: {
			type: String
		},
		poster: {
			type: String,
			default: ''
		},
		name: {
			type: String,
			default: '未知音频'
		},
		author: {
			type: String,
			default: '未知作者'
		},
		autoplay: {
			type: Boolean
			// default: false
		},
		loop: {
			type: Boolean
			// default: false
		},
		// #ifdef MP-WEIXIN || MP-BAIDU || MP-TOUTIAO || MP-LARK || MP-JD || MP-KUAISHOU
		obeyMuteSwitch: {
			type: Boolean
			// default: true
		},
		// #endif
		// 初始化回调
		initAudio: {
			type: Function
		},
		// 自定义组件标识
		index: {
			type: [String, Number]
		}
	},
	data() {
		return {
			audioTimeUpdate: '00:00',
			videoIsPlay: false
			// innerAudioContext: ''
		};
	},
	watch: {
		play(n) {
			// this.videoIsPlay = play
			if (n) {
				!this.videoIsPlay && this.audioPlay();
			} else {
				this.videoIsPlay && this.audioPause();
			}
		},
		src() {
			this.audioDestroy();
			this.contextInit();
		}
	},
	created() {
		this.contextInit();
	},
	beforeDestroy() {
		this.audioDestroy();
	},
	methods: {
		audioPlay() {
			this.innerAudioContext && this.innerAudioContext.play();
		},
		audioPause() {
			this.innerAudioContext && this.innerAudioContext.pause();
		},
		audioOnPlay() {
			this.videoIsPlay = true;
			this.$emit('update:play', true);
		},
		audioOnPause() {
			this.videoIsPlay = false;
			this.$emit('update:play', false);
		},
		audioDestroy() {
			this.innerAudioContext && this.innerAudioContext.destroy && this.innerAudioContext.destroy();
			this.audioPropInit();
		},
		audioControls() {
			if (this.videoIsPlay) {
				this.audioPause();
			} else {
				this.audioPlay();
			}
		},
		handleBtnClick() {
			// this.videoIsPlay = !this.videoIsPlay
			this.audioControls();
			this.$emit('update:play', this.videoIsPlay);
		},
		contextInit() {
			let that = this;
			that.$emit('update:play', false);
			if (!that.src) {
				if (process.env.NODE_ENV !== 'production') {
					console.warn(`[luch-audio warn]: 'src' 为空`);
				}
				return;
			}
			let innerAudioContext = uni.createInnerAudioContext();
			innerAudioContext.autoplay = that.autoplay;
			innerAudioContext.loop = that.loop;
			// #ifdef MP-WEIXIN || MP-BAIDU || MP-TOUTIAO || MP-LARK || MP-JD || MP-KUAISHOU
			if (typeof that.obeyMuteSwitch === 'boolean') {
				innerAudioContext.obeyMuteSwitch = that.obeyMuteSwitch;
			}
			// #endif
			innerAudioContext.onPlay((...arg) => {
				that.audioTimeUpdate = sToHs(Math.floor(innerAudioContext.currentTime));
				that.audioOnPlay();
				that.$emit('onPlay', ...arg);
			});
			innerAudioContext.onPause((...arg) => {
				that.audioOnPause();
				that.$emit('onPause', ...arg);
			});
			innerAudioContext.onEnded((...arg) => {
				that.audioOnPause();
				that.$emit('onEnded', ...arg);
			});
			innerAudioContext.onTimeUpdate((...arg) => {
				that.audioTimeUpdate = sToHs(Math.floor(innerAudioContext.currentTime));
				that.$emit('onTimeUpdate', ...arg);
			});
			innerAudioContext.onError((...arg) => {
				that.$emit('onError', ...arg);
			});
			innerAudioContext.src = that.src;
			if (that.initAudio) {
				that.initAudio(innerAudioContext, { src: that.src, index: that.index });
			}
			this.innerAudioContext = innerAudioContext;
		},
		audioPropInit() {
			this.audioTimeUpdate = '00:00';
			this.videoIsPlay = false;
		}
	}
};
</script>

<style lang="scss" scoped>
.luch-audio--text-eill {
	/*超出省略号*/
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}

.luch-audio__warp {
	display: flex;
	overflow: hidden;
	height: 67px;
	border-radius: 3px;
	border: 1px solid #e0e0e0;
}
.luch-audio__cover {
	position: relative;
	flex-shrink: 0;
	width: 65px;
	height: 100%;
	&.luch-audio--cover-bg {
		background-color: #e6e6e6;
	}
	.luch-audio__cover-img {
		width: 100%;
		height: 100%;
	}
	.luch-audio__play-btn {
		position: absolute;
		left: 50%;
		top: 50%;
		width: 24px;
		height: 24px;
		transform: translateX(-50%) translateY(-50%);
		border-radius: 50%;
		background-size: 100% 100%;
		background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFAAAABQCAYAAACOEfKtAAAACXBIWXMAAAsTAAALEwEAmpwYAAAGymlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDUgNzkuMTYzNDk5LCAyMDE4LzA4LzEzLTE2OjQwOjIyICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdEV2dD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlRXZlbnQjIiB4bWxuczpkYz0iaHR0cDovL3B1cmwub3JnL2RjL2VsZW1lbnRzLzEuMS8iIHhtbG5zOnBob3Rvc2hvcD0iaHR0cDovL25zLmFkb2JlLmNvbS9waG90b3Nob3AvMS4wLyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOSAoV2luZG93cykiIHhtcDpDcmVhdGVEYXRlPSIyMDE5LTA2LTA0VDE3OjU1OjA3KzA4OjAwIiB4bXA6TWV0YWRhdGFEYXRlPSIyMDE5LTA2LTA1VDA5OjUwOjUwKzA4OjAwIiB4bXA6TW9kaWZ5RGF0ZT0iMjAxOS0wNi0wNVQwOTo1MDo1MCswODowMCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo4OWI0YTlkMi02ZWIzLTUyNDQtYjM4OS03MzdmYTA1OTM1ZWEiIHhtcE1NOkRvY3VtZW50SUQ9ImFkb2JlOmRvY2lkOnBob3Rvc2hvcDoyODIyOGJlOC0zZDgxLTlhNDMtOGJjMy02NDA2NTQzYWIwMzQiIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDo0YjA1NDBlYy1mMjE1LWU4NDEtYjkwYy0xZjBiNGQ3OGJkODkiIGRjOmZvcm1hdD0iaW1hZ2UvcG5nIiBwaG90b3Nob3A6Q29sb3JNb2RlPSIzIiBwaG90b3Nob3A6SUNDUHJvZmlsZT0ic1JHQiBJRUM2MTk2Ni0yLjEiPiA8eG1wTU06SGlzdG9yeT4gPHJkZjpTZXE+IDxyZGY6bGkgc3RFdnQ6YWN0aW9uPSJjcmVhdGVkIiBzdEV2dDppbnN0YW5jZUlEPSJ4bXAuaWlkOjRiMDU0MGVjLWYyMTUtZTg0MS1iOTBjLTFmMGI0ZDc4YmQ4OSIgc3RFdnQ6d2hlbj0iMjAxOS0wNi0wNFQxNzo1NTowNyswODowMCIgc3RFdnQ6c29mdHdhcmVBZ2VudD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTkgKFdpbmRvd3MpIi8+IDxyZGY6bGkgc3RFdnQ6YWN0aW9uPSJzYXZlZCIgc3RFdnQ6aW5zdGFuY2VJRD0ieG1wLmlpZDpiNzc4YjE2NC1mODFhLWU5NDktOThmMC00OGQ0MDZlNmU3ZjYiIHN0RXZ0OndoZW49IjIwMTktMDYtMDRUMTc6NTU6MDcrMDg6MDAiIHN0RXZ0OnNvZnR3YXJlQWdlbnQ9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE5IChXaW5kb3dzKSIgc3RFdnQ6Y2hhbmdlZD0iLyIvPiA8cmRmOmxpIHN0RXZ0OmFjdGlvbj0ic2F2ZWQiIHN0RXZ0Omluc3RhbmNlSUQ9InhtcC5paWQ6ODliNGE5ZDItNmViMy01MjQ0LWIzODktNzM3ZmEwNTkzNWVhIiBzdEV2dDp3aGVuPSIyMDE5LTA2LTA1VDA5OjUwOjUwKzA4OjAwIiBzdEV2dDpzb2Z0d2FyZUFnZW50PSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOSAoV2luZG93cykiIHN0RXZ0OmNoYW5nZWQ9Ii8iLz4gPC9yZGY6U2VxPiA8L3htcE1NOkhpc3Rvcnk+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+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');
		&.luch-audio--btn-pause {
			background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFAAAABQCAYAAACOEfKtAAAACXBIWXMAAAsTAAALEwEAmpwYAAAGymlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDUgNzkuMTYzNDk5LCAyMDE4LzA4LzEzLTE2OjQwOjIyICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdEV2dD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlRXZlbnQjIiB4bWxuczpkYz0iaHR0cDovL3B1cmwub3JnL2RjL2VsZW1lbnRzLzEuMS8iIHhtbG5zOnBob3Rvc2hvcD0iaHR0cDovL25zLmFkb2JlLmNvbS9waG90b3Nob3AvMS4wLyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOSAoV2luZG93cykiIHhtcDpDcmVhdGVEYXRlPSIyMDE5LTA2LTA0VDE3OjU0OjM1KzA4OjAwIiB4bXA6TWV0YWRhdGFEYXRlPSIyMDE5LTA2LTA1VDA5OjUwOjI0KzA4OjAwIiB4bXA6TW9kaWZ5RGF0ZT0iMjAxOS0wNi0wNVQwOTo1MDoyNCswODowMCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDoyZGU0NjY1NS05N2I2LTBjNGMtYTQ4NS0wN2E1ZjY5ZjU1YjciIHhtcE1NOkRvY3VtZW50SUQ9ImFkb2JlOmRvY2lkOnBob3Rvc2hvcDpiYTUxOWZkZi0xYWQwLTYwNDUtOGIyNS1hNTZlOGM0YzhkZmQiIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpiZjRlOTFiYS1iYTBjLWVkNDMtOTc1Yi01YTcwODhmNzdiMWUiIGRjOmZvcm1hdD0iaW1hZ2UvcG5nIiBwaG90b3Nob3A6Q29sb3JNb2RlPSIzIiBwaG90b3Nob3A6SUNDUHJvZmlsZT0ic1JHQiBJRUM2MTk2Ni0yLjEiPiA8eG1wTU06SGlzdG9yeT4gPHJkZjpTZXE+IDxyZGY6bGkgc3RFdnQ6YWN0aW9uPSJjcmVhdGVkIiBzdEV2dDppbnN0YW5jZUlEPSJ4bXAuaWlkOmJmNGU5MWJhLWJhMGMtZWQ0My05NzViLTVhNzA4OGY3N2IxZSIgc3RFdnQ6d2hlbj0iMjAxOS0wNi0wNFQxNzo1NDozNSswODowMCIgc3RFdnQ6c29mdHdhcmVBZ2VudD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTkgKFdpbmRvd3MpIi8+IDxyZGY6bGkgc3RFdnQ6YWN0aW9uPSJzYXZlZCIgc3RFdnQ6aW5zdGFuY2VJRD0ieG1wLmlpZDoxY2U4ZTUwMi1iYzUwLThkNDktYTU0ZC04OWVjOGUzMDI5ZTIiIHN0RXZ0OndoZW49IjIwMTktMDYtMDRUMTc6NTQ6MzUrMDg6MDAiIHN0RXZ0OnNvZnR3YXJlQWdlbnQ9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE5IChXaW5kb3dzKSIgc3RFdnQ6Y2hhbmdlZD0iLyIvPiA8cmRmOmxpIHN0RXZ0OmFjdGlvbj0ic2F2ZWQiIHN0RXZ0Omluc3RhbmNlSUQ9InhtcC5paWQ6MmRlNDY2NTUtOTdiNi0wYzRjLWE0ODUtMDdhNWY2OWY1NWI3IiBzdEV2dDp3aGVuPSIyMDE5LTA2LTA1VDA5OjUwOjI0KzA4OjAwIiBzdEV2dDpzb2Z0d2FyZUFnZW50PSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOSAoV2luZG93cykiIHN0RXZ0OmNoYW5nZWQ9Ii8iLz4gPC9yZGY6U2VxPiA8L3htcE1NOkhpc3Rvcnk+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+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');
		}
	}
}
.luch-audio__audio-con {
	position: relative;
	flex: 1;
	display: flex;
	width: 0;
	padding: 0 15px;
	align-items: center;
	background-color: #fcfcfc;
	.luch-audio__audio-info {
		width: 100%;
	}
	.luch-audio__audio-title {
		display: block;
		padding-bottom: 7px;
		padding-right: 25px;
		font-size: 14px;
		color: #353535;
	}
	.luch-audio__audio-author {
		display: block;
		font-size: 12px;
		color: #888888;
	}
	.luch-audio__audio-time {
		position: absolute;
		right: 15px;
		top: 6px;
		font-size: 13px;
		color: #9d9d9d;
	}
}
</style>
