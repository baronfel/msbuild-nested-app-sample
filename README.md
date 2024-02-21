# MSBuild sample for 'embedding' child apps in a parent app

This sample uses a bit of MSBuild logic to 'embed' child apps in subfolders of a parent application.

This might be useful if you need to publish multiple apps as a single unit, or if you use a kind of out-of-proc command/orchestration model.

## Explanation

The premise is this: you have a parent app, and you have a bunch of child apps in subfolders. You want to publish the parent app, and you want to include the child apps in the published output. However, the child apps need to be isolated from each other for dependency load/management reasons.

So we can ask each of the child apps to Publish itself, and then get the list of files that was part of that publish:

```xml
<MSBuild Projects="@(ChildApp)" Targets="PublishItemsOutputGroup">
    <Output TaskParameter="TargetOutputs" ItemName="_ChildAppPublishItems"/>
</MSBuild>
```

From there, we can take those published items and tell our project to also include them in its publish output, but putting each apps files in a subfolder:

```xml
<ItemGroup>
    <ResolvedFileToPublish 
    Include="@(_ChildAppPublishItems)" 
    RelativePath="$([System.IO.Path]::GetFileNameWithoutExtension(%(_ChildAppPublishItems.MSBuildSourceProjectFile)))\%(_ChildAppPublishItems.RelativePath)" 
    CopyToPublishDirectory="PreserveNewest" />
</ItemGroup>
```

Finally, we have to decide _when_ to do this. We need to do it before the SDK copies our apps' publishable files, so we need to fire before `CollectFilesToPublish` is completed:

```xml
<Target Name="CollectChildApps" BeforeTargets="ComputeFilesToPublish">
```