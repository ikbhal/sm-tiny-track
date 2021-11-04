<script>
import {db} from './firebase.js';
import {writable} from 'svelte/store';
import {onMount} from 'svelte';

let todayDateString = new Date().toISOString().slice(0, 10);
export let trackDate= todayDateString;
export let notes;
let tracks = writable([]);

async function loadTrackList() {
	console.log("load track list");
	let trackRef= db.collection('sm-tiny-track');
  	let allTracks= await trackRef.get();
	let trackListCopy = [];
	for(const doc of allTracks.docs){
		let docData = doc.data();
		let copy= {
			id:doc.id,
			trackDate: docData.trackDate,
			notes: docData.notes
		};
		trackListCopy.push(copy);
  	}
	// tracks = [... trackListCopy];
	tracks.set(trackListCopy);
}

function handleSubmit() {
	console.log("handle submit");
	let data = {trackDate, notes};
	db.collection('sm-tiny-track')
	.doc(trackDate)
	.set(data)
	.then(() =>{
		console.log("successfully added sm-tiny-track form");
		loadTrackList();
	})
	.catch(err => {
		console.error("Unable to add sm tiny track form , error: ", error);
	});
}
onMount ( async () => {
	loadTrackList();
})
</script>

<main>
<h2>Sumiya Tiny Track</h2>
<p>Track Namaaz, Walk, Vitamins ...</p>
<p> n - namaaz done, w - walk done, v vitamins ..</p>

<form on:submit|preventDefault={handleSubmit}>
	Date: <input type="date" bind:value={trackDate}/><br/>
	<input type="text" bind:value={notes}/><br/>
	<button type="submit">Save</button>
</form>

<hr/>

<h3> Tiny Track List</h3>
<div class="tinyTrackList">
{#each $tracks as track}
<div class="track">
	{track.trackDate} - {track.notes}
</div>
{/each}
</div>
</main>

<style>
main{
	padding: 20px;
	margin: 10px;
}
</style>