<script>
	import { fade } from 'svelte/transition';
	let modules = {};
	let timetable = {};
	fetch(`http://${window.location.host}/modules.json`)
		.then(response => response.json())
		.then(data => modules = data);
	fetch(`http://${window.location.host}/timetable.json`)
		.then(response => response.json())
		.then(data => timetable = data);

	let subject = '';
	let module = '';
	let color = '';
	let showWeekend = true;
	let codes = ["A", "B", "C", "D", "E", "F", "G"];
	let days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
	let selected = localStorage.getItem("selected") === null ? {} : JSON.parse(localStorage.getItem("selected"));

	function toggleWeekend () {
		showWeekend = !showWeekend;
		if (showWeekend) {
			codes = ["A", "B", "C", "D", "E", "F", "G"];
			days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
		} else {
			codes = ["A", "B", "C", "D", "E"];
			days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"];
		}
	} 

	function add() {
		if (subject !== '' && modules !== ''){
			let details = modules[subject].filter(x => x.number === module);
			selected[subject + "_" + module] = {"subject": subject, "number": details[0].number, "name": details[0].title, "color" : color};
		} else {
			alert("No subject or module selected");
		}
	}

	function del (x) {
		let temp = selected;
		delete temp[x];
		selected = temp;
	}

	function generate () {
		localStorage.setItem("selected", JSON.stringify(selected));
		var ex = document.getElementsByClassName("entry");
		while (ex[0]){
			ex[0].parentNode.removeChild(ex[0]);
		}
		
		Object.keys(selected).forEach(key => {
			timetable[key].forEach(entry => {
				var cell = document.getElementById(entry.timecode);
				var toAdd = '';
				var weeks = JSON.parse("["+ entry.weeks +"]");
				var range = weeks.length >= 20 ? '' : Math.min(...weeks) === Math.max(...weeks) ? `[${Math.min(...weeks)}]` :`[${Math.min(...weeks)}-${Math.max(...weeks)}]`; 
				if (entry.compulsory === "") {
					toAdd = `<p class="entry" style="margin:0; color: ${selected[key].color};">
								(${entry.activity}) ${selected[key].name} ${range}
							</p>`;
				} else {
					toAdd = `<p class="entry" style="margin:0; color: ${selected[key].color}; font-size:small">
								<u>(${entry.activity}) ${selected[key].name}</u>
							</p>`
				}

				cell.innerHTML += toAdd;
			})
		});
	}

	function print () {
		var myWindow = window.open("");
		myWindow.document.write(`
		<head>
			<title>Your Timetable</title>
			<link rel="stylesheet" href="style.css">
		</head>
		<body>
			${document.getElementById("timetable").outerHTML}
		</body>`); 
	}
</script>

<header>
	<h3>Durham 2020-21 Timetable</h3>
	<p>
		<a target ="_blank" href="https://www.maths.dur.ac.uk/clash_checker/help.html">Help</a> / 
		<a target ="_blank" href="http://timetable.dur.ac.uk/">Official DU Timetable</a>
	</p>
</header>

<fieldset>
	<p><strong>Manage Modules</strong></p>
	<hr>
	<form on:submit|preventDefault="{add}">
		<label for="sub">Subject</label>
		<select id="sub" bind:value="{subject}">
			<option value=''>Select</option>
			{#each Object.keys(modules) as subject}
				<option>{subject}</option>
			{/each}
		</select>
		
		<span style="margin-left: 0.5rem"></span>
		
		<label for="mod">Module</label>
		<select id="mod" bind:value="{module}">
			<option value=''>Select</option>
			{#if subject !== ''}
				{#each modules[subject] as mod}
					<option value="{mod.number}">{`${mod.number} ${mod.title}`}</option>
				{/each}
			{/if}
		</select>

		<span style="margin-left: 0.5rem"></span>
		
		<label for="col">Colour</label>
		<select id="col" bind:value="{color}">
			<option value="black">Select</option>
			{#each ["crimson", "coral", "seagreen", "teal", "steelblue", "navy", "indigo"] as clr}
				<option value={clr} style="color: {clr}; font-weight: bold;">{clr}</option>
			{/each}
		</select>

		<span style="margin-left: 0.5rem"></span>
		
		<button type="submit">Add</button>
	</form>

	{#if Object.keys(selected).length !== 0}
		<div style="margin: 1rem 1rem" transition:fade>
			{#each Object.keys(selected) as x}
				<div style="margin-bottom: 0.4rem">
					<span on:click={() => {del(x)}} style="cursor: pointer;">🗑</span>
					<span style="color: {selected[x].color}">{selected[x].number} {selected[x].name}</span>
				</div>
			{/each}
		</div>
		<button on:click={generate}>Update</button>
		<button on:click={print}><span style="font-size:0.8rem">🖨</span> Print</button>
		<button on:click={toggleWeekend}>{showWeekend === true ? 'Hide Weekend' : 'Show Weekend'}</button>
	{/if}
</fieldset>

<section>
	<br>
	<table id="timetable">
		<thead>
			<th style="border:none; background: white;"></th>
			{#each days as day}
				<th>{day}</th>
			{/each}
		</thead>
		<tbody>
			<tr>
				<td>08:00</td>
				{#each codes.map(letter => letter + "1") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>09:00</td>
				{#each codes.map(letter => letter + "2") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>10:00</td>
				{#each codes.map(letter => letter + "3") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>11:00</td>
				{#each codes.map(letter => letter + "4") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>12:00</td>
				{#each codes.map(letter => letter + "5") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>01:00</td>
				{#each codes.map(letter => letter + "6") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>02:00</td>
				{#each codes.map(letter => letter + "7") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>03:00</td>
				{#each codes.map(letter => letter + "8") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>04:00</td>
				{#each codes.map(letter => letter + "9") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>05:00</td>
				{#each codes.map(letter => letter + "10") as code}
					<td id={code}></td>
				{/each}
			</tr>
			<tr>
				<td>06:00</td>
				{#each codes.map(letter => letter + "11") as code}
					<td id={code}></td>
				{/each}
			</tr>
		</tbody>
	</table>
</section>

<br>

<blockquote>
	<p>Notice</p>
	<ul>
		<li>Compulsory events are underlined. If there is more than one event in a cell is underlined then there might be a clash.</li>
		<li>In a cell, entry text is in the format "(X) Module Name [a-b]" where "(X)" is the <a href="https://www.maths.dur.ac.uk/clash_checker/help.html">nature of the activity</a> and "[a-b]" shows the range of teaching weeks the activity is supoose to take place.</li>
		<li>This utility is just a hobby project and therefore has not been robustly tested please use 
			<a href="http://www.maths.dur.ac.uk/clash_checker/module_checker.html">Timetable Compatibility Utility</a> to check the timetable generated here is correct.
		</li>
	</ul>
</blockquote>

<br>

<footer style="font-size:small; color: gray">
	<hr>
	<p>Made by Atharva.<br>Use Firefox to support Mozilla's mission of a healthy and open internet.</p>
</footer>

<style>

	td:hover {
		background-color: aliceblue;
	}
</style>