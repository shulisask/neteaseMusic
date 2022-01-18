<template>
	<view class="detail">
		<view class="fixbg" :style="{'background-image':'url('+songDetail.al.picUrl+')'}"></view>
		<musichead :title="songDetail.name" :icon="true" color="white"></musichead>
		
		<view class="container" v-show="!isLoading">
			<scroll-view scroll-y="true" >
				<!-- 头部图标布局 -->
				<view class="detail-play" @tap="handleToPlay">
					<image :src="songDetail.al.picUrl" :class="{'detail-play-run':isPlayRotate}"></image>
					<text class="iconfont" :class="iconplay"></text>
					<view></view>
				</view>
				<!-- 歌词布局 -->
				<view class="detail-lyric">
					<view class="detail-lyric-warp" :style="{transform:'translateY('+ -(lyricIndex-1)*82 +'rpx)'}">
						<view class="detail-lyric-item" v-for="(item,index) in songLyric" :key="index"  :class="{active:lyricIndex==index}">{{item.lyric}}</view>
					</view>
				</view>
				<!-- 喜欢这首歌的人也听布局 -->
				<view class="detail-like">
					<view class="detail-like-head">喜欢这首歌的人也听</view>
					<view class="detail-like-item" v-for="(item,index) in songSimi" :key="index" @tap="handleToSimi(item.id)">
						<view class="detail-like-img">
							<image :src="item.album.picUrl" mode=""></image>
						</view>
						<view class="detail-like-song">
							<view>{{item.name}}</view>
							<view>
								<image v-if="item.privilege.flag==128" src="../../static/dujia.png" mode=""></image>
								<image v-if="item.privilege.maxbr==999000" src="../../static/sq.png" mode=""></image>
								{{item.album.artists[0].name}} - {{item.name}}
							</view>
						</view>
						<text class="iconfont iconbofang"></text>
					</view>		
				</view>
				<!-- 评论区 -->
				<view class="detail-comment">
					<view class="detail-comment-head">精彩评论</view>
					<view class="detail-comment-item" v-for="(item,index) in songHotComments" :key="index">
						<!-- 头像区域 -->
						<view class="detail-comment-img">
							<image :src="item.user.avatarUrl"></image>
						</view>
						<!-- 内容区域 -->
						<view class="detail-comment-content">
							<view class="detail-comment-title">
								<!-- 名称时间 -->
								<view class="detail-comment-name">
									<view>{{item.user.nickname}}</view>
									<view>{{item.time | formatTime}}</view>
								</view>
								<!-- 点赞数 -->
								<view class="detail-comment-like">
									{{item.likedCount | formatCount}} <text class="iconfont iconlike"></text>
								</view>
							</view>
							<!-- 评论 -->
							<view class="detail-comment-text">{{item.content}}</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import musichead from '../../components/musichead/musichead.vue'
	import {songDetail,songSimi,songComment,songLyric,songUrl} from '../../common/api.js'
	
	export default {
		data() {
			return {
				songDetail:{
					al:{
						picUrl:""
					},
					name:""
				},
				songSimi:[],
				songHotComments:[],
				songLyric:[],
				lyricIndex:0,
				iconplay:"iconpause",
				isPlayRotate:true,
				isLoading:true
			}
		},
		onLoad(options) {
			this.getMusic(options.songId)
		},
		onUnload() {
			this.clearLyricIndex();
			// #ifdef H5
				this.bgAudioManager.destroy();
			// #endif
		},
		onHide(){
			this.clearLyricIndex();
			// #ifdef H5
				this.bgAudioManager.destroy();
			// #endif
		},
		methods: {
			getMusic(songId){
				
				this.$store.commit("NEXT_ID",songId);
				
				uni.showLoading({
					title:"加载中...",
				})
				this.isLoading=true;
				
				Promise.all( [songDetail(songId),songSimi(songId),songComment(songId),songLyric(songId),songUrl(songId) ]).then((res)=>{
					// 0歌曲详情 1相似歌曲 2评论 3歌词 4歌曲地址
					if(res[0][1].data.code=="200"){
						this.songDetail=res[0][1].data.songs[0];
					}
					if(res[1][1].data.code=="200"){
						this.songSimi=res[1][1].data.songs;
					}
					if(res[2][1].data.code=="200"){
						this.songHotComments=res[2][1].data.hotComments;
					}
					if(res[3][1].data.code=="200"){
						const lyric=res[3][1].data.lrc.lyric;
						const result = [];
						const re = /\[([^\]]+)\]([^[]+)/g;
						lyric.replace(re,($0,$1,$2)=>{
							result.push({ time : this.formatTimeToSec($1) , lyric : $2 });
						});
						this.songLyric=result;
					}
					if(res[4][1].data.code=="200"){
						
						// #ifdef MP-WEIXIN
							this.bgAudioManager = uni.getBackgroundAudioManager();
							this.bgAudioManager.title = this.songDetail.name;
						// #endif
						
						// #ifdef H5
							if(!this.bgAudioManager){
								this.bgAudioManager = uni.createInnerAudioContext();
							}
							this.iconplay="iconbofang1";
							this.isPlayRotate = false;
						// #endif
						
						// console.log(this.songLyric)

						this.bgAudioManager.src=res[4][1].data.data[0].url || "";
						this.listenLyricIndex();
						// 开始
						this.bgAudioManager.onPlay(()=>{
							this.iconplay="iconpause",
							this.isPlayRotate=true;
							this.listenLyricIndex()
						})
						// 暂停
						this.bgAudioManager.onPause(()=>{
							this.iconplay="iconbofang1";
							this.isPlayRotate=false;
							this.clearLyricIndex();
						})
						// 下一首歌播放
						this.bgAudioManager.onEnded(()=>{
							this.getMusic(this.$store.state.nextId)
						})
					}
					this.isLoading=false;
					uni.hideLoading();
				})
			},
			formatTimeToSec(value){
				const arr=value.split(":")
				return (Number(arr[0] * 60) + Number(arr[1])).toFixed(2);
			},
			handleToPlay(){
				if(this.bgAudioManager.paused){
					this.bgAudioManager.play();
				}else{
					this.bgAudioManager.pause();
				}
			},
			listenLyricIndex(){
				clearInterval(this.timer);
				this.timer=setInterval(()=>{
					for(let i=0;i<this.songLyric.length;i++){
						if(this.bgAudioManager.currentTime>this.songLyric[this.songLyric.length-1].time){
							this.lyricIndex=this.songLyric.length-1;
							break;
						}
						if(this.bgAudioManager.currentTime>this.songLyric[i].time&&this.bgAudioManager.currentTime<this.songLyric[i+1].time){
							this.lyricIndex=i;
						}
					}
					// console.log(this.lyricIndex)
				},500)
			},
			clearLyricIndex(){
				clearInterval(this.timer)
			},
			handleToSimi(songId){
				this.getMusic(songId)
			}
			
		},
		components:{
			musichead,
		}
	}
