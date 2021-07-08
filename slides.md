---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
---

# WebGl 入门指南

-johnnyzwu

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# 可以学到什么？

从入门到放弃

- **Hello world** - 怎么用 WebGL 画一个点
- **数据输入** - 如何通过鼠标在指定位置画一个点
- **输入多条数据** - 如何用鼠标画一条线
- **构成三维模型的基本单位** - 三角形绘制与基本变幻
- **纹理** - 渲染图片
- **三维世界** - 绘制一个三维立方体
- **更加真实** - 给三维物体添加光照效果
- **未来** - WebGPU 介绍

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent; 
  -moz-text-fill-color: transparent;
}
</style>

---

# Hello World

如何画一个点

<img src="/img/point.png" class="w-100 mx-auto rounded shadow"/>

---

# index.ts  

```ts
import { createProgramFromSource } from "../../utils";
import vertSource from "./index.vert";
import fragSource from "./index.frag";

function start() {
  const canvas = document.querySelector("canvas");
  canvas.width = 500;
  canvas.height = 500;

  // 获取webgl上下文
  const gl = canvas.getContext("webgl2");

  //设置canvas背景色
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  // 清空canvas
  gl.clear(gl.COLOR_BUFFER_BIT);

  // 用一对着色器创建webgl program
  const program = createProgramFromSource(gl, vertSource, fragSource);
  gl.useProgram(program);

  // 绘制点
  gl.drawArrays(gl.POINTS, 0, 1);
}

start();
```

---
layout: two-cols
---

# createShader

```ts
export function createShader(
  gl: WebGL2RenderingContext,
  type: number,
  source: string
) {
  const shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);

  const isOk = gl.getShaderParameter(shader, gl.COMPILE_STATUS);

  if (isOk) return shader;

  console.log(
    `create ${type === gl.VERTEX_SHADER ? "vert" : "frag"} Shader failed:`,
    gl.getShaderInfoLog(shader)
  );
  gl.deleteShader(shader);
}

```

::right::
# createProgram

```ts
export function createProgram(
  gl: WebGL2RenderingContext,
  vertexShader: WebGLShader,
  fragmentShader: WebGLShader
) {
  const program = gl.createProgram();
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);

  gl.linkProgram(program);

  const isOk = gl.getProgramParameter(program, gl.LINK_STATUS);

  if (isOk) return program;

  console.log("createProgram failed:", gl.getProgramInfoLog(program));
  gl.deleteProgram(program);
  gl.deleteShader(vertexShader);
  gl.deleteShader(fragmentShader);
}
```

---

# createProgramFromSource

```ts
export function createProgramFromSource(
  gl: WebGL2RenderingContext,
  vertSource: string,
  fragSource: string
) {
  const vertShader = createShader(gl, gl.VERTEX_SHADER, vertSource);
  const fragShader = createShader(gl, gl.FRAGMENT_SHADER, fragSource);

  return createProgram(gl, vertShader, fragShader);
}
```

---
layout: two-cols
---

# index.vert


```c
#version 300 es

void main() {

  gl_Position = vec4(0, 0, 0, 1);
  gl_PointSize = 10.0;
}



```

::right::

# index.frag

```c
#version 300 es

precision highp float;

out vec4 outColor;

void main() {

  outColor = vec4(1, 1, 0, 1);
}
```

