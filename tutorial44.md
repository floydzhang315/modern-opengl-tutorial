# 第四十四课 GLFW

## 背景

在第一课中我们就已经说过 OpenGL 并不会直接处理 Windows 相关操作，这一部分功能都是由其他 API （如 GLX、WGL 等） 负责。为了简单起见我们使用了 GLUT 处理 Windows API 的调用，由于 GLUT 是一个跨平台的库，所以我们的程序也可以移植到不同的操作系统中。到目前为止我们的程序中都只使用了 GLUT 库，而现在我们将会介绍另外一个比较流行的库，这个库的功能和 GLUT 相似，它叫做 GLFW，并且托管在 www.glfw.org 上。这两个库之间的区别之一就在于 GLFW 更加先进而且有活力，但是 GLUT 则显得比较老旧，而且它的开发维护现在也基本上停滞了。 GLFW 有很多特性你可以在它的主页上进一步了解。  

由于这一课中没有什么数学背景，所有我们可以直接去参看代码即可。这里我做出的改变是对 glut\_backend.h 和 glut\_backend.cpp 中对 Windows 接口和键盘按键等事件的封装进行了抽象，抽象出了一个 backend API，通过这个 API 我们可以方便的在 GLUT 接口和 GLFW 接口之间转换，并且为我们之后的学习提供了很大的灵活性。  

## 代码

```
(ogldev_glfw_backend.cpp:24)
 #define GLFW_DLL
 #include
```

在这里包含 GLFW 头文件，要在 Windows 下使用 GLFW 的DLL 需要用到宏 'GLFW\_DLL' 。

```
(ogldev_glfw_backend.cpp:168)
void GLFWBackendInit(int argc, char** argv, bool WithDepth, bool WithStencil)
{
    sWithDepth = WithDepth;
    sWithStencil = WithStencil;
    if (glfwInit() != 1) {
        OGLDEV_ERROR("Error initializing GLFW");
        exit(1);
    }
    int Major, Minor, Rev;
    glfwGetVersion(&Major, &Minor, &Rev);
    printf("GLFW %d.%d.%d initialized\n", Major, Minor, Rev);
    glfwSetErrorCallback(GLFWErrorCallback);
}
```

GLFW 的初始化十分简单，需要注意的是参数 argc/argv 虽然并没有被使用，但是为了保证与 GLUT 接口的一致性，我们还是保留了他们。除此之外在 GLFW 的初始化中输出了 GLFW 库的版本信息并对初始化的错误进行了处理，如果出现了什么问题都会输出错误信息并终止程序。  

```
(ogldev_glfw_backend.cpp:195)
bool GLFWBackendCreateWindow(uint Width, uint Height, bool isFullScreen, const char* pTitle)
{
    GLFWmonitor* pMonitor = isFullScreen ? glfwGetPrimaryMonitor() : NULL;
    s_pWindow = glfwCreateWindow(Width, Height, pTitle, pMonitor, NULL);
    if (!s_pWindow) {
        OGLDEV_ERROR("error creating window");
        exit(1);
    }
    glfwMakeContextCurrent(s_pWindow);
    // Must be done after glfw is initialized!
    glewExperimental = GL_TRUE;
    GLenum res = glewInit();
    if (res != GLEW_OK) {
        OGLDEV_ERROR((const char*)glewGetErrorString(res));
        exit(1);
    } 
    return (s_pWindow != NULL);
}
```

在上面的函数中我们创建了窗口并做了一些窗口的初始化工作，glfwCreateWindow 函数的前三个参数的用处十分明显，第四个参数定义使用的显示设备，'GLFWmonitor' 是 GLFW 中的一个表示物理显示设备的对象，GLFW 支持多显示器，在多显示器的情况下 glfwGetMonitors 函数会返回一个包含所有可用显示器的列表。如果我们传递一个 NULL 进去，我们会创建一个普通的窗口；如果我们传递一个显示器对象指针进去（我们可以使用 glfwGetPrimaryMonitor 函数获得默认显示器）我们可以得到一个全屏窗口。第五个参数则是用于数据共享方面的操作，这个就超出了我们这一课的范围了。  

