---
title: "Chapter 01: Getting Started"
description: Setup your development environment for shader development
---



- Rebuilding on save 
- Reloading on save
- Reapply parameters


```xml
  <ItemGroup Condition="'$(WatchFx)'=='true'">
    <Watch Include="Content/**/*.fx" Exclude="**/*.cs"/>
    <Compile Update="**/*.cs" Watch="false" />
  </ItemGroup>

  <Target Name="HocusPocus" DependsOnTargets="RunContentBuilder">
    <Message Text="Rebuilding Content!" Importance="High"/>
    <!-- <Message Text="%(ContentReference.ContentOutputDir) $(PlatformResourcePrefix)%(ExtraContent.Identity)%(ExtraContent.Filename)%(ExtraContent.Extension)" Importance="High"/> -->
    <Copy 
      SourceFiles="%(ExtraContent.Identity)" 
      DestinationFiles="$(OutDir)%(ExtraContent.ContentDir)%(ExtraContent.RecursiveDir)%(ExtraContent.Filename)%(ExtraContent.Extension)"
      SkipUnchangedFiles="true"
      />
  </Target>

```


```sh
dotnet watch build --property WatchFx=true --property AutoRestoreMGCBTool=false -- --target:HocusPocus
```


Mgcb Targets
https://github.com/MonoGame/MonoGame/blob/develop/Tools/MonoGame.Content.Builder.Task/MonoGame.Content.Builder.Task.targets