// The purpose of this program is to reduce the amt of data needed in order to produce an image
// The compression is done through combining similiar rbg regions into one.
// For instance, in a rainbow image, we can collectively chunk the red, blue, green, etc colors into being one region. This way, we don't have to store the infornation of every pixel and can for instance, store the infromation of every 2x2 regions.
// The restriction of when we would chunk a region depends on an "error limit"
// Using quad trees, every region will have an error value.
// The error value corresponds to diversitry of coliors within a regions. (Higher error with more color variaty and less with more blended / near the same colors)
// Within a region, if the error is greater than the error limit, we don't chunk and instead create a new quadtree
// Else, we chunk the region into the regions "average color"


let qTree;
let img;
let colors = [];  // Stores rgb values of every pixel of the image
let objCnt = 0;  // Records the number of "chunks" needed to re-create the image
function preload(){
  img = loadImage("gnome.jpg")  // Change the image here
}

function setup() {
  let imgW = 400;
  let imgH = 400;
  img.resize(imgW,imgH);
  createCanvas(imgW, imgH);  // We want the image to fully cover the screen
  // Slower implementation of getting colors (Could later reference the pixels() array)
  // for (let y = 0; y < img.height; y++){
  //   let row = [];
  //   for (let x = 0; x < img.width; x++){
  //     row.push(img.get(x, y));
  //   }
  //   colors.push(row);
  // }

  // Faster implmentation using pixels array
  colors = [];
  image(img, 0, 0)
  loadPixels();  // Record all rgb values on screen (all rbg values of the image in this case) onto a  1-D array
  let imgLength = imgW * imgH * 4; // The length of the 1-D array (Note that the array store red, green, blue, alpha)
  let row = [];
  for (let i = 0; i < imgLength; i+=4){    
    if ((i % (imgW * 4)) == 0 && (i > 0)){
      colors.push(row);
      row = []; 
    }
    row.push([pixels[i], pixels[i + 1], pixels[i + 2]]);  // Note that we don't care for the alpha value
  }
  colors.push(row);  
  
  createCanvas(800, 800);  // Now we don't care about the image fully covering the screen
  image(img, img.width, 0);
  
  qTree = new QuadTree(0,0,img.width,img.height, 300, 0) // Edit error limit here
  qTree.update(colors);  // Insert colors into quad-tree and begin chunking
    
  qTree.draw();  // Draw compressed image 

  print(objCnt, imgW * imgH)  // Compares the # of chunks (Regions where pixels have the same color) vs the # of pixels between the compressed and orignal image

}

function calculateError(x, y, w, h, colors){  // Calculate the error value of a region
  // Function firstly finds the average rbg color of the region
  // It then calculates the error value through checking how much every pixel on the screen differs from the average color
  
  let vals = calculateAvg(x, y, w, h, colors);
  let r = vals[0]; let g = vals[1]; let b = vals[2];
  let errorSum = 0;
  
  for (let ny = y; ny < (y + h); ny++){
    for (let nx = x; nx < (x + w); nx++){
      
      // Note that the reason we do squared is to accentuate color variation
      // Smaller color varaiations return smaller error values while larger vairations would be have near exponentially larger error return values
      // The point is that we don't want a linear model for error values. Instead, we want a model that REALLY points out on large color varations. 
      errorSum += pow(abs(colors[ny][nx][0] - r), 2);
      errorSum += pow(abs(colors[ny][nx][1] - g), 2); 
      errorSum += pow(abs(colors[ny][nx][2] - b), 2);  
      
      // Incase we want to have a linear model for calculating error value
      // errorSum += pow(abs(colors[ny][nx][0] - r), 1);
      // errorSum += pow(abs(colors[ny][nx][1] - g), 1); 
      // errorSum += pow(abs(colors[ny][nx][2] - b), 1); 
    }
  }
  
  let error = errorSum / (3 * w * h);  // Average out the "total error" in order to get an "average error"
  return error;
}

function calculateAvg(x, y, w, h, colors){  // Find the average rbg color of a region
  let r = 0; let b = 0; let g = 0;
  for (let ny = y; ny < (y + h); ny++){
    for (let nx = x; nx < (x + w); nx++){
      r += colors[ny][nx][0];
      g += colors[ny][nx][1];
      b += colors[ny][nx][2];
    }
  }
  
  r /= (w * h);
  g /= (w * h);
  b /= (w * h);
  
  return [r, g, b];
}

function draw() {
}

class QuadTree{
  constructor(x, y, w, h, errorLim, depth){
    this.children = [false, false, false, false];  // Children quadtrees
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
    this.errorLim = errorLim; 
    this.depth = depth;
    this.maxDepth = 500;  // An additional restriction which prevents us to chunk near pixel size regions (If chunks are too small, we wouldn't be doing much compression)
  }
  
  split(){
    // Refer to Diagram.png for a semi-explination of why we do this
    // TLDR, some pixels are not accounted for when width or height are decimals
    // The solution is to split the quadtree nearly halfway to get an integer width and height
    let highx; let highy; let lowx; let lowy;
    let quadW; let quadH;
    
    if (this.w / 2 != int(this.w / 2)){
      highx = int(this.w/2) + 1;
      lowx = int(this.w/2);
    }
    else{
      highx = this.w/2;
      lowx = this.w/2;
    }
    
    if (this.h / 2 != int(this.h / 2)){
      highy = int(this.h/2) + 1;
      lowy = int(this.h/2);
    }
    else{
      highy = this.h/2;
      lowy = this.h/2;
    }
    
    
    this.children[0] = new QuadTree(this.x + highx, this.y, lowx, highy, this.errorLim, this.depth + 1);
    this.children[1] = new QuadTree(this.x, this.y, highx, highy, this.errorLim, this.depth + 1);
    this.children[2] = new QuadTree(this.x, this.y + highy, highx, lowy, this.errorLim, this.depth + 1);
    this.children[3] = new QuadTree(this.x + highx, this.y + highy, lowx, lowy, this.errorLim, this.depth + 1);
    
  }
  
  update(colors){
    if (calculateError(this.x, this.y, this.w, this.h, colors) > this.errorLim && this.depth < this.maxDepth){
      this.split(); 
      for (let child of this.children){
        child.update(colors); 
      }
    }
    else{
      this.color = calculateAvg(this.x, this.y, this.w, this.h, colors);
    }
  }
  
  draw(){
    objCnt += 1;
    if (this.children[0] != false){
      for (let child of this.children){
        child.draw(); 
      }
    }
    else{
      noStroke()
      fill(this.color[0], this.color[1], this.color[2]);
      // Here, we can basically reconstruct the image in whatever shape we want (triagles, rectangles, abstract designs, etc)
      rect(this.x, this.y,this.w, this.h); 
      // circle(this.x + this.w/2, this.y + this.h/2, this.w, this.h)
    }
  } 
}
