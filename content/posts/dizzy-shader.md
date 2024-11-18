+++
title = 'Godot Dizzy Shader'
date = 2024-08-23T17:28:05+02:00
draft = true
tags = ["GLSL", "Godot"]
+++
Shader to make a stunned/dazzled/drunkish effect in 2D.

Based on one of the awesome shaders in the [Godot Retro shaders pack](https://github.com/ahopness/GodotRetro) (License CC0).

```glsl
shader_type canvas_item;

uniform float strength: hint_range(0, 5) = 1.5;
uniform sampler2D SCREEN_TEXTURE: hint_screen_texture, filter_linear_mipmap;

void fragment() {
	vec2 uv = FRAGCOORD.xy / (1.0 / SCREEN_PIXEL_SIZE).xy;
	float xOffset = snoise(vec2(TIME * 0.13, uv.y * 2.0)) * 0.003 * strength;
	float yOffset = snoise(vec2(TIME * 0.37, uv.x * 4.0)) * 0.008 * strength;
	vec3 color = texture(SCREEN_TEXTURE, vec2(uv.x + xOffset, uv.y + yOffset)).rgb;
	COLOR = vec4(color, 1.0);
}

vec3 mod289vec3(vec3 x) {
	return x - floor(x * (1.0 / 289.0)) * 289.0;
}

vec2 mod289vec2(vec2 x) {
	return x - floor(x * (1.0 / 289.0)) * 289.0;
}

vec3 permute(vec3 x) {
	return mod289vec3(((x * 34.0) + 1.0) * x);
}

float snoise(vec2 v) {
	const vec4 C = vec4(
		0.211324865405187, // (3.0-sqrt(3.0)) / 6.0
		0.366025403784439, // 0.5 * (sqrt(3.0) - 1.0)
		-0.577350269189626, // -1.0 + 2.0 * C.x
		0.024390243902439, // 1.0 / 41.0
    );
	// First corner
	vec2 i = floor(v + dot(v, C.yy));
	vec2 x0 = v - i + dot(i, C.xx);
	// Other corners
	vec2 i1;
	i1 = (x0.x > x0.y) ? vec2(1.0, 0.0) : vec2(0.0, 1.0);
	vec4 x12 = x0.xyxy + C.xxzz;
	x12.xy -= i1;
	// Permutations
	i = mod289vec2(i);	// Avoid truncation effects in permutation
	vec3 p = permute(permute(i.y + vec3(0.0, i1.y, 1.0)) + i.x + vec3(0.0, i1.x, 1.0));
	vec3 m = max(0.5 - vec3(dot(x0, x0), dot(x12.xy, x12.xy), dot(x12.zw, x12.zw)), 0.0);
	m = m * m;
	m = m * m;
	// Gradients: 41 points uniformly over a line, mapped onto a diamond.
	// The ring size 17*17 = 289 is close to a multiple of 41 (41*7 = 287)
	vec3 x = 2.0 * fract(p * C.www) - 1.0;
	vec3 h = abs(x) - 0.5;
	vec3 ox = floor(x + 0.5);
	vec3 a0 = x - ox;
	// Normalise gradients implicitly by scaling m
	// Approximation of: m *= inversesqrt(a0*a0 + h*h);
	m *= 1.79284291400159 - 0.85373472095314 * (a0 * a0 + h * h);
	// Compute final noise value at P
	vec3 g;
	g.x = a0.x * x0.x + h.x * x0.y;
	g.yz = a0.yz * x12.xz + h.yz * x12.yw;
	return 130.0 * dot(m, g);
}

```
