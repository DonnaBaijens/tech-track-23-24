<script>
	import { redirect } from "@sveltejs/kit";
    import * as d3 from "d3";
    import { onMount } from "svelte";


    let happiness2015Data = [];
    let happiness2016Data = [];
    let happiness2017Data = [];
    let happiness2018Data = [];
    let happiness2019Data = [];

    
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

    onMount(async () => {
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
            fetchJSONData(url2019, happiness2019Data),
        ]);

        // After fetching the data, you can now generate the chart
        generateChart(happiness2015Data);
    });

    const generateChart = (selectedData) => {
        const chartWidth = 1500;
        const chartHeight = 500;
        const padding = 50;

        const xScale = d3
            .scaleLinear()
            .domain([0, d3.max(selectedData, d => d['Happiness Rank'])])
            .range([padding, chartWidth - padding * 2]);

        const xaxis = d3.axisBottom(xScale);
        xaxis.ticks(10).tickSize(5);

        const svg = d3.select("#chart")
            .attr("width", chartWidth)
            .attr("height", chartHeight);

        const xAxisGroup = svg.append("g")
            .attr("id", "xAxis")
            .attr("transform", `translate(0, ${chartHeight - 200})`)
            .call(xaxis);

        const smileys = svg.selectAll(".smileys")
            .data(selectedData.reverse())
            .join("g")
            .attr("class", "smileys")
            .attr("transform", d => `translate(${xScale(d['Happiness Rank'])}, ${chartHeight - 200})`);

        smileys.append("circle")
            .attr("r", 0)
            .transition()
            .duration(2800)
            .attr("r", d => xScale(d['Happiness Score'] / 1.5))
            .attr("fill", 'url(#smileyPattern)');
    }


    const handleClick = (event) => {
        const selectedYear = event.target.value;
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

        generateChart(selectedData);
    };

</script>    

<div id = "smileyContainer">
    <svg id="chart">
    </svg>
</div>


<svg width="0" height="0">
    <defs>
        <pattern id="smileyPattern" width="100%" height="100%" patternContentUnits="objectBoundingBox">
            <circle cx="0.5" cy="0.5" r="0.4" fill="yellow" stroke="black" stroke-width="0.02"/>
            <circle cx="0.35" cy="0.35" r="0.05" fill="black"/>
            <circle cx="0.65" cy="0.35" r="0.05" fill="black"/>
            <path d="M0.4 0.6 Q0.5 0.7 0.6 0.6" fill="transparent" stroke="black" stroke-width="0.02"/>
        </pattern>
    </defs>
</svg>

<div>
	<button on:click={handleClick} value="2015">2015</button>
	<button on:click={handleClick} value="2016">2016</button>
	<button on:click={handleClick} value="2017">2017</button>
	<button on:click={handleClick} value="2018">2018</button>
	<button on:click={handleClick} value="2019">2019</button>
</div>








