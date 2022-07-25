<template>
	<div id="app">
		<div class="container" ref="container">
			<div class="wrapper" ref="wrapper" >
				<div class="wrapper-item">
					<div v-for="item in lastWeekList" :key="item.timeStamp" class="inner">
						{{ item.showDate }}
					</div>
				</div>
				<div class="wrapper-item">
					<div v-for="item in weekList" :key="item.timeStamp" class="inner">
						{{ item.showDate }}
					</div>
				</div>
				<div class="wrapper-item">
					<div v-for="item in nextWeekList" :key="item.timeStamp" class="inner">
						{{ item.showDate }}
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>

export default {
	name: 'App',
	data() {
		return {
			weekName: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
			currentDay: null,
			oneDayTimestamp: 1000 * 60 * 60 * 24,
			weekList: [],
			lastWeekList: [],
			nextWeekList: [],
			containerWidth: 0,
			startX: 0,
			distanceX: 0,
			isMove: false,
			pageIndex: 1
		}
	},
	mounted() {
		this.currentDay = this.getStartTime(new Date())
		this.weekList = this.getWeekList(new Date(this.currentDay))
		this.getLastWeekList()
		this.getNextWeekList()
		const wrapperStyle = this.$refs.wrapper.style
		this.containerWidth = this.$refs.container.offsetWidth
		wrapperStyle.width = (this.containerWidth * 3) + 'px'
		wrapperStyle.transform = `translateX(${-this.containerWidth}px)`
		this.bindEvent()
	},
	methods: {
		// 获取当天0点的时间戳
		getStartTime(time) {
			const nowTimeDate = new Date(time)
			return nowTimeDate.setHours(0, 0, 0, 0)
		},
		// 时间戳转日期格式
		timestampToTime(timestamp) {
			let date = new Date(timestamp)
			let Y = date.getFullYear() + '-'
			let M =
				(date.getMonth() + 1 < 10
					? '0' + (date.getMonth() + 1)
					: date.getMonth() + 1) + '-'
			let D = date.getDate()
			return Y + M + D
		},
		// 获取一周的日期
		getWeekList(date) {
			let week = []
			for (let i = 0; i < 7; i++) {
				let weekObj = {
					name: this.weekName[i],
					date: '',
					timeStamp: '',
				}
				week.push(weekObj)
			}
			let today = this.getStartTime(Number(date)) // 当天时间戳
			let today_week = date.getDay() // 当天星期几
			if (today_week === 0) {
				// 若当天为周日
				week[6].timeStamp = today
			} else {
				week[today_week - 1].timeStamp = today
			}
			
			let leftNum = today_week - 1 // 本周内今天的前几天的数量
			let rightNum = 7 - today_week // 本周内今天的后几天的数量
			
			for (let left = 0; left < leftNum; left++) {
				week[left].timeStamp =
					today - (today_week - left - 1) * 1000 * 60 * 60 * 24
			}
			for (let right = 0; right < rightNum; right++) {
				week[right + today_week].timeStamp =
					today + (right + 1) * 1000 * 60 * 60 * 24
			}
			week.map((el) => {
				el.date = this.timestampToTime(el.timeStamp)
				const [, month, day] = el.date.split('-')
				el.showDate = month + '/' + day
			})
			return JSON.parse(JSON.stringify(week))
		},
		getLastWeekList() {
			this.lastWeekList = this.getWeekList(new Date(this.currentDay - 7 * this.oneDayTimestamp))
		},
		getNextWeekList() {
			this.nextWeekList = this.getWeekList(new Date(this.currentDay + 7 * this.oneDayTimestamp))
		},
		handlerTouchStart(e){
			this.startX = e.touches[0].clientX
		},
		handlerTouchMove(e){
			const moveX = e.touches[0].clientX
			
			this.distanceX = moveX - this.startX
			this.setTranslate(- this.containerWidth * this.pageIndex + this.distanceX)
			this.isMove = true
		},
		setTranslate(tranX) {
			this.$refs.wrapper.style.transition = "all .1s"
			this.$refs.wrapper.style.transform = `translateX(${tranX}px)`
		},
		handlerTouchEnd(){
			if (this.isMove) {
				if(Math.abs(this.distanceX) >= this.containerWidth / 3) {
					if (this.distanceX > 0) {
						this.pageIndex --
						this.currentDay = this.currentDay - 7 * this.oneDayTimestamp
					}
					
					if (this.distanceX < 0) {
						this.pageIndex ++
						this.currentDay = this.currentDay + 7 * this.oneDayTimestamp
					}
				}
				this.setTranslate(- this.pageIndex * this.containerWidth)
				// this.nextWeekList = this.weekList
				// this.weekList = this.lastWeekList
				// this.currentDay = this.weekList[0].timeStamp
				// this.getLastWeekList()
				// this.pageIndex = 1
				
			}
			
			this.startX = 0
			this.isMove = false
			this.distanceX = 0
		},
		bindEvent(){
			const {wrapper} = this.$refs
			wrapper.addEventListener('touchstart', this.handlerTouchStart, false)
			wrapper.addEventListener('touchmove', this.handlerTouchMove, false)
			wrapper.addEventListener('touchend', this.handlerTouchEnd, false)
		}
	}
}
</script>

<style>
#app {
	width: 100%;
	font-size: 12px;
}

.container {
	width: 100%;
	height: 52px;
	position: relative;
	overflow: hidden;
}

.wrapper {
	display: flex;
	position: absolute;
	height: 100%;
	flex-direction: row;
}

.wrapper-item {
	width: 100%;
	flex: 1;
	display: grid;
	grid-template-columns: repeat(7, 1fr);
}

.inner {
	display: flex;
	align-items: center;
	justify-content: center;
}
</style>
