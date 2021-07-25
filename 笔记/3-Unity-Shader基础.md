Shader其实就是渲染流水线中的某些特定阶段，如顶点着色器阶段、片元着色器阶段。

unity的出现改善了Shader的状况，它提供了一个地方能够让开发者更加轻松地管理着色器代码以及渲染设置（如开启/关闭混合、深度测试、设置渲染顺序等），而不需要管理多个文件和函数。

Unity提供这个“方便的地方”就是Unity Shader。



# Unity Shader 概述

## 一对好兄弟：材质和Unity Shader

在Unity中我们需要使用**材质（Material）**和**Unity Shader**才能达到需要的效果。

【流程】

1. 创建一个材质；
2. 创建一个Unity Shader，并把它赋给上一步中创建的材质；
3. 把材质赋给要渲染的对象；
4. 在材质面板中调整Unity Shader的属性，以得到满意的结果。

Unity Shader定义了渲染所需的各种代码（如顶点着色器和片元着色器）、属性（如使用哪些纹理）和指令（渲染和标签设置等），而材质则允许我们调节这些属性，并将其最终赋给某些模型。

## Unity中的材质

Unity中的材质需要结合一个GameObject的**Mesh**或者**Particle Systems**组件来工作。

创建一个新的材质：Assets->Create->Material

可以通过把材质直接拖曳到**Scene**视图中的对象上来实现，或者在该对象的**Mesh Renderer**组件中直接赋值。

## Unity中的Shader

创建Unity Shader：Assets->Create->Shader

Unity一共提供了4种Unity Shader模板：

- Standard Surface Shader（产生一个包含了标准光照模型的表面着色器模板）
- Unlit shader（产生一个不包含光照（但包含雾效）的基本的顶点/片元着色器）
- Image Effect Shader（为我们实现各种屏幕后处理效果提供了一个基本模板）
- Compute Shader（产生一种特殊的Shader文件，意在利用GPU的并行性来进行一些与常规渲染流水线无关的计算）

Unity Shader必须与材质结合起来才能发挥作用。

Unity Shader本质上是一个文本文件，和Unity中的很多外部文件类似，Unity Shader也有**导入设置（Import Settings）**面板，在**Project**视图中选中某个Unity Shader即可看到。

在导入设置面板上：

- **Default Maps：**指定该Unity Shader使用默认纹理，当任何材质第一次使用该Unity Shader时，这些纹理就会自动被赋予到相应的属性上。
- **Surface shader：**是否是一个表面着色器
- **Fixed function：**是否是一个固定函数着色器。

还有一些信息时和我们在Unity Shader中的标签设置有关，例如：是否会投射阴影、使用渲染队列、LOD值等。

导入面板还可以方便地查看其使用的渲染队列（Render queue）、是否关闭批处理（Disable batching）、属性列表（Properties）等信息。



# Unity Shader基础：ShaderLab

>计算机科学中的任何问题都可以通过增加一层抽象来解决。——大卫·惠勒

通常情况下，为了自定义渲染效果往往需要和很多文件和设置打交道，这些过程很容易消磨掉初学者的耐心。同时一些细节问题也往往需要开发者花费较多的时间去解决。

Unity为了解决上述问题，提供了一层抽象——Unity Shader。

我们和这层抽象打交道的途径就是使用Unity提供的一种专门为Unity Shader服务的语言——ShaderLab。

## ShaderLab

>ShaderLab is a friend you can afford.——Nicholas Francis

Unity Shader是Unity为开发者提供的高层级的渲染抽象层。

- 没有使用Unity Shader（左图）：开发者需要和很多文件和设置打交道，才能让画面呈现出想要的效果。
- 使用Unity Shader（右图）：开发者只需要使用ShaderLab来编写Unity Shader文件就可以完成所有的工作。

<img src="https://huris.oss-cn-hangzhou.aliyuncs.com/blog/%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9B%BE%E5%BD%A2%E5%AD%A6/Unity%20Shader%E5%9F%BA%E7%A1%80/ShaderLab%E4%BD%BF%E7%94%A8%E6%AF%94%E8%BE%83.png">

