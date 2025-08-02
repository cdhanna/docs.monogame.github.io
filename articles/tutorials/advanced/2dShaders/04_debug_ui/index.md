In the previous chapter, you set up a _hot-reload_ system that allows you to edit shader code without needing to restart the game. In this chapter, we will continue in the theme of reducing the need to restart your application by adding a _debug user interface_ for each `Effect` that will enable you to adjust configuration and properties of the shader at runtime. 

## Adding a Debug UI Library

A common approach to building debug UI's in games is to use an _Immediate Mode_ system. An immediate mode UI redraws the entire UI from scratch every frame. Immediate mode UIs make developing developer-facing debug tools easy. A popular library is called `DearImGui`, which has a dotnet C# port called `ImGui.NET`. 

To add `ImGUI.NET`, add the following Nuget package reference to the _MonoGameLibrary_ project,
```xml
<PackageReference Include="ImGui.NET" Version="1.91.6.1" />
```

In order to render the `ImGui.NET` UI in MonoGame, we need a few supporting classes that convert the `ImGui.NET` data into MonoGame's graphical representation. There is a [sample project](https://github.com/ImGuiNET/ImGui.NET/tree/master/src/ImGui.NET.SampleProgram.XNA) on `ImGui.NET`'s public repository that we can copy for our use cases. 

Create a new folder in the _MonoGameLibrary_ project called _ImGui_ and copy paste the following files into the folder, 
- The [`ImGuiRenderer.cs`](https://github.com/ImGuiNET/ImGui.NET/blob/v1.91.6.1/src/ImGui.NET.SampleProgram.XNA/ImGuiRenderer.cs)
- The [`DrawVertDeclaration.cs`](https://github.com/ImGuiNET/ImGui.NET/blob/v1.91.6.1/src/ImGui.NET.SampleProgram.XNA/DrawVertDeclaration.cs)

There is `unsafe` code in the `ImGui` code, like this snippet, so you will need to enable `unsafe` code in the `MonoGameLibrary.csproj` file. Add this property.
```xml
<AllowUnsafeBlocks>true</AllowUnsafeBlocks>
```

In order to play around with the new UI tool, we will set up a small debug UI for the existing `_grayscaleEffect` in the main `GameScene`. As we experiment with `ImGui`, we will build towards a re-usable debug UI for future shaders. To get started, we need to have an instance of `ImGuiRenderer`. Similar to how there is a single `static SpriteBatch` , we will create a single `static ImGuiRenderer` to be re-used throughout the game. 

In the `Core.cs` file, add the following property to the `Core` class.

```csharp
/// <summary>  
/// Gets the ImGui renderer used for debug UIs.  
/// </summary>  
public static ImGuiRenderer ImGuiRenderer { get; private set; }
```

And then to initialize the instance, in the `Initialize()` method, add the following snippet,
```csharp
// Create the ImGui renderer.  
ImGuiRenderer = new ImGuiRenderer(this);  
ImGuiRenderer.RebuildFontAtlas();
```

Similar to `SpriteBatch`'s `.Begin()` and `.End()` calls, the `ImGuiRenderer` has a start and end function call. In the `GameScene` class, add these lines to end of the `.Draw()` method.
```csharp
// Draw debug UI
Core.ImGuiRenderer.BeforeLayout(gameTime);  
// draw the debug UI here  
Core.ImGuiRenderer.AfterLayout();
```

`ImGui` draws by adding draggable windows to the screen. To create a simple window that just prints out `"Hello World"`, use the following snippet,
```csharp
// Draw debug UI  
Core.ImGuiRenderer.BeforeLayout(gameTime);  
ImGui.Begin("Demo Window");  
ImGui.Text("Hello world!");  
ImGui.End();  
Core.ImGuiRenderer.AfterLayout();
```

![Figure 03.1: a simple ImGui window](./gifs/imgui-hello-world.gif)

## 