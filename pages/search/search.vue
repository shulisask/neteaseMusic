<template>
	<view class="search">
		<musichead title="搜索" :icon="true" :iconblack="true"></musichead>
		<view class="container">
			<scroll-view scroll-y="true" >
				<!-- 搜索框 -->
				<view class="search-search">
					<text class="iconfont iconsearch"></text>
					<input type="text" placeholder="搜索歌曲" v-model="searchWord" @confirm="handleToSearch(searchWord)" @input="handleToSuggest"/>
					<text class="iconfont iconguanbi" v-show="searchType!==1" @tap="handleToClose"></text>
				</view>
				<block v-if="searchType===1">
					<!-- 历史记录 -->
					<view class="search-history">
						<view class="search-history-head">
							<text>历史记录</text>
							<text class="iconfont iconlajitong" @tap="handleToClear"></text>
						</view>
						<view class="search-history-list">
							<view v-for="(item,index) in searchHistory" :key="index" @tap="handleToWord(item)">{{item}}</view>
						</view>
					</view>
				</block>
				<block v-else-if="searchType===2">
					<view class="search-result">
						<view class="search-result-item" v-for="(item,index) in searchList" :key="index" @tap="handleToDetail(item.id)">
							<view class="search-result-word">
								<view>{{item.name}}</view>
								<view>{{item.ar[0].name}} - {{item.al.name}}</view>
							</view>
							<text class="iconfont iconbofang"></text>
						</view>
					</view>
				</block>
				<block v-else-if="searchType===3">
					<view class="search-suggest">
						<view class="search-suggest-head">搜索"{{searchWord}}"</view>
						<view class="search-suggest-item" v-for="(item,index) in searchSuggest" :key="index" @tap="handleToWord(item.keyword)">
							<text class="iconfont iconsearch"></text>{{item.keyword}}
						</view>
					</view>
				</block>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import musichead from "../../components/musichead/musichead.vue"
	import "@/common/iconfont.css"
	import {searchWord,searchSuggest} from "../../common/api.js"
	
	export default {
		data() {
			return {
				searchWord:"",
				searchHistory:[],
				searchType:1,
				searchList:[],
				searchSuggest:[],
			}
		},
		onLoad() {
			uni.getStorage({
				key:"searchHistory",
				success: (res) => {
					this.searchHistory=res.data;
				}
			})
		},
		components:{
			musichead
		},
		methods: {
			handleToWord(word){
				this.searchWord=word;
				this.getSearchList(word)
			},
			handleToSearch(word){
				this.searchHistory.unshift(word);
				this.searchHistory=[...new Set(this.searchHistory)]
				if(this.searchHistory.length>10){
					this.searchHistory=10;
				}
				uni.setStorage({
					key:"searchHistory",
					data:this.searchHistory
				});
				this.getSearchList(word);
			},
			handleToClear(){
				uni.removeStorage({
					key:"searchHistory",
					success: (res) => {
						this.searchHistory=[];
					}
				});
			},
			getSearchList(word){
				searchWord(word).then(res=>{
					if(res[1].data.code===200){
						this.searchList=res[1].data.result.songs;
						this.searchType=2;
					}
				})
			},
			handleToClose(){
				this.searchWord="";
				this.searchType=1;
			},
			handleToDetail(songId){
				uni.navigateTo({
					url:'/pages/detail/detail?songId='+songId
				})
			},
			handleToSuggest(event){
				// 拿到当前输入的值
				let value=event.detail.value;
				if(!value){
					this.searchType=1;
					return;
				}
				searchSuggest(value).then(res=>{
					if(res[1].data.code===200){
						this.searchSuggest=res[1].data.result.allMatch;
						this.searchType=3;
					}
				});
			}
		}
	}
</script>

<style scoped>
	.search-search{
		height: 70rpx;
		margin: 50rpx 30rpx 30rpx 30rpx;
		background: #f7f7f7;
		border-radius: 50px;
		display: flex;
		align-items: center;
	}
	.search-search text{
		margin: 0 26rpx;
	}
	.search-search input{
		flex: 1;
		font-size: 26rpx;
	}
	
	.search-history{
		margin: 0 30rpx;
		font-size: 26rpx;
	}
	.search-history-head{
		display: flex;
		margin-bottom: 36rpx;
		justify-content: space-between;
	}
	.search-history-list{
		display: flex;
		flex-wrap: wrap;
	}
	.search-history-list view{
		padding: 16rpx 28rpx;
		border-radius: 40rpx;
		margin-right: 30rpx;
		margin-bottom: 30rpx;
		background: #f7f7f7;
	}
	
	.search-result{ border-top: 2rpx #e5e5e5 solid; padding:30rpx;}
	.search-result-item{ display: flex; align-items: center; border-bottom: 2rpx #e5e5e5 solid; padding-bottom:30rpx; margin-bottom: 30rpx;}
	.search-result-item text{ font-size:50rpx;}
	.search-result-word{ flex:1;}
	.search-result-word view:nth-child(1){ font-size:28rpx; color:#3e6694;}
	.search-result-word view:nth-child(2){ font-size:26rpx;}
		
	.search-suggest{ border-top: 2rpx #e5e5e5 solid; padding:30rpx; font-size:26rpx; }
	.search-suggest-head{ color:#537caa; margin-bottom: 40rpx;}
	.search-suggest-item{ color:#666666; margin-bottom: 70rpx;}
	.search-suggest-item text{ color:#c2c2c2; font-size:26rpx; margin-right:26rpx;}
</style>