在Unity中，所有的Unity Shader都是用ShaderLab来编写的。

ShaderLab是Unity提供的编写Unity Shader的一种说明性语言，它使用了一些嵌套在花括号内部的**语义（syntax）**来描述一个Unity Shader文件的结构。

这些结构包含了许多渲染所需的数据，例如`Properties`语句块中定义了着色器所需的各种属性，这些属性将会出现在材质面板中。

从设计上来说，**ShaderLab类似于CgFX和Direct3D Effects（.FX）语言**，它们都定义了要显示一个材质所需的所有东西，而**不仅仅是着色器代码**。

Unity Shader的基础结构如下：

```c++
Shader "ShaderName"{
    Properties{
        // 属性   
    }
    SubShader{
        // 显卡A使用的子着色器
    }
    SubShader{
        // 显卡B使用的子着色器
    }
    Fallback "VertexLit"
}
```

Unity在背后会**根据使用的平台来把这些结构编译成真正的代码和Shader文件**，而开发者只需要和Unity Shader打交道即可。



# Unity Shader的结构

## 给Shader起个名字

每个Unity Shader文件的第一行都需要通过`Shader`语义来指定该**Unity Shader**的名字（由字符串定义）。

当为材质选择使用的Unity Shader时，这些名称就会出现在材质面板的下拉列表里。

通过在字符串中添加斜杠（"/"），可以控制Unity Shader在材质面板中出现的位置。

```c++
Shader "Custom/MyShader"
```

则这个Unity Shader在材质面板中的位置就是：Shader->Custom->MyShader。

## 材质和Unity Shader的桥梁：Properties

`Properties`语义块中包含了一系列**属性（Property）**，这些属性将会出现在材质面板中。

`Properties`语义块的定义通常如下：

```c++
Properties{
    Name {"display name", PropertyType} = DefaultValues
    Name {"display name", PropertyType} = DefaultValues
    // 更多属性
}
```

声明这些属性是为了在材质面板中能够方便地调整各种材质属性。

如果需要在Shader中访问它们，则需要使用每个属性的**名字（Name）**，在Unity中，这些属性的名字通常由一个下划线开始。

**显示的名称（display name）**则是出现在材质面板上的名字。

我们需要为每个属性指定它的**类型（PropertyType）**，常见的类型如下表所示：

|    属性类型    |       默认值的定义语法        |                例子                |
| :------------: | :---------------------------: | :--------------------------------: |
|      Int       |            number             |         _Int("Int",Int)=2          |
|     Float      |            number             |     _Float("Float",Float)=1.5      |
| Range(min,max) |            number             | _Range("Range",Range(0.0,5.0))=3.0 |
|     Color      | (number,number,number,number) |       _Color("Color",Color)        |
|     Vector     | (number,number,number,number) | _Vector("Vector",Vector)=(2,3,6,1) |
|       2D       |      "defaulttexture"{}       |         _2D("2D",2D)=""{}          |
|      Cube      |      "defaulttexture"{}       |    _Cube("Cube",Cube)="white"{}    |
|       3D       |      "defaulttexture"{}       |       _3D("3D",3D)="black"{}       |

- **Int**、**Float**、**Range**这些数字类型的属性，默认是一个单独的数字；
- **Color**和**Vector**这类属性，默认值是用圆括号包围的一个四维向量；
- **2D**、**Cube**、**3D**这3种纹理类型，默认值是通过一个字符串后跟一个花括号来指定的，其中，字符串要么是空的，要么是内置的纹理名称，如“white”、“black”、“gray”或者“bump”。

展示所有属性类型的例子：

``` c++
Shader "Custom/ShaderLabProperties"{
    Properties{
        // Numbers and Sliders
        _Int("Int", Int) = 2
        _Float("Float", Float) = 1.5
        _Range("Range", Range(0.0, 5.0)) = 3.0
        
        // Colors and Vectors
        _Color("Color", Color) = (1,1,1,1)
        _Vector("Vector", Vector) = (2,3,6,1)
        
        // Textures
        _2D("2D", 2D) = "" {}
        _Cube("Cube", Cube) = "white" {}
        _3D("3D", 3D) = "black" {}
    }
    FallBack "Diffuse"
}
```

