---
layout: post
title: WebMCP in Blazor Diagram Component | Syncfusion®
description: Learn how to enable WebMCP in the Syncfusion Blazor Diagram component, use the built-in tools, register custom tools, and control tool execution from AI agents.
platform: diagram-sdk
control: Diagram Component
documentation: ug
---

# WebMCP in Blazor Diagram Component

The Blazor Diagram component supports the Web Model Context Protocol (WebMCP), which allows AI agents and other WebMCP clients running in the browser to discover and use the diagram through a set of named tools. This guide explains how to enable WebMCP, understand the built-in tools, add custom tools, and control how tools run.

## What is WebMCP?

WebMCP lets a web page expose its functionality as **tools** that AI assistants, LLM extensions, and other WebMCP clients can discover and call in the browser. Each tool has a name, a description, and a JSON input schema that describes the arguments it accepts.

### Benefits of enabling WebMCP in the Diagram

* **Built-in tools:** Common diagram operations such as creating nodes and connectors, aligning, grouping, applying layouts, zooming, and exporting are available without additional code.
* **Configurable tool set:** Rename, re-describe, or disable built-in tools using Blazor markup.
* **Custom tools:** Expose application-specific operations and handle them in C#.
* **Execution control:** Inspect, cancel, confirm, or override every tool call before it runs.

### How AI agents interact with the Diagram

1. When WebMCP is enabled, the Diagram registers its tools with the browser.
2. An AI agent discovers the registered tools and reads their descriptions and input schemas.
3. The agent calls a tool with a JSON arguments object.
4. The Diagram raises the [OnWebMcpExecute](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_OnWebMcpExecute) callback before the tool runs. The application can inspect the call, cancel it, request user confirmation, or return its own result.
5. If the call is allowed and not overridden, the tool runs against the diagram and the result is returned to the agent.

## Enable WebMCP

Set the [EnableWebMcp](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_EnableWebMcp) property of `SfDiagramComponent` to `true`. Optionally, add [DiagramWebMcpSettings](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpSettings.html) to give the tool set a name.

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Width="100%" Height="600px" EnableWebMcp="true">
    <DiagramWebMcpSettings Name="diagram" />
