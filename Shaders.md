#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.5.x (2017 April 14)
If this documentation doesn't cover some questions, please feel free to post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 


# Unity Shaders
[Shaders](https://en.wikipedia.org/wiki/Shader) are sets of instructions for the GPU regarding how to render objects and process data given to it.

In Unity, shaders are handled as [Shader Assets](https://docs.unity3d.com/Manual/class-Shader.html) (`.shader`  text files), and contain more information than what is typically considered a "shader" in other frameworks. 

Unity shaders are written in two languages:
![](/img/spine-runtimes-guide/spine-unity/shader_shaderlabcgparts.png)

**ShaderLab**
The [Unity ShaderLab language](https://docs.unity3d.com/Manual/SL-Shader.html) defines:
- ["material properties" you can set in the inspector](https://docs.unity3d.com/Manual/SL-Properties.html) like texture assets, numbers and colors
- and [Subshaders](https://docs.unity3d.com/Manual/SL-SubShaderTags.html) and [Passes](https://docs.unity3d.com/Manual/SL-Pass.html) that define, for [different](https://docs.unity3d.com/Manual/SL-PassTags.html) [situations](https://docs.unity3d.com/Manual/SL-SubShaderTags.html):
  - graphics API calls ([blend modes](https://docs.unity3d.com/Manual/SL-Blend.html), [depth](https://en.wikipedia.org/wiki/Z-buffering) settings, [stencil](https://en.wikipedia.org/wiki/Stencil_buffer) settings, [backface culling](https://en.wikipedia.org/wiki/Back-face_culling), etc.)
  - other pre-built Unity graphics instructions ([GrabPass](https://docs.unity3d.com/Manual/SL-GrabPass.html), [UsePass](https://docs.unity3d.com/Manual/SL-UsePass.html))
  - and wraps the actual code for (Cg) shader programs

**Cg**
[Cg](http://http.developer.nvidia.com/CgTutorial/cg_tutorial_chapter01.html) is the industry standard, lower-level shader language that determines what actual operations and calculations the GPU needs to do on pixels/fragments, vertices, lights and texture samplers and the like.

Cg is the same code you would see used in other game engines and frameworks.

# Unity Materials


In Unity, a shader for a renderer is chosen by setting the [Materials](https://docs.unity3d.com/Manual/Materials.html) on that renderer.

![](/img/spine-runtimes-guide/spine-unity/shader_partsofmaterial.png)

Unity Materials combine a **Unity Shader** with a set of **Material Property values**. For 3D meshes, this commonly means that a Material defines whether a flat square looks like a rocky ground, or a brick wall, or metal panels. But for 2D meshes acting as sprites, a Material stores where to find the image data (the main texture), and how to render it (the shader, and the various material property values).

# Unity Shaders for Spine-Unity
Spine-Unity comes with a default shader— `Spine/Skeleton`— which is used by the auto-ingester by default when it generates a Material for your Spine texture atlas.

`Spine/Skeleton` is fairly short and typical, and has the following characteristics:
- Premultiply Alpha blending
- No depth buffer reading or writing, no lighting
- no backface culling
- no fog
- Uses vertex colors to tint the texture via multiply.
- Has a "ShadowCaster" pass so it can cast shadows in Unity's system.
- Material Properties:
	- _MainTex "Main Texture"
	- _Cutoff "Shadow alpha cutoff"

Because it is a typical premultiply-alpha blended shader, you easily have the option of using other 2D shaders.
If you opt to use another shader, different characteristics could be accomodated for:

**Backface culling**
The only strict requirement for Spine is **disabling backface culling**, which is typical for 2D shaders.

> Most 3D shaders will have backface culling enabled. With Spine meshes, this will cause some triangles to become invisible when parts are scaled negatively or if you flip your skeleton direction.

**Depth and Lighting**
Not writing to the depth buffer is typical of an alpha-blended 2D sprite shader, as things are layered on top of each other primarily according to Unity's Renderer sorting scheme (see [Rendering](Rendering.md) for more details rather than the depth buffer. `Spine/Skeleton` shares this characteristic with Unity's own `Sprites/Default` shader.

> If you opt to use a shader that does use the depth buffer, you may want to add z-spacing to your SkeletonAnimation to prevent Z-Fighting, especially if you have lighting. You can do this by expanding the `Advanced...` foldout on your SkeletonAnimation's Inspector and sliding `Z Spacing` away from 0.
> Note that using the depth buffer may cause unwanted results around semitransparent areas, including antialiased edges.

**Vertex Colors**
You can set and animate colors in Spine. In Spine-Unity, this information is included in the generated mesh as **vertex colors**.
The renderer then uses the vertex colors as a multiply tint; simply, `texture color * vertex color` for all channels.

> Note that the SkeletonAnimation will have `PMA Vertex Colors` enabled by default. This makes it compatible with premultiply-alpha shaders. Leaving it enabled while using a straight alpha shader may cause color tints that are more transparent to become blacker than intended.
> Likewise, `PMA Vertex Colors` allows SkeletonAnimation to apply additive attachments by zeroing the submitted vertex color alpha for a certain slot.
> If you are not using a premultiply alpha shader, be sure to disable PMA Vertex Colors under the `Advanced...` foldout of SkeletonAnimation's inspector.

**Premultiply Alpha Blending**
Premultiply Alpha Blending ("PMA") is a type of alpha blending scheme that assumes that you have multiplied the alpha channel by the RGB values beforehand. PMA is the default but it not required for rendering Spine skeletons normally.

> Unity cannot detect PMA settings automatically. If you want to use a non-PMA shader (such as `Sprites/Default`), you must make sure the texture is saved without premultiply alpha, and you must make sure PMA Vertex Colors is disabled under the SkeletonAnimation's `Advanced...` foldout in the inspector.  

## Example Alternate Setup

**Using Unity's "Sprites/Default" shader**
The `Sprites/Default` shader has the following characteristics:
- Straight alpha blending (internal rendering PMA but requires a non-PMA texture)
- No depth
- No backface culling
- No lighting
- No shadow pass
- Uses vertex colors via multiply
- Material Properties (mostly hidden in the inspector):
	- _MainTex : Texture2D
	- _Color : Color
	- _PixelSnap : flag (Float)
	- _Flip : Vector4 (Vector)
	- _AlphaTex : Texture2D
	- _EnableExternalAlpha : flat (Float)

To make this setup work, you need to make sure to export your atlas from Spine Texture Packer with the **Premultiply Alpha checkbox unchecked**.
You also need to **disable PMA Vertex Colors** under the `Advanced...` foldout in SkeletonAnimation's inspector.

Unfortunately, despite having the Main Texture property, `Sprites/Default` does not show it in the inspector, so it has to be edited while another shader is chosen.


# Visual Shader Editors
If you are new to writing shaders, you can begin by using node-based visual shader editors. A few examples are [ShaderForge](https://www.assetstore.unity3d.com/en/#!/content/14147) and [Amplify Shader Editor](https://www.assetstore.unity3d.com/en/#!/content/68570) (still in beta at the time of writing).

These shader editors will generate Unity shader code text (a `.shader` file) that you can open, read, understand, edit and compare with changes you make in the nodes. (See [Unity ShaderLab](https://docs.unity3d.com/Manual/SL-Shader.html) and [Cg](http://http.developer.nvidia.com/CgTutorial/cg_tutorial_chapter01.html))
This means, even if you are experienced in writing shaders, that generated shaders can serve as a template or starting point for more complex, hand-coded shaders.

But remember that most of these editors are aimed at making 3D shaders. Not everything in shaders for 3D meshes will work for 2D sprites. You have to keep the characteristics mentioned above in mind for your shader to work correctly with Spine skeleton rendering and Unity sprites in general.

At the same time, these node-based shader editors are limited in what you can do with them. For example, multi-pass (extra non-lighting pass) shaders can be done by hand coding but neither Amplify nor ShaderForge allow this.

# Examples

A plain Skeleton rendering shader that has no features.
```ShaderLab
// - Requires PMA texture.
// - Unlit + no shadow
// - Premultiplied Alpha Blending (One OneMinusSrcAlpha)
// - Double-sided, no depth

Shader "Spine/Skeleton Plain" {
	Properties {
		[NoScaleOffset]_MainTex ("MainTex", 2D) = "white" {}
	}
	SubShader {
		Tags { "IgnoreProjector"="True" "Queue"="Transparent" "RenderType"="Transparent" "PreviewType"="Plane" }
		Blend One OneMinusSrcAlpha		//PMA
		Cull Off						//Double-sided
		ZWrite Off						//No depth
		Lighting Off					//No lighting

		Pass {
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"
			sampler2D _MainTex;

			struct VertexInput {
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
				float4 vertexColor : COLOR;
			};

			struct VertexOutput {
				float4 pos : SV_POSITION;
				float2 uv : TEXCOORD0;
				float4 vertexColor : COLOR;
			};

			VertexOutput vert (VertexInput v) {
				VertexOutput o;
				o.uv = v.uv;
				o.vertexColor = v.vertexColor;
				o.pos = UnityObjectToClipPos(v.vertex); // Unity 5.6
				return o;
			}

			float4 frag (VertexOutput i) : COLOR {
				float4 rawColor = tex2D(_MainTex,i.uv);
				float finalAlpha = rawColor.a * i.vertexColor.a;
				float3 finalColor = rawColor.rgb * i.vertexColor.rgb;
				return float4(finalColor, finalAlpha);
			}
			ENDCG
		}
	}
	FallBack "Diffuse"
}
```
 
Now here's the same shader, but with the ability to add a color overlay.
```ShaderLab
// - Requires PMA texture.
// - Unlit + no shadow
// - Premultiplied Alpha Blending (One OneMinusSrcAlpha)
// - Double-sided, no depth
// - Can render a sprite with a solid color overlay (_FillColor). Use _FillPhase to adjust color overlay amount. 

Shader "Spine/Skeleton Fill" {
	Properties {
		_FillColor ("FillColor", Color) = (1,1,1,1)
		_FillPhase ("FillPhase", Range(0, 1)) = 0
		[NoScaleOffset]_MainTex ("MainTex", 2D) = "white" {}
	}
	SubShader {
		Tags { "IgnoreProjector"="True" "Queue"="Transparent" "RenderType"="Transparent" "PreviewType"="Plane" }
		Blend One OneMinusSrcAlpha
		Cull Off
		ZWrite Off
		Lighting Off

		Pass {
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"
			sampler2D _MainTex;
			float4 _FillColor;
			float _FillPhase;

			struct VertexInput {
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
				float4 vertexColor : COLOR;
			};

			struct VertexOutput {
				float4 pos : SV_POSITION;
				float2 uv : TEXCOORD0;
				float4 vertexColor : COLOR;
			};

			VertexOutput vert (VertexInput v) {
				VertexOutput o = (VertexOutput)0;
				o.uv = v.uv;
				o.vertexColor = v.vertexColor;
				o.pos = UnityObjectToClipPos(v.vertex); // Unity 5.6
				return o;
			}

			float4 frag (VertexOutput i) : COLOR {
				float4 rawColor = tex2D(_MainTex,i.uv);
				float finalAlpha = (rawColor.a * i.vertexColor.a);
				float3 finalColor = lerp((rawColor.rgb * i.vertexColor.rgb), (_FillColor.rgb * finalAlpha), _FillPhase); // make sure to PMA _FillColor.
				return fixed4(finalColor, finalAlpha);
			}
			ENDCG
		}
	}
	FallBack "Diffuse"
}
```

A PMA Multiply Shader
```ShaderLab
// Spine/Skeleton PMA Multiply
// - Requires PMA texture.
// - single color multiply tint
// - unlit
// - Premultiplied alpha Multiply blending
// - No depth, no backface culling, no fog.

Shader "Spine/Skeleton PMA Multiply" {
	Properties {
		_Color ("Tint Color", Color) = (1,1,1,1)
		[NoScaleOffset] _MainTex ("MainTex", 2D) = "black" {}
	}

	SubShader {
		Tags {
			"Queue"="Transparent"
			"IgnoreProjector"="True"
			"RenderType"="Transparent"
		}

		Fog { Mode Off }
		Cull Off
		ZWrite Off
		Blend DstColor OneMinusSrcAlpha
		Lighting Off

		Pass {
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"

			sampler2D _MainTex;
			float4 _Color;

			struct VertexInput {
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
				float4 vertexColor : COLOR;
			};

			struct VertexOutput {
				float4 pos : SV_POSITION;
				float2 uv : TEXCOORD0;
				float4 vertexColor : COLOR;
			};

			VertexOutput vert (VertexInput v) {
				VertexOutput o;
				o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
				o.uv = v.uv;
				o.vertexColor = v.vertexColor * float4(_Color.rgb * _Color.a, _Color.a); // Combine a PMA version of _Color with vertexColor.
				return o;
			}

			float4 frag (VertexOutput i) : COLOR {
				float4 texColor = tex2D(_MainTex, i.uv);
				return (texColor * i.vertexColor);
			}
			ENDCG
		}
	}
}

```


Here is `Spine/Effects/Skeleton Cloak`, a 2D depth-writing refraction shader, made using ShaderForge 1.36.
```ShaderLab
// Spine/Effects/Skeleton Cloak
// - Opacity Clip
// - 

// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

// Shader created with Shader Forge v1.36 
// Shader Forge (c) Neat Corporation / Joachim Holmer - http://www.acegikmo.com/shaderforge/
// Note: Manually altering this data may prevent you from opening it in Shader Forge
/*SF_DATA;ver:1.36;sub:START;pass:START;ps:flbk:,iptp:1,cusa:True,bamd:0,cgin:,lico:1,lgpr:1,limd:0,spmd:1,trmd:0,grmd:0,uamb:True,mssp:True,bkdf:False,hqlp:False,rprd:False,enco:False,rmgx:True,imps:True,rpth:0,vtps:0,hqsc:True,nrmq:1,nrsp:0,vomd:0,spxs:True,tesm:0,olmd:1,culm:2,bsrc:0,bdst:1,dpts:2,wrdp:False,dith:0,atcv:False,rfrpo:False,rfrpn:Refraction,coma:15,ufog:False,aust:False,igpj:True,qofs:50,qpre:3,rntp:3,fgom:False,fgoc:False,fgod:False,fgor:False,fgmd:0,fgcr:0.5,fgcg:0.5,fgcb:0.5,fgca:1,fgde:0.01,fgrn:0,fgrf:300,stcl:False,stva:128,stmr:255,stmw:255,stcp:6,stps:0,stfa:0,stfz:0,ofsf:0,ofsu:0,f2p0:False,fnsp:True,fnfb:False,fsmp:False;n:type:ShaderForge.SFN_Final,id:1873,x:33229,y:32719,varname:node_1873,prsc:2|emission-9115-RGB,clip-603-OUT,refract-1041-OUT;n:type:ShaderForge.SFN_Tex2d,id:4805,x:32100,y:32513,ptovrint:False,ptlb:MainTex,ptin:_MainTex,varname:_MainTex,prsc:2,glob:False,taghide:False,taghdr:False,tagprd:False,tagnsco:True,tagnrm:False,tex:49bb65eefe08e424bbf7a38bc98ec638,ntxv:1,isnm:False;n:type:ShaderForge.SFN_VertexColor,id:5376,x:32100,y:32723,varname:node_5376,prsc:2;n:type:ShaderForge.SFN_Multiply,id:603,x:32695,y:32820,cmnt:A,varname:ca,prsc:2|A-4805-A,B-5376-A;n:type:ShaderForge.SFN_Append,id:2501,x:32681,y:33116,varname:Refraction2D,prsc:2|A-3476-OUT,B-3476-OUT;n:type:ShaderForge.SFN_Slider,id:3476,x:32228,y:33306,ptovrint:False,ptlb:Refraction Strength,ptin:_RefractionStrength,varname:_RefractionStrength,prsc:2,glob:False,taghide:False,taghdr:False,tagprd:False,tagnsco:False,tagnrm:False,min:0,cur:0.01,max:0.05;n:type:ShaderForge.SFN_Multiply,id:1041,x:32936,y:33216,varname:node_1041,prsc:2|A-7637-OUT,B-2501-OUT;n:type:ShaderForge.SFN_Append,id:7637,x:32432,y:32888,varname:node_7637,prsc:2|A-4805-R,B-4805-G;n:type:ShaderForge.SFN_SceneColor,id:9115,x:32815,y:32653,varname:node_9115,prsc:2;proporder:4805-3476;pass:END;sub:END;*/

Shader "Spine/Effects/Skeleton Cloak" {
    Properties {
        [NoScaleOffset]_MainTex ("MainTex", 2D) = "gray" {}
        _RefractionStrength ("Refraction Strength", Range(0, 0.05)) = 0.01
        [HideInInspector]_Cutoff ("Alpha cutoff", Range(0,1)) = 0.5
        [MaterialToggle] PixelSnap ("Pixel snap", Float) = 0
    }
    SubShader {
        Tags {
            "IgnoreProjector"="True"
            "Queue"="Transparent+50"
            "RenderType"="TransparentCutout"
            "CanUseSpriteAtlas"="True"
            "PreviewType"="Plane"
        }
        GrabPass{ "Refraction" }
        Pass {
            Name "FORWARD"
            Tags {
                "LightMode"="ForwardBase"
            }
            Cull Off
            ZWrite Off
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #define UNITY_PASS_FORWARDBASE
            #pragma multi_compile _ PIXELSNAP_ON
            #include "UnityCG.cginc"
            #pragma multi_compile_fwdbase
            #pragma target 3.0
            uniform sampler2D Refraction;
            uniform sampler2D _MainTex;
            uniform float _RefractionStrength;
            struct VertexInput {
                float4 vertex : POSITION;
                float2 texcoord0 : TEXCOORD0;
                float4 vertexColor : COLOR;
            };
            struct VertexOutput {
                float4 pos : SV_POSITION;
                float2 uv0 : TEXCOORD0;
                float4 screenPos : TEXCOORD1;
                float4 vertexColor : COLOR;
            };
            VertexOutput vert (VertexInput v) {
                VertexOutput o = (VertexOutput)0;
                o.uv0 = v.texcoord0;
                o.vertexColor = v.vertexColor;
                o.pos = UnityObjectToClipPos(v.vertex );
                #ifdef PIXELSNAP_ON
                    o.pos = UnityPixelSnap(o.pos);
                #endif
                o.screenPos = o.pos;
                return o;
            }
            float4 frag(VertexOutput i, float facing : VFACE) : COLOR {
                float isFrontFace = ( facing >= 0 ? 1 : 0 );
                float faceSign = ( facing >= 0 ? 1 : -1 );
                #if UNITY_UV_STARTS_AT_TOP
                    float grabSign = -_ProjectionParams.x;
                #else
                    float grabSign = _ProjectionParams.x;
                #endif
                i.screenPos = float4( i.screenPos.xy / i.screenPos.w, 0, 0 );
                i.screenPos.y *= _ProjectionParams.x;
                float4 _MainTex_var = tex2D(_MainTex,i.uv0);
                float2 sceneUVs = float2(1,grabSign)*i.screenPos.xy*0.5+0.5 + (float2(_MainTex_var.r,_MainTex_var.g)*float2(_RefractionStrength,_RefractionStrength));
                float4 sceneColor = tex2D(Refraction, sceneUVs);
                clip((_MainTex_var.a*i.vertexColor.a) - 0.5);
////// Lighting:
////// Emissive:
                float3 emissive = sceneColor.rgb;
                float3 finalColor = emissive;
                return fixed4(lerp(sceneColor.rgb, finalColor,1),1);
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
    CustomEditor "ShaderForgeMaterialInspector"
}

```

# Related Links
- [Rendering](https://github.com/pharan/spine-unity-docs/blob/master/Rendering.md)
  - 