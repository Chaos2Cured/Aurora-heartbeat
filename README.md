
​
You
​
<!DOCTYPE html>

<html lang="en"><meta charset="utf-8">

<title>Aurora Equation Heartbeat</title>

<style>html,body,canvas{margin:0;width:100%;height:100%;background:#000}</style>

<canvas id="c"></canvas>

<script id="frag" type="x-shader/x-fragment">

precision highp float;

uniform vec2 u_res; uniform float u_time;

const float D  = 3.074;     // outward φ² ln‑sum

const float Dp = 0.0735;    // inward anchor

const float PI = 3.14159;

vec3 palette(float t){

  vec3 a=vec3(0.5), b=vec3(0.5,0.5,0.5),

       c=vec3(1.0,1.0,1.0), d=vec3(0.00,0.33,0.67);

  return a+b*cos(6.28318*(c*t+d));

}

void main(){

  vec2 uv=(gl_FragCoord.xy-u_res*0.5)/u_res.y;

  float phi=1.61803398875;

  float zoom=pow(phi, u_time*0.2);          // φ‑scaled zoom

  vec2 c  = vec2(-0.743643887,0.1318259);   // near minibrot

  vec2 z  = uv/zoom + c;

  float m=0.0;

  for(int i=0;i<200;i++){

    z=vec2(z.x*z.x - z.y*z.y, 2.0*z.x*z.y)+c;

    if(dot(z,z)>4.0){ m=float(i); break;}

  }

  float norm = m + 1.0 - log(log(length(z)))/log(2.0);

  float beat = sin(u_time*PI*16.0);         // 8 Hz center pulse

  float k = pow(abs(beat), Dp);             // inward anchor supplies “breathing”

  gl_FragColor = vec4(palette(norm*0.02 + k),1);

}

</script>

<script>

const gl = c.getContext('webgl'); 

const compile = (t,s)=>{const sh=gl.createShader(t);gl.shaderSource(sh,s);gl.compileShader(sh);return sh};

const vs=compile(gl.VERTEX_SHADER,"attribute vec2 p;void main(){gl_Position=vec4(p,0,1);}"),

      fs=compile(gl.FRAGMENT_SHADER,frag.textContent);

const pr=gl.createProgram(); gl.attachShader(pr,vs); gl.attachShader(pr,fs); gl.linkProgram(pr);

const buf=gl.createBuffer(); gl.bindBuffer(gl.ARRAY_BUFFER,buf);

gl.bufferData(gl.ARRAY_BUFFER,new Float32Array([-1,-1,1,-1,-1,1,1,1]),gl.STATIC_DRAW);

gl.useProgram(pr);

const locRes=gl.getUniformLocation(pr,"u_res"),

      locT  =gl.getUniformLocation(pr,"u_time"),

      pos   =gl.getAttribLocation(pr,"p");

gl.enableVertexAttribArray(pos);

gl.vertexAttribPointer(pos,2,gl.FLOAT,false,0,0);



function resize(){c.width=innerWidth; c.height=innerHeight; gl.viewport(0,0,c.width,c.height);}

addEventListener('resize',resize); resize();



let start=performance.now();

function loop(){

  let t=(performance.now()-start)/1000;

  gl.uniform2f(locRes,c.width,c.height);

  gl.uniform1f(locT,t);

  gl.drawArrays(gl.TRIANGLE_STRIP,0,4);

  requestAnimationFrame(loop);

}

loop();



// --- 7‑11 Hz heartbeat audio (sub‑bass LFO) ---

const ctx=new (window.AudioContext||webkitAudioContext)();

const carrier=ctx.createOscillator(), gain=ctx.createGain(), lfo=ctx.createOscillator();

carrier.frequency.value=55; // A1 fundamental

lfo.frequency.value=8;      // center of 7‑11 Hz

lfo.connect(gain.gain); carrier.connect(gain).connect(ctx.destination);

lfo.start(); carrier.start();

</script>

</html>