</SfDiagramComponent>
```

* `EnableWebMcp` is `false` by default. When it is `false`, no tools are registered.
* When `EnableWebMcp` is `true`, **all built-in tools are registered automatically**. You do not need to declare them.
* The [Name](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpSettings.html#Syncfusion_Blazor_Diagram_DiagramWebMcpSettings_Name) property is used as a prefix when the tools are registered in the browser. Set a unique value when a page hosts more than one diagram.
* The tools are unregistered when `EnableWebMcp` is set to `false` or when the component is disposed.

## Built-in WebMCP tools

The built-in tools are identified by the [DiagramWebMcpToolCommand](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html) enum. The following table lists each value, its purpose, and a typical usage scenario. The enum members and their detailed descriptions are listed in the [API reference](#api-reference).

### Diagram information

| Enum value | Purpose | Typical usage scenario |
|---|---|---|
| [GetDiagramCatalog](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_GetDiagramCatalog) | Returns the supported shapes, enums, and shape-specific properties. | An agent discovers which shapes it can use before creating a diagram. |

### Diagram creation

| Enum value | Purpose | Typical usage scenario |
|---|---|---|
| [CreateDiagramNode](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_CreateDiagramNode) | Creates one or more nodes. | Generate the steps of a flowchart from a text description. |
| [CreateDiagramConnector](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_CreateDiagramConnector) | Creates connectors between nodes or ports. | Link the generated steps in order. |
| [CreateUmlSequenceDiagram](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_CreateUmlSequenceDiagram) | Creates or updates a UML sequence diagram. | Generate a sequence diagram from a described interaction. |
| [ApplyLayoutToNodes](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ApplyLayoutToNodes) | Applies a layout to the nodes and connectors. | Arrange a generated diagram automatically. |

### Diagram modification

| Enum value | Purpose | Typical usage scenario |
|---|---|---|
| [DragNode](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_DragNode) | Moves a node or connector by a translation. | Reposition an element on request. |
| [ArrangeAndAlignObjects](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ArrangeAndAlignObjects) | Aligns, distributes, or resizes selected objects. | Tidy up a group of nodes. |
| [ArrangeZOrder](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ArrangeZOrder) | Changes the stacking order of selected objects. | Bring an overlapped node to the front. |
| [ManageGroupsAndHierarchies](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageGroupsAndHierarchies) | Groups or ungroups diagram objects. | Combine related nodes into a single group. |
| [ManageAnnotationsAndLabels](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageAnnotationsAndLabels) | Adds, updates, or removes annotations on a node or connector. | Label or relabel elements. |
| [ManagePorts](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManagePorts) | Adds, removes, or distributes ports on a node or connector. | Add connection points to a node. |
| [DeleteFromDiagram](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_DeleteFromDiagram) | Deletes specified, selected, or all diagram objects. | Remove obsolete elements. |

### Diagram management

| Enum value | Purpose | Typical usage scenario |
|---|---|---|
| [ManageNodeSelection](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageNodeSelection) | Selects, clears, or queries the selection. | Select elements before applying an operation. |
| [EnableEditingMode](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_EnableEditingMode) | Enables or disables diagram editing. | Switch the diagram between read-only and editable. |
| [ManageUndoRedo](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageUndoRedo) | Performs undo, redo, history grouping, or history clearing. | Let an agent revert its own changes. |
| [ManageClipboardOperations](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageClipboardOperations) | Performs copy, cut, and paste on the selection. | Duplicate part of a diagram. |
| [NavigateAndZoomDiagram](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_NavigateAndZoomDiagram) | Zooms, pans, or fits content in the viewport. | Bring the relevant area into view. |

### Import and export

| Enum value | Purpose | Typical usage scenario |
|---|---|---|
| [ExportAndImportDiagrams](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ExportAndImportDiagrams) | Saves, exports, imports, loads, or prints the diagram. | Save the diagram or export it as an image. |

### Custom

| Enum value | Purpose | Typical usage scenario |
|---|---|---|
| [Custom](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_Custom) | Marks an application-defined tool. | Expose application logic to an agent. See [Create custom tools](#create-custom-tools). |

N> All built-in tools are available automatically when `EnableWebMcp` is `true`. To rename, re-describe, or hide them, see [Customize built-in tools](#customize-built-in-tools).

## Customize built-in tools

By default, every built-in tool is registered. To control the tool set, declare [DiagramWebMcpTools](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTools.html) inside `DiagramWebMcpSettings` and add one [DiagramWebMcpTool](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTool.html) for each tool to configure. A built-in entry can override the tool name and description, or remove the tool from registration by setting `Enabled` to `false`.

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Width="100%" Height="600px" EnableWebMcp="true">
    <DiagramWebMcpSettings Name="flowchart">
        <DiagramWebMcpTools>
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.GetDiagramCatalog" />

            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.CreateDiagramNode"
                               Name="addProcessNode"
                               Description="Creates a flowchart step in the diagram." />

            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.DeleteFromDiagram"
                               Enabled="false" />
        </DiagramWebMcpTools>
    </DiagramWebMcpSettings>
</SfDiagramComponent>
```

N> The tool still runs its built-in behavior after it is renamed, because it is identified by the `Command` value and not by its name. In `OnWebMcpExecute`, check `args.Command` to identify a built-in tool.

## Create custom tools

To expose an application-specific operation, add a `DiagramWebMcpTool` with `Command="@DiagramWebMcpToolCommand.Custom"`. A custom tool requires:

* **Name:** the unique name that the agent uses to call the tool.
* **Description:** a plain-text description that helps the agent decide when to use the tool.
* **InputSchema:** a JSON Schema string that describes the arguments.

