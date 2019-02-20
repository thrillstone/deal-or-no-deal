<template>
	<div id="app">
		<game v-if="gameStart" @select="selectCase"></game>
		<bank v-if="bankStart" :selectedCase="selectedCase" :cases="cases"></bank>
		<button v-if="!gameStart && !bankStart" @click="startGame()">Start Game</button>
		<button v-if="!gameStart && !bankStart" @click="startBank()">Start Bank</button>
	</div>
</template>

<script>
import solace from "./lib/solclient.js";
import Game from "./components/Game";
import Bank from "./components/Bank";

export default {
	name: 'app',
	components: {
		game: Game,
		bank: Bank
	},
	data: function() {
		return {
			gameStart: false,
			bankStart: false,
			selectedCase: "None",
			cases: []
		}
	},
	created: function() {
		try {
			let factoryProps = new solace.SolclientFactoryProperties();
			factoryProps.profile = solace.SolclientFactoryProfiles.version10;
			solace.SolclientFactory.init(factoryProps);
			this.client = solace.SolclientFactory.createSession({
				url: "wss://mr85s7y8ur59.messaging.solace.cloud:20548",
				vpnName: "msgvpn-lq2f85anse9",
				userName: "solace-cloud-client",
				password: "bqnu7ana1s8nmrd85e49uopku2"
			});
		} catch (error) {
			console.log(error);
			throw error;
		}

		this.client.on(solace.SessionEventCode.UP_NOTICE, () => {
			console.log("connected");
			this.client.subscribe(solace.SolclientFactory.createTopicDestination(">"),
				false, // request confirmation
				">", // correlation key so we know which subscription suceedes
				1000 // subscribe timeout
			);
		});

		this.client.on(solace.SessionEventCode.MESSAGE, (message) => {
			console.log(message.getBinaryAttachment());
			let m = JSON.parse(message.getBinaryAttachment());
			this[m.type](m.data);
		});
		this.client.connect();
	},
	methods: {
		initialize: function(data) {
			this.selectedCase = data.selectedCase;
			this.cases = data.cases;
		},
		selectCase: function(data) {
			console.log(data);
			var message = solace.SolclientFactory.createMessage();
			message.setDestination(solace.SolclientFactory.createTopicDestination("select"));
			message.setBinaryAttachment(JSON.stringify({
				type: "initialize",
				data: data
			}));
			message.setDeliveryMode(solace.MessageDeliveryModeType.DIRECT);
			this.client.send(message);
		},
		startGame: function() {
			this.gameStart = true;
		},
		startBank: function() {
			this.bankStart = true;
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
button {
	padding: 3vh 3vw;
	font-size: 5vh;
	min-width: 20vw;
	background: linear-gradient(#00c895, #00ad93);
	color: white;
	cursor: pointer;
	border-radius: 5px;
}
</style>
