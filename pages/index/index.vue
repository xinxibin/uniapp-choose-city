<template>
	<view class="content">
		<!-- <view class="top-view"></view> -->
		<ss-select-city :hotCitys="hotCitys" v-model="locationValue" :locationValue="locationValue" @on-select="selectValue" />
	</view>
</template>

<script>
	import ssSelectCity from '@/components/ss-select-city/ss-select-city.vue'
	export default {
		components: {
			ssSelectCity
		},
		data() {
			return {
				hotCitys: ['杭州', '天津', '北京', '上海', '深圳', '广州', '成都', '重庆', '厦门'],
				locationValue: '正在定位...',
				// 页面来源
				sourctType: ''
			}
		},
		onLoad(options) {
			let that = this
			this.sourctType = options.sourctType
			if (this.sourctType == 'pushvideo' || this.sourctType == 'pushvideo') {
				uni.setNavigationBarTitle({
					title: '选择工作地点'
				})
			} 
			setTimeout(function() {
				that.getLocation()
			}, 1000)
		},
		methods: {
			getLocation: function() {
				let that = this
				uni.getLocation({
					type: 'wgs84',
					geocode: true,
					success: function(res) {
						console.log(res);
						var cityTitle = (res.address && res.address.city) || '';
						if (cityTitle.length > 0) {
							let lastChares = cityTitle.substring(cityTitle.length-1);
							if (lastChares == "市")  {
								cityTitle = cityTitle.substring(0, cityTitle.length - 1);
							}
							that.locationValue = cityTitle
							uni.$emit('upadteLoaction', cityTitle)
						} else {
							let latitude = res.latitude
							let longitude = res.longitude
							uni.request({
								header: {
									"Content-Type": "application/text"
								},
								url: 'http://apis.map.qq.com/ws/geocoder/v1/?location=' + latitude + ',' + longitude +
									'&key=52UBZ-W6564-G3GUG-XQRQM-IXOU6-DCFOR',
								success(re) {
									console.log("中文位置")
									console.log(JSON.stringify(re))
									if (re.statusCode === 200) {
										let tmpCity = (re.data.result && re.data.result.address_component && re.data.result.address_component.city) ||
											'';
										let lastChares = tmpCity.substring(tmpCity.length - 1);
										if (lastChares == "市") {
											tmpCity = tmpCity.substring(0, tmpCity.length - 1);
										}
										that.locationValue = tmpCity
										uni.$emit('upadteLoaction', tmpCity)
									}
								}
							});
						}
					},
					fail: (err) => {
						console.log(err);
					}
				});
			},
			selectValue(city) {
				console.log(city)
				this.locationValue = city
				// 发职位
				if (this.sourctType == "publish") {
					uni.$emit('publishCity', city)
				} else if (this.sourctType == 'pushvideo') {
					console.log("发视频选择地点");
					getApp().globalData.pushVideoLocation = city
				} else { // 更新首页位置
					uni.$emit('updateHomeLocation', city)
				}
				
				uni.showToast({
					title:city,
					icon: 'none'
				})
			}
		}
	}
</script>

<style>
	.content {
		/* padding-top: 30rpx; */
		background-color: rgba(34, 33, 33, 1);
	}
	
</style>
