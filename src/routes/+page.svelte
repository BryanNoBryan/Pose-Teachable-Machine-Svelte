<!-- PARTIAL CREDIT TO NOAH GLASER -->
<!-- https://www.youtube.com/watch?v=vriVyygztLM -->

<script>
	import { onMount } from 'svelte';

	import * as tmPose from '@teachablemachine/pose';

	// @ts-ignore
	let videoEl;
	// @ts-ignore
	/**
	 * @type {string}
	 */
	let errorMessage;
	// @ts-ignore
	/**
	 * @type {tmPose.CustomPoseNet}
	 */
	let model;
	let loading = true;
	let percentage = '';
	let name = '';

	const URL = 'https://teachablemachine.withgoogle.com/models/NESqpYDCu/';
	const modelURL = URL + 'model.json';
	const metadataURL = URL + 'metadata.json';

	onMount(async () => {
		try {
			model = await tmPose.load(modelURL, metadataURL);
			const stream = await navigator.mediaDevices.getUserMedia({ video: true });

			// @ts-ignore
			videoEl.srcObject = stream;
			// @ts-ignore
			videoEl.play();
			setInterval(predict, 400);
			loading = false;
		} catch (e) {
			console.error(e, 'camera access denied');
			errorMessage = 'Camera Access Denied';
		}
	});

	async function predict() {
        console.log('b4');
        console.log(model.getTotalClasses());
        // @ts-ignore
        const { pose, posenetOutput } = await model.estimatePose(videoEl);
        const predictions = await model.predict(posenetOutput);
        console.log(predictions);

        const [choosenPrediction] = predictions.sort(
            (a, b) => b.probability - a.probability
        );

		if (choosenPrediction) {
			percentage = (choosenPrediction.probability * 100).toFixed(2) + '%';
			name = classNameToLabel(choosenPrediction.className);
		}
	}

	// @ts-ignore
	function classNameToLabel(className) {
		switch (className) {
			case 'No Tilt':
				return 'No Tilt';
            case 'Left Tilt':
				return 'Left Tilt';
            case 'Right Tilt':
				return 'Right Tilt';
			default:
				return 'Nothing';
		}
	}
</script>

<main>
	<h1>Machine Learning Poses Bryan</h1>
	<video bind:this={videoEl} width="600" height="480" />

	{#if errorMessage}
		<h2 style="color: red;">{errorMessage}</h2>
	{:else if loading}
		<h2>Loading ...</h2>
	{:else if percentage && name}
		<h2>AI {percentage} certain it's a {name}</h2>
	{/if}
</main>

<style>
	main {
		width: 100%;
		height: 100vh;
		padding: 0;
		box-sizing: border-box;
		position: absolute;
	}

	video {
		display: block;
		margin: 20px auto;
        height: 400px;
	}

	h1,
	h2 {
		text-align: center;
	}

	h1 {
		font-size: 40px;
	}

	h2 {
		font-size: 30px;
	}
</style>
