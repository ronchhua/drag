<template>
  <div id="app">
    <canvas id = "canvas-container" v-on:mousedown="checkIfClickedElement" v-on:mousemove="moveElement" v-on:mouseup="dropElement" height="754" width="1536"></canvas> <!--  v-on:mousemove="checkMouseHover"-->
    <img src="./assets/clipart2025153.png" id="blueCircle" style="display: none;">
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
  },
  data() {
    
    return {

      canvas: null,
      mouse: {
        x: null,
        y: null,
        radius: null,
      },
      images: [
        {
          imageName: 'Blue Circle',
          x: 50,
          y: 50,
          width: 100,
          height: 100,
          circle: {
            x: 100,
            y: 100,
            radius: 50,
          },
          userClicked: false,
        },


      ],

    }
  },
  mounted() {

    var htmlElement = document.getElementById("canvas-container");
    var drawingContext = htmlElement.getContext("2d");
    this.canvas = drawingContext;

    this.canvas.width = window.innerWidth;   // Why doesn't this work
    this.canvas.height = window.innerHeight;  // Why doesn't this work

    this.drawCanvasElements();

  },
  methods: {

    drawCanvasElements() {

      this.canvas.beginPath();

      var blueCircleImage = document.getElementById("blueCircle");
      blueCircleImage.addEventListener('load', () => {
        this.canvas.drawImage(blueCircleImage, this.images[0].x, this.images[0].y, this.images[0].width, this.images[0].height);
      });

      this.canvas.closePath();

    },

    // If a user clicks down, we check if it is clicking on one of our canvas elements.

    checkIfClickedElement(event) {

      this.mouse['x'] = event.offsetX;
      this.mouse['y'] = event.offsetY;

      var circleElement = this.images[0].circle;

      var xDistance = (event.offsetX - circleElement.x);
      var yDistance = (event.offsetY - circleElement.y);

      var distance = Math.sqrt(Math.pow(xDistance, 2) + Math.pow(yDistance, 2));

      if(distance <= circleElement['radius']) {
        this.images[0].userClicked = true;
      }

    },

    // If a user is moving his mouse in the canvas, we first check if a user had clicked on an image/canvas element
    // and if yes, then we move the image along with the mouse.

    moveElement(event) {

      this.mouse['x'] = event.offsetX;
      this.mouse['y'] = event.offsetY;

      var mouseX = this.mouse.x;
      var mouseY = this.mouse.y;

      if(this.images[0].userClicked == true) {

        var circleRadius = this.images[0].circle.radius;
        var blueCircleImage = document.getElementById("blueCircle");

        this.canvas.drawImage(blueCircleImage, mouseX-circleRadius, mouseY-circleRadius, this.images[0].width, this.images[0].height);
        // try to align it so that the radius center of image is on the mouse
      }

    },

    // If a user stops pressing down on the mouse, then we set whatever element it was holding to false
    // or if the user wasn't holding an element, we simply just set every canvas element to false.

    dropElement(event) {

      this.images[0].userClicked = false;
      console.log(event);
      console.log('Mouse let go');
    }


  }


}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

  margin: 0px;
  padding: 0px;

}

#canvas-container {
  position:absolute;  /* Specify position, top and left to not have a scrollbar */
  top: 0; left: 0;

  /*
   width: 100%;     Messes up image sizes. The canvas simply shows the image stretched to the full viewport
   height: 100vh;   This just messes with the dimensions of the DOM elements
  */

}
</style>
