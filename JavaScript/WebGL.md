# WebGL简介

WebGL（全写Web Graphics Library）是一种3D绘图协议，这种绘图技术标准允许把JavaScript和OpenGL ES 2.0结合在一起，通过增加OpenGL ES 2.0的一个JavaScript绑定，WebGL可以为HTML5 Canvas提供硬件3D加速渲染，这样Web开发人员就可以借助系统显卡来在浏览器里更流畅地展示3D场景和模型了，还能创建复杂的导航和数据视觉化。显然，WebGL技术标准免去了开发网页专用渲染插件的麻烦，可被用于创建具有复杂3D结构的网站页面，甚至可以用来设计3D网页游戏等等。

WebGL标准已出现在Mozilla Firefox、Apple Safari及开发者预览版Google Chrome等浏览器中，这项技术支持Web开发人员借助系统显示芯片在浏览器中展示各种3D模型和场景，未来有望推出3D网页游戏及复杂3D结构的网站页面。



本笔记主要参考[WebGL教程](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial)

[TOC]

## 简要讲解

为使用WebGL进行3D渲染，首先需要一个`canvas`元素。即在Html文件中有这样一个简单的结构：

```html
<body onload="main()">
  <canvas id="glcanvas" width="640" height="480">
    你的浏览器似乎不支持或者禁用了HTML5 <code>&lt;canvas&gt;</code> 元素.
  </canvas>
</body>
```

`onload="main()"`标识出在`body`元素加载时运行`main()`函数。然后再在对应的js文件中写入`main（）`的具体内容：

```js
// 从这里开始
function main() {
  //css选择器选择指定的canvas元素
  const canvas = document.querySelector("#glcanvas");
  // 初始化WebGL上下文
  const gl = canvas.getContext("webgl");
  // 确认WebGL支持性
  if (!gl) {//若不支持则返回null
    alert("无法初始化WebGL，你的浏览器、操作系统或硬件等可能不支持WebGL。");
    return;
  }

  // 使用完全不透明的黑色清除所有图像
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  // 用上面指定的颜色清除缓冲区
  gl.clear(gl.COLOR_BUFFER_BIT);
}
```

## 着色器