有时想在材质面板上显示更多类型的变量，例如使用布尔变量来控制Shader中使用哪种计算。

Unity允许我们重建默认的材质编辑面板，以提供更多自定义的数据类型。

## 重量级成员：SubShader

每一个Unity Shader文件可以包含多个SubShader语义块，但最少要有一个。

当Unity需要加载这个Unity Shader时，Unity会扫描所有的SubShader语义块，然后选择第一个能够在目标平台上运行的SubShader。

如果都不支持的话，Unity就会使用`Fallback`语义指定的Unity Shader。

SubShader语义块中包含的定义如下：

```c++
SubShader{
    // 可选的
    [tag]
    
    // 可选的
    [RenderSetup]
    
    Pass{
    
    }
    // Other Passes
}
```

SubShader定义了一系列Pass以及可选的**状态（RenderSetup）**和**标签（Tags）**。

每个Pass定义了一次完整的渲染流程，如果Pass数目过多，往往会造成渲染性能下降。（尽量使用最小数目的Pass）

可以在Pass中设置RenderSetup和Tags，如果在Pass外设置，则会用于所有的Pass。

### 状态设置（RenderSetup）

ShaderLab提供了一系列渲染状态的设置指令，这些指令可以设置显卡的各种状态（例如是否开启混合/深度测试等）。

| 状态名称 |                          设置指令                           |                 解释                 |
| :------: | :---------------------------------------------------------: | :----------------------------------: |
|   Cull   |                    Cull Back\|Front\|Off                    | 设置剔除模式：剔除背面/正面/关闭剔除 |
|  ZTest   | ZTest Less Greater\|LEqual\|GEqual\|Equal\|NotEqual\|Always |       设置深度测试时使用的函数       |
|  ZWrite  |                       ZWrite On\|Off                        |          开启/关闭深度写入           |
|  Blend   |                  Blend SrcFactor DstFactor                  |          开启并设置混合模式          |

### 标签（Tags）

SubShader的标签（Tags）是一个**键值对（Key/Value Pair）**，它的键和值都是字符串类型。

这些键值对是SubShader和渲染引擎之间的沟通桥梁，它们用来告诉Unity的渲染引擎：SubShader我希望怎样以及何时渲染这个对象。

标签的结构如下：

```C++
Tags{"TagName1"="Value1" "TagName2"="Value2"}
```

|       标签类型       |                             说明                             |                例子                 |
| :------------------: | :----------------------------------------------------------: | :---------------------------------: |
|        Queue         | 控制渲染顺序，指定该物体属于哪一个渲染队列，通过这种方式可以保证所有的透明物体可以在所有不透明物体后面被渲染，也可以自定义使用的渲染队列来控制物体的渲染顺序。 |     Tags{"Queue"="Transparent"}     |
|      RenderType      | 对着色器进行分类，例如是否为透明着色器，可以被用于着色器替换（Shader Replacement）功能。 |     Tags{"RenderType"="Opaque"}     |
|   DisableBatching    |    可以通过该标签来直接指明是否对该SubShader使用批处理。     |   Tags{"DisableBatching"="True"}    |
| ForceNoShadowCasting |          控制使用该SubShader的物体是否会投射阴影。           | Tags{"ForceNoShadowCasting"="True"} |
|   IgnoreProjector    | 若该标签值为“True”，则使用该SubShader的物体将不会受Projector影响。通常用于半透明物体 |   Tags{"IgnoreProjector"="True"}    |
|  CanUseSpriteAtlas   |     当该SubShader是用于sprites时，将该标签设置为“False”      |  Tags{"CanUseSpriteAtlas"="False"}  |
|     PreviewType      | 指明材质面板将如何预览该材质。默认情况下，材质将显示为一个球形，可以通过把该标签的值设为“Plane”“SkyBox”来改变预览类型 |     Tags{"PreviewType"="Plane"}     |

注意：上述标签仅可以在SubShader中声明，而不可以在Pass块中声明。

### Pass语义块

Pass语义块包含的语义如下：

