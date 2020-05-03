<template>
	<view class="wrapper">
		<map id="map" :latitude="latitude" :longitude="longitude" :polyline="aniPolyline" :scale="scale" style="width: 100vw; height: 100vh;"
		 show-location>
		</map>
		<view class="btn" style="background-color: #FFFFFF;padding: 10px;margin: 10px;">
			<view style="margin: 10px auto;">
				<text>里程数 : {{metters}}</text>
				<text> \n时间 : {{time}} </text>
				<text>\n配速 : {{speed}} </text>
			</view>
			<button type="default" @click="start">开始跑步</button>
			<button type="default" @click="stop">停止跑步</button>
			<!-- <button type="default" @click="openLocation">打开位置</button> -->
			<button type="default" @click="clearMsg">结束跑步</button>
		</view>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				mapCtx: '',
				latitude: 0,
				longitude: 0,
				time: "00:00:00",
				h: 0,
				m: 0,
				s: 0,
				sec: 0,
				aniPointsArr: [],
				aniPoints: [
					// 	{
					// 	latitude: 25.075309,
					// 	longitude: 102.712213,
					// }
				],
				aniPostions: [],
				aniPolyline: [],
				aniPolylineId: '',
				km: 0,
				metters: 0,
				inter: null
			}
		},
		onLoad() {
			// 使用 my.createMapContext 获取 map 上下文
			this.mapCtx = uni.createMapContext('map');
			my.showLoading();
			uni.getLocation({
				success:(res=>{
					console.log(res.latitude + res.longitude)
					my.hideLoading();
					this.latitude = res.latitude
					this.longitude = res.longitude
				}),
				fail:(res=>{
					my.hideLoading();
					my.alert({
						title: '定位失败'
					});
				})
			});
			this.aniPolyline = [{
				points: this.aniPoints,
				color: '#f00',
				width: 6,
				dottedLine: false,
				iconWidth: 6,
			}]
			// this.getLocation()
		},
		computed: {
			speed() {
				if (this.metters) {
					const dateFamt = n => n > 10 ? n : '0' + n
					var time, route, speed, search, rep, fltNum, fltStr, m, s, speedStr
					time = (this.h * 60) + this.m + (this.s / 60)
					route = this.metters
					speed = time / route
					search = speed.toString()
					rep = search.replace('.', "'")
					fltNum = search.search("'")
					fltStr = search.slice(fltNum)
					m = parseInt(speed)
					s = Number(fltStr) * 6
					speedStr = `${m}'${dateFamt(s)}"`
				}
				return speedStr ? speedStr : `0'00"`
			}
		},
		// updated() {
		// 	console.log("有用吗？")
		// 	// this.setPoly()
		// },
		methods: {
			// 调用支付宝地图定位
			getLocation(){
				my.getLocation({
					success: (res) => {
						console.log("支付宝地图----更新的位置",res.latitude + res.longitude)
						this.latitude = res.latitude
						this.longitude = res.longitude
					},
					fail: (res => {
						console.log("失败原因:", res)
						uni.hideLoading();
						uni.alert({
							title: '定位失败'
						});
					})
				})
			},
			// 开始跑步
			start() {
				const dateFamt = n => n > 10 ? n : '0' + n
				if (this.intervalId != null) return;
				this.intervalId = setInterval(() => {
					//实时更新定位
					this.getLocation()
					//计时器 累加时间 跑步时间
					this.sec++;
					let date = new Date(0, 0);
					date.setSeconds(this.sec);
					let h = date.getHours(),
						m = date.getMinutes(),
						s = date.getSeconds();
					this.h = h
					this.m = m
					this.s = s
					this.time = dateFamt(h) + ":" + dateFamt(m) + ":" + dateFamt(s);
				}, 1000);
				this.setPoly()
			},
			// 停止跑步
			stop() {
				clearInterval(this.intervalId)
				clearInterval(this.inter)
				this.intervalId = this.inter = null
			},
			//结束跑步
			clearMsg() {
				this.time = "00:00:00"
				this.metters = 0,
					clearInterval(this.intervalId)
				clearInterval(this.inter)
				this.intervalId = this.inter = null
			},
			// 开始跑步
			setPoly() {
				console.log('开始跑步+++++++++++++++++++++++++++==')
				if (this.inter != null) return
				this.inter = setInterval(() => {
					let covers, len, len_index
					let points = this.aniPoints;
					len = points.length
					len_index = points.length - 1
					this.aniPoints.push({
						latitude: this.latitude,
						longitude: this.longitude
					})
					console.log("更新到的位置 this.aniPoints:", this.aniPoints);
					for (var i = 0; i < this.aniPoints.length; i++) {
						this.aniPolylineId = this.aniPoints[i].__ob__.dep.id
					}
					console.log("更新到的id this.aniPolylineId:", this.aniPolylineId);
					console.log("画线集合 this.aniPolyline:", this.aniPolyline);
					console.log("points========", points);
					this.mapCtx.updateComponents({
						longitude: points.latitude,
						latitude: points.longitude,
						polyline: this.aniPolyline,
					});
					//当数组的长度大于2 从右往左取
					if (len > 2) {
						let lat1, lat2, lng1, lng2
						lat1 = points[len_index - 1]['latitude']
						lng1 = points[len_index - 1]['longitude']
						lat2 = points[len_index]['latitude']
						lng2 = points[len_index]['longitude']
						this.km = (this.GetDistance(lat1, lng1, lat2, lng2)).toFixed(2)
						this.metters += Number(this.km);
					}
				}, 5000)
			},

			//更新缩放比例
			scaleChange() {
				this.scale = 18
			},

			GetDistance(lat1, lng1, lat2, lng2) {
				const Rad = d => d * Math.PI / 180 //经纬度转换成三角函数中度分表形式。
				var radLat1 = Rad(lat1);
				var radLat2 = Rad(lat2);
				var a = radLat1 - radLat2;
				var b = Rad(lng1) - Rad(lng2);
				var s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2) + Math.cos(radLat1) * Math.cos(radLat2) * Math.pow(
					Math.sin(b / 2), 2)));
				s = s * 6378.137;
				s = (Math.round(s * 10000) / 10000) //输出为公里
				return s
			},
		}
	}
</script>

<style lang="scss">
	.wrapper {
		position: relative;

		#map,
		.btn {
			position: absolute;
			top: 0;
			left: 0;

			button {
				margin: 10px 0;
			}
		}
	}
</style>
