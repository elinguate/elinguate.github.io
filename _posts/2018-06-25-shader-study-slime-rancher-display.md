---
title: Slime Rancher Display
tags: [shaders, unity]
style: fill
color: dark
description: a short shader study on Slime Rancher's gorgeous pixelated displays - replicating the effect in Unity w/ Amplify
comments: true
---

I think the most fitting shader to begin with has to be the one that inspired me to write this series in the first place - Slime Rancher's pixel displays. If you're intrigued by the game after our examination you can check it out on [Steam](https://store.steampowered.com/app/433340/Slime_Rancher/), and thanks to [Alan Zucconi](https://www.alanzucconi.com/) for tackling [this shader well before me](https://www.alanzucconi.com/2016/05/04/lcd-shader/), and inspiring me to delve deeper into it! We'll be recreating the shader behind it piece-by-piece in Unity with a little help from [Amplify Shader Editor](https://assetstore.unity.com/packages/tools/visual-scripting/amplify-shader-editor-68570), but the techniques applied can be used in whatever engine you happen to be using.


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/01.webp" caption="The game has a lovely low-poly look - filled with rich colours and cute slimes!" %}


Slime Rancher is a really beautiful game - with a distinct aesthetic style that reminds me of why I like low poly art. It's stylistic yet simple, and declutters the game's visual feedback to ensure that you're engaging with the gameplay, not just all of the noise on screen.

But it's the small notes that really draw out the uniqueness of their style. Hulking back-ends with dainty glass displays and bulky dais-based buttons exemplify the sort of off-brand future we're in. Even the content displayed on the screens is equally time appropriate, with constantly glitching graphics, and an especially awesome close-up look. As you approach the display the monitor appears to reveal more detail - showing off the arrays of bulky RGB pixels that underlay the image presented.


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/02.webp" caption="Slime Rancher's displays show stacked pixels when up close, but they smoothly fade away as we move back from the screen." %}


These pixels are much bigger than their real world counterparts, and yet they don't distort or muddy the normal image when you're further back from the screen. This is the sort of effect that you might not glance twice at as you're playing through Slime Rancher, but its a brilliant touch, that when noticed, makes you immediately appreciate whoever implemented it to be so effortlessly unique. There are a bunch of effects at play here, so let's tackle them one at a time.

#### Texture Pixelation

This one is a simple one, but it definitely helps to sell the dream of our very own retro TV - pixelation. When we chuck a texture onto a material normally, it will try to draw as much of the texture's detail as it can fit on the object (and screen!). We don't want that - we want our object to look like a display - and displays have a finite number of pixels to draw to. Regardless of the original texture's resolution, we need to pixelate our image to a fixed pixel resolution, and we'll do that by manipulating the UV coordinates.

UVs are what tell each part of our 3D mesh where to look in our 2D textures - mapping our 3D space to a normalized 2D space. They do so by acting as a two-dimensional lookup (coordinate). On a simple quad (what we'll be using), the bottom-left corner of the quad maps to (0.0, 0.0), and the top-right corner is (1.0, 1.0). Crucially these values tell us where to sample our textures - and by moving these values we can also move the texture we're drawing.


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/03.webp" caption="Above you can see the UV coordinates for a quad - red being our X value (U), and green being our Y value (V). The space in between the corners is interpolated cleanly, resulting in smooth continuous values." %}


To start off with, we're going to set our texture's import settings to point sampling - because we don't want our texture to innately smooth any of our samples. Then we need to modify our UVs - we don't want to have the smooth, seamless UV space displayed above - we instead want to 'jump' our UVs from pixel to pixel. To do this we're going to multiply our UV coordinates by a pixel resolution and then floor them. This will turn our continuous UV space (0.0-1.0) to discrete integer steps (1, 2, 3, 4, ...). With all that floating point precision between each step removed we just need to revert it back to our original UV space - and we can do as such by dividing it by the resolution from earlier. Through doing this we will still cover the same amount of space in our UVs - but we end up doing so with discrete steps, rather than through continuous change.

One subtle detail that we're unlikely to notice, but that we should get right is how we're sampling. We've correctly bound our UVs to discrete steps, but we haven't really considered what we're sampling within each pixel block. By using floor in the previous example, each pixel block is returning the bottom-left corner's value. We can see this in the UVs below - our bottom-left hand pixel is pure black - meaning we're returning (0.0, 0.0) as our value. This isn't ideal, because we want each pixel block to be returning a value that points to its center - not one of its corners.


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/04.webp" caption="The X points to what we're returning within the block - the pixel block should always be returning the center of itself, not a corner!" %}


Unfortunately, this issue is also apparent if we use ceiling or round instead of floor in our calculations. The former returns the opposite corner (top-right), and the latter creates odd half pixel blocks at the edges. At a high resolution this sort of issue might not be obvious, but we can (and should!) quickly fix this by simply offsetting our values. With the floor and ceiling functions we just need to add or subtract half a pixel block's UV space respectively - shifting our sampling right into the center. With round our fix is slightly more complex, as we need to shift the UV coordinates before we discretize it, and then shift it again afterward.

```cs
uv = (floor(uv * pixRes) / pixRes) + (0.5 * pixRes);
uv = (ceil(uv * pixRes) / pixRes) - (0.5 * pixRes);
uv = (round((uv + 0.5 * pixRes) * pixRes) / pixRes) - (0.5 * pixRes);
```

Once we've done all of this, we should have a nice little UV set that we can actually apply to a texture - and in doing this we'll end up with a pretty pixelated image. Because of how we've calculated the 'pixel' UVs on the fly, we don't have to go back to Photoshop or another image editing tool to further pixelate our image - we can adjust the resolution it's displayed at dynamically, and without editing the [underlying texture](slimerancher.gamepedia.com/File:IconPlortPink.png) of our adorable little plort!


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/05.webp" caption="Slime Rancher's cute pink plorts can be morphed from their original 1024x1024 resolution to whatever we need." %}


#### Hidden Pixels

This is functionally a very simple effect - but there are some subtle nuances that will be important to nail to really sell this. To start with, we have to focus on fading the pixels on and off as we get closer to the display. Like with all things in programming, there are lots of different ways of tackling this. One sneaky way of performing this fade-in is via mipmaps.

Mipmaps are a handy, underappreciated feature that passively help us render our textures optimally. When we're rendering to the screen in a realtime application, our textures aren't going to always be displayed at the same resolution as they are brought in at. 4K textures are great, but when they have repeating patterns in them (I'm looking at you mesh grate) they can produce some unflattering artifacts when placed on an object that's only occupying a couple hundred pixels on the screen.

To solve this, mipmaps come to the rescue. By generating downsampled versions of our texture and automatically swapping and blending between these as the size of the texture increases or decreases in size in our viewport, we end up with a seamless experience that ensures we don't end up with jarring artifacts disturbing our immersion. You can see how even a simple dot being repeated is easily broken without mipmaps enabled, but with them the detail is blended away naturally.


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/06.webp" caption="The weird artifacts on the left as we zoom out are Moiré patterns, caused by the intersection of the sampling pattern and the texture's built-in pattern." %}


By leveraging this for our pixel texture we can automatically perform the seamless reveal of the pixels with minimal difficulty. We just have to grab a pixel texture (a cropped version of [this](https://commons.wikimedia.org/wiki/File:LCD_pixels_RGB.jpg) works fine), adjust some import settings (ensure Generate Mip Maps is ticked), and hook up a basic texture sample of it to our emission output (or chuck it on a default unlit material). And just like that we're done! Naturally, to make it look better you can play around with the mipmap settings - in Unity I went with Box filtering, Fadeout on, and a medium fade range - but do whatever gives you the results you like the best!


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/07.webp" caption="And just like that a bunch of pretty pixels fade away to grey-ness. " %}


One unintended side effect of this is that our smaller mipmaps are inherently darker than we'd like. You can see this as we pan backwards and the pixels blur out to a grey-ish colour. This is due to how mipmaps are generated - they're downsampled versions of our original texture, so inevitably a mixture of red, green, blue, and black is going to average out to form a grey like this. To fix it we're just going to compensate as we blend between mipmaps by multiplying our texture output. We'll calculate the distance between the camera and the world position of the pixel, and use that to determine a brightness multiplier. 

Now that we've got our pixels fading away, we need to add the original texture to the display as well - we're not just interested in seeing a white monitor! Rather than simply adding it, we're going to multiply it by our pixel texture output, as this will simulate the pink plort (or whatever image you've chosen) stemming from the pixels themselves. With all this of put together, our setup is already looking pretty good!


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/08.webp" caption="By blending through mipmaps we get a clean look from afar, as well as a nice blend to our pixels up close." %}


#### Glitch Effect

Now that we have our display working properly we can start breaking it. We can see that once in a while the display glitches out - turning the display to a low-res pixelated mess. We've already played around with pixelation to get the texture displaying like a display, so now we just need to extend our previous effect to (occasionally and selectively) amp it up to 11! To do this we need to build a moving band, and this will help us lerp between the default pixelation and our special blend of retro pixel death.

Firstly we need something that moves in time, and for this we'll start with a simple sine node. By taking in our time parameter and passing it into a sine node we get a nice curve spanning from -1 to 1. This is great, but it's currently universal - it doesn't care where on the texture we are, it will always return a fixed value. To change this we need to make sure the sine output considers our UV height. Just adding it onto the time parameter works fine for now. I've performed a bunch of additional functions below to get a pattern that I'm happy with - feel free to replicate it or build your own!


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/09.webp" caption="The frequency modulates our UV value resulting in more bands, and our speed modulates how fast it progresses across the texture. A clamp to remap setup at the end rebinds the wave to create a small crest and lots of empty space." %}


With our new band lerping between resolutions we have a cool effect, but it lacks the 'smoothness' of Slime Rancher's visuals, and seems to come stacked with unintended artifacts. So what did we do wrong? Well for starters, our gradient band is perfectly smooth - its using raw UV data rather than our pixelated, discretized set. This results in our pixel blocks being non-homogeneous (they won't always agree on what to sample!), and given the context of a display, this doesn't really work. The pixel as a whole should always agree on what colour it's trying to show, so we'll have to fix this. Rather than passing in our raw UV data, we can instead pass in our normal pixelated UV data, this will gives us consistency over each default pixel block. 


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/10.webp" caption="This is getting there, but still causes some odd artifacts." %}


As well, we're also letting our lerp smoothly move between pixel sizes. Examining the UV coordinates or the texture on its own makes it clear what's wrong with this - because of these smooth, floating point changes we end up with curved elements in our final image. To stop this, we can floor our output pixel value to ensure that they only progress in integer amounts.

It's certainly looking better but we're still a bit off the sharpness of the original. We've fixed those smooth curves, and yet we still have some odd thin pixels being generated. Although we're guaranteeing that each pixel block is locked together (and will only be able to use an integer resolution), we haven't ensured that our new pixel block size will fit with the others around it. When we think of pixelation normally, we're really thinking about the level of subdivisions increasing or decreasing. We want a block of 16 pixels to become 4 mega pixels, and those 4 mega pixel to form into a super pixel (or vice versa). This means that we're inherently dealing with an exponential problem - we expect our resolution to double or half in size at each step. So long as we're dealing with resolutions that are a power of 2, we can easily bind the resolution to the last exponential step. By calculating the log of our output resolution (with base 2), flooring, and then raising 2 to the power of it, we ensure that we'll only end up with grid-friendly resolution steps.

#### Wrapping Up

With all this done, we finally have our shader wrapped up, and can enjoy it in action! Naturally, there's still plenty we can do to make this more impressive, and I've taken the liberty of doing some of those final polish steps for the image down below. None of them are particularly complex in and of themselves, but its the little touches that help make your shader unique and worthwhile, so never forget to polish!


{% include elements/figure.html image="../../assets/images/shader_studies/slime_rancher/11.webp" caption="With a render texture hooked up and some minor improvements under the hood, our shader has finally come to fruition." %}


If you're interested in the shader code that's behind what's displayed above, feel free to look through the source [here](https://github.com/elinguate-old/Shader_Studies)!

Got a burning question, a great shader you want me to tackle next, or some feedback about how I can improve? Post down below or get in touch, and thanks for following along!