```c++
Pass{
    [Name]
    [Tags]
    [RenderSetup]
    // Other code
}
```

可以在Pass中定义该Pass的名称，例如

```c++
Name "MyPassName"
```

之后，可以使用ShaderLab的UsePass命令来直接使用其他Unity Shader中的Pass，例如：

```c++
UsePass "MyShader/MYPASSNAME"
```

注意，Unity内部会把所有Pass的名称转换成大写字母的表示，因此，在使用UsePass命令时必须使用大写形式的名字。

Pass标签：

|    标签类型    |                             说明                             |                  例子                   |
| :------------: | :----------------------------------------------------------: | :-------------------------------------: |
|   LightMode    |            定义该Pass在Unity的渲染流水线中的角色             |     Tags{"LightMode"="ForwardBase"}     |
| RequireOptions | 用于指定当满足某些条件时才渲染该Pass，它的值是一个由空格分隔的字符串。目前，Unity支持的选项有：SoftVegetation。后续版本会增加更多的选项。 | Tags{"RequireOptions"="SoftVegetation"} |

## Fallback（留一条后路）

Fallback在各个SubShader语义块后面，它用于告诉Unity，“如果上面所有的SubShader在这块显卡上都不能运行，那么就使用这个最低级的Shader吧！”

语义定义：

```c++
Fallback "name"
// 或者
Fallback Off
```

可以给Fallback起名字，也可以关闭Fallback功能。



# Unity Shader的形式

Unity Shader最重要的任务是指定各种着色器所需的代码。

这些着色器代码可以写在SubShader语义块中（表面着色器的做法），也可以写在Pass语义块中（顶点/片元着色器和固定函数着色器的做法）。

```c++
Shader "MyShader"{
    Properties{
        // 所需的各种属性
    }
    SubShader{
        // 真正意义上的Shader代码会出现在这里
        // 表面着色器（Surface Shader）或者
        // 顶点/片元着色器（Vertex/Fragment Shader）或者
        // 固定函数着色器（Fixed Function Shader）
    }
    SubShader{
        // 和上一个SubShader类似
    }
}
```

## 表面着色器（Surface Shader）

Unity自创的一种着色器代码类型，需要的代码量很少，Unity在背后做了很多工作（背后仍旧把它转换成对应的顶点/片元着色器），但渲染的代价比较大。

```c++
Shader "Custom/Simple Surface Shader"{
    SubShader {
        Tags{"RenderType" = "Opaque"}
        CGPROGRAM
        #pragma surface surf Lambert
        struct Input{
            float4 color : COLOR;
        };
        void surf(Input IN, inout SurfaceOutput o){
            o.Albedo = 1;
        }
        ENDCG
    }
    Fallback "Diffuse"
}
```

表面着色器定义在SubShader语义块（而非Pass语义块）中的**CGPROGRAM**和**ENDCG**之间。

原因：表面着色器不需要开发者关心使用多少个Pass、每个Pass如何渲染等问题，Unity会在背后为我们做好这些事情。

**CGPROGRAM**和**ENDCG**之间的代码是使用**CG/HLSL**编写的，即，我们需要把CG/GLSL语言嵌套在ShaderLab语言中。

如果想和各种光源打交道，使用表面着色器，但需要小心它在移动平台的性能表现。

## 顶点/片元着色器（Vertex/Fragment Shader）

Unity中可使用CG/HLSL语言来编写顶点/片元着色器。

```c++
Shader "Custom/Simple VertexFragment Shader"{
    SubShader {
        Pass{
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            
            float4 vert(float4 v: POSITION): SV_POSITION{
                return mul(UNITY_MATRIX_MVP, v);
            }
            
            fixed4 frag(): SV_Target{
                return fixed4(1.0, 0.0, 0.0, 1.0);
            }
            ENDCG
        }
    }
}
```

顶点/片元着色器的代码也需要定义在**CGPROGRAM**和**ENDCG**之间。

但需要注意，顶点/片元着色器时写在Pass语义块内，而非SubShader内的。

如果有很多自定义的渲染效果，则使用顶点/片元着色器。

