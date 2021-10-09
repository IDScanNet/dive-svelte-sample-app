<script>
	import '../node_modules/@idscan/idvc/dist/css/idvc.css';
	import { onMount } from 'svelte';
	import IDVC from '@idscan/idvc';

	let publicKey = 'REPLACE ME WITH YOUR PUBLIC KEY';
    	let backendServerUrl = 'REPLACE ME WITH YOUR BACKEND SERVICE URL';
    	let licenseKey = 'REPLACE ME WITH YOUR LICENSE KEY';
	
	onMount( () => {
			new IDVC({
			el: 'videoCapturingEl',
			networkUrl: '/assets/networks',
			tapBackSide : true,
			licenseKey: licenseKey,
			steps: [
				{type: 'front', name: 'Front Scan'},
				{type: 'face', name: 'Selfie'}
			],
			submit (data) {
				let backStep = data.steps.find(item => item.type === 'back')
				let trackString = (backStep && backStep.trackString) ? backStep.trackString : ''

				let request = {
					frontImageBase64: data.steps.find (item => item.type === 'front').img.split (/:image\/(jpeg|png);base64,/)[2],
					backOrSecondImageBase64: backStep.img.split (/:image\/(jpeg|png);base64,/)[2],
					faceImageBase64: data.steps.find (item => item.type === 'face').img.split (/:image\/(jpeg|png);base64,/)[2],
					documentType: data.documentType,
					trackString: trackString
				}
				
				fetch ('https://dvs2.idware.net/api/Request', {
					method: 'POST',
					headers: {
						'Content-Type': 'application/json;charset=utf-8',
						'Authorization': `Bearer ${publicKey}`
					},
					body: JSON.stringify (request)
				}).then (response => response.json ())
					.then (response => {
						fetch (backendServerUrl, {
							method: 'POST',
							headers: {
								'Content-Type': 'application/json;charset=utf-8'
							},
							body: JSON.stringify ({
								requestId: response.requestId
							})
						}).then (response => response.json ())
							.then (data => {
								console.log(data);
							})
					}).catch(() => {
					
				})
			}
			});
	});
</script>

<main>
	<h1>DVS Demo Application</h1>
	<div id="videoCapturingEl"></div>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
