# diamond-webgl
This is a real-time rendering engine for diamonds and gemstones. It allows precise control of the cut, and customization of various optical properties to simulate countless types of materials. It can also generate ASET images. The precision is more than enough to create the familiar hearts and arrows patterns of ideal cut diamonds.

The engine runs fully on GPU and uses both rasterization and ray-tracing, as well as post-processing techniques such as bloom and screen-space antialiasing.

It supports many gem shapes, and can render several gems simultaneously.

<div style="background: rgba(200,200,200,0.25);padding: 8px;border-radius: 8px;box-shadow: 0 0 6px rgba(0,0,0,0.5);">If you want to see the latest version of the engine or want to discuss commercial use, please go to my dedicated website:
<div style="text-align:center;margin-top:1em">
    <a href="https://www.3djewelryviewer.com">
        <p>3djewelryviewer.com ↗</p>
        <img src="docs/3djewelryviewer/illustration_500.jpg" alt="Illustration for 3djewelryviewer.com" style="display:block;margin:.5em auto 0;"/>
    </a>
    </div>
</div>

See it live [here](https://piellardj.github.io/diamond-webgl/).

[![Donate](https://raw.githubusercontent.com/piellardj/piellardj.github.io/master/images/readme/donate-paypal.svg)](https://www.paypal.com/donate/?hosted_button_id=AF7H7GEJTL95E)

## Preview

![Example of ring](src/readme/ring1.png)

![Round brilliant cut diamond](src/readme/bloom_final.png)

![Example of ring](src/readme/ring2.png)

![Examples of cuts](src/readme/models.png)

![Hearts pattern](src/readme/hearts.png)

![Arrows pattern](src/readme/ASET_arrows.png)

![Geometry](src/readme/geometry.png)

![Emerald](src/readme/emerald.png)

## Diamond
### Diamond cut
The beauty of a diamond resides not only in the purity of the gem, but also in the way it is cut. The purity affects mostly light absorption, while the cut affects the way light is reflected. The goal is to reflect as much light as possible towards the viewer (that is, towards the top of the gem) so that the diamond appears the brightest.

One of the most popular diamond cuts is the brilliant cut. Subtle variations in the proportions change greatly the way light is reflected and can make the difference between a mediocre and an ideal diamond. It is described by a few key lengths:

<div style="text-align:center">
    <img alt="Diagram of a round brilliant cut" src="src/readme/diamond_brilliant_cut.png"/>
    <p>
        <i>Diagram of a round brilliant cut.</i>
    </p>
</div>

### ASET evaluation
A common tool to evaluate the quality of the cut of a diamond is the Angular Spectrum Evaluation Tool (ASET). Such an image helps to check the way the diamond gives light back, which depends on the cut quality and the clarity of the gem.

ASET images can either be taken with in with an ASET scope, or be computed. Here is what it represents:

<div style="text-align:center">
    <img alt="Explanation of ASET" src="src/readme/ASET_meaning.png"/>
    <p>
        <i>This is what an ASET image represents.</i>
    </p>
</div>

On an ASET image, the blue is the light that comes directly from above, the green from the sides, and the red from between the two. The black parts are parts that don't reflect the light at all.
The ASET image of a high quality diamond should:
- be as bright as possible
- exhibit a good symmetry
- have as much red as possible
- have blue areas that are very distinct from the red and green ones. This contributes to creating good contrasts between bright and dark areas.

To generate an ASET image, orthographic projection should be used to avoid deformations due to perspective projection, especially when being too close to the gem. This project is accurate enough to generate realistic ASET images in real time. For instance, here are the visualizations of a diamond with an ideal brilliant cut, and their decomposition:

<div style="text-align:center">
    <img alt="Arrows visible with ASET" src="src/readme/ASET_arrows_decomposition.png"/>
    <p>
        <i>Viewed from top, the arrows of the diamond are visible in the blue component of ASET.</i>
    </p>
</div>

<div style="text-align:center">
    <img alt="Hearts visible with ASET" src="src/readme/ASET_hearts_decomposition.png"/>
    <p>
        <i>Viewed from bottom, the hearts of the diamond are visible in the red component of ASET.</i>
    </p>
</div>

## Implementation details

### Geometrical optics
The behaviour of light rays as they go through the diamond can be described by a few simple rules.

#### Refractive index
A medium such as diamond is defined by a refractive index n, which is defined as:
<table>
    <tr>
        <td>
            <img alt="Refractive index definition" src="src/readme/formula_refractive_index.png"/>
        </td>
        <td>
            <ul>
                <li>c is the speed of light in void</li>
                <li>v the speed of light in the medium</li>
            </ul>
        </td>
    </tr>
</table>

This is why a refractive index is always greater than 1. I approximated air to have a refractive index of 1.

The speed of light in the medium is defined as:
<table>
    <tr>
        <td>
            <img alt="Light speed formula" src="src/readme/formula_light_speed.png"/>
        </td>
        <td>
            <ul>
                <li>λ is the wavelength</li>
                <li>f is the frequency</li>
            </ul>
        </td>
    </tr>
</table>

The perceived color is linked to the frequency, which is independent of media. This formula shows that for a given medium, each color has a unique refractive index. This is how a prism turns white light into a visible rainbow.

#### Snell's law
When a incident ray r<sub>i</sub> hits an boundary between two media (for instance a side of the diamond), it is split in two:
- a part of the ray (r<sub>r</sub>) is reflected
- another part (r<sub>t</sub>) is transmitted through the medium

<div style="text-align:center">
    <img alt="Illustration of ray splitting" src="src/readme/reflection_refraction.png" width="400px"/>
</div>

Here all angles are considered to be in [0, π/2] and are defined relatively to the local normal of the surface.
The reflected ray has the same angle as the incident ray (θ<sub>i</sub> = θ<sub>r</sub>).
The relationship between θ<sub>i</sub> and θ<sub>t</sub> is described by Snell's law:

![Snell's formula](src/readme/formula_snell.png)

This formula shows that:
- if the ray is entering a medium with a higher refraction index, the transmitted ray will be closer to the normal than the incident ray;
- if the ray is entering a medium with a lower refraction index, it is the other way around.

#### Fresnel
The intensity of the incident ray is split between the reflected ray and the transmitted ray (assuming there is no loss):

<table>
    <tr>
        <td>
            <img alt="Ray splitting formula" src="src/readme/formula_energy_split.png"/>
        </td>
        <td>
            <ul>
                <li>F is the intensity of the incident ray</li>
                <li>F<sub>r</sub> is the intensity of the reflected ray</li>
                <li>F<sub>t</sub> is the intensity of the transmitted ray</li>
            </ul>
        </td>
    </tr>
</table>

Fresnel gives us these coefficients:

![Fresnel formula](src/readme/formula_fresnel.png)

This formula shows that the closer to the normal the incident ray is, the less reflective the surface is. This phenomenon is very natural and can be observed everywhere: if you are by a lake and look at the water at your feet, you clearly see the ground underneath the surface (no reflection due to perpendicular incident ray), however if you look in the distance, you only see the sky reflected by the water surface (high reflection due almost parallel incident ray).


#### Total reflection

If the light ray is entering a medium with a lower refractive index, the transmitted ray will be further away to the surface than the incident ray.

The critical angle (θ<sub>c</sub>) defines the maximum angle the incident ray can have to create a transmitted ray. Beyond this angle, the entirety of the incident ray is reflected. This is called a total reflection.

<div style="text-align:center">
    <img alt="Total reflection illustration" src="src/readme/total_reflection.png" width="400px"/>
</div>

This angle is simply provided by the Snell formula when θ<sub>t</sub> = π/2:

![Formula of the critial angle](src/readme/formula_critical_angle.png)

#### Beer absorption
No medium is completely transparent: the light is a partially absorbed by the material it is traveling through. The more opaque the medium, the more light is absorbed. This is described by Beer's law:

<table>
    <tr>
        <td>
            <img alt="Beer's law formula" src="src/readme/formula_beer.png"/>
        </td>
        <td>
            <ul>
                <li>L is the loss of intensity</li>
                <li>α is the absorbance of the medium</li>
                <li>d is the distance traveled through the medium</li>
            </ul>
        </td>
    </tr>
</table>

### Post processing

#### Sparkle effect
A small sparkle effect is performed. It is essentially a bloom with a bidirectional blur:
1. rendering to an off-screen texture, at full size
2. copying this texture into a smaller one (to make the next part cheaper) and extracting the bright parts
3. blurring the small texture (I don't use a gaussian blur, but simply blur in two directions in one pass to create a sparkle effect)
4. finally, combining both the full-sized and small texture

<div style="text-align:center">
    <div>
        <img src="src/readme/bloom_chain.png"/>
    </div>
    <p>
        <i>These are the steps for the sparkle post-processing effect.</i>
    </p>
</div>

#### Antialiasing
In this scene there are 2 types of aliasing:
- on the edges of the geometry. This one is due to the rasterization.
- inside the diamond. This one is due to the processing of each fragment which might lead to neighbour fragments having very different colors.

On my machine, asking for antialiasing with the `antialias` [WebGL flag](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/getContext) only removes the first type of aliasing. This makes me think it uses a kind of MSAA because a SSAA would also remove the fragment aliasing. I am not sure it is part of the WebGL spec, it is probably implementation-dependent.

When doing post-processing, I first render a scene to an off-screen texture. Unfortunately, WebGL 1 does not support antialiasing when rendering to a texture. To antialias the scene, I have two options:
- do a kind of SSAA where I use a texture twice larger than the screen, which I downsize back to screen dimensions later. I cannot do this because it would lead to way too many fragments being processed (each fragment takes a lot of processing power in a ray-tracing application).
- or apply myself antialiasing as a step of post-processing without upsizing any buffer. This is the approach I chose.

I implemented a simplified FXAA algorithm:
1. to avoid treating each color channel separately, I first turn the image into greyscale by computing the luminance of each pixel. The luminance is defined as `(0.299 x red) + (0.587 x green) + (0.114 x blue)`, as suggested by W3C [here](https://www.w3.org/TR/AERT/#color-contrast).
2. I then determine which areas need antialiasing: for each texel, I sample its luminance and the luminance of the 8 closest neighbours.
3. I then sort the neighbours into 2 categories: the ones that look like the central texel, and the others
4. I then use this binary categorization to determine the direction if the edge (if there is one): mostly horizontal or vertical. Below is an example of a texel (in red) that is part of an horizontal edge because the difference between (1, 2, 3) and (6, 7, 8) is greater than the difference between (1, 4, 6) and (3, 5, 8).\
![Antialiasing explanation](src/readme/antialiasing_direction_horizontal.png)
5. Finally, I apply a blur in that direction.

This algorithm provides a good antialiasing in one pass with only 9 texture fetches and cheap computation. To improve it, I would need to sample a larger neighbourhood, but I don't think it is worth it. If you want to go further, [here is a great description of FXAA](http://blog.simonrodriguez.fr/articles/30-07-2016_implementing_fxaa.html).

Here is the result I obtain:

<div style="text-align:center">
    <img alt="Antialiasing result" src="src/readme/antialiasing_magnified_4x.png"/>
    <p>
        <i>Source image on the left, antialiased image on the right. Notice how most edges are antialiased, while still preserving sharp details such as dots and vertical/horizontal/diagonal lines.</i>
    </p>
</div>

## Other approaches
This ray tracing project is quite processing-heavy for the GPU because for each fragment and each ray rebound, we have to check intersection with all facets of the gem. For a typical diamond, it is 89 intersections. (In reality it is a bit less than this because the shader tries to skip facets that are behind.) The overall complexity is roughly FRAGMENTS_COUNTxREBOUNDS_COUNTxFACETS_COUNTS.

Another approach would be to first generate a cube map of the diamond sides seen from the center of the gem, and then sample this texture directly in the direction of the ray. This way, the complexity is only FRAGMENTS_COUNTxREBOUNDS_COUNT. However, this leads to 2 issues:
- first, the precision is dependent of the resolution of the cube map
- then, this approach is by essence not accurate, because the diamond is not a sphere, so its projection would be deformed.  An adjustment mechanism would be necessary, which would cost more texture fetches. 