在我们使用 GL 命令之前我们需要将创建的窗口设置为当前窗口，我们使用 glfwMakeContextCurrent 函数来完成这个操作，最后我们初始化 GLFW。  

```
(ogldev_glfw_backend.cpp:238)
while (!glfwWindowShouldClose(s_pWindow)) { 
    // OpenGL API calls go here... 
    glfwSwapBuffers(s_pWindow); 
    glfwPollEvents(); 
}
```

与 GLUT 不同的是 GLFW 中并没有提供一个主渲染循环函数，因此我们使用上面的代码来构造这样一个循环并将其封装为 GLFWBackendRun() 函数的一部分，s\_pWindow 指向之前我们使用 glfwCreateWindow() 函数创建的窗口，为了使主程序能够终止这个循环，我们可以在主循环中调用 GLFWBackendLeaveMainLoop() 函数，在这个函数中封装了 glfwSetWindowShouldClose 函数的调用。  

```
(ogldev_glfw_backend.cpp:122)
static void KeyCallback(GLFWwindow* pWindow, int key, int scancode, int action, int mods)
{ 
}
static void CursorPosCallback(GLFWwindow* pWindow, double x, double y)
{
}
static void MouseCallback(GLFWwindow* pWindow, int Button, int Action, int Mode)
{
}
static void InitCallbacks()
{
    glfwSetKeyCallback(s_pWindow, KeyCallback);
    glfwSetCursorPosCallback(s_pWindow, CursorPosCallback);
    glfwSetMouseButtonCallback(s_pWindow, MouseCallback);
}
```

上面的函数是键盘事件和鼠标事件的响应函数，如果你想在你的程序中专门使用 GLFW，你可以参一些 GLFW 的文档了解下它里面按键、操作以及模式等的值。在我们的课程中我创建了一系列的枚举变量用于描述 GLFW 中的按键和鼠标事件。在 GLUT 中我也做了同样的处理，这使得程序能够更好的通用，也能够方便的从一套接口转到另一套接口中（如 GLUT 和 GLFW 之间的转换）。  

```
(ogldev_glfw_backend.cpp:)
void GLFWBackendTerminate()
{
    glfwDestroyWindow(s_pWindow);
    glfwTerminate();
}
```

这是 GLFW 资源释放部分，首先我们删除创建的窗口，之后我们终止 GLFW 库的使用并且释放其所有资源。在这些代码调用之后对 GLFW 的任何调用都不会起作用，这也是为什么我们是在主函数的 最后才调用这个函数。  

```
(ogldev_backend.h)
enum OGLDEV_BACKEND_TYPE {
    OGLDEV_BACKEND_TYPE_GLUT,
    OGLDEV_BACKEND_TYPE_GLFW
};
void OgldevBackendInit(OGLDEV_BACKEND_TYPE BackendType, int argc, char** argv, bool WithDepth, bool WithStencil);
void OgldevBackendTerminate();
bool OgldevBackendCreateWindow(uint Width, uint Height, bool isFullScreen, const char* pTitle);
void OgldevBackendRun(ICallbacks* pCallbacks);
void OgldevBackendLeaveMainLoop();
void OgldevBackendSwapBuffers();
```

我们已经创建了一套新的接口，即上面这个头文件。我们用这些接口替换了之前我们使用的 GLUT 中的特定代码，他们的实现都在 Common 项目的 ogldev\_backend.cpp 文件中，在实现的时候会才会最终调用 GLUT 或者 GLFW 的具体接口。你可以在 OgldevBackendInit()  函数中选择具体使用的接口。  

由于这一课中并没有什么新的东西呈现，所以我们使用了一个 Sponza 模型，这个模型在测试全局光照时经常用到。  
