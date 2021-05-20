<script>
	export let name;
	import { createEventDispatcher } from "svelte";
	import * as tf from '@tensorflow/tfjs';
	import * as use from '@tensorflow-models/universal-sentence-encoder';
	import * as mlpca from 'ml-pca';
	import * as d3 from "d3";
	import { scaleLinear, scaleOrdinal } from "d3-scale";
	import { extent } from "d3-array";	
	import 'milligram';
	import { bubble } from "svelte/internal";

	let ploting_pca = [
		{
			"x": 0.05575651771753806,
			"y": -0.6184774874554108,
			"author": "Smartphones "
		},
		{
			"x": 0.14286922634545163,
			"y": -0.5718559371758866,
			"author": "Smartphones "
		},
		{
			"x": 0.036885030927237515,
			"y": -0.4819638861088219,
			"author": "Smartphones "
		},
		{
			"x": 0.16407547002371187,
			"y": 0.3713582969324919,
			"author": "Weather "
		},
		{
			"x": 0.38280988531201127,
			"y": 0.36232403687060954,
			"author": "Weather "
		},
		{
			"x": 0.3702584002014175,
			"y": 0.408940611927864,
			"author": "Weather "
		},
		{
			"x": 0.13721003123848025,
			"y": -0.045553241063201425,
			"author": "Food and health "
		},
		{
			"x": 0.14774882944613382,
			"y": 0.09597778974645513,
			"author": "Food and health "
		},
		{
			"x": 0.1444092502360126,
			"y": 0.1647556819580047,
			"author": "Food and health "
		},
		{
			"x": -0.7803127108855517,
			"y": 0.15548253490219968,
			"author": " Asking about age "
		},
		{
			"x": -0.8017099305624423,
			"y": 0.15901159946569637,
			"author": " Asking about age "
		}
	];

	let data;

	let model = use.load();
	model.then(m => model = m);

	let sentences = [
		'Hello.',
		'How are you?'
	];
	let authors = ["joe", "trump"];

	let embedding;
	let reducing;
	

	// prediction 
	function predict(){
		embedding = model.embed(sentences).then(embeddings => {
			return embeddings.array().then(array => {
				const pca = new mlpca.PCA(array);
				const pcs_reduction = pca.predict(array);
				ploting_pca = pcs_reduction.data.map((r,index) => {
					return {'x':r[0], 'y':r[1], 'author':authors[index]}
				});
				console.log(ploting_pca);
				data = draw();
			})
		})
	}

	const height = 400;
	const width = 400;
	const buffer = 10;
	const colors = [ 'rgba(255, 99, 132, 1)', 'rgba(54, 162, 235, 1)', 'rgba(255, 206, 86, 1)', 'rgba(75, 192, 192, 1)', 'rgba(153, 102, 255, 1)', 'rgba(255, 159, 64, 1)' ];
	const axisSpace = 50;

	
	function draw(){
		console.log("lets draw")
		let xExtent = extent(ploting_pca, (d) => d.x);
		let yExtent = extent(ploting_pca, (d) => d.y);

		let xScale = scaleLinear()
		.domain(xExtent)
		.range([buffer + axisSpace, width - buffer]);
		
		let yScale = scaleLinear()
		.domain(yExtent)
		.range([height - buffer - axisSpace, buffer]);

		let authors = new Set(ploting_pca.map((d) => d.author));
		let colorScale = scaleOrdinal().domain(Array.from(authors)).range(colors);

		return {
			data: ploting_pca,
			xScale,
			yScale,
			colorScale,
			authors: Array.from(authors),
		};
	}
	data = draw();
	let value = "Smartphones \n I like my phone \nMy phone is not good. \nYour cellphone looks great.\n\nWeather \nWill it snow tomorrow? \nRecently a lot of hurricanes have hit the US \n Global warming is real \n\nFood and health \nAn apple a day, keeps the doctors away \nEating strawberries is healthy \nIs paleo better than keto? \n\n Asking about age \nHow old are you? \nwhat is your age";
	function clean(){
		let new_authors = [];
		let new_sentences = []
		value.split('\n\n').map(group => {
			let author;
			group.split('\n').map((s, index) => {
				if(index == 0 ){
					author = s 
				}else{
					new_authors.push(author)
					new_sentences.push(s)
				}
			})
		})

		sentences = new_sentences
		authors = new_authors
		predict();

	}
</script>
<svelte:head>
	<title>USE - PCA</title>
	<meta name="robots" content="noindex nofollow" />
	<html lang="en" />
</svelte:head>
<main>
<div class="container">
  <div class="row">
    <h3> Universal Sentence Encoder PCA Visualizer</h3>
  </div>
 <div class="row intro" style="">
	<p>The idea of this visualization is to see where different text appears in a latent space. Text with similar meaning is closer to each other.</p>
	<p> Universal Sentence Encoder encodes text into high dimensional (512) vector  using a Transofrmer (in this case).</p>
	<p> Principal Component Analysis (PCA) is a used for dimensionality reductions by projecting each point onto first principal components</p>
	<p> <strong>How to use:</strong> The first line of each block is the label, the other lines are sentences with that label.</p>

</div>
  

  <div class="row">
    <div class="column">
		<textarea bind:value={value} rows="20" cols="50"></textarea>
		
		{#await embedding}
			embedding .. 
		{:then} 
			<button class="button" on:click="{clean}">analyze</button>
		{/await}
	</div>

    <div class="column">
		<svg {height} {width} style="background-color:#808080;">
			{#each data.data as item}
			<circle
				r="3"
				cx={data.xScale(item.x)}
				cy={data.yScale(item.y)}
				fill={data.colorScale(item.author)} />
			{/each}

			{#each data.xScale.ticks(5) as tick}
			<g transform={`translate(${data.xScale(tick)} ${height - 20})`}>
				<line y1="-5" y2="0" stroke="black" />
				<text y="20" text-anchor="middle">{tick}</text>
			</g>
			{/each}

			{#each data.yScale.ticks(5) as tick}
			<g transform={`translate(0 ${data.yScale(tick)})`}>
				<line x1="35" x2="40" stroke="black" />
				<text x="30" dominant-baseline="middle" text-anchor="end">{tick}</text>
			</g>
			{/each}

			<g transform={`translate(${width - 25}, ${height - 100})`}>
			{#each data.authors as author, i}
				<g transform={`translate(0 ${i * 20})`}>
				<rect height="10" width="10" fill={data.colorScale(author)} />
				<text dominant-baseline="middle" y="5" x="20">{author}</text>
				</g>
			{/each}
			</g>
			
		</svg>	
	</div>
  </div>
</div>
</main>


<style>
	main {
		text-align: center;
		margin: 0 auto;
	}

	h3 {
		color: #9b4dca;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
	svg {
		width: 100% !important;
	}
	textarea{
		height: 50vh;
	}
	.intro{
		flex-direction: column;
		text-align: left;
	}
	.intro p {
		margin-bottom: 1rem;
	}
</style>
