<html>
<head>
    <style>
         body {
            text-align: center;
            font-family: Arial, sans-serif;
            background-color: #ffffff;
            margin: 0;
            padding: 0;
        }
        h1 {
            margin-top: 20px;
        }
        #webcam-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-top: 20px;
        }
        #highest-prediction {
            margin-top: 20px;
        }
        button {
            padding: 10px 20px;
            font-size: 18px;
            background-color: #0077cc;
            color: #fff;
            border: none;
            cursor: pointer;
            margin: 10px;
        }
        @media screen and (max-width: 480px) {
            h1 {font-size: 24px;}
            button {font-size: 16px;}
        }
    </style>
</head>
<body>
    <h1>Identifikasi Spesies Rumput Laut</h1>
    <h2>Dekatkan kamera hingga bagian rumput laut terlihat jelas</h2>
    <h3>Tekan tombol fiksasi lalu tahan kamera selama 15 detik untuk mendapatkan hasil tetap</h3>
    <button type="button" onclick="init()">Mulai</button>
    <button type="button" onclick="calculateHighestProbability()">Fiksasi Identifikasi</button>
    <button type="button" onclick="flipCamera()">Ganti Kamera</button>
    <div id="webcam-container">
        <video autoplay playsinline muted id="webcam"></video>
    </div>
    <div id="highest-prediction"></div>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>
    <script type="text/javascript">
        const URL = "https://teachablemachine.withgoogle.com/models/ncWRi_xq6/";
        let model, webcam, highestPrediction;
        let highestClass = "";
        let highestProbabilityInFifteenSecond = 0;
        let calculating = false;
        let currentCamera = 0;
        let videoDevices = [];

        async function init() {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";
            model = await tmImage.load(modelURL, metadataURL);

            videoDevices = await navigator.mediaDevices.enumerateDevices();
            const constraints = { video: true };

            try {
                const stream = await navigator.mediaDevices.getUserMedia(constraints);
                const videoElement = document.getElementById("webcam");
                videoElement.srcObject = stream;
                videoElement.play();
                webcam = videoElement;
                window.requestAnimationFrame(loop);
                document.getElementById("webcam-container").appendChild(videoElement);
                highestPrediction = document.getElementById("highest-prediction");
            } catch (error) {
                console.error("Error accessing webcam:", error);
            }
        }

        async function loop() {
            if (webcam) {
                await predict();
                window.requestAnimationFrame(loop);
            }
        }

        async function predict() {
            if (webcam) {
                const prediction = await model.predict(webcam);
                let highestProbability = 0;
                for (let i = 0; i < prediction.length; i++) {
                    if (prediction[i].probability > highestProbability) {
                        highestProbability = prediction[i].probability;
                        highestClass = prediction[i].className;
                    }
                }
                const highestPredictionText = `Highest Probability: ${highestClass} (${(highestProbability * 100).toFixed(2)}%)`;
                highestPrediction.innerHTML = highestPredictionText;
                if (calculating) {
                    if (highestProbability > highestProbabilityInFifteenSecond) {
                        highestProbabilityInFifteenSecond = highestProbability;
                    }
                }
            }
        }

        function calculateHighestProbability() {
            calculating = true;
            setTimeout(() => {
                calculating = false;
                alert(`Highest Probability in 15 second: ${highestClass} (${(highestProbabilityInFifteenSecond * 100).toFixed(2)}%)`);
                highestProbabilityInFifteenSecond = 0;
            }, 15000);
        }

        async function flipCamera() {
            if (webcam) {
                currentCamera = (currentCamera + 1) % videoDevices.length;
                const constraints = {
                    video: { deviceId: { exact: videoDevices[currentCamera].deviceId } }
                };
                try {
                    const stream = await navigator.mediaDevices.getUserMedia(constraints);
                    webcam.srcObject = stream;
                    webcam.play();
                } catch (error) {
                    console.error("Error flipping camera:", error);
                }
            }
        }
    </script>
</body>
</html>
