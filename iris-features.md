# Iris Features

### All the following features are exclusive to Iris.

## Table of Contents

1. [New Programs](#new-programs)
2. [Enhanced Custom Textures](#enhanced-custom-textures-iris-15)
3. [Defines / Feature Flags](#defines--feature-flags)
4. [Uniforms](#uniforms)
5. [Shader Properties](#shader-properties)
6. [Custom Entity ID's](#custom-entity-ids)
7. [Item and Armor Detection](#item-and-armor-detection)
8. [Dimension Folders](#dimension-folders)
9. [Color Spaces](#color-spaces-iris-164)
10. [Reverse Shadow Culling](#reversed-shadow-culling-iris-166)
11. [Light Block Voxelization](#light-block-voxelization)
12. [Hybrid-Deferred Entities](#hybrid-deferred-entities)
13. [Separate Hardware Shadow Samplers](#separate-hardware-shadow-samplers)
14. [Shader Storage Buffer Objects](#shader-storage-buffer-objects)
15. [Custom Images](#custom-images)
16. [Extended Shadowcolor](#extended-shadowcolor)

# New Programs

## Terrain Solid (Iris 1.6)

This program renders all solid terrain, that is terrain with no transparency or cutouts. When using a core profile you can safely avoid using `discard` in this program, which preserves early z testing.

### Declaration

```
gbuffers_terrain_solid
```

## Terrain Cutout (Iris 1.6)

This program renders all cutout terrain, that is terrain with cutouts and no other transparency. When using core profile, `discard` should be used below a threshold alpha to achieve the cutout.

### Declaration

```
gbuffers_terrain_cutout
```

## Entity Translucent (Iris 1.5)

This program renders all entities that are marked translucent and have blending enabled. If this program is not used, all entities will be rendered with `gbuffers_entities`.

### Declaration

```
gbuffers_entities_translucent
```

## Block Entity Translucent (Iris 1.6)

This program renders all block entities that are marked translucent and have blending enabled. If this program is not used, all block entities will be rendered with `gbuffers_block`.

### Declaration

```
gbuffers_block_translucent
```

## Particles (Iris 1.6)

This program renders all non-translucent particles. If this program is not used, all particles will be rendered with `textured_lit`.

### Declaration

```
gbuffers_particles
```

## Particles Translucent (Iris 1.6)

This program renders all translucent particles with blending enabled. If this program is not used, all translucent particles will be rendered with `particles`.

This can be combined with [Particle Ordering](#particle-ordering-iris-15) to render translucent particles after the deferred pass.

### Declaration

```
gbuffers_particles_translucent
```

## Begin Program (Iris 1.6)

The begin program is a new composite pass that runs before the shadow pass, and is intended to be used for setting up any data that is needed for the shadow pass. It can be used as a normal composite.

### Declaration

```
begin
```

## Setup Program (Iris 1.6)

The setup pass can only be used as a compute pass, and is only run once, during the pack load or when the screen size changes. However, you can use a_z suffixes to have multiple compute passes.

### Declaration

```
setup.csh
setup_a.csh ... setup_z.csh
```

# Enhanced Custom Textures (Iris 1.5)

Instead of using the OptiFine format of replacing a color texture with a custom texture, you can define entirely custom textures to use in programs.
This completely sidesteps the obsolete requirements of sacrificing a color texture. However, this does not change the amount of textures (16/32 depending on machine) you can use in a program at a given time. Enhanced custom textures are avaliable from any program, similar to custom images.

### Example

```properties
customTexture.name = <path> <type> <internalFormat> <dimensions> <pixelFormat> <pixelType>
```

```glsl
uniform sampler2D name;
```

# Defines / Feature Flags

## IS_IRIS (Iris 1.6)

You can check for the define `IS_IRIS` to check for Iris support.

## Feature Flags

Feature flags are a new system in Iris to query the existence of certain features. To activate them use `iris.features.required` to show an error if your PC or Iris doesn't support a feature, or use `iris.features.optional` to get a define with the feature name `IRIS_FEATURE_X` if the feature is supported.

The currently added feature flags are:

`SEPARATE_HARDWARE_SAMPLERS` (required for [Separate Hardware Shadow Samplers](#separate-hardware-shadow-samplers))

`PER_BUFFER_BLENDING`

`COMPUTE_SHADERS`

`ENTITY_TRANSLUCENT` (recommended, but not required for [Entity Translucent](#entity-translucent--iris-15-))

`SSBO` (required for [Shader Storage Buffer Objects](#shader-storage-buffer-objects))

`CUSTOM_IMAGES` (required for [Custom Images](#custom-images))

`HIGHER_SHADOWCOLOR` (required for [Extended Shadowcolor](#extended-shadowcolor))

`REVERSED_CULLING` (recommended, but not required for [Reverse Shadow Culling](#reversed-shadow-culling-iris-166))

`BLOCK_EMISSION_ATTRIBUTE` (recommended, but not required for [Block Emission: at_midBlock.w] as of [iris-1.7.0-snapshotmc1.20.4-a787322])

# Uniforms

## Lightning bolt position (Iris 1.2.5)

This value reads the position of a lightning bolt currently being rendered. If there are none, zero is returned. If there is at least one, `w` is set to 1. If there are multiple,
a random one is chosen.

### Declaration

```glsl
uniform vec4 lightningBoltPosition;
```

## Thunder strength (Iris 1.3)

This value controls the "thunder strength", equivalent to Optifine's rainStrength for thunder.
### Declaration

```glsl
uniform float thunderStrength;
```

## Player health, air, armor, and hunger (Iris 1.2.7 - 1.6.15)

These are multiple declarations to read player health, air, and hunger. Armor was added in 1.6.15.

The uniforms marked with `current` are 0-1. To get them as a full value, multiply them by their `max` values.

### Declaration

```glsl
uniform float currentPlayerHealth;
uniform float maxPlayerHealth;
uniform float currentPlayerAir;
uniform float maxPlayerAir;
uniform float currentPlayerHunger;
uniform float maxPlayerHunger;
uniform float currentPlayerArmor;
uniform float maxPlayerArmor;
```

## Camera uniforms & Eye Position (Iris 1.4)

These uniforms read multiple aspects of the camera.

`eyePosition` stores the world space position of the player's head model. When in first person view, this is equivalent to `cameraPosition`. However in third person mode the two will differ as the camera and player's head are now in different locations.

### Declaration

```glsl
uniform bool firstPersonCamera;
uniform bool isSpectator;
uniform vec3 eyePosition;
```

## Additional Player Model Uniforms (Iris 1.6.11)

These uniforms read multiple aspects of the player model and camera.

`relativeEyePosition` reads the world space offset from the player model's head position to the camera's position(ie `cameraPosition` - `eyePosition`).

`playerLookVector` reads the world alligned direction the player model's head is facing. This facing direction is not affected by animations such as swimming.

`playerBodyVector` reads the world alligned direction the player model's body is facing, although this behavior is currently broken and reads the same value as `playerLookVector`.

### Declaration
```glsl
uniform vec3 relativeEyePosition;
uniform vec3 playerLookVector;
uniform vec3 playerBodyVector;
```

## World Info Uniforms (Iris 1.5)

These uniforms read many aspects of the dimension, such as the minimum/maximum height and ambient lighting.


### Declaration

```glsl
uniform int bedrockLevel;
uniform int heightLimit;
uniform bool hasCeiling;
uniform bool hasSkylight;
uniform float ambientLight;
```

## Logical Height Uniform (Iris 1.6)

This uniform reads the logical height of the current dimension, which refers to the maximum height to which chorus fruits and nether portals can bring players within the dimension.

### Declaration

```glsl
uniform int logicalHeightLimit;
```

## Cloud Height Uniform (Iris 1.6.9)

This uniform reads the height of vanilla clouds in the current dimension in blocks. Value is `NaN` for dimensions without clouds.

### Declaration

```glsl
uniform float cloudHeight;
```

## Player sneaking, sprinting, hurt, invisible, and burning uniforms (Iris 1.5)

These boolean uniforms are `true` while the condition they are named after is active.

`is_hurt` is `true` for a short time after the player is hurt for any reason, then returns to `false`.

`is_invisible` is `true` both when using an invisibility potion and when in spectator mode.

### Declaration

```glsl
uniform bool is_sneaking;
uniform bool is_sprinting;
uniform bool is_hurt;
uniform bool is_invisible;
uniform bool is_burning;
```

## Player is on ground (Iris 1.6.5)

This boolean uniform is `true` when the player is not flying is an on the ground, and `false` otherwise.

### Declaration

```glsl
uniform bool is_on_ground;
```

## Color Space Uniforms (Iris 1.6.4)

This value reads the color space used when displaying to the screen. 0 is sRGB, 1 is DCI_P3, 2 is Display P3, 3 is REC2020, 4 is Adobe RGB.

### Declaration
```glsl
uniform int currentColorSpace;
```

## Biome Uniforms (Iris 1.6.11)

These uniforms are used to identify and read aspects of the biome the player is currently in. These uniforms are defined the same as when using custom uniforms with these variables.

`biome` identifies the biome currently occupied by the player. It's value can be compared with the same predefined constants as custom unfiorms using biome, for example: `BIOME_PLAINS`, `BIOME_RIVER`, `BIOME_DESERT`, `BIOME_SWAMP`, etc.

`biome_category` identifies the biome category currently occupied by the player. It's value can be compared with the same predefined constants as custom unfiorms, the following are recognized:

`CAT_NONE`, `CAT_TAIGA`, `CAT_EXTREME_HILLS`, `CAT_JUNGLE`, `CAT_MESA`, `CAT_PLAINS`, `CAT_SAVANNA`, `CAT_ICY`, `CAT_THE_END`, `CAT_BEACH`, `CAT_FOREST`, `CAT_OCEAN`, `CAT_DESERT`, `CAT_RIVER`, `CAT_SWAMP`, `CAT_MUSHROOM`, `CAT_NETHER`

`biome_precipitation` tells what type of precipitation occurs in this biome. 0 is no precipitation, 1 is rain, 2 is snow. The following defines can also be used: `PPT_NONE`, `PPT_RAIN`, `PPT_SNOW`.

`rainfall` and `temperature` measure aspects of the biome as defined by Minecraft internally, and range in value form 0 to 1.

### Declaration
```glsl
uniform int biome;
uniform int biome_category;
uniform int biome_precipitation;
uniform float rainfall;
uniform float temperature;
```

## System Time Uniforms (Iris 1.6.11)

These uniforms allow shaders to access the OS reported date and time.

`currentDate` is in the following format: ivec3(year, month, day)

`currentTime` is in the following format: ivec3(hour, minute, second)

`currentYearTime` is in the following format: ivec2(seconds_ellapsed_in_year, seconds_remaining_in_year)

### Declaration
```glsl
uniform ivec3 currentDate;
uniform ivec3 currentTime;
uniform ivec2 currentYearTime;
```

# Shader Properties

## Particle ordering (Iris 1.5)

This controls how particles will be rendered. There are 3 possible options for this value.

`mixed`: Opaque particles are rendered before the deferred pass, and translucent particles are rendered after.

This is the default if **no deferred passes** are present.


`after`: All particles are rendered after the deferred pass.

This is the default if there are any deferred passes.

`before`: All particles are rendered before the deferred pass.

### Declaration

```
particles.ordering = mixed
```

### Valid Declaration Locations

* `shaders.properties`

## Explicit Shadow Pass Enable/Disable  (Iris 1.4.3)

Normally Optifine/Iris will check if the shadow buffers are bound and used to determine when to enable/disable the shadow pass. Iris offers an additional explicit control for enabling/disabling the pass. Thic can be useful for voxelization which doesn't write to a shadowtex/shadowcolor buffer (for example SSBOs or custom images).

### Declaration

```
shadow.enabled = true
shadow.enabled = false
```

### Valid Declaration Locations

* `shaders.properties`

## Entity shadow distance multiplier (Iris 1.2.1)

This value controls the bounds for entity shadows to be rendered. By default, it is the value set for terrain. Any floating point number that is not 1 is multiplied by the terrain distance to get the final shadow distance multiplier.

### Declaration

```glsl
const float entityShadowDistanceMul = 1.0f;
```

### Value Range

* Minimum Value: 0.01
* Default Value: Terrain value
* Out-of-range values behavior: Disable

### Valid Declaration Locations

* ❌ Vertex Shader (*.vsh)
* ❌ Geometry Shader (*.gsh)
* ✔️ Fragment Shader (*.fsh)


## shadowPlayer directive (Iris 1.2.5)

This value controls if the player should have a shadow rendered. This is forced on if shadowEntities (default true) is enabled.
This also controls shadows of any entities the player is riding.

### Declaration

```
shadowPlayer = true
```

### Valid Declaration Locations

* `shaders.properties`


## Concurrent Compute (Iris 1.4)

This value controls if compute shaders within the same pass are allowed to run concurrently. For more information, reference [Concurrency Between Compute Passes](passes/compute.md#concurrency-between-compute-passes).

### Declaration

```
allowConcurrentCompute = true
```

### Valid Declaration Locations

* `shaders.properties`

# The following are exclusive to Iris 1.6.

# Custom Entity ID's

Iris hardcodes some custom entity ID's to detect specific things.

- `minecraft:entity_shadow`: The circular shadow under an entity when there is no shadow map.
- `minecraft:entity_flame`: The flame when an entity is on fire.
- `minecraft:zombie_villager_converting`: A zombie villager undergoing conversion.
- `minecraft:player_cape`: Player cape (without elytra).
- `minecraft:elytra_with_cape`: Player cape (with elytra).

# Item and Armor Detection

Iris allows detecting items and armor during rendering on *anything*. 

Using `uniform int currentRenderedItemId;`, you can detect items and armor rendered in the level at the point of render.

There are some new ID's that can be detected alongside items and armor:

`trim_material` to detect armor trims on armor. (For example, `trim_emerald`).

# Dimension Folders

When `dimension.properties` is added to the shader pack root, behavior relating to dimensions change.
Iris will no longer resolve any dimensions for you, and you are expected to resolve you own. The syntax for dimension.properties is as follows:

`dimension.<folderName> = <dimensionNamespace>:<dimensionPath>`

- `<folderName>`: The name of the folder containing the shaders used for the given dimension.
- `<dimensionNamespace>`: The namespace for the dimension, vanilla shaders will use `minecraft`.
- `<dimensionPath>`: The internal name of the dimension.

*You can use `*` as a value to fallback all dimensions.*

The following example sets the shaders for the vanilla Nether dimension to the `netherShaders` folder:

`dimension.netherShaders = minecraft:the_nether`

# Color Spaces (Iris 1.6.4)

Iris 1.6.4 added support for additional color spaces beyond sRGB (DCI_P3, Display P3, REC2020, and Adobe RGB). This does not allow for outputting HDR values, it simply applies a tonemapping operation in the given color space instead of sRGB.

By default, Iris will assume all shaders output sRGB, and if a different color spaces is selected it will convert the sRGB output to that color space for display. If `supportsColorCorrection = true` is in shaders.properties however, this conversion will be left up to the shader. In all scenarios, the chosen colorspace is avaliable through the uniform `currentColorSpace`.

# Reversed Shadow Culling (Iris 1.6.6)

Setting `shadow.culling = reversed` in shaders.properties will create an area around the player where geometry in the shadow pass will not be culled. Outside this area, geometry will be culled as if shadow culling was enabled. This "unculled" distance is controled with `const float voxelDistance`, and the culled distance is controlled as normal with `const float shadowDistance` (and cannot be lower than `voxelDistance`). This feature is intended for packs which utilize both voxelization and a shadow map.

# Light Block Voxelization

Using `voxelizeLightBlocks` in shaders.properties, you can now voxelize light blocks in the shadow or main pass.

Light blocks will be rendered as a single (degenerate) invisible quad with all points centered on the middle of the block. (`at_midBlock` will be 0.) The ID will correspond as normal to the light block,
and UV will be 0. `lmcoord.xy` will both be the value of the light made by the light block.

# Hybrid Deferred Entities

Using the new `gbuffers_entities_translucent` and `gbuffers_block_translucent` programs, you can now render entities and blocks in a hybrid deferred manner.

If `separateEntityDraws` is true in `shaders.properties`, entity draws will behave differently. During the main render, translucent entities and block entities will wait to be drawn until **after** the deferred pass,
and then will be drawn in the `gbuffers_entities_translucent` and `gbuffers_block_translucent` programs.

# Separate Hardware Shadow Samplers

Separate hardware shadow samplers can be enabled using the [feature flag](#feature-flags) for it.

When enabled and shadow hardware filtering is enabled via the hardware constant, `shadowtex0` and `shadowtex1` will no longer function as hardware samplers.

Instead, you can use `shadowtex0HW` and `shadowtex1HW` to sample using hardware shadow filtering and software at the same time

# Extended Shadowcolor

If the [Feature Flag](#feature-flags) for extended shadowcolor is set, `shadowcolor2` through `shadowcolor7` is enabled.

These can be drawn to via `DRAWBUFFERS` or `RENDERTARGETS` in the shadow pass or shadow composites.

# Shader Storage Buffer Objects

Shader Storage Buffer Objects (or SSBO's) are buffers that can store large amounts of data between shader invocations, and even between frames.

For more information about the limits of SSBO's in OpenGL, read https://www.khronos.org/opengl/wiki/Shader_Storage_Buffer_Object.

SSBO's can hold a guaranteed maximum of 128MB, and on most drivers can hold as much as the GPU physically alllows.

Iris will give an error and fail to load a shader if allocating an SSBO would otherwise use up too much VRAM.

To allocate an SSBO for a shaderpack, put the following in shaders.properties, where index can be between 0 and 8:

`bufferObject.<index> = <byteSize> <isRelative> <scaleX> <scaleY>`

To define the SSBO as fixed size, simply exclude the last three options and fill `<byteSize>` with the size of the SSBO.

SSBOs can also be defined as screen-sized (Iris 1.6.6), where their size is relative to the screen dimensions. This is useful for storing data per-pixel. Fill `<isRelative>` with `true`, `<scaleX>` and `<scaleY>` then define the multipliers for the number of "pixels" in the SSBO relative to each screen dimension (i.e. 1.0 would mean the same dimension as the screen), and `<byteSize>` defines the number of bytes per "pixel". The total size of the SSBO is calculated as `int(viewWidth * scaleX) * int(viewHeight * scaleY) * byteSize`.

To use an SSBO in a shader, you must define it's layout. Here is an example definition of a SSBO, where bufferName can be any name:

**This layout must be the same across all shaders, otherwise the data will get corrupted.**

```glsl
layout(std430, binding = index) buffer bufferName {
    vec4 someData; // 16 bytes
    float someExtraData; // 4 bytes
};

void main() {
    someData = vec4(0); // Other shaders will see this
    gl_FragColor = vec4(someExtraData); // Read from any previous shaders, or the previous frame if it was never overwritten
}
```

You can optionally make all the variables of an SSBO local to avoid redefinitions. To do this, declare an accessor for the SSBO, as the following.

```glsl
layout(std430, binding = index) buffer bufferName {
    vec4 someData; // 16 bytes
    float someExtraData; // 4 bytes
} bufferAccess;

void main() {
    bufferAccess.someData = vec4(0); // Other shaders will see this
    gl_FragColor = vec4(bufferAccess.someExtraData); // Read from any previous shaders, or the previous frame if it was never overwritten
}
```

# Custom Images

Custom images allow to write up to 8 custom full images of data.

For more info on allowed formats and restrictions, along for their use in shaders, read https://www.khronos.org/opengl/wiki/Image_Load_Store.

To declare a custom image, use the following in shaders.properties:

```
image.cimage1 = samplerAccess format internalFormat pixelType <shouldClearOnNewFrame> <isRelative> <relativeX/absoluteX> <relativeY/absoluteY> <absoluteZ>
```

For example, to declare a RGBA32F custom image half the screen size that does not clear, use the following:

```
image.cimage1 = cSampler1 rgba rgba32f float false true 0.5 0.5
```

And to declare a RGBA8 3D custom image with dimensions of 512x512x512 that clears every frame:

```
image.cimage1 = cSampler1 rgba rgba8 float true false 512 512 512
```

This image can be accessed in 2 ways:

The image can be written and read per-pixel by declaring it as an `image`:

```glsl
uniform image2D cimage1;

void main() {
    vec4 previousValue = imageLoad(cimage1, vec2(0, 0)); // Reads from first pixel in the image
    imageStore(cimage1, vec2(0, 0), vec4(1, 0, 0, 1)); // Writes to first pixel in the image
}
```

Or the image can be read with filtering as a `sampler`:

```glsl
uniform sampler2D cSampler1;
varying float viewWidth;
varying float viewHeight;

void main() {
    gl_FragColor = texture2D(cSampler1, gl_FragCoord.xy / vec2(viewWidth, viewHeight)); // Samples the current pixel of the image with smooth linear filtering.
}
```
