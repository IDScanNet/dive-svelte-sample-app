<script>
	import '../node_modules/@idscan/idvc/dist/css/idvc.css';
	import { onMount } from 'svelte';
	import IDVC from '@idscan/idvc';

	let publicKey = 'pk_f123d212-fd2c-4d22-aae2-e3eb61765646';
    let backendServerUrl = 'https://us-central1-idscan-backend.cloudfunctions.net';
    let licenseKey = 'eyJwZGY0MTdrZXkiOiJNYTR4b1JBTHlCaU5yRmFLVGpLRG8vN0ZKQnJPdlpMU3VjbDFLZXV6cEliUSsxVXZWeS95MUVwSUFleUF2bE5HUlhGblQ0ZTl0K2RQK2wxMjlYYkxtZHRHRmVabHBRUVZOSGU5dGVBcEo2VzFwSGpWOFJJL2d1WW1YWEgzZ3htQlBaNC9sTUdJdVQzdWZHOFZpUjl2dFc0SFlhUTZGTU5aUFhKeUdvODZsSEU9IiwiaW1hZ2VQcm9jZXNzaW5nS2V5IjoiUmduMktPemNZaXNsd01YUkNxSC8ra3ZPNnBOanFUUlJlZDlTdWMrbXY3WHZkSTJtU3dFLzdIRk1qOFd3ZkZ6STExZmJneC94eDRrV2ppdmJtdWxhNXJYNzhxS1lZWnpsb05hVjI5bFZMSUlsNHJ3VjZycC9iU3VBL2dOMkxMbXRkcUZrbkhwQk5nNHdWbE9LRENodTZVNm5FRmZla1FuU1QxSjc0RDFpcVEwPSJ9';
	
	onMount( () => {
			new IDVC({
			el: 'videoCapturingEl',
			networkUrl: '/assets/networks',
			tapBackSide : true,
			licenseKey: licenseKey,
			steps: [
				{type: 'front', name: 'Front Scan'},
				{type: 'back', name: 'Back Scan'},
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
						fetch (backendServerUrl + '/api/ValidationRequests/complete/', {
							method: 'POST',
							headers: {
								'Content-Type': 'application/json;charset=utf-8'
							},
							body: JSON.stringify ({
								requestId: response.requestId,
								documentType: response.documentType
							})
						}).then (response => response.json ())
							.then (data => {
								
								alert((data.payload.isDocumentSuccess) ? 'Document valid' : 'Document invalid')
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