[什么是片段着色器](https://thebookofshaders.com/01/?lan=ch)

---

# WebGL 顶点坐标


<img src="http://www.yanhuangxueyuan.com/WebGL_course/icon/coordinate.png" class="w-100 mx-auto rounded shadow"/>

---

# 数据输入

如何通过鼠标绘制线段

<img src="/img/line.png" class="w-100 mx-auto rounded shadow"/>

---

# 顶点着色器

```c
#version 300 es

in vec4 a_Position;

void main() {
  gl_Position = a_Position;
  gl_PointSize = 10.0;
}
```

---
layout: two-cols
---

# start

```ts
function start() {
  const canvas = document.querySelector("canvas");
  const gl = canvas.getContext("webgl2");

  const program = createProgramFromSource(gl, vertSource, fragSource);
  gl.useProgram(program);

  const buffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer);

  const a_Position = gl.getAttribLocation(program, "a_Position");

  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);
  gl.enableVertexAttribArray(a_Position);

  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);

  const points = [];

  canvas.addEventListener("click", (e) => {
    handleClick(e, points, gl);
  });
}
```

::right::

# handleClick

```ts
function handleClick(
  e: MouseEvent,
  points: number[],
  gl: WebGL2RenderingContext
) {
  const { clientX, clientY } = e;
  const canvas = e.target as HTMLCanvasElement;
  const { left, top } = canvas.getBoundingClientRect();

  const x = ((clientX - left) / canvas.width) * 2 - 1;
  const y = (((clientY - top) / canvas.height) * 2 - 1) * -1;
  points.push(x);
  points.push(y);
  const pointsLen = Math.floor(points.length / 2);

  const data = new Float32Array(points);
  gl.bufferData(gl.ARRAY_BUFFER, data, gl.STATIC_DRAW);
  gl.clear(gl.COLOR_BUFFER_BIT);

  if (pointsLen < 2) {
    gl.drawArrays(gl.POINTS, 0, pointsLen);
  } else {
    gl.drawArrays(gl.LINE_STRIP, 0, pointsLen);
  }
}
```

<!-- 这是一条备注 -->
---

# 缓冲区对象
1. 创建缓冲区对象 : `gl.createBuffer`
2. 绑定缓冲区对象：`gl.bindBuffer`
3. 将数据写入缓冲区对象：`gl.bufferData`
4. 将缓冲区对象分配给一个attribute变量：`gl.vertexAttribPointer`
5. 开启attribute变量：`gl.enableVertexAttribArray`

---

# 三角形
一切的基石

<img src="/img/triangle.png" class="w-100 mx-auto rounded shadow"/>


---
layout: two-cols
---
# class Triangle

```ts {all|2-6|15|23}
class Triangle {
  readonly defaultPositionData = [
    { x: -0.5, y: -0.5 },
    { x: 0.5, y: -0.5 },
    { x: 0, y: 0.5 },
  ];

  xTranslateCount = 0;
  yTranslateCount = 0;
  rotate = 0;
  gl: WebGL2RenderingContext;

  constructor(shaderSource: { vertSource: string; fragSource: string }) {}

  init() {}

  handleTranslateX() {}

  handleTranslateY() {}

  handleRotate() {}

  draw() {}
}
```

::right::

# method draw

```ts {all|18}
draw() {
    const finalData = this.defaultPositionData
      .map(({ x, y }) => ({
        x: x * Math.cos(this.rotate) - y * Math.sin(this.rotate),
        y: x * Math.sin(this.rotate) + y * Math.cos(this.rotate),
      }))
      .map(({ x, y }) => ({
        x: x + this.xTranslateCount,
        y: y + this.yTranslateCount,
      }))
      .reduce((all, { x, y }) => [...all, x, y], []);

    const gl = this.gl;

    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(finalData), gl.STATIC_DRAW);

    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.TRIANGLES, 0, 3);





  }
```

---

# 变换三角形
- 平移
- 旋转
- 缩放

---
layout: two-cols
---
# 平移

<img src="/img/translate.png" class="w-100 mx-auto rounded shadow"/>

::right::

# 公式

- $x^{\prime} = x + Tx$
- $y^{\prime} = y + Ty$
- $z^{\prime} = z + Tz$


```ts
handleTranslateX() {
    const xInput = document.querySelector<HTMLInputElement>("#xTranslate");
    xInput.addEventListener("input", (e: Event) => {
      const el = e.target as HTMLInputElement;
      const value = +el.value;

      this.xTranslateCount = value;

      this.draw();
    });
  }
```

```ts
map(({ x, y }) => ({
        x: x + this.xTranslateCount,
        y: y + this.yTranslateCount,
      }))
```

---
layout: two-cols
---
# 旋转

<img src="/img/rotate.png" class="w-100 mx-auto rounded shadow"/>

## 未旋转前
- $x = r \; cos \, \alpha$
- $y = r \; sin \, \alpha$

::right::

# 公式


## 旋转后
- $x^{\prime} = r \; cos \, (\alpha + \beta)$
- $y^{\prime} = r \; sin \, (\alpha + \beta)$

## 三角函数两角和公式
- $sin (a \pm b) = sin \, a \; cos \, b \pm cos \, a \; sin \, b$
- $cos (a \pm b) = cos \, a \; cos \, b \pm sin \, a \; sin \, b$

## 最终
- $x^{\prime} = x \; cos \, \beta - y \; sin \, \beta$
- $y^{\prime} = x \; sin \, \beta + y \; cos \, \beta$

---

# handleRotate

```ts
handleRotate() {
    const rInput = document.querySelector<HTMLInputElement>("#rotate");
    rInput.addEventListener("input", (e: Event) => {
      const el = e.target as HTMLInputElement;
      const value = +el.value;

      this.rotate = value * (Math.PI / 180);

      this.draw();
    });
  }
```
# draw
 
```ts
map(({ x, y }) => ({
        x: x * Math.cos(this.rotate) - y * Math.sin(this.rotate),
        y: x * Math.sin(this.rotate) + y * Math.cos(this.rotate),
      }))
```

---

# 缩放

<img src="/img/scale.png" class="w-100 mx-auto rounded shadow"/>

## 公式

- $x^{\prime} = S_x \times x$
- $y^{\prime} = S_y \times y$
- $z^{\prime} = S_z \times z$

---

# 矩阵
矩阵乘法


<img src="https://www.shuxuele.com/algebra/images/matrix-multiply-a.svg" class="w-100 mx-auto rounded shadow"/>

<p class="mx-auto w-100">(1, 2, 3) • (7, 9, 11) = 1×7 + 2×9 + 3×11 = 58</p>

<img src="https://www.shuxuele.com/algebra/images/matrix-multiply-c.svg" class="w-100 mx-auto rounded shadow"/>

[矩阵乘法-数学乐](https://www.shuxuele.com/algebra/matrix-multiplying.html)

---
layout: two-cols
---
# 旋转矩阵

$$
 \begin{bmatrix}
   x^{\prime} \\
   y^{\prime} \\
   z^{\prime}
  \end{bmatrix}
   =
   \begin{bmatrix}
   a & b & c \\
   d & e & f \\
   g & h & i
  \end{bmatrix}
  \times
   \begin{bmatrix}
   x \\
   y \\
   z
  \end{bmatrix}
$$

- $x^{\prime} = ax + by + cz$
- $y^{\prime} = dx + ey + fz$
- $z^{\prime} = gx + hy + iz$

## 旋转计算公式

- $x^{\prime} = x \; cos \, \beta - y \; sin \, \beta$
- $y^{\prime} = x \; sin \, \beta + y \; cos \, \beta$
- $z^{\prime} = z$

::right::



## 推倒得出
$$
a = cos \beta
\qquad
b = - sin \beta
\qquad
c = 0
$$

$$
d = sin \beta
\qquad
e = cos \beta
\qquad
f = 0
$$

$$
g = 0
\qquad
h = 0
\qquad
i = 1
$$

## 最终矩阵

$$
  \begin{bmatrix}
   x^{\prime} \\
   y^{\prime} \\
   z^{\prime}
  \end{bmatrix}
   =
   \begin{bmatrix}
   cons \beta & - sin \beta & 0 \\
   sin \beta & cos \beta & 0 \\
   0 & 0 & 1
  \end{bmatrix}
  \times
   \begin{bmatrix}
   x \\
   y \\
   z
  \end{bmatrix}
$$

---

# 纹理映射
贴图，将一张图片贴到一个几何图形的表面

<img src="/img/green.jpg" class="w-100 mx-auto  shadow"/>

---

# 绘制矩形
三角带（TRIANGLE_STRIP）

<img src="/img/triangle-strip.png" class="w-100 mx-auto  shadow"/>

```ts
const data = new Float32Array([-1.0, 1.0, -1.0, -1.0, 1.0, 1.0, 1.0, -1.0]);
gl.bufferData(gl.ARRAY_BUFFER, data, gl.STATIC_DRAW);

// .......

gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
```

---

# 纹理坐标与webgl坐标


<img src="/img/texture-zb.jpeg" class="w-150 mx-auto  shadow"/>

---

# 图像坐标与纹理坐标

<img src="/img/image-texture-zb.jpg" class="w-150 mx-auto  shadow"/>


---
layout: two-cols
---

# 顶点着色器

```c
#version 300 es

in vec4 a_position;
in vec2 a_texCoord;
out vec2 v_texCoord;

void main() {

  gl_Position = a_position;

  v_texCoord = a_texCoord;
}


```

::right::

# 片段着色器

```c
#version 300 es

precision highp float;

uniform sampler2D u_image;

in vec2 v_texCoord;

out vec4 outColor;

void main() {
  outColor = texture(u_image, v_texCoord);
}

```

---

# 设置纹理坐标与顶点坐标

```ts
class ImageAdjust {
  initPosition(program: WebGLProgram) {
    const gl = this.gl;

    const data = new Float32Array([-1.0, 1.0, -1.0, -1.0, 1.0, 1.0, 1.0, -1.0]);

    // ...

    gl.bufferData(gl.ARRAY_BUFFER, data, gl.STATIC_DRAW);
  }

  initTexCoord(program: WebGLProgram) {
    const gl = this.gl;

    const data = new Float32Array([0.0, 1.0, 0.0, 0.0, 1.0, 1.0, 1.0, 0.0]);

    //...
    gl.bufferData(gl.ARRAY_BUFFER, data, gl.STATIC_DRAW);
  }

}

```

---

# 配置加载纹理

```ts

class ImageAdjust {
  initTexture(program: WebGLProgram, image: HTMLImageElement) {
    const gl = this.gl;

    const texture = gl.createTexture();
    gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, 1);
    gl.activeTexture(gl.TEXTURE0);
    gl.bindTexture(gl.TEXTURE_2D, texture);
    
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
    
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGB, gl.RGB, gl.UNSIGNED_BYTE, image);

    const imageLocation = gl.getUniformLocation(program, "u_image");

    gl.uniform1i(imageLocation, 0);
  }

}

```

---

# 图片处理
没有写

1. RGB, HSL, HSV
2. 帧缓冲

---

# 3D

