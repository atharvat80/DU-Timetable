<script>
import { detach_between_dev } from 'svelte/internal';

	import { fade } from 'svelte/transition';

	// Fetch necessary data files
	let modules = {};
	let timetable = {};
	let weeks = {}
	fetch(`${document.baseURI}modules.json`)
		.then(response => response.json())
		.then(data => modules = data);
	fetch(`${document.baseURI}timetable.json`)
		.then(response => response.json())
		.then(data => timetable = data);
	fetch(`${document.baseURI}teaching_weeks.json`)
		.then(response => response.json())
		.then(data => weeks = data);

	let subject = '';
	let module = '';
	let color = '';
	let edit = false;
	let showWeekend = true;
	let days = {
		"A": ["Monday", 0],
		"B": ["Tuesday", 1],
		"C": ["Wednesday", 2],
		"D": ["Thursday", 3],
		"E": ["Friday", 4],
		"F": ["Saturday", 5],
		"G": ["Sunday", 6]
	};

	let hours = ["08:00", "09:00", "10:00", "11:00", "12:00", "01:00", "02:00", "03:00", "04:00", "05:00", "06:00"];
	let selected =  {};
	let events = {};

	function add() {
		if (subject !== '' && modules !== ''){
			let details = modules[subject].filter(x => x.number === module);
			selected[subject + "_" + module] = {"subject": subject, "number": details[0].number, "name": details[0].title, "color" : color};
			generate();
		} else {
			alert("No subject or module selected");
		}
	}

	function del (x) {
		delete selected[x];
		selected = selected;
	}

	function delEntry (code, event) {
		const index = events[code].indexOf(event);
		if (index > -1) {
			events[code].splice(index, 1);
		}
		events = events;
	}

	function getCode(day, hour){
		return day + (hours.indexOf(hour) + 1)
	}

	function getRange(weeks){
		let w = JSON.parse("["+weeks+"]");
		let min = Math.min(...w);
		let max = Math.max(...w);
		return w.length >= 20 ? '' : min === max ? '[' + min +']'  :'[' + min + '-' + max +']' 
	}

	function generate () {
		events = {};
		Object.keys(selected).forEach(key => {
			timetable[key].forEach(entry => {
				entry["module"] = key;
				events[entry.timecode] = events[entry.timecode] === undefined ? [entry] : [...events[entry.timecode], entry];
			})
		});
	}

	function print () {
		var myWindow = window.open("");
		myWindow.document.write(`
		<head>
			<title>Your Timetable</title>
			<link rel="stylesheet" href="style.css">
			<style>
				.close {display: none};
			</style>
		</head>
		<body>
			${document.getElementById("timetable").outerHTML}
		</body>`); 
	}

	function toggleWeekend () {
		showWeekend = !showWeekend;
		if (showWeekend) {
			days["F"] = ["Saturday", 5];
			days["G"] = ["Sunday", 6];
		} else {
			delete days["F"];
			delete days["G"];
		}
		days = days;
	}

	// Everything relating to ics export
	Date.prototype.addDays = function(days) {
		var date = new Date(this.valueOf());
		date.setDate(date.getDate() + days);
		return date;
	}
	Date.prototype.addHours = function(h) {
		this.setTime(this.getTime() + (h*60*60*1000));
		return this;
	}

	function exp () {
		let btn = document.getElementById("export");
		btn.innerHTML = "Generating...";
		btn.setAttribute('disabled', true);
		let cal = ics();
		let timeCode = {"1": 8, "2": 9, "3": 10, "4": 11, "5": 12, "6": 13, "7": 14, "8": 15, "9": 16, "10": 17, "11": 18}
		Object.keys(events).forEach(cell => {
			events[cell].forEach(entry => {
				let w = JSON.parse("["+entry.weeks+"]");
				w.forEach(week => {
					let begin = new Date(weeks[week]);
					begin = begin.addDays(days[entry.timecode.slice(0, 1)][1]);
					begin.setHours(timeCode[entry.timecode.slice(1)], 0, 0);
					let end = new Date(begin).addHours(1);
					cal.addEvent(`(${entry.activity}) ${selected[entry.module].name}`, '', '', begin, end);
				})
			})
		})
		btn.innerHTML = "Export as ics";
		btn.removeAttribute('disabled');
		cal.download("Durham")
	}
</script>

