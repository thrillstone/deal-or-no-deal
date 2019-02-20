
<template>
	<div id="bank">
		<div class="big selected-value" :class="{show: visible}">
			${{selectedCase.value}}
		</div>
		<div v-for="c of sorted" :key="c.index" class="case-bank" :class="c.selected ? 'selected' : ''">
			<span v-if="visible">{{c.index}}</span> ${{c.value}}
		</div>
		<div class="big">${{ev}}</div>
		<button @click="show()">Show</button>
	</div>
</template>

<script>
export default {
	name: 'bank',
	props: ["selectedCase", "cases"],
	data: function() {
		return {
			visible: false
		}
	},
	created: function() {
	},
	methods: {
		show: function() {
			this.visible = !this.visible;
		}
	},
	computed: {
		ev: function() {
			let remaining = this.cases.filter(v => !v.selected);
			return (remaining.reduce((result, value) => result + value.value, 0) / remaining.length).toFixed(2);
		},
		sorted: function() {
			return this.cases.sort((a, b) => a.value - b.value);
		}
	}
}
</script>

<style>
#game {
	display: flex;
	align-items: center;
	justify-content: center;
}
.big {
	font-size: 5vh;
}
.case-bank {
	width: 30vw;
	height: 3vh;
	border: 1px solid grey;
	margin: 1vw;
	line-height: 3vh;
	font-size:2vh;
	vertical-align: middle;
	cursor: pointer;
}
.selected {
	background: #00c895;
	cursor: auto;
}
.active {
	background: #474747;
	color: white;
}
.selected-value {
	color: white;
	border: 1px solid grey;
}
.show {
	color: black;
}
</style>
