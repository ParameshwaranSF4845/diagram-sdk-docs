---
layout: post
title: WebMCP in Blazor Diagram Component | Syncfusion®
description: Learn how to enable WebMCP in the Blazor Diagram component, use the built-in tools, register custom tools, and control tool execution from MCP clients and AI agents.
platform: diagram-sdk
control: Diagram Component
documentation: ug
---

# WebMCP in Blazor Diagram Component

## Overview

### What is WebMCP?

The Web Model Context Protocol (WebMCP) lets a web page expose its functionality as a set of named **tools** that MCP clients and AI agents running in the browser can discover and call. Each tool has a name, a description, and a JSON input schema that tells the caller which arguments it accepts.

### Benefits of enabling WebMCP in the Diagram component

* Built-in tools cover the common diagram operations, such as creating nodes and connectors, moving, aligning, grouping, deleting, applying layouts, zooming, and exporting. No additional code is required to expose them.
* AI agents can read and modify the diagram through structured tool calls instead of simulating pointer or keyboard interactions.
* The application keeps full control. Every tool call can be inspected, cancelled, confirmed by the user, or answered with a custom result before it runs.
* Application-specific operations can be exposed as custom tools alongside the built-in ones.

### How MCP clients and AI agents interact with diagrams

1. The Diagram component registers its tools with the browser when WebMCP is enabled.
2. An MCP client or AI agent discovers the registered tools and reads their descriptions and input schemas.
3. The agent calls a tool with a JSON arguments object.
4. The Diagram component raises the [OnWebMcpExecute](#handle-tool-execution) callback, so the application can inspect, cancel, confirm, or override the call.
5. If the call is not cancelled or overridden, the tool runs against the diagram and the result is returned to the agent.

## Enable WebMCP

Set the [EnableWebMcp](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_EnableWebMcp) property of the `SfDiagramComponent` to `true`.

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Height="600px" EnableWebMcp="true">
</SfDiagramComponent>
```

* `EnableWebMcp` is `false` by default. When it is `false`, no tools are registered.
* When it is `true`, the diagram registers its tools so that MCP clients and AI agents can discover them.
* **All built-in tools become available automatically.** You do not need to declare them. To rename, describe differently, or hide individual built-in tools, use [DiagramWebMcpSettings](#customize-built-in-tools).
* Tools are unregistered when `EnableWebMcp` is set to `false` or the component is disposed.

## Built-in WebMCP Tools

The built-in tools are identified by the [DiagramWebMcpToolCommand](#diagramwebmcptoolcommand) enum. The registered tool name is the camel-case form of the enum member name, for example `DeleteFromDiagram` is registered as `deleteFromDiagram`.

### Diagram Information Tools

| Tool name | Command | Description | Typical usage |
|---|---|---|---|
| `getDiagramCatalog` | `GetDiagramCatalog` | Returns the supported diagram shapes, enums, and shape-specific properties. | An agent calls this first to learn which shapes and property values it can use before creating nodes. |
| `manageNodeSelection` | `ManageNodeSelection` | Selects, clears, or queries the diagram selection. | Find out which objects are selected, or select objects before running an operation on them. |

### Diagram Creation Tools

| Tool name | Command | Description | Typical usage |
|---|---|---|---|
| `createDiagramNode` | `CreateDiagramNode` | Creates one or more diagram nodes. | Generate a flowchart or organization chart from a text description. |
| `createDiagramConnector` | `CreateDiagramConnector` | Creates a connector between two existing nodes or ports. | Link nodes created earlier. |
| `createUmlSequenceDiagram` | `CreateUmlSequenceDiagram` | Creates or updates a UML sequence diagram model. | Build a sequence diagram from a described interaction. |
| `applyLayoutToNodes` | `ApplyLayoutToNodes` | Applies a layout to the diagram nodes and connectors. | Arrange generated nodes automatically. |

### Diagram Modification Tools

| Tool name | Command | Description | Typical usage |
|---|---|---|---|
| `dragNode` | `DragNode` | Moves an existing node or connector by a translation. | Reposition an element. |
| `arrangeAndAlignObjects` | `ArrangeAndAlignObjects` | Aligns, distributes, or resizes selected objects. | Tidy up selected nodes. |
| `arrangeZOrder` | `ArrangeZOrder` | Changes the stacking order of selected objects. | Bring an element to the front or send it to the back. |
| `manageGroupsAndHierarchies` | `ManageGroupsAndHierarchies` | Groups or ungroups objects in place or by ID. | Combine related nodes into a group. |
| `manageAnnotationsAndLabels` | `ManageAnnotationsAndLabels` | Adds, updates, or removes annotations or labels on a node or connector. | Set or change element text. |
| `managePorts` | `ManagePorts` | Adds, removes, or distributes ports on a node or connector. | Add connection points to a node. |
| `deleteFromDiagram` | `DeleteFromDiagram` | Deletes the specified, selected, or all diagram objects. | Remove elements. This is a destructive tool, so consider a [confirmation dialog](#show-confirmation-dialog). |

### Diagram Management Tools

| Tool name | Command | Description | Typical usage |
|---|---|---|---|
| `enableEditingMode` | `EnableEditingMode` | Enables or disables diagram editing in boolean or string mode. | Switch the diagram between read-only and editable. |
| `manageUndoRedo` | `ManageUndoRedo` | Performs undo, redo, history grouping, or history clearing. | Revert an agent's change. |
| `manageClipboardOperations` | `ManageClipboardOperations` | Performs copy, cut, and paste on the current selection. | Duplicate parts of a diagram. |
| `navigateAndZoomDiagram` | `NavigateAndZoomDiagram` | Navigates the viewport by zooming, panning, or fitting content. | Fit the whole diagram in view after generation. |

### Import/Export Tools

| Tool name | Command | Description | Typical usage |
|---|---|---|---|
| `exportAndImportDiagrams` | `ExportAndImportDiagrams` | Saves, exports, imports, loads, or prints the diagram. | Save the diagram to JSON, export it as an image, or load a saved diagram. |

### Customize built-in tools

Declare [DiagramWebMcpSettings](#diagramwebmcpsettings) as a child of the diagram to adjust built-in tools. When no entries are declared, every built-in tool is registered. A built-in entry can override the exposed `Name` and `Description`, or remove the tool from registration by setting `Enabled` to `false`. Dispatch is based on the `Command`, so renaming a tool does not change its behavior.

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Height="600px" EnableWebMcp="true">
    <DiagramWebMcpSettings>
        <DiagramWebMcpTools>
            <DiagramWebMcpTool Command="DiagramWebMcpToolCommand.DeleteFromDiagram" Enabled="false" />
            <DiagramWebMcpTool Command="DiagramWebMcpToolCommand.CreateDiagramNode"
                               Name="addShape"
                               Description="Adds one or more shapes to the diagram." />
        </DiagramWebMcpTools>
    </DiagramWebMcpSettings>
</SfDiagramComponent>
```

## Retrieve Registered Tools

The [GetWebMcpToolsAsync](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_GetWebMcpToolsAsync_System_Collections_Generic_IEnumerable_System_String__) method of `SfDiagramComponent` returns the metadata of the tools currently registered for the diagram.

* **Purpose:** Inspect which built-in and custom tools are available, including any renamed or disabled tools.
* **Parameter:** `toolNames` (optional `IEnumerable<string>`). When provided, only the tools with the given names are returned.
* **Return value:** A task that returns a read-only list of tool metadata. Each item exposes `Name`, `Description`, and `InputSchema`. `Name` is the bare tool name, without any registration prefix.

The following example lists the registered tools.

```cshtml
@using Syncfusion.Blazor.Diagram

<button @onclick="ListToolsAsync">List tools</button>

<SfDiagramComponent @ref="_diagram" Height="600px" EnableWebMcp="true">
</SfDiagramComponent>

<ul>
    @foreach (var tool in _tools)
    {
        <li><b>@tool.Name</b> - @tool.Description</li>
    }
</ul>

@code {
    private SfDiagramComponent _diagram;
    private IReadOnlyList<IDiagramWebMcpTool> _tools = new List<IDiagramWebMcpTool>();

    private async Task ListToolsAsync()
    {
        _tools = await _diagram.GetWebMcpToolsAsync();
    }
}
```

To retrieve specific tools only, pass their names.

```csharp
var tools = await _diagram.GetWebMcpToolsAsync(new[] { "createDiagramNode", "deleteFromDiagram" });
```

> Call this method after the diagram has rendered, for example from a button click or `OnAfterRenderAsync`. Tools are registered after the first render.

## Create Custom Tools

Register an application-specific tool by adding a [DiagramWebMcpTool](#diagramwebmcptool) with `Command="DiagramWebMcpToolCommand.Custom"` inside `DiagramWebMcpSettings`. A custom tool requires:

* **Name** - a unique tool name.
* **Description** - explains to the agent what the tool does and when to use it.
* **InputSchema** - a JSON Schema string that describes the arguments.

A custom tool has no built-in behavior. Its execution is handled in the [OnWebMcpExecute](#handle-tool-execution) callback, which must set `Result`.

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Height="600px" EnableWebMcp="true" OnWebMcpExecute="HandleWebMcpExecuteAsync">
    <DiagramWebMcpSettings>
        <DiagramWebMcpTools>
            <DiagramWebMcpTool Command="DiagramWebMcpToolCommand.Custom"
                               Name="getDiagramSummary"
                               Description="Returns a short summary of the diagram."
                               InputSchema="@_summarySchema" />
        </DiagramWebMcpTools>
    </DiagramWebMcpSettings>
</SfDiagramComponent>

@code {
    private const string _summarySchema = """
    {
      "type": "object",
      "properties": {},
      "additionalProperties": false
    }
    """;

    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        if (args.Command == DiagramWebMcpToolCommand.Custom && args.Name == "getDiagramSummary")
        {
            args.Result = "{\"success\": true, \"message\": \"Summary is available.\"}";
        }

        return Task.CompletedTask;
    }
}
```

## Handle Tool Execution

The [OnWebMcpExecute](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_OnWebMcpExecute) callback is triggered **before a built-in or custom tool executes**. It receives a [DiagramWebMcpExecuteEventArgs](#diagramwebmcpexecuteeventargs) object, which lets the application decide what happens next:

| Member | Use it to |
|---|---|
| `Command` | Identify the built-in command, or `Custom` for a custom tool. |
| `Name` | Read the tool name. For custom tools this is the name you supplied. For built-in tools it is the command enum member name, for example `DeleteFromDiagram`. |
| `Arguments` | Read the arguments sent by the caller. Whole numbers are provided as `long`, fractional numbers as `double`, and objects and arrays as dictionaries and lists. |
| `Cancel` | Stop the tool from running. |
| `ShowConfirmationDialog` | Ask the user to confirm before a built-in tool runs. |
| `CancellationResponse` | Set the message returned to the caller when execution is cancelled or confirmation is denied. |
| `Result` | Return a JSON payload to the caller. Required for custom tools. For built-in tools it replaces the built-in result. |

```cshtml
<SfDiagramComponent Height="600px" EnableWebMcp="true" OnWebMcpExecute="HandleWebMcpExecuteAsync">
</SfDiagramComponent>

@code {
    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        Console.WriteLine($"WebMCP tool requested: {args.Command} ({args.Name})");
        return Task.CompletedTask;
    }
}
```

The callback runs for every tool call, so keep the logic fast and check `Command` or `Name` before acting.

## Cancel Tool Execution

Set `Cancel` to `true` to prevent a built-in or custom tool from executing. The tool does not run, and the caller receives the text in `CancellationResponse`.

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

Typical use cases:

* Block operations that the current user is not authorized to perform.
* Keep the diagram read-only while a review or approval is in progress.
* Enforce application rules, such as a maximum number of nodes.

> To remove a built-in tool permanently, set `Enabled="false"` on its [DiagramWebMcpTool](#customize-built-in-tools) entry. Agents will then not see the tool at all.

## Show Confirmation Dialog

Set `ShowConfirmationDialog` to `true` to ask the user for confirmation in the browser before a built-in tool runs. If the user declines, the tool does not run and the caller receives `CancellationResponse`.

```cshtml
@code {
    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        if (args.Command == DiagramWebMcpToolCommand.DeleteFromDiagram)
        {
            args.ShowConfirmationDialog = true;
            args.CancellationResponse = "The user declined to delete the selected objects.";
        }

        return Task.CompletedTask;
    }
}
```

Use confirmation for destructive or hard-to-reverse operations such as `deleteFromDiagram`, or for `exportAndImportDiagrams` when it loads a diagram that replaces the current content. Confirmation applies to built-in tools only. It does not apply to custom tools, so implement your own check inside the callback for those.

## Override Tool Execution

Set `Result` to return your own response instead of running the built-in logic. Because the callback runs before execution, it can intercept any tool and customize its behavior.

```cshtml
@code {
    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        if (args.Command == DiagramWebMcpToolCommand.GetDiagramCatalog)
        {
            args.Result = "{\"success\": true, \"shapes\": [\"Rectangle\", \"Ellipse\", \"Diamond\"]}";
        }

        return Task.CompletedTask;
    }
}
```

* For a **built-in tool**, setting `Result` skips the built-in execution and returns your payload. Leave `Result` unset to let the built-in tool run normally.
* For a **custom tool**, `Result` is mandatory because there is no built-in behavior.
* `Result` must be a valid JSON string. If the JSON object has a `success` member, its value is reported as the success state of the call, so return `"success": false` to report an error.

## Complete Custom Tool Example

The following example registers a custom `addNamedNode` tool that adds a node with a label to the diagram, and validates its arguments before doing so. It also asks for confirmation before `deleteFromDiagram` runs.

```cshtml
@using Syncfusion.Blazor.Diagram
@using System.Text.Json

