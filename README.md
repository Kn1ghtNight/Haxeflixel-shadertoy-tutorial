# Haxeflixel and Shadertoy
A small and stinky tutorial on how to hardcode shaders!!!!

**(NOTE: I do not have a lot of experience with glsl, so some information might be wrong.)**

## STEP 1. Find a Shader.

We are going to use shadertoy for this. So first, open your web browser and go to [Shadertoy](https://shadertoy.com).

As you can see we have a few shaders on our screen.

![image](https://user-images.githubusercontent.com/89167668/226473399-53835dd7-4f62-43f3-92d4-2587c3498642.png)

We aren't going to click on any of them but instead go to the search bar. Let's search VHS for example.

![image](https://user-images.githubusercontent.com/89167668/226475082-c7ab672e-1081-4ec1-9788-7f2e0db00b4d.png)

Lets choose this one right here. "20151110_VHS by FMS_Cat"

![image](https://user-images.githubusercontent.com/89167668/226475188-19653fda-2222-43da-b63d-53880d8e1eba.png)

## STEP 2. Convert it to a HaxeFlixel shader.

As you can see we have our shader on the left, and the code for it on the right.

![image](https://user-images.githubusercontent.com/89167668/226476895-1b709d02-1fc2-4557-80c5-b9a31f21eb87.png)

We now make a file called VhsShader.hx for example.

You can use this shader class template as an example.

```haxe
import flixel.system.FlxAssets.FlxShader;

class VhsShader extends FlxShader//https://www.shadertoy.com/view/XtBXDt
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

When converting our shader we need to modify a few things.

We need to delete the parameters in mainImage.

It should now look like this.

![image](https://user-images.githubusercontent.com/89167668/226479920-cd47645a-2aa6-478c-bf8a-d87498da83a3.png)

We are going to pres Ctrl + A to select all of it and copy it to our clipboard.

Our shader code is pretty much done, all we need to do is paste this in to our shader class.

It should look like this now.

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

## STEP 3: Adding the shader.
Now in PlayState.hx (or whatever class you are applying the shader in), put:

```haxe
import openfl.filters.ShaderFilter;
import VhsShader;
```

Near the imports.

In our vars, put:

```haxe
	var shader:VhsShader;
```

then in our stage, or function create, put:

```haxe
			shader = new VhsShader();
			camGame.setFilters([new ShaderFilter(shader)]);
```

Now if we recompile and play, our shader is on camGame!

![image](https://user-images.githubusercontent.com/89167668/226491010-ba29e8a9-0a17-4ad2-9c64-82196c7fe95f.png)

Now you might notice, our shader is not animated.

in function update, put:

```haxe
shader.update(elapsed);
```

And now our shader is animated!!!

![2023-03-20 20-05-33](https://user-images.githubusercontent.com/89167668/226491713-a075633e-25a0-4401-a683-690b7a8656d3.gif)

ey thanke for reading this tutorial i had to be not silly to write all of this!!