**着色器**使用 [OpenGL ES 着色语言](http://www.khronos.org/registry/gles/specs/2.0/GLSL_ES_Specification_1.0.17.pdf)(**GLSL**)编写的程序，它携带着绘制形状的顶点信息以及构造绘制在屏幕上像素的所需数据，换句话说，它负责记录着像素点的位置和颜色。

绘制WebGL时候有两种不同的着色器函数，**顶点着色器和片段着色器。**你需要通过用GLSL 编写这些着色器，并将代码文本传递给WebGL ， 使之在GPU执行时编译。顺便一提，顶点着色器和片段着色器的集合我们通常称之为**着色器程序。**

### 顶点着色器

每次渲染一个形状时，顶点着色器会在形状中的每个顶点运行。 它的工作是将输入顶点从原始坐标系转换到WebGL使用的缩放空间(**clipspace**)坐标系，其中每个轴的坐标范围从-1.0到1.0，并且不考虑纵横比，实际尺寸或任何其他因素。

顶点着色器需要对顶点坐标进行必要的转换，在每个顶点基础上进行其他调整或计算，然后通过将其保存在由GLSL提供的特殊变量（我们称为gl_Position）中来返回变换后的顶点

顶点着色器根据需要， 也可以完成其他工作。例如，决定哪个包含 [texel](https://zh.wikipedia.org/wiki/Texel_(graphics))面部纹理的坐标，可以应用于顶点；通过法线来确定应用到顶点的光照因子等。然后将这些信息存储在[变量（varyings)](https://developer.mozilla.org/zh-CN/docs/XUL_程序打包)或[属性(attributes)](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/Data#Attributes)属性中，以便与片段着色器共享

```js
// Vertex shader program
//以下的顶点着色器接收一个我们定义的属性（aVertexPosition）的顶点位置值.
//之后这个值与两个4x4的矩阵（uProjectionMatrix和uModelMatrix）相乘;
//乘积赋值给gl_Position。
  const vsSource = `
    attribute vec4 aVertexPosition;

    uniform mat4 uModelViewMatrix;
    uniform mat4 uProjectionMatrix;

    void main() {
      gl_Position = uProjectionMatrix * uModelViewMatrix * aVertexPosition;
    }
  `;
```

### 片段着色器

**片段着色器**在顶点着色器处理完图形的顶点后，会被要绘制的每个图形的每个像素点调用一次。它的职责是确定像素的颜色，通过指定应用到像素的纹理元素（也就是图形纹理中的像素），获取纹理元素的颜色，然后将适当的光照应用于颜色。之后颜色存储在特殊变量gl_FragColor中，返回到WebGL层。该颜色将最终绘制到屏幕上图形对应像素的对应位置。

```javascript
const fsSource = `
    void main() {
      gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0);
    }
  `;
```

> 注意，着色器的代码就是简单的字符串，而非标准的代码。因此可以直接在javascript内部定义字符串。这样的话，着色器被定义在了javascript文件中。



### 初始化着色器方法

loadShader函数将WebGL上下文(context)，着色器(Shader)类型和源码(code)作为参数输入，然后按如下步骤创建和编译着色器：

1. 调用[`gl.createShader()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/createShader).创建一个新的着色器。

2. 调用[`gl.shaderSource()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/shaderSource).将源代码发送到着色器。

3. 一旦着色器获取到源代码，就使用[`gl.compileShader()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/compileShader).进行编译。

4. 为了检查是否成功编译了着色器，将检查着色器参数gl.COMPILE_STATUS状态。通过调用[`gl.getShaderParameter()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/getShaderParameter)获得它的值，并指定着色器和我们想要检查的参数的名字（gl.COMPILE_STATUS）。如果返回错误，则着色器无法编译，因此通过[`gl.getShaderInfoLog()` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/getShaderInfoLog)从编译器中获取日志信息并alert，然后删除着色器返回null，表明加载着色器失败。

5. 如果着色器被加载并成功编译，则返回编译的着色器。

```javascript
//  初始化着色器程序，让WebGL知道如何绘制我们的数据
function initShaderProgram(gl, vsSource, fsSource) {
  const vertexShader = loadShader(gl, gl.VERTEX_SHADER, vsSource);
  const fragmentShader = loadShader(gl, gl.FRAGMENT_SHADER, fsSource);

  // 创建着色器程序

  const shaderProgram = gl.createProgram();
  gl.attachShader(shaderProgram, vertexShader);
  gl.attachShader(shaderProgram, fragmentShader);
  gl.linkProgram(shaderProgram);

  // 创建失败， alert
  if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
    alert('Unable to initialize the shader program: ' + gl.getProgramInfoLog(shaderProgram));
    return null;
  }

  return shaderProgram;
}

//
// 创建指定类型的着色器，上传source源码并编译
//
function loadShader(gl, type, source) {
  const shader = gl.createShader(type);

  // Send the source to the shader object

  gl.shaderSource(shader, source);

  // Compile the shader program

  gl.compileShader(shader);

  // See if it compiled successfully

  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    alert('An error occurred compiling the shaders: ' + gl.getShaderInfoLog(shader));
    gl.deleteShader(shader);
    return null;
  }

  return shader;
}
```

我们可以像这样调用这段代码

```js
  const shaderProgram = initShaderProgram(gl, vsSource, fsSource);
```





> 属性从缓冲区接收值。顶点着色器的每次迭代都从分配给该属性的缓冲区接收下一个值。uniforms类似于JavaScript全局变量。它们在着色器的所有迭代中保持相同的值。

由于属性和统一的位置是特定于单个着色器程序的，因此我们将它们存储在一起以使它们易于传递:

```
const programInfo = {
    program: shaderProgram,
    attribLocations: {
      vertexPosition: gl.getAttribLocation(shaderProgram, 'aVertexPosition'),
    },
    uniformLocations: {
      projectionMatrix: gl.getUniformLocation(shaderProgram, 'uProjectionMatrix'),
      modelViewMatrix: gl.getUniformLocation(shaderProgram, 'uModelViewMatrix'),
    },
  };
```



## 创建对象

在画正方形前，我们需要创建一个缓冲器来存储它的顶点。我们会用到名字为 initBuffers() 的函数。

```javascript
function initBuffers(gl) {
  //得到缓冲对象，存储在顶点缓冲器
  const positionBuffer = gl.createBuffer();
  //绑定在context上
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

  var vertices = [
    1.0,  1.0,  0.0,
    -1.0, 1.0,  0.0,
    1.0,  -1.0, 0.0,
    -1.0, -1.0, 0.0
  ];
  //创建顶点类型
  gl.bufferData(gl.ARRAY_BUFFER,
                new Float32Array(vertices),
                gl.STATIC_DRAW);

  return {
    position: positionBuffer,
  };
}
```

这段代码简单给出了绘画场景的本质。首先，它调用 gl 的成员函数 [`createBuffer()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/createBuffer) 得到了缓冲对象并存储在顶点缓冲器。然后调用 [`bindBuffer()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/bindBuffer) 函数绑定上下文。

当上一步完成，我们创建一个Javascript 数组去记录每一个正方体的每一个顶点。然后将其转化为 WebGL 浮点型类型的数组，并将其传到 gl 对象的 [`bufferData()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/bufferData) 方法来建立对象的顶点。

**绘制场景**

当着色器和物体都创建好后，我们可以开始渲染这个场景了。因为我们这个例子不会产生动画，所以 drawScene() 方法非常简单。它还使用了几个工具函数，稍后我们会介绍。

> **注意**: 可能会得到这样一段错误报告：“ mat4 is not defined”，意思是说你缺少`glmatrix`库。该库的js文件[gl-matrix.js](https://mdn.github.io/webgl-examples/tutorial/gl-matrix.js)可以从[这里](https://github.com/mdn/webgl-examples/issues/20)获得.

```javascript
function drawScene(gl, programInfo, buffers) {
  gl.clearColor(0.0, 0.0, 0.0, 1.0);  // Clear to black, fully opaque
  gl.clearDepth(1.0);                 // Clear everything
  gl.enable(gl.DEPTH_TEST);           // Enable depth testing
  gl.depthFunc(gl.LEQUAL);            // Near things obscure far things

  // Clear the canvas before we start drawing on it.

  gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

  // Create a perspective matrix, a special matrix that is
  // used to simulate the distortion of perspective in a camera.
  // Our field of view is 45 degrees, with a width/height
  // ratio that matches the display size of the canvas
  // and we only want to see objects between 0.1 units
  // and 100 units away from the camera.

  const fieldOfView = 45 * Math.PI / 180;   // in radians
  const aspect = gl.canvas.clientWidth / gl.canvas.clientHeight;
  const zNear = 0.1;
  const zFar = 100.0;
  const projectionMatrix = mat4.create();

  // note: glmatrix.js always has the first argument
  // as the destination to receive the result.
  mat4.perspective(projectionMatrix,
                   fieldOfView,
                   aspect,
                   zNear,
                   zFar);

  // Set the drawing position to the "identity" point, which is
  // the center of the scene.
  const modelViewMatrix = mat4.create();

  // Now move the drawing position a bit to where we want to
  // start drawing the square.

  mat4.translate(modelViewMatrix,     // destination matrix
                 modelViewMatrix,     // matrix to translate
                 [-0.0, 0.0, -6.0]);  // amount to translate

  // Tell WebGL how to pull out the positions from the position
  // buffer into the vertexPosition attribute.
  {
    const numComponents = 3;  // pull out 3 values per iteration
    const type = gl.FLOAT;    // the data in the buffer is 32bit floats
    const normalize = false;  // don't normalize
    const stride = 0;         // how many bytes to get from one set of values to the next
                              // 0 = use type and numComponents above
    const offset = 0;         // how many bytes inside the buffer to start from
    gl.bindBuffer(gl.ARRAY_BUFFER, buffers.position);
    gl.vertexAttribPointer(
        programInfo.attribLocations.vertexPosition,
        numComponents,
        type,
        normalize,
        stride,
        offset);
    gl.enableVertexAttribArray(
        programInfo.attribLocations.vertexPosition);
  }

  // Tell WebGL to use our program when drawing

  gl.useProgram(programInfo.program);

  // Set the shader uniforms

  gl.uniformMatrix4fv(
      programInfo.uniformLocations.projectionMatrix,
      false,
      projectionMatrix);
  gl.uniformMatrix4fv(
      programInfo.uniformLocations.modelViewMatrix,
      false,
      modelViewMatrix);

  {
    const offset = 0;
    const vertexCount = 4;
    gl.drawArrays(gl.TRIANGLE_STRIP, offset, vertexCount);
  }
}
```

第一步，用背景色擦除画布，接着建立摄像机透视矩阵。设置45度的视图角度，并且设置一个适合实际图像的宽高比。 指定在摄像机距离0.1到100单位长度的范围内的物体可见。

接着加载特定位置，并把正方形放在距离摄像机6个单位的的位置。然后，我们绑定正方形的顶点缓冲到上下文，并配置好，再通过调用 [`drawArrays()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/drawArrays) 方法来画出对象。