Custom tools run through the [OnWebMcpExecute](#control-tool-execution) callback. The callback must set `args.Result` to the JSON string that is returned to the agent.

```cshtml
@using System.Text.Json
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent @ref="_diagram"
                    Width="100%"
                    Height="600px"
                    EnableWebMcp="true"
                    OnWebMcpExecute="HandleWebMcpExecuteAsync">
    <DiagramWebMcpSettings Name="diagram">
        <DiagramWebMcpTools>
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.Custom"
                               Name="getDiagramSummary"
                               Description="Returns the number of nodes and connectors in the diagram."
                               InputSchema="@_summarySchema" />
        </DiagramWebMcpTools>
    </DiagramWebMcpSettings>
</SfDiagramComponent>

@code {
    private SfDiagramComponent? _diagram;

    private readonly string _summarySchema = JsonSerializer.Serialize(new
    {
        type = "object",
        properties = new { }
    });

    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        if (args.Command == DiagramWebMcpToolCommand.Custom && args.Name == "getDiagramSummary")
        {
            args.Result = JsonSerializer.Serialize(new
            {
                success = true,
                nodes = _diagram?.Nodes.Count ?? 0,
                connectors = _diagram?.Connectors.Count ?? 0
            });
        }

        return Task.CompletedTask;
    }
}
```

## Control tool execution

The [OnWebMcpExecute](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_OnWebMcpExecute) callback is triggered **before** a built-in or custom tool executes. It receives a [DiagramWebMcpExecuteEventArgs](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html) instance, which lets the application inspect the call and decide what happens next.

| Member | Use |
|---|---|
| [Command](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Command) | Identifies the tool being called. This is `Custom` for custom tools. |
| [Name](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Name) | The tool name, without the registration prefix. |
| [Arguments](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Arguments) | The arguments sent by the agent. Whole numbers are `long`, fractional numbers are `double`, and nested objects are dictionaries. |
| [Cancel](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Cancel) | Set to `true` to [cancel](#cancel-tool-execution) the call. |
| [ShowConfirmationDialog](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_ShowConfirmationDialog) | Set to `true` to [ask the user to confirm](#request-user-confirmation) a built-in tool. |
| [CancellationResponse](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_CancellationResponse) | The message returned to the agent when the call is cancelled or declined. |
| [Result](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Result) | A JSON string returned to the agent. Required for custom tools. For built-in tools, it [overrides](#override-tool-execution) the built-in result. |

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Width="100%"
                    Height="600px"
                    EnableWebMcp="true"
                    OnWebMcpExecute="HandleWebMcpExecuteAsync">
    <DiagramWebMcpSettings Name="diagram" />
</SfDiagramComponent>

@code {
    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        Console.WriteLine($"WebMCP tool requested: {args.Command} ({args.Name})");

        if (args.Arguments != null)
        {
            foreach (var argument in args.Arguments)
            {
                Console.WriteLine($"{argument.Key} = {argument.Value}");
            }
        }

        return Task.CompletedTask;
    }
}
```

### Cancel tool execution

Set `args.Cancel` to `true` to prevent a built-in or custom tool from running. The agent receives the text in `args.CancellationResponse`. Use cancellation to enforce application rules, such as permissions or read-only review modes.

```cshtml
@code {
    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        if (args.Command == DiagramWebMcpToolCommand.DeleteFromDiagram)
        {
            args.Cancel = true;
            args.CancellationResponse = "Deleting diagram objects is not allowed.";
        }

        return Task.CompletedTask;
    }
}
```

### Request user confirmation

Set `args.ShowConfirmationDialog` to `true` to ask the user to confirm a built-in tool in the browser before it runs. If the user declines, the tool does not run and the agent receives `args.CancellationResponse`. Use confirmation for destructive or hard-to-undo operations, such as deleting objects or clearing the history.

```cshtml
@code {
    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        if (args.Command == DiagramWebMcpToolCommand.DeleteFromDiagram)
        {
            args.ShowConfirmationDialog = true;
            args.CancellationResponse = "The user declined to delete the objects.";
        }

        return Task.CompletedTask;
    }
}
```

N> `ShowConfirmationDialog` applies to built-in tools only.

### Override tool execution

Set `args.Result` to a JSON string to replace the result of a built-in tool. The callback runs before the built-in execution, so the application can customize the behavior of any tool.

```cshtml
@using System.Text.Json

@code {
    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        if (args.Command == DiagramWebMcpToolCommand.GetDiagramCatalog)
        {
            args.Result = JsonSerializer.Serialize(new
            {
                allowedShapes = new[] { "Rectangle", "Ellipse", "Diamond" }
            });
        }

        return Task.CompletedTask;
    }
}
```

## Get the registered tools

The [GetWebMcpToolsAsync](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_GetWebMcpToolsAsync_System_Collections_Generic_IEnumerable_System_String__) method returns the metadata of the tools that are currently registered for the diagram. The optional `toolNames` parameter filters the result by tool name. The method returns a read-only list of [IDiagramWebMcpTool](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.IDiagramWebMcpTool.html) items, and each item exposes `Name`, `Description`, and `InputSchema`.

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent @ref="_diagram" Width="100%" Height="600px" EnableWebMcp="true">
    <DiagramWebMcpSettings Name="diagram" />
</SfDiagramComponent>

<button @onclick="ListToolsAsync">List registered tools</button>

@code {
    private SfDiagramComponent? _diagram;

    private async Task ListToolsAsync()
    {
        if (_diagram == null)
        {
            return;
        }

        IReadOnlyList<IDiagramWebMcpTool> tools = await _diagram.GetWebMcpToolsAsync();

        foreach (IDiagramWebMcpTool tool in tools)
        {
            Console.WriteLine($"{tool.Name}: {tool.Description}");
        }
    }
}
```

