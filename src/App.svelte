<script>
	import '../node_modules/@idscan/idvc/dist/css/idvc.css';
	import { onMount } from 'svelte';
	import IDVC from '@idscan/idvc';

		
	onMount( () => {		
 		new IDVC({
            el: 'videoCapturingEl',
            licenseKey: '',
            isShowManualSwitchButton: true,
            showSubmitBtn: true,
            isShowVersion: true,
            tapOnVideo: false,
            tapBackSide: false,
            minPDFframes: 1,
            parseMRZ: false,
            tapFace: false,
            enableLimitation: false,
            autoContinue: true,
            resizeUploadedImage: 1500,
            showForceCapturingBtn: false,
            fixFrontOrientAfterUpload: false,
            enableFlash: false,
            capturingMode: '4',
            steps: [
                { type: 'front', name: 'Front Scan' },
                { type: 'face', name: 'Selfie' },
            ],
            useCDN: true,
            networkUrl: '/assets/networks',
            showPreviewForOneStep: true,
            priority: 'auto',
            realFaceMode: 'all',
            types: ['ID', 'Passport', 'PassportCard', 'GreenCard', 'InternationalId' ],
            strictAllowedTypes: false,
            enableGeolocation: false,
            displayParsedData: false,
            onChange(data) {
                console.log(data);
            },
            onCameraError(data) {
                console.log(data);
            },
            onReset(data) {
                 console.log(data);
            },
            onRetakeHook(data) {
                console.log(data);
            },
            submit(data) {   
                let frontImage,backImage,faceImage;
                let trackString;
                let frontStep, backStep, faceStep;

                switch(data.documentType){
                // Drivers License and Identification Card
                case 1:
                        frontImage = data.steps
                        .find((item) => item.type === "front")
                        .img.split(/:image\/(jpeg|png);base64,/)[2];


                        backStep = data.steps.find((item) => item.type === "back");
                        trackString = backStep && backStep.trackString ? backStep.trackString : "";
                        backImage = backStep.img.split(/:image\/(jpeg|png);base64,/)[2];


                        faceImage = data.steps
                        .find((item) => item.type === "face")
                        .img.split(/:image\/(jpeg|png);base64,/)[2];

                        break;
                    // US and International Passports
                    case 2:
                        frontStep = data.steps.find((item) => item.type === "front");
                        trackString = frontStep && frontStep.mrzText ? frontStep.mrzText : "";
                        frontImage = frontStep.img.split(/:image\/(jpeg|png);base64,/)[2];

                        faceImage = data.steps
                        .find((item) => item.type === "face")
                        .img.split(/:image\/(jpeg|png);base64,/)[2];
                    
                        break;
                    // US Passport Cards,US Green Cards,International IDs with 3 line MRZs  
                    case 3:
                    case 6:
                    case 7:
                        frontImage = data.steps
                            .find((item) => item.type === "front")
                            .img.split(/:image\/(jpeg|png);base64,/)[2];

                        backStep = data.steps.find((item) => item.type === "back");
                        trackString = backStep && backStep.mrzText ? backStep.mrzText : "";
                        backImage = backStep.img.split(/:image\/(jpeg|png);base64,/)[2];


                        faceImage = data.steps
                            .find((item) => item.type === "face")
                            .img.split(/:image\/(jpeg|png);base64,/)[2];
                        break;
                default:

                }

                let request = {
                    frontImageBase64: frontImage,
                    backOrSecondImageBase64: backImage,
                    faceImageBase64: faceImage,
                    documentType: data.documentType,
                    trackString: trackString,
                    userAgent: window.navigator.userAgent,
                    captureMethod: data.captureMethod,
                    verifyFace: true,
                };

                fetch(
                    "https://us-central1-idscan-backend.cloudfunctions.net/api/Verify",
                    {
                    method: "POST",
                    headers: {
                        "Content-Type": "application/json;charset=utf-8",
                    },
                    body: JSON.stringify(request),
                    }
                )
                .then((response) => response.json())
                .then((data) => {
                        console.log(data);
                })
                .catch((err) => {
                    console.err(err);
                });
            },
        });
	});
</script>

<main>
	<h1>DVS Demo Application</h1>
	<div id="videoCapturingEl"></div>
</main>

