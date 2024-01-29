let capture;
let capturewidth = 640;
let captureheight = 480;

let emotions = ["neutral", "happy", "sad", "angry", "fearful", "disgusted", "surprised"];
let detections = [];

let audioContext;
let oscillators = {};

function setup() {
  createCanvas(capturewidth, captureheight);

  capture = createCapture(VIDEO);
  capture.position(0, 0);
  capture.hide();

  const faceOptions = { withLandmarks: true, withExpressions: true, withDescriptors: false };

  faceapi = ml5.faceApi(capture, faceOptions, faceReady);

  // Initialize Web Audio API
  audioContext = new (window.AudioContext || window.webkitAudioContext)();

  // Setup oscillators for each emotion
  for (let i = 0; i < emotions.length; i++) {
    let emotion = emotions[i];
    oscillators[emotion] = createOscillator(getFrequency(emotion));
  }
}

function createOscillator(frequency) {
  let oscillator = audioContext.createOscillator();
  oscillator.type = 'sine';
  oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime);
  oscillator.connect(audioContext.destination);
  return oscillator;
}

function getFrequency(emotion) {
  
  const frequencyMap = {
    neutral: 210, 
    happy: 432,
    sad: 220,
    angry: 440,
    fearful: 660,
    disgusted: 528,
    surprised: 520,
  };

  return frequencyMap[emotion] || 210; 
}

function faceReady() {
  faceapi.detect(gotFaces);
}

function gotFaces(error, result) {
  if (error) {
    console.log(error);
    return;
  }
  detections = result;
  faceapi.detect(gotFaces);
}

function draw() {
  background(0);
  capture.loadPixels();

  fill('green');
  if (detections.length > 0) {
    for (let i = 0; i < detections.length; i++) {
      const points = detections[i].landmarks.positions;
      for (let j = 0; j < points.length; j++) {
        circle(points[j]._x, points[j]._y, 5);
      }

      const currentEmotion = getDominantEmotion(detections[i].expressions);
      updateOscillator(currentEmotion);

      for (let k = 0; k < emotions.length; k++) {
        const thisEmotion = emotions[k];
        const thisEmotionLevel = detections[i].expressions[thisEmotion];
        text(thisEmotion + " value: " + thisEmotionLevel, 40, 30 + 30 * k);
        rect(40, 30 + 30 * k, thisEmotionLevel * 100, 10);
      }
    }
  }
}

function getDominantEmotion(expressions) {
  let dominantEmotion = 'neutral';
  let dominantValue = expressions.neutral;

  for (let emotion of emotions) {
    if (expressions[emotion] > dominantValue) {
      dominantValue = expressions[emotion];
      dominantEmotion = emotion;
    }
  }

  return dominantEmotion;
}

function updateOscillator(currentEmotion) {
  // Stop all oscillators
  for (let emotion in oscillators) {
   
    oscillators[emotion] = createOscillator(getFrequency(emotion));
  }

  // Start the oscillator corresponding to the current emotion
  oscillators[currentEmotion].start();
}
