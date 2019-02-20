
<template>
	<div id="game">
		<div class="left values">
			<div class="value" :class="value.selected ? 'selected' : ''" v-for="value of leftValues" :key="value.value">
				<span>${{value.value}}</span>
			</div>
		</div>
		<div class="right values">
			<div class="value" :class="value.selected ? 'selected' : ''" v-for="value of rightValues" :key="value.value">
				<span>${{value.value}}</span>
			</div>
		</div>
		<div class="cases">
			<div class="case" :class="{selected: c.selected, active: c === selectedCase, highlight: activeRound && activeRound.lastRound}" v-for="c of cases" :key="c.index" @click="selectCase(c)">
				{{c.index + 1}}
			</div>
		</div>
		<div v-if="!waitForDeal && activeRound" class="round">
			Round {{activeRound.round + 1}}. Select {{activeRound.selections + 1}} of {{activeRound.cases}}
		</div>
		<div v-if="waitForDeal" class="round">
			<button @click="dealOrNoDeal(true)">Deal</button>
			<button @click="dealOrNoDeal(false)">No Deal</button>
		</div>
		<div v-if="gameOver" class="game-over">
			<div v-if="!finalCase">You took the deal!</div>
			<div v-if="finalCase">You got ${{finalCase.value}}</div>
			<div>
				<button @click="newGame()">New Game</button>
			</div>
			</div>
	</div>
</template>

<script>
export default {
	name: 'game',
	data: function() {
		return {
			waitForDeal: false,
			leftValues: [
				{
					value: 0,
					selected: false
				}, {
					value: 1,
					selected: false
				}, {
					value: 2,
					selected: false
				}, {
					value: 3,
					selected: false
				}, {
					value: 4,
					selected: false
				}, {
					value: 5,
					selected: false
				}, {
					value: 6,
					selected: false
				}, {
					value: 7,
					selected: false
				}
			],
			rightValues: [
				{
					value: 8,
					selected: false
				}, {
					value: 9,
					selected: false
				}, {
					value: 10,
					selected: false
				}, {
					value: 12,
					selected: false
				}, {
					value: 15,
					selected: false
				}, {
					value: 20,
					selected: false
				}, {
					value: 25,
					selected: false
				}, {
					value: 30,
					selected: false
				}
			],
			cases: [],
			selectedCase: null,
			rounds: [],
			activeRound: {},
			gameOver: false
		}
	},
	created: function() {
		this.newGame();
	},
	methods: {
		newGame: function() {
			this.gameOver = false;
			this.cases = [];
			let values = this.leftValues;
			values = values.concat(this.rightValues);
			values = this.shuffle(values);
			values.forEach((value, index) => {
				value.selected = false;
				this.cases.push({
					index: index,
					value: value.value,
					selected: false
				});
			});
			this.activeRound = null;
			this.selectedCase = null;
			this.waitForDeal = false;
			this.rounds = [
				{
					round: 0,
					cases: 7,
					selections: 0
				}, {
					round: 1,
					cases: 4,
					selections: 0
				}, {
					round: 2,
					cases: 2,
					selections: 0
				}, {
					round: 3,
					cases: 1,
					selections: 0
				}, {
					round: 4,
					cases: 1,
					selections: 0,
					lastRound: true
				}
			];
		},
		shuffle: function(array) {
			var currentIndex = array.length, temporaryValue, randomIndex;

			// While there remain elements to shuffle...
			while (0 !== currentIndex) {
				// Pick a remaining element...
				randomIndex = Math.floor(Math.random() * currentIndex);
				currentIndex -= 1;

				// And swap it with the current element.
				temporaryValue = array[currentIndex];
				array[currentIndex] = array[randomIndex];
				array[randomIndex] = temporaryValue;
			}

			return array;
		},
		dealOrNoDeal: function(deal) {
			if (!deal) {
				this.waitForDeal = deal;
			} else {
				this.gameOver = true;
				this.finalCase = null;
			}
		},
		selectCase: function(c) {
			if (this.waitForDeal) {
				return;
			}
			if (this.activeRound && !this.activeRound.lastRound && this.selectedCase === c) {
				return;
			}
			if (c.selected) {
				return;
			}
			if (!this.selectedCase) {
				let data = {
					selectedCase: c,
					cases: this.cases
				}
				this.selectedCase = c;
				this.activeRound = this.rounds[0];
				this.$emit("select", data);
				return;
			}
			c.selected = true;
			let value = this.leftValues.find(v => {
				return v.value === c.value;
			});
			if (!value) {
				value = this.rightValues.find(v => {
					return v.value === c.value;
				});
			}
			value.selected = true;
			if (this.activeRound.lastRound) {
				this.gameOver = true;
				this.finalCase = c;
			} else {
				this.activeRound.selections++;
				if (this.activeRound.selections === this.activeRound.cases) {
					this.activeRound = this.rounds[this.activeRound.round + 1];
					if (!this.activeRound.lastRound) {
						this.waitForDeal = true;
					}
				}
			}
			let data = {
				selectedCase: this.selectedCase,
				cases: this.cases
			}
			this.$emit("select", data);
		}
	}
}
</script>

<style>
#game {
	margin-top: 10vh;
	display: flex;
	align-items: center;
	justify-content: center;
}
.left {
	position: absolute;
	left: 0;
	top: 0;
}
.right {
	position: absolute;
	right: 0;
	top: 0;
}
.values {
	display: flex;
	flex-direction: column;
	border: 1px solid grey;
}
.value {
	height: 12.5vh;
	width: 10vw;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 6vh;
	vertical-align: middle;
	line-height: 12.5vh;
}
.cases {
	width: 50vw;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-wrap: wrap;
}
.case {
	width: 10vw;
	height: 12.5vh;
	border: 1px solid grey;
	margin: 1vw;
	line-height: 12.5vh;
	font-size: 10vh;
	vertical-align: middle;
	cursor: pointer;
}
.highlight {
	border: 2px solid red;
}
.selected {
	background: #00c895;
	cursor: auto;
	border: unset;
}
.active {
	background: #474747;
	color: white;
}
.round {
	position: absolute;
	bottom: 0;
	left: 50%;
	margin-bottom: 10vh;
	font-size: 5vh;
	transform: translateX(-50%);
}
.round button {
	margin-right: 3vw;
}
.round button:last-child {
	margin-right: 0;
}
.game-over {
	position: fixed;
	background: white;
	width: 100vw;
	height: 100vh;
	top: 0;
	left: 0;
}
</style>
