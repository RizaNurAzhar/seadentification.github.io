# mangjajing.github.io
<!DOCTYPE html>
<html>
<head>
    <title>Seaweed Species Identifier</title>
    <style>
        body {
            text-align: center;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
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
            h1 {
                font-size: 24px;
            }

            button {
                font-size: 16px;
            }
        }
    </style>
</head>
<body>
    <h1>Seaweed Species Identifier</h1>

    <!-- Start button -->
    <button type="button" onclick="init()">Start</button>

    <!-- Webcam and highest prediction display area -->
    <div id="webcam-container">
        <video autoplay playsinline muted id="webcam"></video>
    </div>
    <div id="highest-prediction"></div>

    <!-- TensorFlow.js and Teachable Machine scripts -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>

    <!-- Your JavaScript code (modified with getUserMedia) -->
    <script type="text/javascript">
        const URL = "https://teachablemachine.withgoogle.com/models/dZkZH7W2Q/";
        let model, webcam, highestPrediction;
        let highestClass = ""; // Declare highestClass here

        async function init() {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";

            model = await tmImage.load(modelURL, metadataURL);

            // Use getUserMedia to access the webcam
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
                        highestClass = prediction[i].className; // Assign highestClass here
                    }
                }

                const highestPredictionText = `Highest Probability: ${highestClass} (${(highestProbability * 100).toFixed(2)}%)`;
                highestPrediction.innerHTML = highestPredictionText;
            }
        }
    </script>
</body>
</html>
