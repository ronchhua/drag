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
          center: {
            x: 100,
            y: 100,
          },
          radius: 50,
          userClicked: false,
          attachedToRail: false,
        },


      ],
      rail: {
        x: 500,
        y: 100,
        width: 700,
        height: 500,
        components: [
          { part: 'top', x1: 500, y1: 100, x2: 500+700, y2: 100 },
          { part: 'left', x1: 500, y1: 100, x2: 500, y2: 100+500 },
          { part: 'right', x1: 500+700, y1: 100, x2: 500+700, y2: 100+500 },
          { part: 'bottom', x1: 500, y1: 100+500, x2: 500+700, y2: 100+500 },
        ],


      }

    }
  },
  mounted() {

    var htmlElement = document.getElementById("canvas-container");
    var drawingContext = htmlElement.getContext("2d");
    this.canvas = drawingContext;

    this.canvas.width = window.innerWidth;   // Why doesn't this work
    this.canvas.height = window.innerHeight;  // Why doesn't this work

    this.initialDrawCanvasElements();

  },
  methods: {

    initialDrawCanvasElements() {

      this.canvas.beginPath();

      var blueCircleImage = document.getElementById("blueCircle");
      blueCircleImage.addEventListener('load', () => {
        this.canvas.drawImage(blueCircleImage, this.images[0].x, this.images[0].y, this.images[0].width, this.images[0].height);
      });

      this.drawRail();

      this.canvas.closePath();
      this.canvas.stroke();

    },

    drawRail() {
      var rail = this.rail;
      this.canvas.strokeRect(rail.x, rail.y, rail.width, rail.height);
      this.canvas.fill();
    },

    // If a user clicks down, we check if it is clicking on one of our canvas elements.
    // If it is within the bounds

    checkIfClickedElement(event) {

      this.mouse['x'] = event.offsetX;
      this.mouse['y'] = event.offsetY;

      var circleCenterCoords = this.images[0].center;

      var xDistance = (event.offsetX - circleCenterCoords.x);
      var yDistance = (event.offsetY - circleCenterCoords.y);

      var distance = Math.sqrt(Math.pow(xDistance, 2) + Math.pow(yDistance, 2));

      if(distance <= this.images[0].radius) {
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

        var circleRadius = this.images[0].radius;
        var blueCircleImage = document.getElementById("blueCircle");

        this.canvas.beginPath();

        // Clear the initial placement of the image
        this.canvas.clearRect(this.images[0].x-3, this.images[0].y-3, this.images[0].width+3, this.images[0].height+3);
        
        // While we move the image, we update it's coordinates to match the image moving with our mouse.
        // Note the circle image is basically a square.

        var circleImage = this.images[0];
        circleImage.x = mouseX-circleRadius;
        circleImage.y = mouseY-circleRadius;
        circleImage.center['x'] = mouseX;
        circleImage.center['y'] = mouseY;

        this.canvas.drawImage(blueCircleImage, mouseX-circleRadius, mouseY-circleRadius, circleImage.width, circleImage.height);

        this.drawRail();

        this.canvas.closePath();
      }

    },

    // If a user stops pressing down on the mouse, then we set whatever element it was holding to false
    // or if the user wasn't holding an element, we simply just set every canvas element to false.

    dropElement(event) {

      this.mouse.x = event.offsetX;
      this.mouse.y = event.offsetY;
      var mouseX = this.mouse.x;
      var mouseY = this.mouse.y;

      if(this.images[0].userClicked) {
        // Basically if the mouse x position + the radius is in the bounds of [x1-circle radius, x2+circle radius]

        var rail = this.rail;
        var circleRadius = this.images[0].radius;
        var foundRail = false;

        rail.components.forEach((comp) => {

          //console.log(mouseX, comp.x1-circleRadius, mouseX, comp.x2+circleRadius, mouseY, comp.y1-circleRadius, mouseY, comp.y2+circleRadius);

          if(mouseX >= comp.x1-circleRadius && mouseX <= comp.x2+circleRadius && mouseY >= comp.y1-circleRadius && mouseY <= comp.y2+circleRadius) {
            
            if(!foundRail) {

              this.images[0].attachedToRail = true;

              var blueCircleImage = document.getElementById("blueCircle");
              this.canvas.beginPath();
              this.canvas.clearRect(this.images[0].x-3, this.images[0].y-3, this.images[0].width+3, this.images[0].height+3);
              
              var circleImage = this.images[0];

              if(comp.part == 'top' || comp.part == 'bottom') {
                circleImage.x = mouseX-circleRadius;
                circleImage.y = comp.y1-circleRadius;   // Could also be y2 because of same height
                circleImage.center['x'] = mouseX;
                circleImage.center['y'] = comp.y1;
              }
              else if(comp.part == 'left' || comp.part == 'right') {
                circleImage.x = comp.x1-circleRadius;   // Could also be x2 because of same x pos
                circleImage.y = mouseY-circleRadius;
                circleImage.center['x'] = comp.x1;
                circleImage.center['y'] = mouseY;
              }

              this.canvas.drawImage(blueCircleImage, circleImage.x, circleImage.y, circleImage.width, circleImage.height);

              //We need to redraw the track because the clearRect() above would clear part of the track that the circle is dragged through
              this.drawRail();

              this.canvas.closePath();

            }

            foundRail = true; // Very likely that element is touching two components of the rail. Whichever one we iterate through first, we'll place it there.
          }

        });

      }

      this.images[0].userClicked = false;
      console.log('Mouse let go');

      


    },




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
