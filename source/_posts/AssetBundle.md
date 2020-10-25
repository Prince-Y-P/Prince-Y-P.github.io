---
title: 【加载】Asset bundle
date: 2020-10-25 02:29:00
categories:
- Unity3D
tags:
- Asset
- 加载
---


## 是什么
#### 从资源的角度
它是一个**存在于硬盘上的文件**。可以称之为**压缩包**。这个压缩包可以认为是一个文件夹，里面包含了多个文件。这些文件可以分为两类：serialized file 和 resource files。（序列化文件和源文件）
* serialized files：资源被打碎放在一个对象中，最后统一被写进一个单独的文件（只有一个）。相当于是一个头文件，里面记录了关于这个AB的相关信息，当我们调用**LoadFromFile**接口加载AB的时候实际上就是去加载的这一部分信息。这部分信息在Profiler里面的SerilizeField选项里。
* resource files：某些二进制资源（图片、声音）被单独保存，方便快速加载

#### 从API的角度
它是一个AssetBundle对象，我们可以通过代码从一个特定的压缩包加载出来的对象。这个对象包含了所有我们当初添加到这个压缩包里面的内容，我们可以通过这个对象加载出来使用。

## 用处
1. AssetBundle是一个压缩包包含模型、贴图、预制体、声音、甚至整个场景，可以在游戏运行的时候被加载；
2. AssetBundle自身保存着互相的依赖关系；
3. 压缩包可以使用LZMA和LZ4压缩算法，减少包大小，更快的进行网络传输；
4. 把一些可以下载内容放在AssetBundle里面，可以减少安装包的大小；

## API
```cs
BuildPipeline.BuildAssetBundles(dir, BuildAssetBundleOptions.None, BuildTarget.StandaloneWindows64);

AssetBundle ab = AssetBundle.LoadFromFile("AssetBundles/scene/wall.unity3d");
GameObject wallPrefab = ab.LoadAsset<GameObject>("CubeWall");
Instantiate(wallPrefab);
```

```cs
AssetBundle.LoadFromMemoryAsync
AssetBundle.LoadFromFile
WWW.LoadFromCacheOrDownload
UnityWebRequest
```

```cs
一般
T objectFromBundle = bundleObject.LoadAsset<T>(assetName);
GameObject
GameObject gameObject =
loadedAssetBundle.LoadAsset<GameObject>(assetName);
所有资源
Unity.Object[] objectArray =
loadedAssetBundle.LoadAllAssets();
```

## 关于打包
### 分组策略
1. 把经常更新的资源放在一个单独的包里面，跟不经常更新的包分离
2. 把需要同时加载的资源放在一个包里面
3. 可以把其他包共享的资源放在一个单独的包里面
4. 把一些需要同时加载的小资源打包成一个包
5. 如果对于一个同一个资源有两个版本，可以考虑通过后缀来区分  v1  v2  v3  unity3dv1 unity3dv2

#### 逻辑实体分组
a. 一个UI界面或者所有UI界面一个包（这个界面里面的贴图和布局信息一个包）
b. 一个角色或者所有角色一个包（这个角色里面的模型和动画一个包）
c. 所有的场景所共享的部分一个包（包括贴图和模型）
#### 按照类型分组
所有**声音**资源打成一个包，所有**shader**打成一个包，所有**模型**打成一个包，所有材质打成一个包
#### 按照使用分组
把在某一时间内使用的所有资源打成一个包。可以按照关卡分，一个关卡所需要的所有资源包括角色、贴图、声音等打成一个包。也可以按照场景分，一个场景所需要的资源一个包

### 依赖打包
(Mat + Cube1) (Mat + Cube2)
(Mat) (Cube1) (Cube2)
Unity会自动识别，共享的依赖如果已经单独打包，则不会重复打包
###  Build AssetBundles

1. Build的路径（随意只要是在硬盘上都可以的）
2. BuildAssetBundleOptions
BuildAssetBundleOptions.None：使用LZMA算法压缩，压缩的包更小，但是加载时间更长。使用之前需要整体解压。一旦被解压，这个包会使用LZ4重新压缩。使用资源的时候不需要整体解压。在下载的时候可以使用LZMA算法，一旦它被下载了之后，它会使用LZ4算法保存到本地上。
BuildAssetBundleOptions.UncompressedAssetBundle：不压缩，包大，加载快
BuildAssetBundleOptions.ChunkBasedCompression：使用LZ4压缩，压缩率没有LZMA高，但是我们可以加载指定资源而不用解压全部。

