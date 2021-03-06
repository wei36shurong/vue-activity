<template>
	<div class="honeycomb-scroller y-scroller" v-scroll="onScroll">
		<honeycomb :class="`row-${row}`" :colWidth="colWidth">
			<div v-for="(user, index) in localUsers">
				<avatar class="md"
				:src="user.info.avatarUrl" 
				></avatar>
			</div>
		</honeycomb>
	</div>
</template>

<script>
import userState from '@/store/user-state'
import Glass from '@/components/Glass'
import Avatar from '@/components/Avatar'
import Honeycomb from '@/components/Honeycomb'
import {getRandomInt} from '@/utils'

let localUsers = []
for (let i = 0; i < 200; i++) {
	localUsers.push(userState)
}

export default {
	name: 'HoneycombScroller',
	components: { Glass, Avatar, Honeycomb },
	data () {
		return {
			colWidth: 65,
			threshhold: 10, // cols
			swapColumn: 3, // cols
			intervalId: '',
			localUsers
		}
	},
	props: {
		users: {...userState},
		newUser: {...userState},
		row: {
			type: Number,
			default: 6
		},
		autoPlay: {
			type: Boolean,
			default: true
		}
	},
	computed: {
		// 还有的空格的index
		emptyUserIndexes() {
			let indexes = []
			this.localUsers.forEach((user, index) => {
				if (user._id) return
				indexes.push(index)
			})
			return indexes
		}
	},
	mounted () {
		if (this.autoPlay) this.autoScroll()
	},
	watch: {
		autoPlay(val) {
			clearInterval(this.autoScrollIntervalId)
			if (val) this.autoScroll()
		},
		users() {
			this.users.filter(user => user.info && user.info.avatarUrl).forEach(user => { this.randomInsertUser(user) })
		},
		newUser(user) {
			this.randomInsertUser(user)
		}
	},
	methods: {
		randomInsertUser (user) {
			const length = this.emptyUserIndexes.length
			// 当空格都被填满，插入到最前
			if (!length) { this.localUsers.unshift(user); return }
			const randomIndex = this.emptyUserIndexes[getRandomInt(0, length - 1)]
			this.localUsers.splice(randomIndex, 1, user)
		},
		autoScroll() {
			// const scroller = this.$el.querySelector('.y-scroller')
			const scroller = this.$el
			if (!scroller.scrollBy) return
			this.autoScrollIntervalId = setInterval(() => {
				scroller.scrollBy({
					left: 4
				})
			}, 20)
		},
		onScroll(event, {scrollLeft}) {
			const currentCol = scrollLeft / this.colWidth
			if (currentCol < this.threshhold) return
			let localUsers = this.localUsers
			const swapNum = this.row * this.swapColumn
			const swap = localUsers.slice(0, swapNum)
			localUsers.splice(0, swapNum)
			localUsers.splice(localUsers.length - 1, 0, ...swap)
			this.localUsers = localUsers
			event.target.scrollLeft = (this.threshhold - this.swapColumn) * this.colWidth
		}
	}
}
</script>

<style lang="less" scoped>
.y-scroller {
	transition-timing-function: ease;
	overflow-x: scroll;
	overflow-y: hidden;
	margin: 0 -22.5px;
	padding: 0 22.5px;
	&::-webkit-scrollbar {
		display: none;
	}
}
</style>
