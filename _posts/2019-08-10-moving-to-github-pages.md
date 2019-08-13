---
layout: post
title: Moving to Github Pages
date: 2019-08-10 17:34:00 -0500
tags: [jekyll, github, html, web]
---

I'm moving all of my old site content over to [Github Pages](https://pages.github.com/). Wordpress is just too much of a pain to deal with.

```java
/**
 *
 * Created by kmilligan on 1/20/18.
 */

@SuppressLint("ViewConstructor")
public class CameraSurfaceView extends SurfaceView implements SurfaceHolder.Callback {
    private static final String TAG = "ImageSurfaceView";
    private Camera camera;

    public CameraSurfaceView(Context context, Camera camera) {
        super(context);
        this.camera = camera;
        SurfaceHolder surfaceHolder = getHolder();
        surfaceHolder.addCallback(this);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        try {
            this.camera.setPreviewDisplay(holder);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        this.camera.startPreview();
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        this.camera.stopPreview();
    }
}
```
