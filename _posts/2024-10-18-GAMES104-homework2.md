---
title: GAMES104-作业2
date: 2024-10-18 16:41:21 +0800
categories: [GAMES104]
tags: [games104] # TAG names should always be lowercase
#media_subpath: 
---

[作业文档](https://cdn.boomingtech.com/games104_static/upload/PA02%EF%BC%9ARendering.pdf)

# GLSL

在 `color_grading.frag` 文件中，使用的是 GLSL (the OpenGL Shading Language) 变量类型、数学函数与 C++ 相似，但是多了很多图形学计算会用到的变量和函数，详见 [Overview of GLSL, the OpenGL Shading Language](https://www.youtube.com/watch?v=uOErsQljpHs&t=32s)

由于作业中用到了 `texture(aTexture, texCoord)` 函数，其实就是查询 `aTexture` 中**坐标**或**颜色**为 `texCoord` 的颜色。

`texture` -> `vec4`

`aTexture` -> `sampler2D or image/animation`

`texCoord` -> `vec2`

# 最终代码
```GLSL
void main()
{
    // 获取 LUT 纹理的尺寸
    highp ivec2 lut_tex_size = textureSize(color_grading_lut_texture_sampler, 0);
    highp float _COLORS      = float(lut_tex_size.y); // LUT的每一维的颜色数量
    highp vec4  color        = subpassLoad(in_color).rgba;
    highp float xsize        = float(lut_tex_size.x);//x总长
    
    highp float max_color = _COLORS - 1.0;
    highp float blueoffset   = max_color * color.b;
    highp float redoffset    = max_color * color.r;
    highp float greenoffset  = max_color * color.g;

    highp float u = (redoffset + floor(redoffset) * _COLORS) / xsize;
    highp float v = greenoffset / _COLORS;


    highp vec2 uv = vec2(u, v);

    highp vec4 color_sampled = texture(color_grading_lut_texture_sampler, uv);

    out_color =  color_sampled;
}

```
主要就是将 `color` 的范围 $[0, 1]$ **放大**到 `LUT` 的 $[u, v]$ 上。

`offset` 算的都是 `rbg` 在一个块的范围下的值。