## 固定函数着色器（Fixed Function Shader）

以上两种Unity Shader形式都使用了**可编程管线**。

由于一些旧设备不支持可编程管线着色器，因此，需要使用**固定函数着色器**来渲染，但这些着色器往往只能完成一些非常简单的效果。

```c++
Shader "Tutorial/Basic" {
    Properties {
        _Color ("Main Color", Color) = (1, 0.5, 0.5, 1)
    }
    SubShader {
        Pass {
            Material {
                Diffuse [_Color]
            }
            Lighting On
        }
    }
}
```

固定函数着色器的代码被**定义在Pass语义块**中，这些代码相当于Pass中的一些渲染设置。

对于固定函数着色器来说，需要**完全使用ShaderLab的语法**（使用ShaderLab的渲染设置命令）来编写，而非使用CG/HLSL。

## 选择哪种Unity Shader形式

- 除非必要（例如在一些旧设备上运行），否则不选固定函数着色器。
- 如果跟各种光源打交道，则使用**表面着色器**，但要小心它在移动平台的性能表现。
- 如果光源较少，例如只有一个平行光，则使用**顶点/片元着色器**（有很多自定义渲染效果，也使用顶点着色器）。



# 疑惑

## Unity Shader != 真正的Shader

Unity Shader指一个**ShaderLab文件**——硬盘上以`.shader`作为文件后缀的一种文件。

Unity Shader（或者说ShaderLab文件）里，我们可以做的事情远多于一个传统意义上的Shader。

- 传统Shader中，只能编写**特定类型的Shader**，例如顶点着色器、片元着色器等。而Unity Shader中，可以在同一文件夹里，同时包含需要的顶点着色器和片元着色器代码。
- 传统的Shader中，**无法设置一些渲染设置**（例如是否开启混合、深度测试等，这些只能由开发者在另外的代码中自行设置）。而Unity Shader中，我们通过一行特定的指令就可以完成这些设置
- 传统Shader中，需要编写冗长的代码来**设置着色器的输入和输出**，小心处理这些输入输出的位置对应关系等。而Unity Shader中，只需要在特定语句块中声明一些属性，就可以依靠材质来方便地改变这些属性。对于模型自带数据（如顶点位置、纹理坐标、法线等），Unity Shader也提供了直接访问的方法，不需要开发者自行编码来传给着色器。

**Unity Shader缺点：**

- 由于Unity Shader的高度封装性，可以编写的Shader类型和语法都被限制了。
- 对于一些类型的Shader，例如曲面细分着色器（Tessellation Shader）、几何着色器（Geometry Shader），Unity的支持就相对差一些。
- 一些高级Shader语法Unity Shader也不支持。

Unity Shader提供了一种**让开发者同时控制渲染流水线中多个阶段的一种方式**，不仅仅是提供Shader代码。

对开发者而已，我们绝大多数时候**只需要和Unity Shader打交道**，而**不需要关心渲染引擎底层的实现细节**。

## Unity Shader和CG/HLSL之间的关系

Unity Shader是用ShdaerLab语言编写的，但对表面着色器和顶点/片元着色器，可以在ShaderLab内部嵌套CS/HLSL语言来编写这些着色器代码，这些CG/HLSL代码嵌套在CGPROGRAM和ENDCG之间。

由于CG和DX9风格的HLSL从写法上几乎是同一种语言，因此在Unity里CG和HLSL是等价的。

通常，CG代码片段是位于Pass语义块内部，

```c++
Pass {
    // Pass的标签和状态设置
    
    CGPROGRAM
    // 编译指令，如:
    #pragma vertex vert
    #pragma fragment frag
    
    // CG 代码
    
    ENDCG
    // 其他一些设置
}
```

## 可以使用GLSL来写吗？

如果用GLSL来写，只能发布到Mac OS X、OpenGL ES 2.0 或者Linux，对于PC、xbox 360这样的仅支持DirectX的平台来说，就要放弃它们了。

和CG/HLSL需要嵌套在CGPROGRAM和ENGCG之间类似，GLSL的代码需要嵌套在GLSLPROGRAM和ENGGLSL之间。