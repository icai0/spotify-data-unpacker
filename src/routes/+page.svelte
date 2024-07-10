<script>
	let fullData = []
	let boundData = []

	let songInfo = new Map()
	let artistInfo = new Map()

	let earliestDate
	let latestDate
	let startDate
	let endDate

	let totalTime = 0
	//total plays is equal to array length

	let startDateInput
	let endDateInput

	const orders = Object.freeze({
		PLAYS: 0,
		DURATION: 1
	})
	let currentOrder = orders.PLAYS

	let showPlays = true
	let showDuration = true

	let displayData = false

	async function onUpload(event) {
		fullData = []
		const files = event.target.files
		try{
			for (const file of files){
				const reader = new FileReader()
				reader.onload = onReaderLoad
				reader.readAsText(file)
			}

			//definitely not the way to do this but I tried the normal ways and gave up
			await new Promise(resolve => setTimeout(resolve, 1000))
			
			fullData.sort((a, b) => a.date.getTime() - b.date.getTime())

			//reset bounds to full data when uploading files
			const firstTrackDate = fullData[0].date
			const lastTrackDate = fullData[fullData.length - 1].date
			startDate = earliestDate = new Date(firstTrackDate.getFullYear(), firstTrackDate.getMonth(), firstTrackDate.getDate(), 0, 0, 0)
			endDate = latestDate = new Date(lastTrackDate.getFullYear(), lastTrackDate.getMonth(), lastTrackDate.getDate(), 23, 59, 59)
			startDateInput = dateToInput(startDate)
			endDateInput = dateToInput(endDate)
			updateData()
			displayData = true
		} catch{
			alert("Wrong JSON file")
			displayData = false
		}
	}

	function onReaderLoad(event){
		const fileInfo = JSON.parse(event.target.result)
		for (const info of fileInfo){
			//spotify only counts a play as 30+ seconds
			if(info.msPlayed >= 30000){
				fullData.push({"date": new Date(info.endTime), "artist": info.artistName, "track": info.trackName, "duration": info.msPlayed})
			}
		}
	}

	function updateData(){
		
		const splitStartDate = startDateInput.split("-")
		const splitEndDate = endDateInput.split("-")
		// -1 because for some reason months are 0 - 11
		startDate = new Date(splitStartDate[0], splitStartDate[1] - 1, splitStartDate[2], 0, 0, 0)
		endDate = new Date(splitEndDate[0], splitEndDate[1] - 1, splitEndDate[2], 0, 0, 0)

		restrictData()

		currentOrder = orders.PLAYS
		songInfo = new Map()
		artistInfo = new Map()
		totalTime = 0

		for (const info of boundData){
			totalTime += info.duration

			if(artistInfo.has(info.artist)){
				artistInfo.set(info.artist, {"plays": artistInfo.get(info.artist).plays + 1, "time": artistInfo.get(info.artist).time + info.duration})
			}else{
				artistInfo.set(info.artist, {"plays": 1, "time": info.duration})
			}
			if(songInfo.has(info.track)){
				songInfo.set(info.track, {"plays": songInfo.get(info.track).plays + 1, "time": songInfo.get(info.track).time + info.duration, "artist": info.artist})
			}else{
				songInfo.set(info.track, {"plays": 1, "time": info.duration, "artist": info.artist})
			}
		}

		reorderData()
		$: console.log(boundData)
	}

	function reorderData(){
		if(currentOrder == orders.PLAYS){
			artistInfo = Array.from(artistInfo).sort((a,b) => b[1].plays - a[1].plays)
			songInfo = Array.from(songInfo).sort((a,b) => b[1].plays - a[1].plays)
		}else{
			artistInfo = Array.from(artistInfo).sort((a,b) => b[1].time - a[1].time)
			songInfo = Array.from(songInfo).sort((a,b) => b[1].time - a[1].time)
		}
	}

	function restrictData(){
		boundData = []
		for (var i = 0; i < fullData.length; i++){
			if (fullData[i].date >= startDate){
				break;
			}
		}
		for (let j = i; j < fullData.length; j++){
			if(fullData[j].date <= endDate){
				boundData.push(fullData[j])
			}else{
				break;
			}
		}
	}

	function dateToInput(date){
		return date.toISOString().split("T")[0]
	}

	function msToTime(ms){
		let seconds = ms/1000
		let minutes = seconds/60
		let hours = Math.floor(minutes/60)
		minutes = Math.floor(minutes%60)
		return hours + " Hours " + minutes + " Minutes"
	}
</script>


