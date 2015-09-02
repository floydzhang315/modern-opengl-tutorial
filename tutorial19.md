#第十九课 镜面反射
##背景

当我们在计算环境光照的时候，唯一的要考虑的因素是光照强度；之后当我们计算到漫反射光照的时候，我们将光照的方向因素考虑进去了。镜面反射光包含这些因素，并且增加一个新的因素——观察者位置。这样做的原因是当光线以某一角度照射到一个平面上时，他会以相同的角度反射出去（在法线的另一边）。如果观察者刚好位于这条反射光线的线路上，那么他接受到的光照强度将大于远离这条线路的观察者。  

镜面反射光的效果就是当观察者以某一个角度看观察对象时会比其他角度亮，并且当你偏离这个角度时亮度会减弱。在真实世界中最好的关于镜面反射的例子就是金属物体了。这类物体有时候看上去不是其本来的颜色，而是一束闪亮的白光直射直射你的眼睛。然而这种在金属中十分常见的特性在其他很多材质中却很少出现（比如木头）。许多物体无论受到任何方向的光照都不会发生这样的反光，也无论观察者从哪个方向看。这表明镜面光的影响因素很大程度上依赖于物体而不是光照本身。  

接下来让我们看一看如何将观察者位置引入镜面反射光的计算之中。让我们看下面这幅图：

![](images\picture191.jpg)

这里我们需要注意5个地方：
	
1. ‘I’ 是照射到表面的入射光线（并且会产生漫反射光）；
2. ‘N’ 是表面法线；
3. ’R‘ 是从表面反射出去的光线。它以 ’I‘ 是关于法线相对称的，但是他们的总的方向是相反的（它朝上而不是朝下）；
4. ’V’ 是从表面入射点到眼睛（表示观察者）的向量； 
5. ‘α’ 是 ‘R’ 与 ‘V’ 之间的夹角；  
  

我们将会借助于 α 角来模拟镜面反射光的现象。我们计算镜面反射光的思路是，当观察者处于镜面光的反射路线上时（ ‘R’ 向量），其强度最大。在这种情况下，‘V’ 与 ‘R’ 是相同的，他们之间的角度为 0。当观察者远离 ‘R’ 时，他们之间的角度会变大，我们希望光照效果效果随着这个角度的变大而逐渐减弱。现在大家应该能够猜到我们接下来就需要使用点乘来计算 ‘α’ 角度的 cosine 值了吧。这个值将会在我们的光照计算公式中当做镜面反射光因子来使用。当 ‘α’ 为 0 的时候，其 cosine 值为 1，它是最大的镜面反射因子。随着 ‘α’ 角度的增加，其 cosine 值会逐渐减小，直到 ‘α’ 增加到 90 度其 cosine 值减小到 0，此时镜面光不会产生效果。当 ‘α’ 角度超过 90 度（或者 -90 度）时，其 cosine 值为负值，当然这种情况下镜面光也不会对结果产生影响。这意味着观察者完全不在反射光的线路上。  

为了计算出 ‘α’，我们需要使用 ‘R’ 和 ‘V’，‘V’ 可以通过用位于世界坐标系中的观察者的位置减去同样位于世界坐标系中的光入射点的位置得到。由于我们的相机本来就是在世界坐标系中定义的，所以我们只需要将其位置信息传入到 shader 中即可。上图是一个简化了的示意图，里面只有一个入射点。在现实世界中，实际上整个平面都会被光线照射到（假设这个平面是朝向光源的）。所以我们需要为每个像素点都计算出他们的镜面光效果（就如同我们在计算漫反射光时所做的那样），而且正因为如此我们需要得到每个像素点在世界坐标系中的位置。这也十分简单——我们可以将顶点变换到世界坐标系之下，之后让光栅器对世界坐标系中的位置进行插值，最后将片元着色器中的结果返回给我们。事实上，这与我们在之前的课程中处理法线时是一样的。  

现在，唯一剩下的事就是通过 ‘I’（由应用程序提供给shader）来计算出反射光线 ‘R’ 了。让我们看看下面这幅图片：  

![](images\picture192.jpg)

