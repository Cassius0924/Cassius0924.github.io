# HoloLens 应用程序渲染不出 Mesh 面片的解决方法

## 问题

当项目在 Unity 测试 Mesh 面片渲染正常，而打包部署到 HoloLens 后就无法正常显示 Mesh 面片了。

失败原因可能出在 Holo 不支持程序代码所用的 Shader 着色器，需要使用 MRTK3 官方提供的 Shader 着色器。

## 解决方法

默认前提你的项目已经安装了 MRTK3 开发包。

MRTK3 官方可用的彩色 Mesh 着色器为 `Mixed Reality Toolkit/Dashed Ray`。

```C#
UnityEngine.Mesh mesh = new UnityEngine.Mesh();
mesh.indexFormat = UnityEngine.Rendering.IndexFormat.UInt32;		#当面片数量超过65535时需要设置此项
GetComponent<MeshFilter>().mesh = mesh;
Material material = new Material(Shader.Find("Mixed Reality Toolkit/Dashed Ray"));
GetComponent<MeshRenderer>().material = material;
```