>  注意使用LZ4压缩，可以获得可以跟不压缩想媲美的加载速度，而且比不压缩文件要小。


## LoadAssetBundles

### LoadFromMemoryAsync
```cs
using UnityEngine;
using System.Collections;
using System.IO;

public class Example : MonoBehaviour
{
    IEnumerator LoadFromMemoryAsync(string path)
    {
        AssetBundleCreateRequest createRequest = AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(path));
        yield return createRequest;
        AssetBundle bundle = createRequest.assetBundle;
        var prefab = bundle.LoadAsset<GameObject>("MyObject");
        Instantiate(prefab);
    }
}
```

### LoadFromFile
```cs
public class LoadFromFileExample : MonoBehaviour {
    function Start() {
        var myLoadedAssetBundle
            = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "myassetBundle"));

        if (myLoadedAssetBundle == null) {
            Debug.Log("Failed to load AssetBundle!");
            return;
        }
        var prefab = myLoadedAssetBundle.LoadAsset.<GameObject>("MyObject");
        Instantiate(prefab);
    }
}
```

### LoadFromCacheOrDownload

```cs
using UnityEngine;
using System.Collections;

public class LoadFromCacheOrDownloadExample : MonoBehaviour
{
    IEnumerator Start ()
    {
            while (!Caching.ready)
                    yield return null;

        var www = WWW.LoadFromCacheOrDownload("http://myserver.com/myassetBundle", 5);
        yield return www;
        if(!string.IsNullOrEmpty(www.error))
        {
            Debug.Log(www.error);
            yield return;
        }
        var myLoadedAssetBundle = www.assetBundle;

        var asset = myLoadedAssetBundle.mainAsset;
    }
}
```
### UnityWebRequest
```cs
IEnumerator InstantiateObject()
{
    string uri = "file:///" + Application.dataPath + "/AssetBundles/" + assetBundleName;
    UnityEngine.Networking.UnityWebRequest request
        = UnityEngine.Networking.UnityWebRequest.GetAssetBundle(uri, 0);
    yield return request.Send();
    AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(request);
    GameObject cube = bundle.LoadAsset<GameObject>("Cube");
    GameObject sprite = bundle.LoadAsset<GameObject>("Sprite");
    Instantiate(cube);
    Instantiate(sprite);
}
```


Demo
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.IO;

public class FromFileABLoader : MonoBehaviour
{

    AssetBundle cubeAB;
    string abPath = "Assets/AssetBundles/waterobjs/";
    // Start is called before the first frame update
    void Start()
    {
        string ABPath = Application.dataPath + "/AssetBundles";

        //LoadABFromFile();
        //StartCoroutine(LoadABFromMemoryAsync());
        LoadABFromMemory();
        //AssetBundle matAB = AssetBundle.LoadFromFile("Assets/AssetBundles/watermat.asbd");
        if (cubeAB != null)
        {
            GameObject cubePrefab = cubeAB.LoadAsset<GameObject>("cube");
            Instantiate(cubePrefab);
        }

        //Object[] objABs = cubeAB1.LoadAllAssets();
        //Debug.Log(objABs[0]);
    }

    // Update is called once per frame
    void Update()
    {
        if(Input.GetKeyDown(KeyCode.M))
        {
            AssetBundle.LoadFromFile("Assets/AssetBundles/watermat.asbd");
        }
    }

    void LoadABFromFile()
    {
        cubeAB = AssetBundle.LoadFromFile(abPath + "cube.asbd");

    }

