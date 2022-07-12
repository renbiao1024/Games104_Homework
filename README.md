# Games104_Homework

## 作业2

### 题目

在小引擎代码中找到 `pilot/engine/shader/glsl/color_grading.frag`，补充此 `shader` 代码中的` main `函数，以实现` ColorGrading `功能。若代码编译成功且实现方法正确，则可以看到进行 `ColorGrading `渲染之后的结果。 

### 思路

- 根据原始颜色在LUT找到对应的颜色位置，做一些插值即可

## 实现

~~~glsl
void main()
{
    highp ivec2 lut_tex_size = textureSize(color_grading_lut_texture_sampler, 0);
    //原始颜色
    highp vec4 color = subpassLoad(in_color).rgba;
    //LUT的长
    highp float _LutSize      = float(lut_tex_size.y);
    //像素大小
    highp float _SpriteSize = 1.0/_LutSize;
    //LUT的宽
    highp float texWidth = float(lut_tex_size.x);
    //找到b的LUT位置
    highp float v = color.b * (_LutSize - 1.0);
    highp float f_v = floor(v);
    highp float c_v = ceil(v);
    //三通道的 b 分别存放 floor color ，ceil color ，alpha
    highp vec3 b = vec3(f_v * _SpriteSize, c_v * _SpriteSize, (v - f_v));
    //找到r，g在LUT 一个block里的位置
    highp float r = color.r * (_LutSize - 1.0) * (1.0/texWidth);
    highp float g = color.g * (_LutSize - 1.0) * _SpriteSize * (1.0/texWidth);
	//插值避免过度明显
    highp vec2 luv = vec2(r + b.x, g);
    highp vec3 lw = texture(color_grading_lut_texture_sampler, luv).rgb;
    highp vec2 huv = vec2(r + b.y, g);
    highp vec3 hw = texture(color_grading_lut_texture_sampler, huv).rgb;
    highp vec3 ret = mix(lw, hw, b.z);
    out_color = vec4(ret.rgb, 1.0);
}
~~~

## 效果

![](.\assets\image-20220712180552067.png)
