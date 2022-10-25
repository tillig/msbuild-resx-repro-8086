# Repro for MSBuild Issue 8086

[Issue #8086](https://github.com/dotnet/msbuild/issues/8086) is about MSBuild warning about `.Designer.cs` files getting included twice in the build.

To see the repro:

1. Clone the repo.
2. Build the project.
    1. The build will generate the `.Designer.cs` file, which is not checked in.
    2. The build will succeed without warnings or errors.
3. Open the repo with VS Code. Look in the `Class1.cs` file - you will see OmniSharp with a red squiggly under the use of `MyResources.SimpleValue` (the generated Designer class member). The `.cs` file is clearly there but OmniSharp won't see it due to the `<Compile Remove>` in the `.csproj`
4. Clean the clone: `git clean -dfx`
5. Edit the `.csproj` file to remove the `<Compile Remove>` directive.
6. Build the project.
    1. The build will generate the `.Designer.cs` file, which is not checked in.
    2. You will see a warning: `CSC : warning CS2002: Source file '/.../src/ResxRepro/MyResources.Designer.cs' specified multiple times [/.../src/ResxRepro/ResxRepro.csproj]`

The branch `issuecomment-1290568321` contains some fixes/updates from [this issue comment](https://github.com/dotnet/msbuild/issues/8086#issuecomment-1290568321) that help resolve a few things:

- Instead of using `ResXFileCodeGenerator` for the resource generator, it uses `MSBuild:Compile`. This is different than what you might see in Visual Studio.
- The generated files go into the `obj` folder so you don't need the `<Compile Remove>` part. This gets rid of the CS2002 warning.
- In order to get OmniSharp to see the generated resources, you need to add to `<Properties>` a line `<CoreCompileDependsOn>PrepareResources;$(CompileDependsOn)</CoreCompileDependsOn>` - this is because OmniSharp calls the build in a way that skips resource generation.

That branch appears to work around some of the challenges, but it does seem like this should be easier (which is tracked in [MSBuild issue 4751](https://github.com/dotnet/msbuild/issues/4751)).