需要记住的是，向量没有起点，而且所有方向和模相同的向量都相等。因此，上图中的位于表面“下面”的向量 ‘I’ 是原始 ‘I’ 向量的拷贝，并且二者是一样的，现在我们的目标计算出向量 ‘R’ 。根据向量相加原则，向量 ‘R’ 等于 'I' + 'V'，‘I’ 是已知的，所以我们需要做的就是找出向量 ‘V’。注意法向量 ‘N’ 的负方向就是 ‘-N’，我们可以在 ‘I’ 和 ‘-N’ 之间使用一个点乘运算就能得到 ‘I’ 在 ‘-N’ 上面的投影的模。这个模正好是 ‘V’ 的模的一半，由于 ‘V’ 与 ‘N’ 有相同的方向，我们可以将这个模乘上 ‘N’ （其模为 1 ）再乘上 2 即可得到 ‘V’。总结一下就是下面的公式：  

![](images\picture193.jpg)  

现在你应该已经了解了上面的数学公式了，现在是时候告诉你一个小诀窍了—— GLSL 提供了一个专门用于上面的计算的内置函数 ‘reflect’。下面我们将会介绍在 shader 中如何使用它们。  

让我们总结一下镜面反射光的计算公式：  

![](images\picture194.jpg)

首先我们用灯光颜色乘上物体表面颜色，这一步与环境光和漫反射光的计算类似。得到的结果我们将其乘上材质的镜面反射强度（ ’M’ ）.一个没有镜面反射能力的材质（如木头材质）其镜面反射强度为 0，并且会使得整个等式的结果变为 0。像金属这样较为闪亮的东西会有一个较高的强度。之后我们再乘上反射光线和指向观察的向量之间夹角的余弦值的 ‘P’ 次方，‘P’ 被称作‘高光指数’或者‘亮度因子’。它的作用是锐化和加强反射光的镜面反射存在的区域的边缘，下面这幅图片展示了当我们把镜面指数设置为 1 时的情景：  

![](images\picture195.jpg)

而下面这幅图片展示了将高光指数设置为 32 时的情景：  

![](images\picture196.jpg)

高光指数也是材质的一个属性，所以不同的对象会有不同的高光指数。

##代码

```
(lighting_technique.h:32)
{
class LightingTechnique : public Technique
public:
...
    void SetEyeWorldPos(const Vector3f& EyeWorldPos);
    void SetMatSpecularIntensity(float Intensity);
    void SetMatSpecularPower(float Power);
private:
...
    GLuint m_eyeWorldPosLocation;
    GLuint m_matSpecularIntensityLocation;
    GLuint m_matSpecularPowerLocation;
```  

这里我们向 LightingTechnique 类中添加了三个新的属性——眼坐标（观察者）、镜面反射强度、高光指数。这三个属性都是与灯光本身无关的。这是因为当同样的光照射在两中不同的材质上面（比如金属与木质）时，他们都会有不一样的反光现象，我们当前对两种材质的模拟还有一定的限制性。所有属于一个绘制命令的三角形得到的值都一样，而当这些三角形分别代表模型中有不同材质性的部分时，这就比较麻烦了。当我们接触到网格加载这一节时，我们将会看到我们可以在建模软件中生成不同的高光值，并且让他们成为顶点缓冲器的一部分（而不是着色器的一个参数）。这使得我们可以在同一个绘制调用中处理有不同高光值的三角形。现在这个简单的方法可以产生我们要的效果了（作为练习，你可以试着将镜面强度和高光强度加到顶点缓冲器中，然后在着色器中进行处理）。

```
(lighting.vs:12)
out vec3 WorldPos0;
void main()
{
    gl_Position = gWVP * vec4(Position, 1.0);
    TexCoord0 = TexCoord;
    Normal0 = (gWorld * vec4(Normal, 0.0)).xyz;
    WorldPos0 = (gWorld * vec4(Position, 1.0)).xyz;
} 
```   

在上面的顶点着色器中，我们只增加了一行新的代码（最后一行）。我们在前面的课程中用于对法线进行变换的世界矩阵现在被用来将顶点的世界坐标传给片元着色器。这里我们会发现一个很有趣的现象，我们对同一顶点坐标（ Position ）进行了两次不同的矩阵变换，并且将结果分别传入到了片元着色器中。全矩阵变换（ world-view-projection matrix ）的结果被存放在系统变量 'gl_Position' 中，之后 GPU 会将其变换到屏幕坐标空间，用它进行实际的光栅化。而部分矩阵变换（仅仅变换到世界坐标系下）的结果被存放在用户定义的属性中，这个属性只用于光栅化阶段的插值，所以每个调用了片元着色器的像素使用的值都是其世界坐标值。这个技术十分常见而非常有用。   

