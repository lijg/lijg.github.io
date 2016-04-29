---
title: 渲染到纹理(RenderToTexture)
date: 2015-03-04 21:09:07
tags: 
- 纹理 
- GPU 
- OpenGL 
- GLES 
- RTT
categories: 
- Linux
---

渲染到纹理(RenderToTexture, RTT)，顾名思义就是把渲染目标从帧缓存变成一个纹理。这样就可以实现使一个场景经过多次GPU运算（中间过程都使用RTT），最后一次完全处理完后再渲染到屏幕上。

在Android中使用RTT的步骤主要有：
#1 初始化
* 1 创建FBO
```
void createFBO(void) {
    // 创建一个FrameBuffer对象
    glGenFramebuffers(1, &m_fbo);
}
```
* 2 创建存储图像的纹理

FBO本身不能存储图像数据，必须将数据绑定到一个纹理上，所以需要创建一个纹理，用于保存数据
```
void createTexture(void) {
    // 创建一个要绑定到FBO的纹理
    glGenTextures(1, &m_tex);
    // 选中当前纹理
    glBindTexture(GL_TEXTURE_2D, m_tex);
    // 设置纹理参数
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
            
    // 该纹理必须先用glTexImage2D提交一次，以使GPU确定纹理大小
    // ！！注意！！ 这里的ScreenWidth和ScreenHeight代表屏幕上实际的像素值！！！
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, ScreenWidth, ScreenHeight, 0, GL_RGBA,
        GL_UNSIGNED_BYTE, NULL);
}
```
* 3 绑定FBO和纹理
```
void bindFBOtoTexture(GLuint tex) {    
    // 绑定FBO和纹理，该纹理保存的是FBO中的COLOR信息
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D,
        m_tex, 0);

    // 检查是否绑定成功
    GLuint status = glCheckFramebufferStatus(GL_FRAMEBUFFER);
    if (status != GL_FRAMEBUFFER_COMPLETE) {
        LOGE("Could not attach texture to framebuffer");
    }
}
```
#2 渲染
* 1 渲染之前，绑定FBO
```
void bindFBO(GLuint tex) {
    // 保存原始绑定的FBO
    glGetIntegerv(GL_FRAMEBUFFER_BINDING, &m_oldFBO);

    // 渲染到新的FBO
    glBindFramebuffer(GL_FRAMEBUFFER, m_fbo);
    
    // 绑定纹理
    bindFBOtoTexture(tex);
}
```
* 2 渲染完后，使用FBO作为下一次处理的输入

```
// 激活一个纹理
glActiveTexture(GL_TEXTURE0);

// 绑定m_fbo到0号纹理
glBindTexture(GL_TEXTURE_2D, m_tex);

// 设置glsl中对应的sample2D使用的纹理号
glUniform1i(m_hTexture, 0);
```

* 3 处理完成后，释放FBO，渲染到屏幕

```
void releaseFBO() {
    glBindFramebuffer(GL_FRAMEBUFFER, m_oldFBO);
}
```