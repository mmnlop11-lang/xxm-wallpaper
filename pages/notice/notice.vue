<template>
	<view class="noticeLayout">
		<!-- 页面加载中，数据未到达 -->
		<view class="loadingLayout" v-if="!noticeList.length && !noData">
			<uni-load-more status="loading"></uni-load-more>
		</view>

		<!-- 公告列表 -->
		<view class="content" v-if="noticeList.length">
			<view
				class="item"
				v-for="item in noticeList"
				:key="item._id"
				@click="routerTo('/pages/notice/detail?id=' + item._id)"
			>
				<view class="card">
					<view class="title">
						<!-- 置顶标记 -->
						<view class="tag" v-if="item.select">
							<uni-tag inverted text="置顶" type="error" />
						</view>
						<view class="font">{{ item.title }}</view>
					</view>
					<view class="info">
						<view class="item">{{ item.author }}</view>
						<view class="item">
							<uni-dateformat :date="item.publish_date" format="yyyy-MM-dd hh:mm"></uni-dateformat>
						</view>
						<view class="item">阅读 {{ item.view_count }}</view>
					</view>
				</view>
			</view>
		</view>

		<!-- 底部加载状态 -->
		<view class="loadingLayout" v-if="noticeList.length || noData">
			<uni-load-more :status="noData ? 'noMore' : 'loading'"></uni-load-more>
		</view>

		<view class="safe-area-inset-bottom"></view>
	</view>
</template>

<script setup>
	import { ref } from 'vue';
	import { onLoad, onReachBottom, onShareAppMessage, onShareTimeline } from '@dcloudio/uni-app';
	import { apiGetNotice } from '@/api/apis.js';
	import { routerTo } from '@/utils/common.js';

	// 公告列表数据
	const noticeList = ref([]);
	// 是否没有更多数据
	const noData = ref(false);
	// 分页参数
	const queryParams = {
		pageNum: 1,
		pageSize: 10
	};

	// 页面加载时获取数据
	onLoad(() => {
		getNoticeList();
	});

	// 获取公告列表
	const getNoticeList = async () => {
		try {
			const res = await apiGetNotice(queryParams);
			// 追加数据到列表
			noticeList.value = [...noticeList.value, ...res.data];
			// 返回数据少于请求条数，说明没有更多了
			if (queryParams.pageSize > res.data.length) {
				noData.value = true;
			}
		} catch (err) {
			console.error('获取公告列表失败', err);
			uni.showToast({
				title: '加载失败，请下拉刷新',
				icon: 'none'
			});
		}
	};

	// 触底加载更多
	onReachBottom(() => {
		if (noData.value) return;
		queryParams.pageNum++;
		getNoticeList();
	});

	// 分享给好友
	onShareAppMessage(() => {
		return {
			title: '咸虾米壁纸 - 公告',
			path: '/pages/notice/notice'
		};
	});

	// 分享朋友圈
	onShareTimeline(() => {
		return {
			title: '咸虾米壁纸 - 公告'
		};
	});
</script>

<style lang="scss" scoped>
	.noticeLayout {
		min-height: 100vh;
		background-color: #f8f8f8;

		.content {
			padding: 20rpx 30rpx;

			.item {
				margin-bottom: 20rpx;

				.card {
					background: #fff;
					border-radius: 16rpx;
					padding: 30rpx;
					box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.04);
					transition: transform 0.2s;

					&:active {
						transform: scale(0.98);
					}

					.title {
						display: flex;
						align-items: flex-start;
						font-size: 32rpx;
						color: #222;
						font-weight: 500;
						line-height: 1.5em;
						padding-bottom: 20rpx;

						.tag {
							transform: scale(0.82);
							transform-origin: left center;
							flex-shrink: 0;
						}

						.font {
							padding-left: 6rpx;
						}
					}

					.info {
						display: flex;
						align-items: center;
						color: #999;
						font-size: 24rpx;
						padding-top: 10rpx;
						border-top: 1rpx solid #f5f5f5;

						.item {
							margin-right: 24rpx;
							margin-bottom: 0;
						}
					}
				}
			}
		}
	}
</style>
