<template>
	<view class="list" v-show="isLoading">
		<view class="fixbg" :style="{'background-image':'url('+playlist.coverImgUrl+')'}"></view>
		<musichead title="歌单" :icon="true" color="white"></musichead>
		
		<view class="container">
			<scroll-view scroll-y="true" >
				<!-- 头部区域 -->
				<view class="list-head">
					<view class="list-head-img">
						<image :src="playlist.coverImgUrl" mode=""></image>
						<text class="iconfont iconyousanjiao">{{playlist.playCount | formatCount}}</text>
					</view>
					<view class="list-head-text">
						<view>{{playlist.name}}</view>
						<view>
							<image :src="playlist.creator.avatarUrl" mode=""></image>{{playlist.creator.nickname}}
						</view>
						<view>{{playlist.description}}</view>
					</view>
				</view>
				
				<!-- 只有小程序才会显示 -->
				<!-- #ifdef MP-WEIXIN -->
					<button class="list-share" open-type="share">
						<text class="iconfont iconicon-"></text>分享给微信好友
					</button><strong></strong>
				<!-- #endif -->
				
				<!-- 内容主体区域 -->
				<view class="list-music">
					<view class="list-music-head">
						<text class="iconfont iconbofang1"></text>
						<text>播放全部</text>
						<text>(共{{playlist.trackCount}}首)</text>
					</view>
					<view class="list-music-item" v-for="(item,index) in playlist.tracks" :key="index" @tap="handleToDetail(item.id)">
						<view class="list-music-top">{{index + 1}}</view>
						<view class="list-music-song">
							<view>{{item.name}}</view>
							<view>
								<image v-if="privileges[index].flag==128" src="../../static/dujia.png" mode=""></image>
								<image v-if="privileges[index].maxbr==999000" src="../../static/sq.png" mode=""></image>
								{{item.ar[0].name}} - {{item.name}}
							</view>
						</view>
						<text class="iconfont iconbofang"></text>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import musichead from '../../components/musichead/musichead.vue'
	import {list} from '../../common/api.js'
	
	
	export default {
		data() {
			return {
				playlist:{
					creator:{
						avatarUrl:"",
						nickname:""
					},
					coverImgUrl:"",
					playCount:"",
					trackCount:"",
					tracks:[],
					name:"",
					description:"",
				},
				privileges:[],
				isLoading:false,
			}
		},
		onLoad(options) {
			uni.showLoading({
				title:"加载中..."
			});
			list(options.id).then(res=>{
				if(res[1].data.code=="200"){
					this.playlist=res[1].data.playlist;
					this.privileges=res[1].data.privileges;
					this.$store.commit("INIT_TOPLISTIDS",res[1].data.playlist.trackIds)
					this.isLoading=true;
					uni.hideLoading()
				}
			})
		},
		methods: {
			handleToDetail(songId){
				uni.navigateTo({
					url:"/pages/detail/detail?songId="+songId,
				})
			}
		},
		comments:{
			musichead
		}
	}
</script>

<style scoped>
	.list-head{
		display: flex;
		margin: 30rpx;
	}
	.list-head-img{
		width: 264rpx;
		height: 264rpx;
		border-radius: 30rpx;
		overflow: hidden;
		position: relative;
		margin-right: 42rpx;
	}
	.list-head-img image{
		width: 100%;
		height: 100%;
	}
	.list-head-img text{
		position: absolute;
		right: 8rpx;
		top: 8rpx;
		color: white;
		font-size: 26rpx;
	}
	.list-head-text{
		width: 300rpx;
		word-wrap:break-word; 
		word-break:break-all; 
		overflow: hidden;
		color: #f0f2f7;
	}
	.list-head-text view:nth-child(1){
		color: white;
		font-size: 34rpx;
	}
	.list-head-text view:nth-child(2){
		display: flex;
		margin: 20rpx 0;
		font-size: 24rpx;
		align-items: center;
	}
	.list-head-text view:nth-child(2) image{
		width: 50rpx;
		height: 50rpx;
		border-radius: 50%;
		margin-right: 14rpx;
	}
	.list-head-text view:nth-child(3){
		line-height: 30rpx;
		font-size: 22rpx;
	}
	.list-share{
		width: 330rpx;
		height: 74rpx;
		margin: 0 auto;
		background: rgba(0,0,0,0.4);
		border-radius: 37rpx;
		color: white;
		text-align: center;
		line-height: 74rpx;
		font-size: 28rpx;
	}
	.list-share text{
		margin-right: 16rpx;
	}
	.list-music{background: white; border-radius: 50rpx 50rpx 0 0; margin-top: 40rpx; overflow: hidden;}
	.list-music-head{height: 50rpx; margin: 30rpx 0 70rpx 20rpx;}
	.list-music-head text:nth-child(1){height: 25rpx; line-height: 25rpx; font-size: 50rpx; margin: 0 10rpx 0 25rpx; vertical-align: middle;}
	.list-music-head text:nth-child(2){font-size: 30rpx;}
	.list-music-head text:nth-child(3){ font-size: 26rpx; color: #b2b2b2}
	.list-music-item{display: flex; margin: 0 30rpx 70rpx 45rpx; align-items: center; color: #959595}
	.list-music-top{width: 60rpx; font-size: 35rpx; line-height: 35rpx;}
	.list-music-song{flex: 1;}
	.list-music-song view:nth-child(1){font-size: 30rpx; color: black;  width: 70vw; overflow: hidden; white-space: nowrap; text-overflow: ellipsis;}
	.list-music-song view:nth-child(2){font-size: 20rpx; align-items: center; width: 70vw; overflow: hidden; white-space: nowrap; text-overflow: ellipsis;}
	.list-music-song view:nth-child(2) image{width: 32rpx; height: 20rpx; margin-right: 10rpx; flex-shrink: 0;}
	.list-music-item text{font-size: 50rpx; color: #c7c7c7;}
</style>