The returned `Name` does not include the prefix that is set through `DiagramWebMcpSettings.Name`.

## Complete custom tool example

The following example registers a custom tool with an input schema, reads its arguments in `OnWebMcpExecute`, adds a node to the diagram, and returns a structured JSON result. It also asks for confirmation before the built-in delete tool runs.

```cshtml
@using System.Text.Json
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent @ref="_diagram"
                    Width="100%"
                    Height="650px"
                    EnableWebMcp="true"
                    OnWebMcpExecute="HandleWebMcpExecuteAsync">
    <DiagramWebMcpSettings Name="workflow">
        <DiagramWebMcpTools>
            <!-- Built-in tool -->
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.GetDiagramCatalog" />

            <!-- Custom tool to add a specialized process step -->
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.Custom"
                               Name="addProcessStep"
                               Description="Creates a new styled step node in the process diagram."
                               InputSchema="@_addStepSchema" />
        </DiagramWebMcpTools>
    </DiagramWebMcpSettings>
</SfDiagramComponent>

@code {
    private SfDiagramComponent? _diagram;

    private readonly string _addStepSchema = JsonSerializer.Serialize(new
    {
        type = "object",
        properties = new
        {
            title = new { type = "string", description = "The step title." },
            x = new { type = "number", description = "The X coordinate." },
            y = new { type = "number", description = "The Y coordinate." }
        },
        required = new[] { "title", "x", "y" }
    });

    private async Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        // Safeguard destructive built-in operations
        if (args.Command == DiagramWebMcpToolCommand.DeleteFromDiagram)
        {
            args.ShowConfirmationDialog = true;
            args.CancellationResponse = "User declined deletion.";
            return;
        }

        // Handle custom tool
        if (args.Command == DiagramWebMcpToolCommand.Custom && args.Name == "addProcessStep")
        {
            if (args.Arguments == null)
            {
                args.Result = JsonSerializer.Serialize(new { success = false, message = "Missing arguments." });
                return;
            }

            string title = args.Arguments.TryGetValue("title", out object? t) ? t?.ToString() ?? "Step" : "Step";
            double x = args.Arguments.TryGetValue("x", out object? xv) && xv != null ? Convert.ToDouble(xv) : 100;
            double y = args.Arguments.TryGetValue("y", out object? yv) && yv != null ? Convert.ToDouble(yv) : 100;

            string nodeId = $"step_{Guid.NewGuid():N}";

            var node = new Node
            {
                ID = nodeId,
                OffsetX = x,
                OffsetY = y,
                Width = 140,
                Height = 60,
                Shape = new BasicShape { Type = NodeShapes.Basic, Shape = BasicShapeType.Rectangle },
                Style = new ShapeStyle { Fill = "#EBF4FF", StrokeColor = "#3182CE", StrokeWidth = 2 },
                Annotations = new DiagramObjectCollection<ShapeAnnotation>
                {
                    new ShapeAnnotation { Content = title, Style = new TextStyle { Color = "#1A202C", Bold = true } }
                }
            };

            if (_diagram != null)
            {
                await _diagram.AddDiagramElementsAsync(new DiagramObjectCollection<NodeBase> { node });
            }

            // Return structured response
            args.Result = JsonSerializer.Serialize(new
            {
                success = true,
                createdNodeId = nodeId,
                offsetX = x,
                offsetY = y
            });
        }
    }
}
```

