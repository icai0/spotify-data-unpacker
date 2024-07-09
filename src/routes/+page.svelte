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

		currentOrder = orders.DURATION
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
</script>

<label for='files'>Choose JSON file(s)</label>
<input type='file' on:change={onUpload} accept='.json' multiple>

{#if displayData}
<br>
<label for='start'>Start date:</label>
<input type='date' bind:value={startDateInput} on:change={updateData} min={dateToInput(earliestDate)} max={endDateInput}>
<br>
<label for='end'>End date:</label>
<input type='date' bind:value={endDateInput} on:change={updateData} min={startDateInput} max={dateToInput(latestDate)}>
{/if}

<style>
	
</style>
