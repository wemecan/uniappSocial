<template>
	<view>
		<view class="flex align-center justify-center" v-if="providerList.length > 0">
			<view style="height: 1rpx;background-color: #dddddd;width: 100rpx;"></view>
			<view class="mx-2 text-muted">社交账号登录</view>
			<view style="height: 1rpx;background-color: #dddddd;width: 100rpx;"></view>
		</view>
		<view class="flex align-center px-5 py-3">

			<view class="flex-1 flex align-center justify-center" v-for="(item,index) in providerList" :key="index" @click="login(item)">
				<view :class="item.icon + ' '+item.bgColor" class="iconfont font-lg text-white flex align-center justify-center rounded-circle"
				 style="width: 100rpx;height: 100rpx;"></view>
			</view>

		</view>
	</view>

</template>

<script>
	export default {
		props: {
			back: {
				type: Boolean,
				default: false
			},
		},
		data() {
			return {
				providerList: []
			}
		},
		methods: {
			login(item) {			
				uni.login({
					provider: item.id,
					success: res => {
						uni.getUserInfo({
							provider: item.id,
							success: (infoRes) => {
								let obj = {
									provider: item.id,
									openid: infoRes.userInfo.openId,
									expires_in: 0,
									nickName: infoRes.userInfo.nickName,
									avatarUrl: infoRes.userInfo.avatarUrl,
								}
								this.loginEvent(obj)
							}
						});
					},
					fail: () => {
						uni.showToast({
							title: '登录失败',
							icon: 'none'
						});
					},
				});
			},
			loginEvent(data) {
				this.$H.post('/user/otherlogin', data)
					.then(res => {
						// 修改vuex的state,持久化存储
						this.$store.commit('login', this.$U.formatUserinfo(res))
						// 返回上一页
					uni.reLaunch({
						url:"../../pages/user/user",
						success:()=>{
							uni.showToast({
								title: '登录成功',
								icon: 'none'
							});
						}
					});
						
					})
			}
		},
		mounted() {
			uni.getProvider({
				service: 'oauth',
				success: (result) => {
					this.providerList = result.provider.map((value) => {
						let providerName = '';
						let icon = ''
						let bgColor = ''
						switch (value) {
							case 'weixin':
								providerName = '微信登录'
								icon = 'icon-weixin'
								bgColor = 'bg-success'
								break;
							case 'qq':
								providerName = 'QQ登录'
								icon = 'icon-QQ'
								bgColor = 'bg-primary'
								break;
							case 'sinaweibo':
								providerName = '新浪微博登录'
								icon = 'icon-xinlangweibo'
								bgColor = 'bg-warning'
								break;
						}
						return {
							name: providerName,
							id: value,
							icon: icon,
							bgColor: bgColor
						}
					});

				},
				fail: (error) => {
					console.log('获取登录通道失败', error);
				}
			});
		}
	}
</script>

<style>

</style>
