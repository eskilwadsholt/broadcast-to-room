<script>
	import { io } from 'socket.io-client';
	import { afterUpdate } from 'svelte';

	const serverURL = 'https://chat-instantly.herokuapp.com';
	//const serverURL = 'http://localhost:5050';
	
	let socket, room, username, msg, users;
	let messages = [];
	let previews = {};
	let connected = false;

	afterUpdate(() => {
		if (element && messages && previews)
			if (messages.length || Object.keys(previews).length)
				element.scrollTop = element.scrollHeight;
	})

	async function joinRoom() {
		const roomID = room ? room : "default";
		const userID = username ? username : "Somebody";
		// Reset message vars
		messages = [];
		previews = {};

		console.debug('Connecting to socket server');
		if (socket) await socket.disconnect();
		
		socket = await io(serverURL, { autoConnect: false });
		socket.auth = { username };
		await socket.connect();
		connected = true;
		console.debug(socket);
		socket.emit('enter-room', { roomID, userID });

		socket.on('new-msg', info => {
			console.debug(info);
			messages.push(info);
			messages = messages;
		});

		socket.on('new-preview', info => {
			console.debug(info);
			previews[info.user] = info;
			previews = previews;
		});

		socket.on("users", userList => { users = userList; console.debug(users); });

		return async () => {
			await socket.disconnect();
			connected = false;
		}
	}

	async function sendMsg() {
		if (!socket) await joinRoom();

		if (socket && msg && username) {
			socket.emit('send-msg', { user: username, msg });
			messages.push({ id: socket.id, user: username, msg });
			messages = messages;
			msg = '';
			sendPreview();
		}
	}

	async function sendPreview() {
		if (!socket) await joinRoom();

		if (socket && username) {
			socket.emit('send-preview', { user: username, msg });
		}
	}

	function sendOrPreview(e) {
		if (e.key === "Enter" && !e.shiftKey) {
			sendMsg();
			e.preventDefault();
			return;
		}
		else if (e.key === "Escape") {
			msg = '';
		}
		sendPreview();
	}

	let element;
</script>

<main class:connected>
	<div class="topbar">
		<div class="field">
			<label for="room">ROOM</label>
			<input type="text" placeholder="Room ID ..."
				bind:value={room}
				on:change={joinRoom}>
		</div>
		<div class="field">
			<label for="user">USER</label>
			<input type="text" placeholder="Enter name ..."
				bind:value={username}
				on:change={joinRoom}>
		</div>
	</div>

	<div class="middle">
		{#if users}
			<div class="users">
				<div class="users-title">USERS</div>
				{#each users ? users : [] as userSocket}
				<div class="username">{userSocket.username}</div>
				{/each}
			</div>
		{/if}
	
		<div class="messages" bind:this={element}>
			{#each messages as { id, user, msg }}
				<div class="message" class:outgoing={ id === socket.id }>
					<div class="user">
						{#if id === socket.id}
						You
						{:else if user}
						{user}
						{:else}
						Somebody
						{/if}
					</div>
					<div class="content bubble"
						class:left={id !== socket.id}
						class:right={id === socket.id}>{@html msg.replaceAll('\n', '<br>')}</div>
				</div>
			{/each}
			{#each Object.values(previews) as { user, msg }}
				{#if msg}
					<div class="preview">
						<div class="user">
							{#if user}
							{user}
							{:else}
							Somebody
							{/if}
						</div>
						<div class="content bubble left">{@html msg.replaceAll('\n', '<br>')}</div>
					</div>
				{/if}
			{/each}
			<div class="bottom-space"></div>
		</div>
	</div>

	<div class="bottombar">
		<textarea name="chat-input" id="chat-input"
			placeholder="Enter message ..."
			cols="30" rows="3"
			bind:value={msg}
			on:keydown={sendOrPreview}></textarea>
		<div class="quad"></div>
		<button on:click={sendMsg}>SEND</button>
	</div>
</main>

<style>
	.middle {
		flex: 1;
		display: flex;
		overflow: auto;
	}
	.users {
		display: flex;
		flex-direction: column;
		align-items: baseline;
		min-width: fit-content;
	}
	.users > * {
		padding: 0 5px;
	}
	.users-title {
		background: #AFA8;
		color: white;
		font-weight: 600;
		width: 100%;
	}
	.bottom-space {
		height: 20px;
	}
	.bubble::before {
		content: "";
		width: 0px;
		height: 0px;
		position: absolute;
		border-bottom: 12px solid transparent;
		top: 10px;
	}
	.bubble.left::before {
		border-right: 18px solid #AAF8;
		left: -16px;
	}
	.bubble.right::before {
		border-left: 18px solid #FAA8;
		right: -16px;
	}
	.message, .preview {
		display: flex;
		justify-content: flex-start;
		width: 100%;
		padding: 5px;
	}
	.message.outgoing {
		flex-direction: row-reverse;
	}
	.user {
		padding: 0 5px;
		border-radius: 100px;
	}
	.content {
		position: relative;
		background: #AAF8;
		text-align: left;
		padding: 5px;
		border-radius: 15px;
		z-index: 5;
		margin: 0 15px;
	}

	.message.outgoing .content {
		background: #FAA8;
	}

	.preview .content::before {
		border-right: 18px solid #AAA8;
	}

	.preview .content {
		background: #AAA8;
		color: #FFF8;
	}

	.messages {
		width: 100%;
		background: #FFF3;
		padding: 10px;
		display: flex;
		flex-direction: column;
		overflow-y: scroll;
		transition: 500ms;
	}
	.bottombar, .topbar {
		display: flex;
		align-items: center;
		padding: 1em;
		width: 100%;
	}
	.topbar {
		justify-content: space-around;
	}
	.quad {
		width: 5px;
	}
	#chat-input {
		flex-grow: 1;
		resize: none;
	}
	button {
		background: #55C;
		color: white;
		font-weight: 600;
	}
	label {
		margin: 10px;
	}

	input[type="text"] {
		width: 80%;
	}
	main {
		text-align: center;
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		background: #5555;
		border: 2px solid red;
	}

	.connected {
		border: 2px solid green;
	}
</style>