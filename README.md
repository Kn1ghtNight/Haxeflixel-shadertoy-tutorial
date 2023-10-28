# HaxeFlixel + Shadertoy Tutorial

This is a "rewrite" of the cool shadertoy tutorial that [Kn1ghtNight](https://github.com/Kn1ghtNight) did.

# STEP 1. Finding a shader

For starters, we're going to find the shader that we'll be using.

So now, we want to open our web browser and go to [Shadertoy](https://shadertoy.com).

As seen in the screenshot shown below, we have a few shaders presented on our screen.

![image](https://github.com/oofienoob/Haxeflixel-shadertoy-tutorial/assets/143152154/8fa16462-a086-4b2c-b794-7c35ed2caced)

We aren't gonna click on any of them though. However, we are gonna click on the search bar which is on the top left of the screen and search up VHS for example.

![image](https://github.com/oofienoob/Haxeflixel-shadertoy-tutorial/assets/143152154/0d9dfb2c-9f20-4306-8681-e5ae6cf9f961)

After looking at the collection of VHS shaders, let's use 20151110_VHS by FMS_Cat as an example. Let's click it, shall we?

![image](https://github.com/oofienoob/Haxeflixel-shadertoy-tutorial/assets/143152154/4edba318-288e-45db-8006-531aaa5905fc)

## STEP 2. Converting to a Haxeflixel Shader

After clicking it, we see the two things, being the showcase of the shader, and its code.

![image](https://github.com/oofienoob/Haxeflixel-shadertoy-tutorial/assets/143152154/4cabafd4-ab45-4dc2-94ae-762bf5ac8377)

Now make a file in the source folder, and let's call it VhsShader.hx for an example.

Open it in your code editing program, and paste the following template used as an example:

```haxe
import flixel.system.FlxAssets.FlxShader;

class VhsShader extends FlxShader //https://www.shadertoy.com/view/XtBXDt
{
    @:glFragmentSource('
    #pragma header
    //shader code goes here
    ')

    public function new()
        {
            super();
            iTime.value = [0.0];
        }
    
        public function update(elapsed:Float)
        {
            iTime.value[0] += elapsed;
        }
    
}
```

Now you want to keep in mind, while we're porting our shader, we need to modify a few things to make it work with Haxe.

We now need to delete the parameters in mainImage.

When done that, it should look like this:

![image](https://github.com/oofienoob/Haxeflixel-shadertoy-tutorial/assets/143152154/b329b550-d03f-463a-8a8f-ce14087b77e3)

We are now gonna press CTRL + A in here so we can copy all the code, and paste it into our .hx file we created.

![image](https://github.com/oofienoob/Haxeflixel-shadertoy-tutorial/assets/143152154/5fc04c87-d505-4f2b-b9ef-76a8324c66c9)

We are almost done with our code, 

As of now, it should look like this:

```haxe
import flixel.system.FlxAssets.FlxShader;

class VhsShader extends FlxShader
{
    @:glFragmentSource('
#pragma header

uniform vec2 iResolution;
uniform float iTime;

#define time iTime

#define PI 3.14159265

uniform bool chromaKey;

vec3 tex2D( sampler2D _tex, vec2 _p ){
    vec3 col = texture2D( _tex, _p ).xyz;
    if ( 0.5 < abs( _p.x - 0.5 ) ) {
        col = vec3( 0.1 );
    }
    return col;
}

float hash( vec2 _v ){
    return fract( sin( dot( _v, vec2( 89.44, 19.36 ) ) ) * 22189.22 );
}

float iHash( vec2 _v, vec2 _r ){
    float h00 = hash( vec2( floor( _v * _r + vec2( 0.0, 0.0 ) ) / _r ) );
    float h10 = hash( vec2( floor( _v * _r + vec2( 1.0, 0.0 ) ) / _r ) );
    float h01 = hash( vec2( floor( _v * _r + vec2( 0.0, 1.0 ) ) / _r ) );
    float h11 = hash( vec2( floor( _v * _r + vec2( 1.0, 1.0 ) ) / _r ) );
    vec2 ip = vec2( smoothstep( vec2( 0.0, 0.0 ), vec2( 1.0, 1.0 ), mod( _v*_r, 1. ) ) );
    return ( h00 * ( 1. - ip.x ) + h10 * ip.x ) * ( 1. - ip.y ) + ( h01 * ( 1. - ip.x ) + h11 * ip.x ) * ip.y;
}

float noise( vec2 _v ){
    float sum = 0.;
    for( int i=1; i<9; i++ )
    {
        sum += iHash( _v + vec2( i ), vec2( 2. * pow( 2., float( i ) ) ) ) / pow( 2., float( i ) );
    }
    return sum;
}

void main(){
    vec2 fragCoord = openfl_TextureCoordv;
    vec2 uv = openfl_TextureCoordv;
    vec2 uvn = uv;
    vec3 col = vec3( 0.0 );

    // tape wave
    uvn.x += ( noise( vec2( uvn.y, time ) ) - 0.5 )* 0.005;
    uvn.x += ( noise( vec2( uvn.y * 100.0, time * 10.0 ) ) - 0.5 ) * 0.01;

    // tape crease
    float tcPhase = clamp( ( sin( uvn.y * 8.0 - time * PI * 1.2 ) - 0.92 ) * noise( vec2( time ) ), 0.0, 0.01 ) * 10.0;
    float tcNoise = max( noise( vec2( uvn.y * 100.0, time * 10.0 ) ) - 0.5, 0.0 );
    uvn.x = uvn.x - tcNoise * tcPhase;

    // switching noise
    float snPhase = smoothstep( 0.0001, 0.0, uvn.y );
    uvn.y += snPhase * 0.3;
    uvn.x += snPhase * ( ( noise( vec2( uv.y * 100.0, time * 10.0 ) ) - 0.5 ) * 0.2 );

    col = tex2D( bitmap, uvn );
    col *= 1.0 - tcPhase;
    col = mix(
        col,
        col.yzx,
        snPhase
    );

    // bloom
    for( float x = -4.0; x < 2.5; x += 1.0 ){
        col.xyz += vec3(
            tex2D(bitmap, uvn + vec2( x - 0.0, 0.0 ) * 7E-3 ).x,
            tex2D(bitmap, uvn + vec2( x - 1.0, 0.0 ) * 7E-3 ).y,
            tex2D(bitmap, uvn + vec2( x - 1.0, 0.0 ) * 7E-3 ).z
        ) * 0.1;
    }
    col *= 0.75;

    // ac beat
    col *= 1.0 + clamp( noise( vec2( 0.0, uv.y + time * 0.2 ) ) * 0.6 - 0.25, 0.0, 0.1 );

    vec4 a = texture2D(bitmap, uv);

    for( float x = -4.0; x < 2.5; x += 1.0 ){
        a += vec4(
            tex2D(bitmap, uvn + vec2( x - 0.0, 0.0 ) * 7E-3 ).x,
            tex2D(bitmap, uvn + vec2( x - 2.0, 0.0 ) * 7E-3 ).y,
            tex2D(bitmap, uvn + vec2( x - 4.0, 0.0 ) * 7E-3 ).z,
            0
        ) * 0.1;
    }

    if(chromaKey){
        gl_FragColor = vec4(col, a.a);
    }else{
        gl_FragColor = vec4(col, 1.0 );
    }
}
    ')

    public function new()
        {
            super();
            iTime.value = [0.0];
        }
    
        public function update(elapsed:Float)
        {
            iTime.value[0] += elapsed;
        }
    
}
```

# STEP 3. Adding the Shader

Open up the state you want to add the shader in, and put these imports in:

```haxe
	import openfl.filters.ShaderFilter;
	import VhsShader;
```

Now, let's go to our variables in said state, and put this in:

```haxe
	var shader:VhsShader;
```

Finally, in our stage, or in the function create, put this in:

Then, in our stage, or in the function create, put:

```haxe
	shader = new VhsShader();
	camGame.setFilters([new ShaderFilter(shader)]);
```

Now if we compile, or recompile, whichever, we now see that the shader is now in camGame!

[You can set the shader to be in any cam, (camGame, camHUD, or camOther).]

![image](https://user-images.githubusercontent.com/89167668/226491010-ba29e8a9-0a17-4ad2-9c64-82196c7fe95f.png)

As you've may noticed, the shader isn't animated.

So, in the function update, paste the following code in there:

```haxe
	shader.update(elapsed);
```

And now when we did that, the shader is now animated!

![2023-03-20 20-05-33](https://user-images.githubusercontent.com/89167668/226491713-a075633e-25a0-4401-a683-690b7a8656d3.gif)

thank you [Kn1ghtNight](https://github.com/Kn1ghtNight) for making the original tutorial again, without you i wouldn't be able to import shaders to haxeflixel

