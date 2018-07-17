# Project Templates

We previously created the solution template, which starts with the file `Root.vstemplate` where we
declare solution template data and settings, and where we specify which projects we will want.

We will now start creating project templates. Like a solution template, a project template is a folder
(directly a child of the solution root folder) with its own `.vstemplate` file. In addition, we will
also create the the `.csproj` file where we will list any references to dependencies and our .cs files.

## Creating the Project Template

Before we create a project template, we need to revisit a project declaration from the solution's
`Root.vstemplate` file:

```xml
...
<ProjectTemplateLink ProjectName="$projectname$" CopyParameters="true">
   BusinessLogicLibraryTemplate\MyTemplate.vstemplate
</ProjectTemplateLink>
```

In particular, we focus on the node's value, in this case, `BusinessLogicLibraryTemplate\MyTemplate.vstemplate`.
The value is the relative path of the project template's `.vstemplate` file. In fact, this