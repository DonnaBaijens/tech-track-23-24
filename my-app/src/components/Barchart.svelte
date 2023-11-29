<script>
	import { redirect } from '@sveltejs/kit';
	import * as d3 from 'd3';
	import { onMount } from 'svelte';

	let happiness2015Data = [];
	let happiness2016Data = [];
	let happiness2017Data = [];
	let happiness2018Data = [];
	let happiness2019Data = [];

    let selectedOption = 'Happiness Score';
	let selectedYear = '2015';

	async function fetchJSONData(url, happinessData) {
		const response = await fetch(url);
		if (response.ok) {
			const data = await response.json();
			happinessData.push(...data); // Append data to the existing array
			console.log(happinessData);
		} else {
			console.error('Failed to fetch the data');
		}
	}

	onMount(() => {
		fetchDataAndUpdateChart();


        const button2015 = document.querySelector("div button:first-of-type")
        button2015.addEventListener('click', () => {
            button2015.classList.remove('selectedbutton');
        })
	});

	async function fetchDataAndUpdateChart() {
		const url2015 = '/data/2015.json';
		const url2016 = '/data/2016.json';
		const url2017 = '/data/2017.json';
		const url2018 = '/data/2018.json';
		const url2019 = '/data/2019.json';

		await Promise.all([
			fetchJSONData(url2015, happiness2015Data),
			fetchJSONData(url2016, happiness2016Data),
			fetchJSONData(url2017, happiness2017Data),
			fetchJSONData(url2018, happiness2018Data),
			fetchJSONData(url2019, happiness2019Data)
		]);

console.log('hoi', happiness2017Data);

		let updatedData2017 = replaceKey(happiness2017Data, 'Happiness.Score', 'Happiness Score');
		updatedData2017 = replaceKey(updatedData2017, 'Health..Life.Expectancy.', 'Health (Life Expectancy)');
		happiness2017Data = updatedData2017;

		let updatedData2018 = replaceKey(happiness2018Data, 'Social support', 'Family');
		updatedData2018 = replaceKey(updatedData2018, 'Country or region', 'Country');
		updatedData2018 = replaceKey(updatedData2018, 'Score', 'Happiness Score');
		updatedData2018 = replaceKey(updatedData2018, 'Healthy life expectancy', 'Health (Life Expectancy)');
		updatedData2018 = replaceKey(updatedData2018, 'Freedom to make life choices', 'Freedom');
		happiness2018Data = updatedData2018;

		let updatedData2019 = replaceKey(happiness2019Data, 'Social support', 'Family');
		updatedData2019 = replaceKey(updatedData2019, 'Country or region', 'Country');
		updatedData2019 = replaceKey(updatedData2019, 'Score', 'Happiness Score');
		updatedData2019 = replaceKey(updatedData2019,'Healthy life expectancy', 'Health (Life Expectancy)');
		updatedData2019 = replaceKey(updatedData2019, 'Freedom to make life choices', 'Freedom');
		happiness2019Data = updatedData2019;

		console.log('deze1', updatedData2017);
		console.log('deze2', updatedData2018);
		console.log('deze3', updatedData2019);

        console.log('dit', updatedData2017)

		// After updating the data, regenerate the chart
		generateChart(happiness2015Data, selectedOption);
	}

	function replaceKey(dataArray, oldKey, newKey) {
		return dataArray.map((oldDataObj) => {
			const newObj = { ...oldDataObj };
			if (oldKey in oldDataObj) {
				newObj[newKey] = newObj[oldKey];
				delete newObj[oldKey];
			}
			return newObj;
		});
	}

	const generateChart = (selectedData, selectedOption) => {
		console.log('generate Chart' + selectedData[0][selectedOption]);
        console.log(selectedData.length);

		d3.select('#chartContainer svg').remove();

		const chartWidth = 1500;
		const chartHeight = 3000;

		const xScale = d3
			.scaleLinear()
			.domain([0, d3.max(selectedData, (d) => d[selectedOption])])
			.range([0, chartWidth - 155]);

		const yScale = d3
			.scaleBand()
			.domain(selectedData.map((d) => d.Country))
			.range([0, chartHeight])
			.paddingInner(0.1);

		const yaxis = d3.axisLeft(yScale);
		yaxis.ticks(5).tickSize(5);

		const svg = d3
			.select('#chartContainer')
			.append('svg')
			.attr('width', chartWidth)
			.attr('height', chartHeight);

		const yAxisGroup = svg
			.append('g')
			.attr('id', 'yAxis')
			.attr('transform', 'translate(145, 0)')
			.call(yaxis);

		const colorScaleQuant = d3
			.scaleQuantize()
			.domain([0, d3.max(selectedData, (d) => d[selectedOption])])
			.range(['purple', 'blue', 'green', 'yellow', 'orange', 'red']);

		const smileyWidth = 17;
		const smileyHeight = 17;

		const defs = svg.append('defs');

		defs
			.append('pattern')
			.attr('id', 'smileyPattern')
			.attr('width', smileyWidth)
			.attr('height', smileyHeight)
			.attr('patternUnits', 'userSpaceOnUse')
			.append('image')
			.attr('xlink:href', '/images/Smiley.svg')
			.attr('width', smileyWidth)
			.attr('height', smileyHeight);

		const bars = svg
			.selectAll('.bar')
			.data(selectedData)
			.join('g')
			.attr('class', 'bar')
			.attr('transform', (d) => `translate(145, ${yScale(d.Country)})`);

		bars
			.append('rect')
            .attr('width', 0)
            .attr('height', yScale.bandwidth())
            .attr('fill', 'url(#smileyPattern)')
            .transition()
            .duration(3000)
            .ease(d3.easeCubicInOut)
			.attr('width', (d) => {
				const numberOfSmileys = Math.floor(xScale(d[selectedOption]) / smileyWidth);
				return numberOfSmileys * smileyWidth;
			})
			.attr('stroke', 'black');

		bars
			.append('text')
			.attr('y', yScale.bandwidth() / 1.2)
			.attr('x', 0)
			.transition()
			.duration(3000)
			.tween('text', function (d) {
				const interpolator = d3.interpolate(0, d[selectedOption]);
				return function (t) {
					if (d[selectedOption] != 0) {
						d3.select(this).text(interpolator(t).toFixed(4)); // Adjust the precision as needed
					} else {
						d3.select(this).text(interpolator(t).toFixed(0));
					}
				};
			})
			.attr('x', (d) => (d[selectedOption] === 0 ? 20 : xScale(d[selectedOption]) - 80))
			// .text(d => d.Family)
			.attr('fill', 'black');
	};



	const handleClick = (event) => {
		selectedYear = event.target.value;
		console.log('Selected year:', selectedYear);

		// Dynamically choose the selected dataset based on the year
		let selectedData;

		switch (selectedYear) {
			case '2015':
				selectedData = happiness2015Data;
				break;
			case '2016':
				selectedData = happiness2016Data;
				break;
			case '2017':
				selectedData = happiness2017Data;
				break;
			case '2018':
				selectedData = happiness2018Data;
				break;
			case '2019':
				selectedData = happiness2019Data;
				break;
			default:
				// Handle default case or provide an error message
				selectedData = [];
				break;
		}

        console.log('selected data:', selectedData)
        console.log('selected option:', selectedOption)

		generateChart(selectedData, selectedOption);
	};

	function handleSelection(event) {
		selectedOption = event.target.value;
		// Add logic to handle the selected option, such as updating the chart
		generateChart(getSelectedData(), selectedOption);
	}

	function getSelectedData() {
		switch (selectedYear) {
			case '2015':
				return happiness2015Data;
			case '2016':
				return happiness2016Data;
			case '2017':
				return happiness2017Data;
			case '2018':
				return happiness2018Data;
			case '2019':
				return happiness2019Data;
			default:
				return [];
		}
	}

    
