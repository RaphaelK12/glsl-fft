# glsl-fft

> GLSL setup for performing a [Fast Fourier Transform][fft]

This module does not interface with WebGL or have WebGL-specific peer dependencies. It only performs the setup work required to invoke the provided fragment shader that performs the Fourier transform.

## Installation

```sh
$ npm install glsl-fft
```

## Usage 

### What does it compute?

This shader computes the 2D [Fast Fourier Transform][fft] of two complex input matrices contained in a single four-channel floating point WebGL texture. The red and green channels contain the real and imaginary components of the first matrix, while the blue and alpha channels contain the real and imaginary components of the second matrix. The results match and are tested against [ndarray-fft][ndarray-fft].

### What is required?

This module is designed for use with [glslify][glslify], though it's not required. It also works relatively effortlessly with [regl][regl], though that's also not required. At minimum, you'll need to use this with no less than two WebGL framebuffers, including input, output, and two buffers to ping-pong back and forth between during the passes. The ping-pong framebuffers may include the input and output framebuffers as long as the parity of the number of steps permits the final output without requiring an extra copy operation.

### Is it fast?

Not particularly. Could be way better. Correctness first, then optimization.

## Example

The code below shows usage of the basic JavaScript API. The [example](./example/index.js) is perhaps more illustrative since it shows a fully worked example for a Gaussian blur. Note in particular the the return value of the JavaScript API is particularly suited for the way [regl][regl] performs multiple draw calls with a single function call, though [regl][regl] is certainly not reqired in order to use this module.

```javascript
var fft = require('glsl-fft');

var forwardTransform = fft({
  width: 4,
  height: 2,
  input: 'a',
  ping: 'b',
  pong: 'c',
  output: 'd',
  forward: true,
  normalization: 'inverse'
});

// => [
//  {input: 'a', output: 'c', horizontal: true, forward: true, resolution: [ 0.25, 0.5 ], normalization: 1, subtransformSize: 2},
//  {input: 'c', output: 'b', horizontal: true, forward: true, resolution: [ 0.25, 0.5 ], normalization: 1, subtransformSize: 4},
//  {input: 'b', output: 'd', horizontal: false, forward: true, resolution: [ 0.25, 0.5 ], normalization: 1, subtransformSize: 2}
// ]
```

An example GLSL fragment shader that performs the FFT given the above passes as uniforms is:

```glsl
precision highp float;

#pragma glslify: fft = require(glsl-fft)

uniform sampler2D src;
uniform vec2 resolution;
uniform float subtransformSize, normalization;
uniform bool horizontal, forward;

void main () {
  gl_FragColor = fft(src, resolution, subtransformSize, horizontal, forward, normalization);
}
```

See [example/index.js](./example/index.js) for a fully worked [Gaussian blur][gaussian] example using [regl][regl].

## JavaScript API

### `require('glsl-fft')(options)`

Perform the setup work required to use the FFT kernel in the fragment shader, `index.glsl`. Input arguments are:

- `input` (`Any`): An identifier or object for the input framebuffer.
- `output` (`Any`): An identifier or object for the final output framebuffer.
- `ping` (`Any`): An identifier or object for the first ping-pong framebuffer.
- `pong` (`Any`): An identifier or object for the second ping-pong framebuffer.
- `forward` (`Boolean`): `true` if the transform is in the forward direction.
- `size` (`Number`): size of the input, equal to the `width` and `height`. Must be a power of two.
- `width` (`Number`): width of the input. Must be a power of two. Ignored if `size` is specified.
- `height` (`Number`): height of the input. Must be a power of two. Ignored if `size` is specifid.
- `normalization`: `'split' | 'inverse'`: If `'split'`, normalize by `1 / √(width * height)` on both the forward and inverse transforms. If `'inverse'`, normalize by `1 / (width * height)` on only the inverse transform. Default value is `'split'`. Provided to avoid catastrophic overflow which may occur during the forward transform with half-float textures. One-way transforms will match [ndarray-fft][ndarray-fft] only if normalization mode is `'inverse'`.

Returns a list of passes. Each object in the list is a set of parameters that must either be used to bind the correct framebuffers or passed as uniforms to the fragment shader.

## GLSL API

### `#pragma glslify: fft = require(glsl-fft)`
### `vec4 fft(sampler2D src, vec2 resolution, float subtransformSize, bool horizontal, bool forward, float normalization)`

Returns the `gl_FragColor` in order to perform a single pass of the FFT comptuation. Uniforms map directly to the output of the JavaScript setup function, with the exception of `src` which is a `sampler2D` for the input framebuffer or texture.

### `#pragma glslify: wavenumber = require(glsl-fft/wavenumber)`
### `vec2 wavenumber(float size)`

Returns `vec2(kx, ky)`, where `kx` and `ky` are the wavenumber of the corresponding fragment of the Fourier Transform. Uses `gl_FragCoord` directly.

## License

&copy; Ricky Reusser 2017. MIT License. Based on the [filtering example][dli] of David Li. See LICENSE for more details.

[glslify]: https://github.com/glslify/glslify
[fft]: https://en.wikipedia.org/wiki/Fast_Fourier_transform
[dli]: https://github.com/dli/filtering
[regl]: https://github.com/regl-project/regl
[ndarray-fft]: https://github.com/scijs/ndarray-fft
[gaussian]: https://en.wikipedia.org/wiki/Gaussian_blur
