<script>
	let fullData = [];
	let songInfo = {};
	let artistInfo = {};
	let earliestDate;
	let latestDate;
	function onUpload(event) {
		fullData = []
		const files = event.target.files
		for (const file of files){
			const reader = new FileReader();
			reader.onload = onReaderLoad;
			reader.readAsText(file);
		}
		fullData.sort((a, b) => a.date - b.date)
		
		$: console.log(fullData)
		$: console.log(fullData.length)
		const firstTrackDate = fullData[0].date
		const lastTrackDate = fullData[fullData.length - 1].date
		earliestDate = new Date(firstTrackDate.getFullYear(), firstTrackDate.getMonth(), firstTrackDate.getDay())
		earliestDate = new Date(lastTrackDate.getFullYear(), lastTrackDate.getMonth(), lastTrackDate.getDay())

		updateData()
	}

	function onReaderLoad(event){
		const fileInfo = JSON.parse(event.target.result);
		for (const info of fileInfo){
			//spotify only counts a play as 30+ seconds
			if(info.msPlayed >= 30000){
				fullData.push({"date": new Date(info.endTime), "artist": info.artistName, "track": info.trackName, "duration": info.msPlayed})
				$: console.log(fullData.length)
			}
		}
	}

	function updateData(){
		artistInfo = {}
		for (const info of fullData){
			if(info.artist in artistInfo){
				artistInfo[info.artist].plays += 1
				artistInfo[info.artist].time += info.duration
			}else{
				artistInfo[info.artist] = {"plays": 1, "time": info.duration}
			}
		}
		$: console.log(artistInfo)
	}
</script>

<label for='files'>Choose JSON file(s)</label>
<input type='file' on:change={onUpload} accept='.json' multiple>
<style>
	
</style>
