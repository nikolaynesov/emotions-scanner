<script>

  import Logo from '~/components/Logo.vue'

  export default {

    components: {
      Logo
    },

    head () {
      return {
        script: [
          { src: 'https://download.affectiva.com/js/3.2.1/affdex.js', defer: true }
        ]
      }
    },

    data() {

      return  {

        width: 640,
        height: 480,
        detector: null,

        results: {
          timestamp: 0,
          faces_count: 0,
          appearance: 0,
          expressions: 0,

          joy: 0,
          sadness: 0,
          disgust: 0,
          contempt: 0,
          anger: 0,
          fear: 0,
          surprise: 0,
          valence: 0,
          engagement: 0,

          emoji: 0

        },

        defaults: {},

        working: false,

        testing: true

      }

    },

    mounted() {
      window.addEventListener("load", (event) => {
        this.log('init');
        this.defaults = _.clone(this.results);
        this.init();
        this.start();
      });
    },

    computed: {

      faceMode() {
        return typeof this.affdex == 'undefined' ? null : this.affdex.FaceDetectorMode.LARGE_FACES ;
      },

      affdex() {
        return window.affdex;
      }

    },

    methods: {

      init() {

        this.log('instantiating...');

        //Construct a CameraDetector and specify the image width / height and face detector mode.
        this.detector = new this.affdex.CameraDetector(
                document.getElementById('affdex_elements'),
                this.width,
                this.height,
                this.faceMode
        );

        //Enable detection of all Expressions, Emotions and Emojis classifiers.
        this.detector.detectAllEmotions();
        this.detector.detectAllExpressions();
        this.detector.detectAllEmojis();
        this.detector.detectAllAppearance();

        this.registerListeners();

      },

      registerListeners() {

        //Add a callback to notify when the detector is initialized and ready for runing.
        this.detector.addEventListener("onInitializeSuccess", () => {
          this.log("The detector reports initialized");
          document.getElementById('face_video_canvas').style.display = "block";
          document.getElementById('face_video').style.display = "none";
        });

        //Add a callback to notify when camera access is allowed
        this.detector.addEventListener("onWebcamConnectSuccess", () => {
          this.log("Webcam access allowed");
        });

        //Add a callback to notify when camera access is denied
        this.detector.addEventListener("onWebcamConnectFailure", () => {
          this.log("Webcam access denied");
        });

        //Add a callback to notify when detector is stopped
        this.detector.addEventListener("onStopSuccess", () => {
          this.log( "The detector reports stopped");
        });

        //Add a callback to receive the results from processing an image.
        //The faces object contains the list of the faces detected in an image.
        //Faces object contains probabilities for all the different expressions, emotions and appearance metrics
        this.detector.addEventListener("onImageResultsSuccess", (faces, image, timestamp) => {

          this.results.faces_count  = faces.length;
          this.results.timestamp    =  timestamp.toFixed(2);

          if (faces.length > 0) {
            let face = faces[0];
            this.parseFaceResults(face);
            this.drawFeaturePoints(image, face.featurePoints);
          }

        });

      },

      start() {
        if (this.detector && !this.detector.isRunning) {
          this.detector.start();
          this.working = true;
        }
        this.log("Clicked the start button");
      },

      stop() {
        this.log("Clicked the stop button");
        this.resetResults();
        if (this.detector && this.detector.isRunning) {
          this.detector.removeEventListener();
          this.detector.stop();
          this.working = false;
        }
      },

      reset() {
        this.log("Clicked the reset button");
        this.resetResults();
        if (this.detector && this.detector.isRunning) {
          this.detector.reset();
        }
      },

      resetResults() {
        this.results = _.clone(this.defaults);
      },

      drawFeaturePoints(img, featurePoints) {

        if(document.getElementById('face_video_canvas') == null) {
          return false;
        }

        var contxt = document.getElementById('face_video_canvas').getContext('2d');
        contxt.strokeStyle = "#FFFFFF";

        for (var id in featurePoints) {
          contxt.beginPath();
          contxt.arc(featurePoints[id].x,
                  featurePoints[id].y, 2, 0, 2 * Math.PI);
          contxt.stroke();

        }

      },

      convertEmotion(val) {
        return val.toFixed ? Number(val.toFixed(0)) : val;
      },

      parseFaceResults(face) {

        this.results.appearance  =  face.appearance;
        this.results.joy         =  this.convertEmotion(face.emotions.joy);
        this.results.sadness     =  this.convertEmotion(face.emotions.sadness);
        this.results.disgust     =  this.convertEmotion(face.emotions.disgust);
        this.results.contempt    =  this.convertEmotion(face.emotions.contempt);
        this.results.anger       =  this.convertEmotion(face.emotions.anger);
        this.results.fear        =  this.convertEmotion(face.emotions.fear);
        this.results.surprise    =  this.convertEmotion(face.emotions.surprise);
        this.results.valence     =  this.convertEmotion(face.emotions.valence);
        this.results.engagement  =  this.convertEmotion(face.emotions.engagement);
        this.results.expressions =  face.expressions;
        this.results.emoji       =  face.emojis.dominantEmoji;

      },

      log(line) {

        if(this.testing) {
          console.log(line);
        }

      }

    }

  }