</script>

<style scoped>
	.detail-play{width: 580rpx; height: 580rpx; background: url(~@/static/disc.png);background-size: cover; margin: 214rpx auto 0 auto; position: relative;}
	.detail-play image{width: 370rpx; height: 370rpx; border-radius: 50%; position: absolute; left: 0; top: 0; bottom: 0; right: 0; margin: auto; animation: 20s linear move infinite; animation-play-state: paused;}
	.detail-play .detail-play-run{animation-play-state: running;}
	@keyframes move{
		from{transform: rotate(0deg);}
		to{transform: rotate(360deg);}
	}
	.detail-play text{width: 100rpx; height: 100rpx; font-size: 100rpx; color: white; position: absolute; left: 0; top: 0; bottom: 0; right: 0; margin: auto;}
	.detail-play view{width: 230rpx; height: 360rpx; background: url(~@/static/needle.png); position: absolute; left: 0; right: 0; margin: auto; background-size: cover; transform: translate(30%,-50%);}
	
	.detail-lyric{font-size: 32rpx; line-height: 82rpx; height: 246rpx; text-align: center; overflow: hidden; color: #34495e;}
	.detail-lyric-warp{transition: 0.5s;}
	.detail-lyric-item{height: 82rpx;}
	.detail-lyric-item.active{color: #006BA3;}
	
	.detail-like{margin: 0 30rpx;}
	.detail-like-head{font-size: 36rpx; color: #2c3e50; margin: 50rpx 0;}
	.detail-like-item{display: flex; align-items: center; margin-bottom: 28rpx;}
	.detail-like-img{width: 82rpx; height: 82rpx; border-radius: 20rpx; overflow: hidden; margin-right: 20rpx;}
	.detail-like-img image{width: 100%; height: 100%;}
	.detail-like-song{flex: 1; color: #34495e;}
	.detail-like-song view:nth-child(1){font-size: 30rpx; color: #2c3e50; margin-bottom: 10rpx;}
	.detail-like-song view:nth-child(2){font-size: 22rpx;}
	.detail-like-song image{width: 26rpx; height: 20rpx; margin-right: 10rpx;}
	.detail-like-item text{font-size: 50rpx;}
	
	.detail-comment{margin: 0 30rpx;}
	.detail-comment-head{font-size: 36rpx; color: #2c3e50; margin: 50rpx 0;}
	.detail-comment-item{margin-bottom: 28rpx; display: flex;}
	.detail-comment-img{width: 64rpx; height: 64rpx; border-radius: 50%; overflow: hidden; margin-right: 18rpx;}
	.detail-comment-img image{width: 100%; height: 100%;}
	.detail-comment-content{flex: 1; color: #34495e}
	.detail-comment-title{display: flex; justify-content: space-between;}
	.detail-comment-name view:nth-child(1){font-size: 26rpx;}
	.detail-comment-name view:nth-child(2){font-size: 20rpx;}
	.detail-comment-like{font-size: 28rpx;}
	.detail-comment-text{font-size: 28rpx; line-height: 40rpx; color: #2c3e50; margin-top: 20rpx; border-bottom: 1px #34495e solid; padding-bottom: 20rpx;}
</style>