<div class='content'>
	<h1>Spotify Data Unpacker</h1>
	<div class='file-input'>
		<label for='files'>Choose JSON file(s):</label>
		<input type='file' on:change={onUpload} accept='.json' multiple>
	</div>
	{#if displayData}
		<div class='options'>
			<div class='option-section'>
				<label for='start'>Start date: </label>
				<input type='date' bind:value={startDateInput} on:change={updateData} min={dateToInput(earliestDate)} max={endDateInput}>
			</div>
			<div class='option-section'>
				<label for='end'>End date: </label>
				<input type='date' bind:value={endDateInput} on:change={updateData} min={startDateInput} max={dateToInput(latestDate)}>
			</div>
			<div class='option-section'>
				<label for='plays'>Show Plays:</label>
				<input type='checkbox' bind:checked={showPlays}/>
			</div>
			<div class='option-section'>
				<label for='time'>Show Time:</label>
				<input type='checkbox' bind:checked={showDuration}/>
			</div>
			<div class='option-section'>
				<h4 class='option-text'>{msToTime(totalTime)}</h4>
			</div>
			<div class='option-section'>
				<h4 class='option-text'>{boundData.length + " Plays"}</h4>
			</div>
			<div class='option-section' id='order-section'>
				<button class='order-button' id='left-button' class:selected-order={currentOrder == orders.PLAYS} on:click={() => currentOrder = orders.PLAYS} on:click={reorderData}>
					Plays
				</button>
				<button class='order-button' id='right-button' class:selected-order={currentOrder == orders.DURATION} on:click={() => currentOrder = orders.DURATION} on:click={reorderData}>
					Time
				</button>
			</div>
		</div>
		<div class='info-area'>
			<div class='info-container'>
				<h2 class='info-title'>Top Artists</h2>
				{#each artistInfo as [artist, data], i}
					<div class='info-box'>
						<div class='info-text'>
						<h4 class='info-subject'>{artist}</h4>
						{#if showPlays}
							<h4>{data.plays} Plays</h4>
						{/if}
						{#if showDuration}
							<h4>{msToTime(data.time)}</h4>
						{/if}
						</div>
						<h1 class='rank-text'>#{i+1}</h1>
					</div>
				{/each}
			</div>
			<div class='info-container'>
				<h2 class='info-title'>Top Songs</h2>
				{#each songInfo as [song, data], i}
					<div class='info-box'>
						<div class='info-text'>
						<h4 class='info-subject'>{song}</h4>
						<h4 class='info-subject'>{data.artist}</h4>
						{#if showPlays}
							<h4>{data.plays} Plays</h4>
						{/if}
						{#if showDuration}
							<h4>{msToTime(data.time)}</h4>
						{/if}
						</div>
						<h1 class='rank-text'>#{i+1}</h1>
					</div>
				{/each}
			</div>
		</div>
	{/if}
</div>


<style>
	:global(body){
		margin: 0;
		width: 100%;
		height: 1500px;
	}
	.content {
		width: 100%;
		height: 100%;
		margin: 0;
		background-color: black;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	h1 {
		color:rgb(30, 215, 96);
		font-family: Verdana;
		margin-bottom: 10px;
		text-align: center;
	}
	h4 {
		color: rgb(255, 255, 255);
		font-family: Verdana;
		font-weight: 200;
		margin: 0;
		margin-top: 5px;
		margin-bottom: 5px;
	}
	.option-text {
		color: rgb(255, 255, 255);
		font-family: Verdana;
		font-weight: 200;
		margin: 0;
		margin-top: 5px;
		margin-bottom: 5px;
		text-align: center;
	}
	label {
		color: rgb(255, 255, 255);
		font-family: Verdana;
		margin-top: 5px;
		margin-bottom: 5px;
		margin-right: 2px;
		text-align: center;
	}
	.file-input {
		border: 2px;
		border-style: solid;
		border-color: rgb(30, 215, 96);
		border-radius: 5px;
		font-family: Verdana;
		margin-top: 5px;
		margin-bottom: 10px;
		background-color: rgb(53, 53, 53);
		padding: 5px;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: center;
		align-items:center;
	}
	input {
		border: 1px;
		border-style: solid;
		border-color: rgb(30, 215, 96);
		border-radius: 5px;
		width: auto;
		background-color: black;
		color: rgb(30, 215, 96);
		accent-color: rgb(30, 215, 96);
		height: 22px;
	}
	.options {
		border: 2px;
		border-style: solid;
		border-color: rgb(30, 215, 96);
		border-radius: 10px;
		background-color: rgb(53, 53, 53);
		padding: 5px;
		margin: 10px;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		width: min(500px, 90%);
	}
	.option-section {
		width: 50%;
		align-content: center;
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
	}
	.info-container {
		border: 2px;
		border-style: solid;
		border-color: rgb(30, 215, 96);
		border-radius: 10px;
		background-color: rgb(53, 53, 53);
		padding: 5px;
		display: flex;
		flex-direction: column;
		width: 48%;
		height: 500px;
		overflow: scroll;
	}
	.info-box{
		width: 100%;
		border: 1px;
		border-style: solid;
		border-color: rgb(30, 215, 96);
		border-radius: 5px;
		width: auto;
		background-color: black;
		margin-bottom: 5px;
		margin-left: 5px;
		margin-right: 5px;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	.info-title{
		color:white;
		font-family: Verdana;
		margin: 0;
		margin-bottom: 5px;
		text-align: center;
	}
	.rank-text{
		color:rgb(30, 215, 96);
		font-family: Verdana;
		margin: 0;
		margin-right: 5px;
	}
	.info-text{
		margin-left: 5px;
	}
	.info-subject {
		color: rgb(30, 215, 96);
		font-family: Verdana;
		font-weight: 200;
		margin: 0;
		margin-top: 5px;
		margin-bottom: 5px;
	}
	.info-area {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		width: 90%;
	}
	.order-button{
		width: 50px;
		height: 30px;
		margin: 0;
		background: black;
		border-color: rgb(30, 215, 96);
		border-style: solid;
		color: rgb(30, 215, 96);
		text-align: center;
	}
	#left-button{
		border-top-left-radius: 5px;
		border-bottom-left-radius: 5px;
	}
	#right-button{
		border-top-right-radius: 5px;
		border-bottom-right-radius: 5px;
	}
	.selected-order{
		background: rgb(0, 75, 0);
	}
	#order-section{
		width: 100%;
	}
</style>
