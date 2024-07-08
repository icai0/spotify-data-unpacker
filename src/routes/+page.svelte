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
	async function onUpload(event) {
		fullData = []
		const files = event.target.files
		
		for (const file of files){
			const reader = new FileReader()
			reader.onload = onReaderLoad
			reader.readAsText(file)
		}
		fullData.sort((a, b) => a.date - b.date)

		//definitely not the way to do this but I tried the normal ways and gave up
		await new Promise(resolve => setTimeout(resolve, 1000))

		//reset bounds to full data when uploading files
		const firstTrackDate = fullData[0].date
		const lastTrackDate = fullData[fullData.length - 1].date
		startDate = earliestDate = new Date(firstTrackDate.getFullYear(), firstTrackDate.getMonth(), firstTrackDate.getDay(), "00:00:01")
		endDate = latestDate = new Date(lastTrackDate.getFullYear(), lastTrackDate.getMonth(), lastTrackDate.getDay(), "23:59:59")
		boundData = fullData

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
		for (const info of fullData){
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
		$: console.log(songInfo)
	}

	function restrictData(){
		//restrict data within the set dates
	}
</script>

<label for='files'>Choose JSON file(s)</label>
<input type='file' on:change={onUpload} accept='.json' multiple>
<style>
	
</style>