<header>
	<h3>Durham 2020-21 Timetable</h3>
	<p>
		<a target ="_blank" href="https://www.maths.dur.ac.uk/clash_checker/help.html">Help</a> / 
		<a target ="_blank" href="http://timetable.dur.ac.uk/">Official DU Timetable</a>
	</p>
</header>

<!-- <fieldset> -->
	<h4>Manage Modules</h4><hr>
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
					<option value="{mod.number}">{mod.number} {mod.title}</option>
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
		<p transition:fade>Module names are editable. Change them as you wish and click update.</p>
		<fieldset transition:fade>
			{#each Object.keys(selected) as x}
				<div style="margin-bottom: 0.4rem">
					<span on:click={() => {del(x)}} style="cursor: pointer;">🗑</span>
					<span style="color: {selected[x].color}">{selected[x].number} 
						<span contenteditable="true" bind:innerHTML="{selected[x].name}">{selected[x].name}</span>
					</span>
				</div>
			{/each}
		</fieldset>
	{/if}
<!-- </fieldset> -->

<section>
	<br>
	<h4>Preview</h4><hr>
	<blockquote>
		<p>Please Note</p>
		<ul>
			<li>This utility is just a hobby project and therefore has not been robustly tested please use 
				<a href="http://www.maths.dur.ac.uk/clash_checker/module_checker.html">Timetable Compatibility Utility</a> to check the timetable generated here is correct.
			</li>
			<li>Compulsory events are underlined. If there is more than one event in a cell is underlined then there might be a clash.</li>
			<li>In a cell, entry text is in the format "(X) Module Name [a-b]" where "(X)" is the <a href="https://www.maths.dur.ac.uk/clash_checker/help.html">nature of the activity</a> and "[a-b]" shows the range of teaching weeks the activity is suppose to take place.</li>
			<li>Remove the events you won't be attending and click "🖨 Print" to print or "Export as ics" to import this to you online calendar.</li>
			<li>Please <u>create a new calendar</u> and then import the ics file download from here to avoid any side effects to your main calendar. Refresh the page after importing.</li>
		</ul>
	</blockquote>
	<p>
		<button on:click={print}><span style="font-size:0.8rem">🖨</span> Print</button>
		<button on:click={toggleWeekend}>{showWeekend === true ? 'Hide Weekend' : 'Show Weekend'}</button>
		<button 
			on:click={Object.keys(selected).length === 0 ? alert("No modules selected") : exp} 
			title="You can import this file in Google calendar to add this timetable to it." 
			id="export">Export as ics</button>
	</p>

	<table id="timetable">
		<thead>
			<th style="border:none; background: white;"></th>
			{#each Object.keys(days) as day}
				<th style="white-space: nowrap;">{days[day][0]}</th>
			{/each}
		</thead>
		<tbody>
			<!-- Create grid -->
			{#each hours as hour}
				<tr>
					<td style="white-space: nowrap;">{hour}</td>
					{#each Object.keys(days) as day}
						<td id={getCode(day,hour)}>
							<!-- Add events to cell if they exist -->
							{#if events[getCode(day,hour)] !== undefined}
								{#each events[getCode(day,hour)] as event}
									<!-- check if the module hasn't been deleted -->
									{#if selected[event.module] !== undefined}
										<p style="color:{selected[event.module].color}" class="entry">
											<span class="close" on:click={() => {delEntry(getCode(day,hour), event)}}>×</span>
											({event.activity}) {@html selected[event.module].name} {getRange(event.weeks)}
										</p>
									{/if}
								{/each}
							{/if}
						</td>
					{/each}
				</tr>
			{/each}
		</tbody>
	</table>
</section>

<br>

<footer style="font-size:small; color: gray">
	<hr>
	<p>Made by Atharva.<br>Use Firefox to support Mozilla's mission of a healthy and open internet.</p>
</footer>


<style>
	.entry{
		background-color: whitesmoke;
		border: #1A1A1A 1px solid;
		border-radius: 0.5rem;
		padding: 0.5rem;
		margin: 0 0 0.5rem 0;
	}

	.close {
		float:right;
		position: relative;
		padding: 0 0.25rem;
		background: red;
		color: white;
		font-size: x-small;
		border-radius: 50%;
		/* border: 0.1rem white solid; */
		top: -15px;
		right: -15px;
		cursor: pointer;
	}
</style>