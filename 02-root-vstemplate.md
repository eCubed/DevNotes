# Root.vstemplate

`Root.vstemplate` is THE xml file that defines our solution's structure. It contains settings that
identify our template like name, description, and version numbers. More importantly, it will contain
declarations that state what projects will be created in our solutions.

## Creating the file Root.vstemplate
We will first need to create a folder that will contain all of our template code. At its root level,
we will need to create the file `Root.vstemplate`. Though filenames are typically case-insensitive,
this file **is case-sensitive**! The R in `Root` indeed does need to be capitalized, or else, Visual Studio
will NOT recognize it. We will need to create this file **outside** Visual Studio. A separate XML editor
is preferred, though Notepad is adequate.

## Basic Xml Structure of Root.vstemplate

```xml
<VSTemplate Version="3.0.0" Type="ProjectGroup"
    xmlns="http://schemas.microsoft.com/developer/vstemplate/2005">
  <TemplateData>
    <!-- Basic settings of our solution template -->
  </TemplateData>
  
  <TemplateContent>    
    <ProjectCollection>
      <!-- Declarations of what project files are in our template -->
    </ProjectCollection>   
  </TemplateContent>
</VSTemplate>
```

The root element of `Root.vstemplate` is the `<VSTemplate>` tag. It will contain the exact attribute-values as
indicated above.

The `<TemplateData>` tag contains basic information and settings about our solution template. We will look at this
node in detail in the following section.

The `<TemplateContent>` tag declares what will be included in our template. Since we are interested only in creating
projects, we will focus on the `<ProjectCollection>` tag where we will list exactly the projects that we want to
include in a generated solution.

## The TemplateData Node

```xml
<TemplateData>
  <Name>Full Backend Site</Name>
  <Description>A multi-project solution template containing the authorization, resource (Web API),
    and file servers as well as dev and production web projects
  </Description>    
  <ProjectType>CSharp</ProjectType>
  <SortOrder>1000</SortOrder>
  <CreateNewFolder>true</CreateNewFolder>
  <CreateInPlace>true</CreateInPlace>
  <DefaultName>FullBackendSite</DefaultName>
  <ProvideDefaultName>true</ProvideDefaultName>
  <LocationField>Enabled</LocationField>
  <EnableLocationBrowseButton>true</EnableLocationBrowseButton>
</TemplateData>
```

There are actually more children tags of `<TemplateData>` than shown, but the ones above are required.

`<Name>` is the friendly name of our project. (Verify: ) This will be the name of a template that will show from the 
`New Project` dialog window once we've "deployed" it. (*Deployed* is in quotes because for our purposes, we will not
actually publish our solution template. We will merely package it for local use.)

`<Description>` is details about our project. (Verify: ) This will show up on the right-hand panel of the `New Project`
dialog window when we choose our solution template. It's helper text.

`<ProjectType>` really is the programming language we'll be working with. We will be developing in C#, so the value
is `CSharp`.

`<CreateNewFolder>` determines whether (Verify: ) the "Create folder for solution" option is automatically checked
in the `New Project` dialog window when we start creating a new solution. We will always set this to `true`.

`<CreateInPlace>` means (if the value is `true`) create all the project folders in their respective places relative
to the solution's root. For some odd reason, the default value of this node is false, and is NOT mentioned in the
official documentation. For all purposes, we want the value to be `true` or else, our projects will not be created.
We found this answer in several Github issue conversations. Yes, we have to explicitly have `<CreateInPlace>true</CreateInPlace>`
in this file, and later, throughout our entire solution.

`<DefaultName>` is the default project name that would be automatically filled in the textbox that we would actually
type in our project name.

The other settings aren't relevant to our needs but need to be present.