## API reference

### [SfDiagramComponent](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html)

| Member | Type | Description | Default |
|---|---|---|---|
| [EnableWebMcp](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_EnableWebMcp) | `bool` | Registers the WebMCP tools for the diagram. | `false` |
| [OnWebMcpExecute](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_OnWebMcpExecute) | `EventCallback<DiagramWebMcpExecuteEventArgs>` | Triggered before a built-in or custom tool executes. | - |
| [GetWebMcpToolsAsync(IEnumerable<string>? toolNames = null)](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_GetWebMcpToolsAsync_System_Collections_Generic_IEnumerable_System_String__) | `Task<IReadOnlyList<IDiagramWebMcpTool>>` | Returns the metadata of the registered tools, optionally filtered by tool name. | - |

### [DiagramWebMcpSettings](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpSettings.html)

| Member | Type | Description | Default |
|---|---|---|---|
| [Name](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpSettings.html#Syncfusion_Blazor_Diagram_DiagramWebMcpSettings_Name) | `string` | Prefix used when the tools are registered in the browser. | `string.Empty` |
| [ChildContent](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpSettings.html#Syncfusion_Blazor_Diagram_DiagramWebMcpSettings_ChildContent) | `RenderFragment?` | Contains the `DiagramWebMcpTools` collection. When it is omitted or empty, all built-in tools are registered. | `null` |

### [DiagramWebMcpTools](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTools.html)

| Member | Type | Description | Default |
|---|---|---|---|
| [ChildContent](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTools.html#Syncfusion_Blazor_Diagram_DiagramWebMcpTools_ChildContent) | `RenderFragment?` | Contains the `DiagramWebMcpTool` entries. | `null` |

### [DiagramWebMcpTool](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTool.html)

| Member | Type | Description | Default |
|---|---|---|---|
| [Command](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTool.html#Syncfusion_Blazor_Diagram_DiagramWebMcpTool_Command) | `DiagramWebMcpToolCommand` | Selects a built-in tool, or `Custom` for an application-defined tool. Each command can be declared only once. | - |
| [Name](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTool.html#Syncfusion_Blazor_Diagram_DiagramWebMcpTool_Name) | `string?` | Overrides the tool name. Required for custom tools. | `null` |
| [Description](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTool.html#Syncfusion_Blazor_Diagram_DiagramWebMcpTool_Description) | `string?` | Overrides the tool description. Required for custom tools. | `null` |
| [InputSchema](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTool.html#Syncfusion_Blazor_Diagram_DiagramWebMcpTool_InputSchema) | `string?` | JSON Schema string that describes the tool arguments. Required for custom tools. | Built-in schema for built-in tools |
| [Enabled](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpTool.html#Syncfusion_Blazor_Diagram_DiagramWebMcpTool_Enabled) | `bool` | When `false`, the tool is not registered. | `true` |

### [DiagramWebMcpToolCommand](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html)

| Member | Description |
|---|---|
| [Custom](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_Custom) | Marks an entry that supplies its own name, schema, and handler. |
| [GetDiagramCatalog](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_GetDiagramCatalog) | Returns the supported diagram shapes, enums, and shape-specific properties. |
| [CreateDiagramNode](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_CreateDiagramNode) | Creates one or more diagram nodes. |
| [CreateDiagramConnector](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_CreateDiagramConnector) | Creates a connector between two existing nodes or ports. |
| [DragNode](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_DragNode) | Moves an existing node or connector by a translation. |
| [ManageNodeSelection](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageNodeSelection) | Selects, clears, or queries the diagram selection. |
| [EnableEditingMode](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_EnableEditingMode) | Enables or disables diagram editing in boolean or string mode. |
| [ArrangeAndAlignObjects](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ArrangeAndAlignObjects) | Aligns, distributes, or resizes selected diagram objects. |
| [DeleteFromDiagram](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_DeleteFromDiagram) | Deletes the specified, selected, or all diagram objects. |
| [ManageUndoRedo](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageUndoRedo) | Performs undo, redo, history grouping, or history clearing. |
| [ManageClipboardOperations](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageClipboardOperations) | Performs clipboard copy, cut, and paste on the current selection. |
| [ExportAndImportDiagrams](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ExportAndImportDiagrams) | Saves, exports, imports, loads, or prints the diagram. |
| [ArrangeZOrder](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ArrangeZOrder) | Changes the stacking order of selected diagram objects. |
| [NavigateAndZoomDiagram](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_NavigateAndZoomDiagram) | Navigates the viewport by zooming, panning, or fitting content. |
| [ManageGroupsAndHierarchies](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageGroupsAndHierarchies) | Groups or ungroups diagram objects in place or by ID. |
| [ManageAnnotationsAndLabels](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManageAnnotationsAndLabels) | Adds, updates, or removes annotations or labels on a node or connector. |
| [ManagePorts](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ManagePorts) | Adds, removes, or distributes ports on a node or connector. |
| [ApplyLayoutToNodes](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_ApplyLayoutToNodes) | Applies a layout to the diagram nodes and connectors. |
| [CreateUmlSequenceDiagram](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpToolCommand.html#Syncfusion_Blazor_Diagram_DiagramWebMcpToolCommand_CreateUmlSequenceDiagram) | Creates or updates a UML sequence diagram model. |

### [DiagramWebMcpExecuteEventArgs](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html)

| Member | Type | Description | Default |
|---|---|---|---|
| [Command](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Command) | `DiagramWebMcpToolCommand` | The command that triggered the call. Read-only. | - |
| [Name](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Name) | `string` | The tool name without the registration prefix. Read-only. | - |
| [Arguments](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Arguments) | `IReadOnlyDictionary<string, object>?` | The arguments supplied by the caller. Read-only. | - |
| [Cancel](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Cancel) | `bool` | Cancels the tool execution. | `false` |
| [ShowConfirmationDialog](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_ShowConfirmationDialog) | `bool` | Requests browser confirmation before a built-in tool runs. | `false` |
| [CancellationResponse](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_CancellationResponse) | `string?` | Message returned when the call is cancelled or declined. | `null` |
| [Result](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.DiagramWebMcpExecuteEventArgs.html#Syncfusion_Blazor_Diagram_DiagramWebMcpExecuteEventArgs_Result) | `string?` | JSON result returned to the caller. Required for custom tools; overrides the result of built-in tools. | `null` |

### [IDiagramWebMcpTool](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.IDiagramWebMcpTool.html)

| Member | Type | Description |
|---|---|---|
| [Name](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.IDiagramWebMcpTool.html#Syncfusion_Blazor_Diagram_IDiagramWebMcpTool_Name) | `string` | The tool name. |
| [Description](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.IDiagramWebMcpTool.html#Syncfusion_Blazor_Diagram_IDiagramWebMcpTool_Description) | `string` | The tool description. |
| [InputSchema](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.IDiagramWebMcpTool.html#Syncfusion_Blazor_Diagram_IDiagramWebMcpTool_InputSchema) | `object?` | The JSON schema that describes the tool input. |

## Best practices

* **Use meaningful tool names and descriptions.** Agents rely on them to choose the right tool.
* **Define clear input schemas.** Specify the types, descriptions, and `required` fields of every custom tool.
* **Validate tool arguments.** Read values from `args.Arguments` with `TryGetValue` and handle missing or unexpected values.
* **Return structured JSON responses.** Include a `success` value and relevant data in `args.Result`.
* **Use confirmation dialogs for destructive operations.** Set `ShowConfirmationDialog` for tools such as delete.
* **Use cancellation to enforce application rules.** Set `Cancel` with a clear `CancellationResponse`.
* **Name each tool set on multi-diagram pages.** Use a unique `DiagramWebMcpSettings.Name` for each diagram.

## See also

* [Getting started with Blazor Diagram](https://blazor.syncfusion.com/documentation/diagram/getting-started-with-web-app)
* [Nodes in Blazor Diagram](https://blazor.syncfusion.com/documentation/diagram/nodes/nodes)
* [Connectors in Blazor Diagram](https://blazor.syncfusion.com/documentation/diagram/connectors/connectors)
* [Automatic layout in Blazor Diagram](https://blazor.syncfusion.com/documentation/diagram/layout/automatic-layout)
* [Undo and redo in Blazor Diagram](https://blazor.syncfusion.com/documentation/diagram/undo-redo)
