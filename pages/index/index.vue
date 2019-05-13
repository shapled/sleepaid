<template>
	<view>
		<view class="content">
			<view class="container">
				<view class="title">
					<text>{{title}}</text>
				</view>
				<view v-if="playing" class="player pause" @click="togglePlay"></view>
				<view v-else class="player play" @click="togglePlay"></view>
				<view class="uni-title">雨声</view>
				<slider :value="rainDefaultVolume" @changing="changeRainVolume" min="0" max="100" activeColor="#e9e9e9" backgroundColor="gray"/>
				<view class="uni-title">钢琴声</view>
				<slider :value="pianoDefaultVolume" @changing="changePianoVolume" min="0" max="100" activeColor="#e9e9e9" backgroundColor="gray"/>
				<view v-if="remain === 100" class="uni-title">一直听</view>
				<view v-else-if="remain === 0" class="uni-title">已停止</view>
				<view v-else class="uni-title">{{remain}}分钟</view>
				<slider :value="remain" @changing="changeRemain" @change="changeRemainDone" min="0" max="100" activeColor="#e9e9e9" backgroundColor="gray"/>
			</view>
		</view>
		<view class="support" @click="showSupport">支持</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				rainPlayer: null,
				pianoPlayer: null,
				title: '助眠小乐',
				playing: false,
				rainDefaultVolume: 40,
				pianoDefaultVolume: 20,
				remain: 100,
				stopTask: null,
				shouldResume: false,
			}
		},
		onLoad() {
		},
		onHide() {
			if(this.playing) {
				this.togglePlay();
				this.shouldResume = true;
			}
		},
		onShow() {
			if(this.shouldResume) {
				this.togglePlay();
				this.shouldResume = false;
			}
		},
		methods: {
			togglePlay() {
				this.playing = !this.playing;
				if(this.playing) {
					this.playRain();
					this.playPiano();
					if(this.remain === 0) {
						this.remain = 100;
					}
					uni.setKeepScreenOn({keepScreenOn: true});
				} else {
					this.rainPlayer.pause();
					this.pianoPlayer.pause();
					uni.setKeepScreenOn({keepScreenOn: false});
				}
			},
			playRain() {
				if(!this.rainPlayer) {
					console.log("初始化 rain player.")
					const innerAudioContext = uni.createInnerAudioContext();
					innerAudioContext.autoplay = false;
					innerAudioContext.loop = true;
					innerAudioContext.volume = this.rainDefaultVolume / 100;
					innerAudioContext.src = 'http://sleepaid.oss.shapled.com/rain2.wav';
					innerAudioContext.onPlay(() => {
					  // console.log('开始播放 rain');
					});
					innerAudioContext.onError((res) => {
					  console.log(res.errMsg);
					  console.log(res.errCode);
					});
					this.rainPlayer = innerAudioContext;
				}
				this.rainPlayer.play();
			},
			playPiano() {
				if(!this.pianoPlayer) {
					console.log("初始化 piano player.")
					const innerAudioContext = uni.createInnerAudioContext();
					innerAudioContext.autoplay = false;
					innerAudioContext.loop = true;
					innerAudioContext.volume = this.pianoDefaultVolume / 100;
					innerAudioContext.src = 'http://sleepaid.oss.shapled.com/piano.mp3';
					innerAudioContext.onPlay(() => {
					  // console.log('开始播放 piano');
					});
					innerAudioContext.onError((res) => {
					  console.log(res.errMsg);
					  console.log(res.errCode);
					});
					this.pianoPlayer = innerAudioContext;
				}
				this.pianoPlayer.play();
			},
			changeRainVolume(e) {
				const volume = e.target.value;
				this.rainDefaultVolume = volume;
				if(this.rainPlayer) {
					this.rainPlayer.volume = volume / 100;
				}
			},
			changePianoVolume(e) {
				const volume = e.target.value;
				this.pianoDefaultVolume = volume;
				if(this.pianoPlayer) {
					this.pianoPlayer.volume = volume / 100;
				}
			},
			changeRemain(e) {
				const remain = e.target.value;
				if(remain === 0) {
					this.remain = 1;
				} else {
					this.remain = remain;
				}
			},
			changeRemainDone(e) {
				if(this.stopTask) {
					clearInterval(this.stopTask);
					this.stopTask = null;
				}
				if(this.remain !== 100) {
					this.stopTask = setInterval(() => {
						this.remain -= 1;
						if(this.remain === 0) {
							clearInterval(this.stopTask);
							this.stopTask = null;
							this.togglePlay();
						}
					}, 60000);
				}
			},
			showSupport() {
				uni.previewImage({
					urls: ["http://sleepaid.oss.shapled.com/support-me2.jpg"]
				})
			}
		}
	}
</script>

<style>
	.content {
		text-align: center;
		height: 100vh;
		position: relative;
	}
	
	.support {
		color: gray;
		position: fixed;
		right: 0;
		bottom: 61.8%;
		border: 10upx solid gray;
		border-right: none;
		border-radius: 50upx 0 0 50upx;
		padding: 10upx;
	}
	
	.content::before {
		content: "";
		display: block;
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: url(http://sleepaid.oss.shapled.com/street.jpg) no-repeat;
		background-size: auto 100%;
		opacity: 0.62;
		z-index: 0;
	}

	.container {
		width: 62%;
		height: 100%;
		margin: 0 auto;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		position: relative;
		z-index: 1;
	}
	
	slider {
		width: 100%;
	}

	.player {
		width: 250upx;
		height: 250upx;
		margin: 50upx 0;
	}
	
	.play {
		background: url(http://sleepaid.oss.shapled.com/play-circle.svg);
		background-repeat: no-repeat;
		background-size: 100% 100%;
	}
	
	.pause {
		background: url(http://sleepaid.oss.shapled.com/timeout.svg);
		background-repeat: no-repeat;
		background-size: 100% 100%;
	}

	.title {
		font-size: 50upx;
		text-shadow: 1px 1px rgba(150, 150, 150, 0.5), -1px -1px rgba(150, 150, 150, 0.5);
	}
	
	.uni-title {
		text-shadow: 1px 1px rgba(150, 150, 150, 0.5), -1px -1px rgba(150, 150, 150, 0.5);
	}
</style>