    IEnumerator LoadABFromMemoryAsync()
    {
        AssetBundleCreateRequest cubeRequest = AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(abPath + "cube.asbd"));
        yield return cubeRequest;
        cubeAB = cubeRequest.assetBundle;
        GameObject cubePrefab = cubeAB.LoadAsset<GameObject>("cube");
        Instantiate(cubePrefab);
    }

    void LoadABFromMemory()
    {
        AssetBundleCreateRequest cubeRequest = AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(abPath + "cube.asbd"));
        cubeAB = cubeRequest.assetBundle;
    }
}
```

### AssetBundle.LoadFromMemoryAsync
> **若把Start()函数声明成返回IEnumrator，编译器会自动将Start的调用处理成协程的模式**
### AssetBundle.LoadFromFile
### WWW.LoadFromCacheOrDownload
Loading an AB from a remote location will automatically cache the AssetBundles. If the AssetBundle is compressed, a worker thread will spin up to decompress the bundle and write it to the cache. Once a bundle has been decompressed and cached, it will load exactly like AssetBundle.LoadFromFile .
### UnityWebRequest
从服务器下载
1. NetBox可以在本地启动服务器，把当前目录作为服务器端的网站


### 同步和异步的优缺点

Ref: https://blog.csdn.net/qq_40093529/article/details/85290686
异步
优点：速度快，与主线程无关，
缺点：调用比较麻烦，因为你不知道啥时候你的资源准备好了，最好的做法也是使用回调，这样回调就会很多，很乱个人感觉管理起来很不舒服。

同步
优点：管理起来方便，而且资源准备好了是可以及时返回的，
缺点：是没有异步快
综合上诉，最终我选择了 同步，因为我不希望代码不整洁，也不希望写太多的回调函数来通知调用者，资源准备妥当了。那么问题来了，如何解决同步的缺点呢。也就是卡主线程。之前一直以为corotine这玩意可以帮到我们。但是当我深入理解了coroutine以后发现他其实也是在主线程中的。最终我选择了使用c# 的多线程机制来解决这个问题。


## 加载Manifests
加载Manifests文件可以处理资源的依赖
AssetBundle assetBundle = AssetBundle.LoadFromFile(manifestFilePath);
AssetBundleManifest manifest =
assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
string[] dependencies = manifest.GetAllDependencies("assetBundle"); //Pass the name of the bundle you want the dependencies for.
foreach(string dependency in dependencies)
{
    AssetBundle.LoadFromFile(Path.Combine(assetBundlePath, dependency));
}


## 卸载
卸载有两个方面
1. 减少内存使用
2. 有可能导致丢失
所以什么时候去卸载资源
#### AssetBundle.Unload(true)

When unloadAllLoadedObjects is true, all objects that were loaded from this bundle will be destroyed as well. If there are GameObjects in your Scene referencing those assets, the references to them will become missing.


卸载所有资源，即使有资源被使用着
	1. 在关卡切换、场景切换
	2. 资源没被用的时候调用
#### AssetBundle.Unload(false)
When unloadAllLoadedObjects is false, compressed file data inside the bundle itself will be freed, but any instances of objects loaded from this bundle will remain intact.

If an application must use AssetBundle.Unload(false), then individual Objects can only be unloaded in two ways:
1. 先去除对不想要的Objects的引用（包括场景和代码当中），然后调用 Resources.UnloadUnusedAssets.
2. 场景切换的时候：Load a scene non-additively. This will destroy all Objects in the current scene and invoke ```Resources.UnloadUnusedAssets``` automatically.

## 文件校验
CRC MD5 SHA1
相同点：
CRC、MD5、SHA1都是通过对数据进行计算，来生成一个校验值，该校验值用来校验数据的完整性。
不同点：
1. 算法不同。CRC采用多项式除法，MD5和SHA1使用的是替换、轮转等方法；
2. 校验值的长度不同。CRC校验位的长度跟其多项式有关系，一般为16位或32位；MD5是16个字节（128位）；SHA1是20个字节（160位）；
3. 校验值的称呼不同。CRC一般叫做CRC值；MD5和SHA1一般叫做哈希值（Hash）或散列值；
4. 安全性不同。这里的安全性是指检错的能力，即数据的错误能通过校验位检测出来。CRC的安全性跟多项式有很大关系，相对于MD5和SHA1要弱很多；MD5的安全性很高，不过大概在04年的时候被山东大学的王小云破解了；SHA1的安全性最高。
5. 效率不同，CRC的计算效率很高；MD5和SHA1比较慢。
6. 用途不同。CRC一般用作通信数据的校验；MD5和SHA1用于安全（Security）领域，比如文件校验、数字签名等。

## 其他问题
1. 依赖包重复问题
a. 把需要共享的资源打包到一起
b. 分割包，这些包不是在同一时间使用的
c. 把共享部分打包成一个单独的包
2. 图集重复问题
在Unity当中，Sprite2D会被打包到一个图集当中（由Packing Tag决定）。如果不指定PackingTag，Sprite会打包到同一个图集当中。一个Sprite打包到AssetBundle中时，它所在的整个图集都会被打包进去。
解决方法：确保同一个图集当中的图片打包到同一个AssetBundle当中去。
3. Android贴图问题
4. iOS文件处理重复fixed in Unity 5.3.2p2.

## UnityAssetBundleBrowserTool
### StreamingAssets
Build的时候，该文件夹下的所有东西会被原封不动地打包到我们的安装包当中。
一般放一些二进制文件、 AssetBundles。
