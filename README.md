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

It does not appear that there is a way to both get build analysis to work _and_ get rid of the CS2002 warning. Using `<Compile Remove>` solves the warning, but it seems like that should also actually remove the `.Designer.cs` file from being compiled... and it doesn't. Conversely, without it, you get a warning that somehow it's being included multiple times, which doesn't seem correct.