<SfDiagramComponent @ref="_diagram"
                    Height="600px"
                    Nodes="@_nodes"
                    EnableWebMcp="true"
                    OnWebMcpExecute="HandleWebMcpExecuteAsync">
    <DiagramWebMcpSettings>
        <DiagramWebMcpTools>
            <DiagramWebMcpTool Command="DiagramWebMcpToolCommand.Custom"
                               Name="addNamedNode"
                               Description="Adds a rectangle node with the given label at the given position."
                               InputSchema="@_addNamedNodeSchema" />
        </DiagramWebMcpTools>
    </DiagramWebMcpSettings>
</SfDiagramComponent>

@code {
    private SfDiagramComponent _diagram;
    private DiagramObjectCollection<Node> _nodes = new DiagramObjectCollection<Node>();
    private int _count;

    private const string _addNamedNodeSchema = """
    {
      "type": "object",
      "properties": {
        "label": { "type": "string", "description": "Text shown in the node." },
        "x": { "type": "number", "description": "Horizontal offset of the node center." },
        "y": { "type": "number", "description": "Vertical offset of the node center." }
      },
      "required": ["label", "x", "y"],
      "additionalProperties": false
    }
    """;

    private async Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        // Ask the user before any object is deleted.
        if (args.Command == DiagramWebMcpToolCommand.DeleteFromDiagram)
        {
            args.ShowConfirmationDialog = true;
            args.CancellationResponse = "The user declined to delete the selected objects.";
            return;
        }

        if (args.Command != DiagramWebMcpToolCommand.Custom || args.Name != "addNamedNode")
        {
            return;
        }

        // Read and validate the arguments.
        if (!args.Arguments.TryGetValue("label", out var labelValue) || labelValue is not string label
            || string.IsNullOrWhiteSpace(label)
            || !args.Arguments.TryGetValue("x", out var xValue)
            || !args.Arguments.TryGetValue("y", out var yValue))
        {
            args.Result = JsonSerializer.Serialize(new
            {
                success = false,
                error = "The 'label', 'x' and 'y' arguments are required."
            });
            return;
        }

        double x = Convert.ToDouble(xValue);
        double y = Convert.ToDouble(yValue);
        string id = "node" + (++_count);

        var node = new Node
        {
            ID = id,
            OffsetX = x,
            OffsetY = y,
            Width = 120,
            Height = 60,
            Annotations = new DiagramObjectCollection<ShapeAnnotation>
            {
                new ShapeAnnotation { Content = label }
            }
        };
        await _diagram.AddDiagramElementsAsync(new DiagramObjectCollection<NodeBase> { node });

        // Return a structured JSON result.
        args.Result = JsonSerializer.Serialize(new
        {
            success = true,
            nodeId = id,
            label,
            x,
            y
        });
    }
}
```

## API Reference

### SfDiagramComponent

| Member | Type | Description | Default |
|---|---|---|---|
| `EnableWebMcp` | `bool` | Enables registration of WebMCP tools for the diagram. | `false` |
| `OnWebMcpExecute` | `EventCallback<DiagramWebMcpExecuteEventArgs>` | Callback invoked before a built-in or custom WebMCP tool executes. | `null` |
| `GetWebMcpToolsAsync(IEnumerable<string> toolNames = null)` | `Task<IReadOnlyList<IDiagramWebMcpTool>>` | Returns the metadata of the registered tools. Pass tool names to filter the result. | - |

### DiagramWebMcpSettings

Child component of `SfDiagramComponent` used to configure the tool set.

| Member | Type | Description | Default |
|---|---|---|---|
| `ChildContent` | `RenderFragment` | Contains the `DiagramWebMcpTools` collection. | `null` |
| `Name` | `string` | Name used to identify the WebMCP tool set. | `null` |

When the `DiagramWebMcpTools` collection is empty or omitted, every built-in tool is registered.

### DiagramWebMcpTools

| Member | Type | Description | Default |
|---|---|---|---|
| `ChildContent` | `RenderFragment` | Contains the `DiagramWebMcpTool` entries. | `null` |

### DiagramWebMcpTool

| Member | Type | Description | Default |
|---|---|---|---|
| `Command` | `DiagramWebMcpToolCommand` | Selects a built-in command, or `Custom` to declare a custom tool. | - |
| `Name` | `string` | Overrides the registered name of a built-in tool. Required for custom tools. | Canonical name of the command |
| `Description` | `string` | Overrides the description of a built-in tool. Required for custom tools. | Canonical description of the command |
| `Enabled` | `bool` | Indicates whether the tool is registered. Set to `false` to omit it. | `true` |
| `InputSchema` | `string` | JSON Schema string describing the input arguments. Required for custom tools. | Built-in schema for built-in commands |

A command can be declared only once. A custom entry without `Name`, `Description`, or `InputSchema` is not valid.

### DiagramWebMcpToolCommand

| Member | Registered tool name |
|---|---|
| `Custom` | Supplied through `Name` |
| `GetDiagramCatalog` | `getDiagramCatalog` |
| `CreateDiagramNode` | `createDiagramNode` |
| `CreateDiagramConnector` | `createDiagramConnector` |
| `DragNode` | `dragNode` |
| `ManageNodeSelection` | `manageNodeSelection` |
| `EnableEditingMode` | `enableEditingMode` |
| `ArrangeAndAlignObjects` | `arrangeAndAlignObjects` |
| `DeleteFromDiagram` | `deleteFromDiagram` |
| `ManageUndoRedo` | `manageUndoRedo` |
| `ManageClipboardOperations` | `manageClipboardOperations` |
| `ExportAndImportDiagrams` | `exportAndImportDiagrams` |
| `ArrangeZOrder` | `arrangeZOrder` |
| `NavigateAndZoomDiagram` | `navigateAndZoomDiagram` |
| `ManageGroupsAndHierarchies` | `manageGroupsAndHierarchies` |
| `ManageAnnotationsAndLabels` | `manageAnnotationsAndLabels` |
| `ManagePorts` | `managePorts` |
| `ApplyLayoutToNodes` | `applyLayoutToNodes` |
| `CreateUmlSequenceDiagram` | `createUmlSequenceDiagram` |

### DiagramWebMcpExecuteEventArgs

| Member | Type | Description | Default |
|---|---|---|---|
| `Command` | `DiagramWebMcpToolCommand` | The command that triggered the call. `Custom` for custom tools. | - |
| `Name` | `string` | Tool name. For built-in tools, the command enum member name. | - |
| `Arguments` | `IReadOnlyDictionary<string, object>` | Arguments supplied by the caller, keyed by argument name. | Empty |
| `Cancel` | `bool` | Cancels the tool execution when `true`. | `false` |
| `ShowConfirmationDialog` | `bool` | Requests browser confirmation before a built-in tool runs. Not applicable to custom tools. | `false` |
| `CancellationResponse` | `string` | Message returned when execution is cancelled or confirmation is denied. | `null` |
| `Result` | `string` | JSON payload returned to the caller. Required for custom tools; overrides the result of built-in tools. | `null` |

### IDiagramWebMcpTool

Read-only metadata returned by `GetWebMcpToolsAsync`.

| Member | Type | Description | Default |
|---|---|---|---|
| `Name` | `string` | The tool name. | - |
| `Description` | `string` | The human-readable tool description. | - |
| `InputSchema` | `string` | JSON schema describing the tool input. | - |

## Best Practices

* **Use meaningful tool names.** Choose short, action-oriented names such as `addNamedNode`, so that agents can select the right tool.
* **Write clear descriptions.** Describe what the tool does and when to use it.
* **Define clear input schemas.** Specify types, descriptions, and `required` fields, and set `additionalProperties` to `false` where appropriate.
* **Validate tool arguments.** Arguments come from an external caller. Check for missing and out-of-range values in `OnWebMcpExecute` before using them.
* **Return structured JSON responses.** Include a `success` flag and a clear `error` message on failure so that agents can recover.
* **Use confirmation dialogs for destructive operations.** Set `ShowConfirmationDialog` for tools such as `deleteFromDiagram`.
* **Use cancellation to enforce application-specific rules.** Set `Cancel` with a helpful `CancellationResponse` for operations that are not permitted.
* **Expose only what is needed.** Disable built-in tools that your application does not want agents to use.
* **Keep the callback fast.** `OnWebMcpExecute` runs before every tool call.

## See Also

* [Undo and Redo](./undo-redo)
* [Serialization](./serialization)
* [Events](./events)
