<template>
	<div id="app">
		<div>
			<div @click="select('mario')"><img :class="selected == 'mario' ? 'selected' : ''" src="mario.png"></div>
			<div @click="select('luigi')"><img :class="selected == 'luigi' ? 'selected' : ''" src="luigi.png"></div>
			<div @click="select('peach')"><img :class="selected == 'peach' ? 'selected' : ''" src="star.png"></div>
			<div @click="select('bowser')"><img :class="selected == 'bowser' ? 'selected' : ''" src="boo.png"></div>
		</div>
		<div class="attacks">
			<div @click="attack('wheelbroken')"><img src="blueShell.png"></div>
			<div @click="attack('spinout')"><img src="banana.png"></div>
			<div @click="attack('reverse')"><img src="mushroom.png"></div>
			<div @click="attack('slowdown')"><img src="lightning.png"></div>
		</div>
	</div>
</template>

<script>
import solace from "./lib/solclient.js";

export default {
	name: 'app',
	data: function() {
		return {
			team: null,
			client: null,
			clientName: null,
			opponent: null,
			selectingTeam: true,
			selected: "mario",
			error: null,
			waiting: false
		}
	},
	created: function() {
		try {
			let factoryProps = new solace.SolclientFactoryProperties();
			factoryProps.profile = solace.SolclientFactoryProfiles.version10;
			solace.SolclientFactory.init(factoryProps);
			this.client = solace.SolclientFactory.createSession({
				url: "wss://mrrwtxvkmotl3.messaging.solace.cloud:443",
				vpnName: "msgvpn-rwtxvkmotk9",
				userName: "solace-cloud-client",
				password: "dgs3c5bfq8kue13oa31fsov6ho"
			});
		} catch (error) {
			console.log(error);
			throw error;
		}

		this.client.on(solace.SessionEventCode.UP_NOTICE, () => {
			this.clientName = this.client.getSessionProperties().clientName;
		});

		this.client.connect();
	},
	methods: {
		select: function(character) {
			this.selected = character;
		},
		attack: function(attack) {
			var message = solace.SolclientFactory.createMessage();
			message.setDestination(solace.SolclientFactory.createTopicDestination(`game/attack`));
			message.setBinaryAttachment(JSON.stringify({
				attack: attack,
				player: this.selected
			}));
			message.setDeliveryMode(solace.MessageDeliveryModeType.DIRECT);
			this.client.send(message);
		}
	}
}
</script>

<style>
#app {
	font-family: 'Avenir', Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	display: flex;
	align-items: center;
	justify-content: center;
}
img {
	padding: 3vh 3vw;
	font-size: 5vh;
	color: white;
	cursor: pointer;
	border-radius: 5px;
	width: 100px;
	height: 100px;
}

img:active {
	border: 1px solid green;
}

img.selected {
	border: 1px solid green;
}

</style>