</script>

<template>

  <section class="container" id="main-container">


    <div v-if="!this.working"  id="start-btn-wrapper">
      <button class="btn btn-primary" id="start" @click="start">Почати сканувати емоції</button>
    </div>

    <div class="row" v-show="this.working">

      <div class="col" id="video-col">

        <div id="emoji">
          <template v-if="results.emoji">{{results.emoji}}</template>
        </div>

        <div id="affdex_elements" style="min-height: 300px;"></div>

      </div>

      <div class="col">

        <div style="overflow: hidden; font-size: 28px">

          <ul id="emotions-list">

            <li>
              радість ({{results.joy}}%):
              <div class="progress">
                <div class="progress-bar progress-bar-striped progress-bar-animated bg-success"
                     :style="'width:' + results.joy + '%'"
                     role="progressbar" :aria-valuenow="results.joy"
                     aria-valuemin="0" aria-valuemax="100">
                  {{results.joy}}%
                </div>
              </div>
            </li>
            <li>
              смуток ({{results.sadness}}%):
              <div class="progress">
                <div class="progress-bar progress-bar-striped progress-bar-animated"
                     :style="'width:' + results.sadness + '%'"
                     role="progressbar" :aria-valuenow="results.sadness"
                     aria-valuemin="0" aria-valuemax="100">
                  {{results.sadness}}%
                </div>
              </div>
            </li>
            <li>
              відраза ({{results.disgust}}%):
              <div class="progress">
                <div class="progress-bar progress-bar-striped progress-bar-animated bg-warning"
                     :style="'width:' + results.disgust + '%'"
                     role="progressbar" :aria-valuenow="results.disgust"
                     aria-valuemin="0" aria-valuemax="100">
                  {{results.disgust}}%
                </div>
              </div>
            </li>
            <li>
              презирство ({{results.contempt}}%):
              <div class="progress">
                <div class="progress-bar progress-bar-striped progress-bar-animated bg-warning"
                     :style="'width:' + results.contempt + '%'"
                     role="progressbar" :aria-valuenow="results.contempt"
                     aria-valuemin="0" aria-valuemax="100">
                  {{results.contempt}}%
                </div>
              </div>
            </li>
            <li>
              гнів ({{results.anger}}%):
              <div class="progress">
                <div class="progress-bar progress-bar-striped progress-bar-animated bg-danger"
                     :style="'width:' + results.anger + '%'"
                     role="progressbar" :aria-valuenow="results.anger"
                     aria-valuemin="0" aria-valuemax="100">
                  {{results.anger}}%
                </div>
              </div>
            </li>
            <li>
              страх ({{results.fear}}%):
              <div class="progress">
                <div class="progress-bar progress-bar-striped progress-bar-animated bg-danger"
                     :style="'width:' + results.fear + '%'"
                     role="progressbar" :aria-valuenow="results.fear"
                     aria-valuemin="0" aria-valuemax="100">
                  {{results.fear}}%
                </div>
              </div>
            </li>
            <li>
              здивування ({{results.surprise}}%):
              <div class="progress">
                <div class="progress-bar progress-bar-striped progress-bar-animated"
                     :style="'width:' + results.surprise + '%'"
                     role="progressbar" :aria-valuenow="results.surprise"
                     aria-valuemin="0" aria-valuemax="100">
                  {{results.surprise}}%
                </div>
              </div>
            </li>
            <li>
              залученість ({{results.engagement}}%):
              <div class="progress">
                <div class="progress-bar progress-bar-striped progress-bar-animated"
                     :style="'width:' + results.engagement + '%'"
                     role="progressbar" :aria-valuenow="results.engagement"
                     aria-valuemin="0" aria-valuemax="100">
                  {{results.engagement}}%
                </div>
              </div>
            </li>

          </ul>

        </div>

      </div>

    </div>

    <div v-show="this.working">

      <div style="padding-top: 30px;">

        <div>
          <div>Час: {{results.timestamp}}</div>
          <div>Кількість осіб: {{results.faces_count}}</div>
        </div>

        <button class="btn btn-danger" id="stop" @click="stop">Припинити</button>
        <button class="btn btn-secondary" id="reset" @click="reset">Скинути</button>

      </div>

    </div>


  </section>

</template>

<style>

  #emotions-list {

    list-style: none;

  }

  #emoji {
    position: absolute;
    top: 20px;
    left: 20px;
    font-size: 140px;
    line-height: 140px;
  }

  #video-col {
    position: relative;
  }

  #start-btn-wrapper {

    border: 1px solid #ccc;
    padding: 20px;
    text-align: center;
    margin-top: 20px;

  }

</style>
