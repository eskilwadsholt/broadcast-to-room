<script>
</script>

<main>
	<div class="topbar">
		<div class="field">
			<label for="room">ROOM</label>
			<input type="text" placeholder="Enter room PIN ...">
		</div>
		<div class="field">
			<label for="user">USER</label>
			<input type="text" placeholder="Enter name ...">
		</div>
	</div>

	<div class="bottombar">
		<input type="text" placeholder="Enter message ..." id="message">
		<div class="quad"></div>
		<button>SEND</button>
	</div>
</main>

<style>
	.bottombar, .topbar {
		position: absolute;
		display: flex;
		align-items: center;
		padding: 1em;
		width: 100%;
	}
	.bottombar {
		bottom: 0;
	}
	.topbar {
		top: 0;
		justify-content: space-around;
	}
	.quad {
		width: 5px;
	}
	#message {
		flex-grow: 1;
	}
	button {
		background: #AAC;
		color: white;
		font-weight: 600;
	}
	label {
		margin: 10px;
	}
	.field {
		display: flex;
	}
	main {
		text-align: center;
		width: 100%;
		height: 100%;
		background: #5555;
	}
</style>