```
(lighting.fs:5)
in vec3 WorldPos0;
uniform vec3 gEyeWorldPos;
uniform float gMatSpecularIntensity;
uniform float gSpecularPower;
void main()
{
    vec4 AmbientColor = vec4(gDirectionalLight.Color, 1.0f) * gDirectionalLight.AmbientIntensity;
    vec3 LightDirection = -gDirectionalLight.Direction;
    vec3 Normal = normalize(Normal0);
    float DiffuseFactor = dot(Normal, LightDirection);
    vec4 DiffuseColor = vec4(0, 0, 0, 0);
    vec4 SpecularColor = vec4(0, 0, 0, 0);
    if (DiffuseFactor > 0) {
        DiffuseColor = vec4(gDirectionalLight.Color, 1.0f) *
            gDirectionalLight.DiffuseIntensity *
            DiffuseFactor;
        vec3 VertexToEye = normalize(gEyeWorldPos - WorldPos0);
        vec3 LightReflect = normalize(reflect(gDirectionalLight.Direction, Normal));
        float SpecularFactor = dot(VertexToEye, LightReflect);
        SpecularFactor = pow(SpecularFactor, gSpecularPower);
    if (SpecularFactor > 0) {
        SpecularColor = vec4(gDirectionalLight.Color, 1.0f) * gMatSpecularIntensity * SpecularFactor;
        }
    }
    FragColor = texture2D(gSampler, TexCoord0.xy) * (AmbientColor + DiffuseColor + SpecularColor);
} 
```

在片元着色器中有一些变动，这里定义了三个新的一致变量用于保存计算镜面高光需要用到的属性（眼位置、镜面反射强度、高光指数）。环境光照的计算与前面两节中一样，之后我们创建了漫反射光和镜面反射光的颜色向量，并初始化为 0。只有当入射光线与平面的夹角小于 90 度时它们才有不同的值，否则为 0。这是通过 DiffuseFactor 判断的（与在漫反射光那一节中一样）。  

下面我们需要做的就是计算出从位于世界坐标系中的入射点到观察者位置（同样处于世界坐标系中）的向量。我们只需要用观察者坐标（观察者坐标是一个一致变量，它对于所有像素都是一样的）减去顶点坐标即可。同时我们要对此向量进行规范化，这是为之后的点乘运算做准备。在那之后我们通过内置函数 'reflect' 计算出反射光向量（你也可以按上面的推导公式来计算）。这个函数接收两个参数——入射光向量和表面法向量。这里需要注意的是我们使用的是原始的指向表面的入射光向量，而不是用于计算漫反射因子的反向量。这从上面的图中可以很明显的看出来。之后我们计算镜面反射因子（specular factor），其值为反射光线与从入射点到观察者的向量之间的夹角的 cosine 值（再次使用点积）。  

只有当这个角度小于 90 度的时候镜面反射的效果才是可见的，因此我们检查这个镜面反射因素的值是否大于 0。最后的高光颜色是通过将光照颜色、材质的镜面反射强度、镜面反射因素相乘得到的。我们将高光颜色、环境光颜色以及漫反射光颜色相加来得到总体的光照颜色。最后我们将这个值与从纹理中的取样值相乘，并将其结果作为像素的最终颜色。  

```
(tutorial19.cpp:134)
m_pEffect->SetEyeWorldPos(m_pGameCamera->GetPos());
m_pEffect->SetMatSpecularIntensity(1.0f);
m_pEffect->SetMatSpecularPower(32);
```

使用镜面反射颜色十分简单，在渲染循环中我们获取相机的位置（早已置于世界坐标系中）并且将其传递给 lighting technique 类，同时我们也设定高光强度和高光指数。剩下的就所有工作就交给 shader 完成了。  

尝试使用不同的高光指数和光照方向来看看他们的效果。你可能需要绕着物体转来找到一个可以看到镜面高光的位置。

##操作结果
![](images\picture197.jpg)
