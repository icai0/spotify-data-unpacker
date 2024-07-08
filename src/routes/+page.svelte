<script>
	let fullData = []
	let boundData = []
	let songInfo = {}
	let artistInfo = {}
	let earliestDate
	let latestDate
	let startDate
	let endDate
	let totalTime = 0

	const orders = Object.freeze({
		PLAYS: 0,
		DURATION: 1
	})
	let currentOrder = orders.PLAYS

	let showPlays = true

	async function onUpload(event) {
		fullData = []
		const files = event.target.files
		
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

		updateData()
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
		restrictData()
		songInfo = {}
		artistInfo = {}
		totalTime = 0
		for (const info of boundData){
			totalTime += info.duration

			//songs with multiple artists
			if(Array.isArray(info.artist)){
				for(const artist in info.artist){
					if(artist in artistInfo){
						artistInfo[artist].plays += 1
						artistInfo[artist].time += info.duration
					}else{
						artistInfo[artist] = {"plays": 1, "time": info.duration}
					}
				}
			}else{
				if(info.artist in artistInfo){
					artistInfo[info.artist].plays += 1
					artistInfo[info.artist].time += info.duration
				}else{
					artistInfo[info.artist] = {"plays": 1, "time": info.duration}
				}
			}
			if(info.track in songInfo){
				songInfo[info.track].plays += 1
				songInfo[info.track].time += info.duration
			}else{
				songInfo[info.track] = {"plays": 1, "time": info.duration}
			}
		}
		$: console.log(fullData)
		$: console.log(boundData)
	}

	function restrictData(){
		boundData = []
		$: console.log(startDate)
		$: console.log(fullData[0].date)

		$: console.log(endDate)
		$: console.log(fullData[fullData.length - 1].date)
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
</script>

<label for='files'>Choose JSON file(s)</label>
<input type='file' on:change={onUpload} accept='.json' multiple>
<style>
	
</style>
