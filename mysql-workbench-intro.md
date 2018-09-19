# MySQL Workbench Introduction

Once we log in to a MySQL server from Workbench, we are given several panes which make up our workspace. We do not need to be familiar with every link, icon, button,
etc. We can look those up later when we need to work with more advanced MySQL features. This article lists only the most common "How do I?"s for a developer or
administrator new to MySQL who has other SQL management experience.

## Creating Databases Schema
In MSSQL, a schema refers to the definition of our tables and their columns. In MySQL, a schema is something totally different - a MySQL schema is *exactly* a
database to which we would create our tables, stored procedures, functions, etc.

So to create a schema, we can go ahead to the top and click the disk stack icon with the plus button. A new tab will be placed in the middle pane, and it will ask
ask us for the schema name or database name, so we enter our database's name. We click the Apply button at the bottom-right of the panel. A window will pop up
containing the exact MySQL query that would be run to create the schema, and its own Apply button. We need to click that Apply button to confirm, and to finally
create the new schema.

Thenafter, creating tables, columns, etc. are very similar to how we would achieve those same tasks in MSSQL.

Notice: Every time we go through the UI to perform database-altering tasks,
we will always be presented with a confirmation window containing the exact MySQL command that would be run. This is good as a learning tool in case we want to
go "hard core" and manually type those commands later.

## Selecting a Schema

Why would we select a schema? This task doesn't really have an equivalent in MSSQL Management Studio. In Workbench, we **always** have to select the schema **first**
to let Workbench know that we're about to alter it. To select a schema, we double-click one in the schemas list on the left. That second click will automatically
expand the schema. Once the schema is selected, all subsequent actions like creating a table, view, stored procedure, etc, will apply to the selected or active
schema.

## Viewing a Table's Design

We can twirl-open any schema (even if not selected/activated), expand its Tables, right-click on the table we want, and select Table Inspector from the context menu.
This will bring up a new tab in the main tile, also having its own set of tabs just for the selected table. The Columns sub-tab will list the table's columns. We 
cannot alter anything here. The entire Table Inspector is view-only.

## Alter a Table's Design

Twirl-open any schema, expand its Tables node, right-click on the desired table, and then select Alter Table from the context menu. The Columns sub-tab will be
automatically selected, which we'll want for this task. We may go through the UI and click around to add, remove, or edit columns as needed. When we're finished,
we click the Apply Button, see the SQL for altering the table, and then click Apply again. We may get warnings or errors depending on what kind of changes we made,
as we would get with MSSQL Management Studio.

## Creating and Altering Triggers

Twirl-open any schema, expand its Tables node, right-click on the desired table, and then select Alter Table from the context menu. The Columns sub-tab will be
automatically selected, which we don't want for this task. We will need to select the Triggers sub-tab, which is at the bottom of the main pane. There is a list
of the kinds of triggers we can create for a table. If we hover one of them, we will see a plus sign. We would click that plus sign to create that particular kind
of trigger. We will then be presented with a text editor prefilled with the appropriate MySQL trigger command. We would typically put code between the BEGIN and
END tags, and then click Apply when we're done. We may get errors due to mistakes or typos in our trigger code. Once the trigger is successfully created, it will
be listed undereath the appropriate trigger type.

To alter the trigger, we simply select it on the same Trigger sub-tab pane, make alterations to the code, and click Apply. Note that no alteration will actually
happen. In MySQL, "altering" means dropping the current trigger and creating a new one with the same name. This mechanism of drop-then-recreate also applies to
stored procedures and functions.

## Creating and Altering Stored Procedures

To create a stored procedure, twirl-open any schema, right-click Stored Procedures, choose Create Stored Procedure. We'll be brought to a text editor were we'll
type the necessary code to create the stored procedure. When we're done, we click Apply. Once successfully created, the new stored procedure will be listed under
Stored Procedures of that schema.

To alter a stored procedure, twirl-open any schema, expand Stored Procedures, hover over the desired stored procedure, and click the small wrench icon. Alternatively,
we may right-click on the stored procedure we want, and choose Alter Stored Procedure. We will be brought to the same text editor we used when we created the stored
procedure previously. We then click on Apply when we're done. If there are no errors, our stored procedure will be "altered".

There are quotes around "altered" because we really did NOT alter a stored procedure. In MySQL, we do NOT alter stored procedures. We actually end up deleting the
old one and replacing it with the new one. Note that there is no ALTER statement in any of the generated MySQL queries on the confirmation screen. It will always
drop the current stored procedure of the same name and create a new one in its place.

## Creating and Altering Functions

Creating and altering functions is the same administrative experience as creating and altering stored procedures, except we work with the Functions "folder" instead
of the Stored Procedures "folder".