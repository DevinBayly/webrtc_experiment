<script>

	let name = 'world';
	let role
	let local
	let remote
	let icecands,icecandsRemote
	let pc
	let remVid
	let channel
	var sdpConstraints = {
  offerToReceiveVideo: true
};
	let candidateList = []
	function onIceCand(e) { 
		console.log("ice cand event",e)
		if (e.candidate){
			candidateList.push(e.candidate)
		} else {
			icecands.value = JSON.stringify(candidateList)
		}
	}
	function onIceState(e) {
	console.log("ice state chage",e)
	}
	function onIceGatherChange(e) {
	console.log("ice gather change",e)
	}
	function dcOpen(e){
		console.log("channel opening",e)
		if (role === 'sender'){
			setInterval(()=>channel.send("hello there over the channel"),2000)
		}
	}
	function dcClose(e) {
		console.log("channel closing",e)
	}
	function dcMessage(e){
		console.log("getting channel message",e)
	}
	async function start() {
		pc = new RTCPeerConnection({
  iceServers: [{
     urls: 'stun:stun.l.google.com:19302'
  }]
})
		pc.addEventListener("icecandidate",onIceCand)
		pc.onconnectionstatechange = onIceState
		pc.onicegatheringstatechange = onIceGatherChange
		

		// check thte role
		
		if (role === "sender"){
			channel = pc.createDataChannel('data');
			channel.onopen=dcOpen
			channel.onclose = dcClose
			const localStream = await navigator.mediaDevices.getUserMedia({video: true});
			localStream.getTracks().forEach(track => {
					pc.addTrack(track, localStream);
					console.log("added all tracks",track, "to local",localStream)
			});

			// do the thigns that make itt possible to start an offer
			pc.createOffer(sdpConstraints).then(sd=> {
				local.value = JSON.stringify(sd)
				pc.setLocalDescription(sd)
			})
			console.log("sent offer")
			
		} else if (role === "receiver"){
			pc.addEventListener('datachannel', event => {
   	channel = event.channel;
				channel.onopen=dcOpen
			channel.onclose = dcClose
				channel.onmessage = dcMessage
});
			const remoteStream = new MediaStream();
			remVid.srcObject = remoteStream;

pc.addEventListener('track', async (event) => {
	console.log("got track event",event)
    remoteStream.addTrack(event.track, remoteStream);
});

			// read the offer out of theh remote ta
			let sd = new RTCSessionDescription(JSON.parse(remote.value))
			pc.setRemoteDescription(sd)
			pc.createAnswer().then(sdLoc=> {
				local.value = JSON.stringify(sdLoc)
				pc.setLocalDescription(sdLoc)
			})
			
		}
	}
	function setRemote() {
		pc.setRemoteDescription(new RTCSessionDescription(JSON.parse(remote.value)))
	}
	function addCandidates() {
		let allRemoteCandidates = JSON.parse(icecandsRemote.value)
		allRemoteCandidates.map(e=> {
			pc.addIceCandidate(e)
		})
		console.log("added all candidates")
		console.log("peer is",pc)
	}
</script>

<label>sender<input type="radio" name="type" bind:group={role} value={'sender'}></label> 
<label>receiver<input type="radio" name="type"bind:group={role} value={'receiver'}></label>
<div>
	my role is "{role}"
</div>
<button on:click={start}>
	start
</button>
<button on:click={setRemote}>
	Set Remote
</button>
<div>
	<div>
		local<textarea bind:this={local}></textarea>
	</div>
	<div>
		remote<textarea bind:this={remote}></textarea>
	</div>
	<div>
		iceCandidatesLocal<textarea bind:this={icecands}></textarea>
	</div>
	<div>
		iceCandidatesRemote<textarea bind:this={icecandsRemote} on:change={addCandidates}></textarea>
	</div>
</div>
<video bind:this={remVid} playsinline autoplay></video>