</script>

<div class="dropdown">
	<select bind:value={selectedOption} on:change={handleSelection}>
		<option value="Happiness Score">Happiness Score</option>
		<option value="Family">Family</option>
		<option value="Freedom">Freedom</option>
		<option value="Health (Life Expectancy)">Health (Life Expectancy)</option>
	</select>
</div>

<div>
	<button class="selectedbutton" on:click={handleClick} value="2015">2015</button>
	<button on:click={handleClick} value="2016">2016</button>
	<button on:click={handleClick} value="2017">2017</button>
	<button on:click={handleClick} value="2018">2018</button>
	<button on:click={handleClick} value="2019">2019</button>
</div>

<div id="chartContainer" />


<style>
    div {
        padding: 10px;
    }

    div button {
        padding: 8px;
        margin: 3px;
        background-color: rgb(238, 238, 46); 
        font-size: 15px;
        font-family: 'Josefin Sans';
        border: 2px black;
        border-radius: 5px;
        cursor: pointer;
        transition: ease-in-out 400ms;
        
    }

    .selectedbutton {
        background-color: rgb(240, 179, 65);
    }


    div button:hover {
        background-color: rgb(240, 179, 65);
        transform: scale(1.5);
    }

    div button:focus {
        background-color: rgb(240, 179, 65);
    }


	.dropdown {
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 40px;
	}

	/* Style the dropdown select */
	select {
		padding: 20px;
        text-align: center;
		font-size: 40px;
		font-family: 'Pacifico'; 
        border: 5px transparent solid;
        border-radius: 30px;	
        background:  rgb(240, 179, 65);
        cursor: pointer;      
        transition: ease-in-out 200ms; 
	}

    div select:hover {
        border: 5px black solid
    }


</style>
