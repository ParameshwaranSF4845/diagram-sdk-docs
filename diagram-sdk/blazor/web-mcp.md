---
layout: post
title: WebMCP in Blazor Diagram Component | Syncfusion®
description: Learn how to enable WebMCP in the Blazor Diagram component, use the 18 built-in tools with complete input schemas, register custom tools, and control tool execution from AI agents and MCP clients.
platform: diagram-sdk
control: Diagram Component
documentation: ug
---

# WebMCP in Blazor Diagram Component

## Overview

### What is WebMCP?

The Web Model Context Protocol (WebMCP) lets a web page expose its functionality as a set of named **tools** that AI assistants, LLM extensions, and MCP clients running in the browser can discover and call via standard browser contracts (`document.modelContext.registerTool`). Each tool exposes a name, a human-readable description, and a JSON input schema that describes the arguments it accepts.

### Benefits of enabling WebMCP in the Diagram component

* **18 built-in tools** cover all primary diagramming tasks: discovering shapes, creating nodes and connectors, moving, selecting, editing mode switching, aligning, deleting, undo/redo, clipboard operations, import/export, z-ordering, panning/zooming, grouping, managing labels and ports, applying automatic layouts, and generating UML sequence diagrams.
* **Declarative configuration**: Selectively expose, rename, re-describe, or disable built-in tools using simple Blazor markup.
* **Custom AI tools**: Register application-specific tools with custom JSON schemas and handle them directly in C#.
* **Lifecycle interception**: Every tool call passes through the `OnWebMcpExecute` event, allowing your application to inspect arguments, enforce security rules, cancel execution, trigger native browser confirmation dialogs, or override results.
* **Browser namespace prefixing**: Use `DiagramWebMcpSettings.Name` to namespace tool sets for multiple diagram instances on the same page.

### How MCP clients and AI agents interact with diagrams

1. When WebMCP is enabled, the Diagram component registers its tools with the browser's model context.
2. An MCP client or AI agent discovers the registered tools, inspecting their canonical descriptions and input schemas.
3. The agent invokes a tool by passing a structured JSON argument payload.
4. Before any tool executes, the Diagram raises the [OnWebMcpExecute](#handle-tool-execution) callback.
5. In the callback, your application can inspect arguments, cancel the call, request browser user confirmation, or provide a custom response.
6. If the call is allowed and not overridden, the Diagram executes the command and returns the JSON result to the caller.

---

## Enable WebMCP

Set the [EnableWebMcp](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.Diagram.SfDiagramComponent.html#Syncfusion_Blazor_Diagram_SfDiagramComponent_EnableWebMcp) property of `SfDiagramComponent` to `true` and add a `<DiagramWebMcpSettings>` tag to configure the toolset namespace:

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Width="100%" Height="700px" EnableWebMcp="true">
    <DiagramWebMcpSettings Name="diagram" />
</SfDiagramComponent>
```

* `EnableWebMcp` defaults to `false`. When disabled, no tools are exposed to the browser.
* When `EnableWebMcp="true"`, all **18 built-in tools** are automatically registered with their canonical names, descriptions, and input schemas.
* The `Name` property of `DiagramWebMcpSettings` defines the prefix used when registering tools in the browser (e.g., `Name="diagram"` produces tool names like `diagram_createDiagramNode` and `diagram_getDiagramCatalog`).
* `GetWebMcpToolsAsync` and the `OnWebMcpExecute` callback receive the **base tool name** without the registration prefix.
* When the diagram component is disposed or `EnableWebMcp` is set to `false`, the tools are automatically unregistered from the browser context.

---

## Selective Tool Configuration

To expose only specific tools, rename them, rewrite their descriptions for an LLM prompt, or disable destructive actions, declare child `<DiagramWebMcpTool>` elements inside `<DiagramWebMcpTools>`:

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Width="100%" Height="700px" EnableWebMcp="true">
    <DiagramWebMcpSettings Name="flowchart">
        <DiagramWebMcpTools>
            <!-- Discover diagram shapes and capabilities -->
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.GetDiagramCatalog" />

            <!-- Customize names and descriptions for the AI agent -->
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.CreateDiagramNode"
                               Name="addProcessNode"
                               Description="Creates a flowchart step in the diagram canvas." />

            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.CreateDiagramConnector"
                               Name="linkSteps"
                               Description="Connects two process steps using orthogonal connectors." />

            <!-- Enable automatic layout calculation -->
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.ApplyLayoutToNodes" />

            <!-- Explicitly disable destructive operations -->
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.DeleteFromDiagram"
                               Enabled="false" />
        </DiagramWebMcpTools>
    </DiagramWebMcpSettings>
</SfDiagramComponent>
```

> **Note:** Dispatch uses the `Command` enum, not the public tool name. Renaming a built-in tool changes its name in the browser and in `args.Name`, but retains built-in execution behavior. Setting `Enabled="false"` omits the tool from registration.

---

## Built-in WebMCP Tools

The Diagram component provides **18 built-in tools** identified by the `DiagramWebMcpToolCommand` enum:

| Tool Name | Enum Member | Annotations | Purpose |
|---|---|---|---|
| `getDiagramCatalog` | `GetDiagramCatalog` | Read-only, Idempotent | Discovers supported shapes, enums, BPMN/UML types, swimlane capabilities, and styling properties. |
| `createDiagramNode` | `CreateDiagramNode` | Standard | Creates one or more nodes with shapes, geometry, headers, styling, and annotations. |
| `createDiagramConnector` | `CreateDiagramConnector` | Standard | Creates straight, orthogonal, or bezier connectors between nodes or ports. |
| `dragNode` | `DragNode` | Standard | Moves existing nodes or connectors by relative coordinate offsets (`tx`, `ty`). |
| `manageNodeSelection` | `ManageNodeSelection` | Idempotent | Selects, inspects, or clears diagram selection (`single`, `multiple`, `all`, `none`). |
| `enableEditingMode` | `EnableEditingMode` | Destructive, Idempotent | Toggles diagram between read-only and interactive read-write modes. |
| `arrangeAndAlignObjects` | `ArrangeAndAlignObjects` | Standard | Aligns, distributes, or resizes selected diagram objects. |
| `deleteFromDiagram` | `DeleteFromDiagram` | Destructive | Deletes specified objects, selected objects, or all diagram elements. |
| `manageUndoRedo` | `ManageUndoRedo` | Destructive | Triggers undo, redo, starts/ends history groups, or clears history stacks. |
| `manageClipboardOperations` | `ManageClipboardOperations` | Destructive | Performs clipboard copy, cut, or paste on the current selection. |
| `exportAndImportDiagrams` | `ExportAndImportDiagrams` | Destructive | Exports/imports JSON or Mermaid formats; exports PNG, JPEG, SVG, or triggers print. |
| `arrangeZOrder` | `ArrangeZOrder` | Standard | Changes object stacking order (`BringToFront`, `SendToBack`, `BringForward`, `SendBackward`). |
| `navigateAndZoomDiagram` | `NavigateAndZoomDiagram` | Idempotent | Navigates the viewport: pan, zoom, reset, fit to page, or bring into view. |
| `manageGroupsAndHierarchies` | `ManageGroupsAndHierarchies` | Standard | Groups or ungroups diagram objects in place or by explicit parent/child IDs. |
| `manageAnnotationsAndLabels` | `ManageAnnotationsAndLabels` | Standard | Adds, updates, or deletes labels and annotations on nodes or connectors. |
| `managePorts` | `ManagePorts` | Standard | Adds, removes, or positions connection ports on nodes or connectors. |
| `applyLayoutToNodes` | `ApplyLayoutToNodes` | Standard | Applies automatic layout algorithms (Hierarchical, Radial, MindMap, OrgChart, etc.). |
| `createUmlSequenceDiagram` | `CreateUmlSequenceDiagram` | Standard | Creates a complete UML sequence diagram with participants, messages, and fragments. |

---

## Built-in Tool Payloads & Input Schemas

Each built-in tool accepts a structured JSON payload. Below are the canonical argument payloads for each tool.

### 1. `getDiagramCatalog`
Discovers supported shapes, enums, categories, and properties before generating diagram elements.

```json
{
  "category": "flow"
}
```
* **Supported categories:** `"all"`, `"basic"`, `"flow"`, `"bpmn"`, `"uml"`, `"swimlane"`, `"connector"`.

### 2. `createDiagramNode`
Creates one or more diagram nodes with shape details, positions, dimensions, styling, and annotations.

```json
{
  "nodes": [
    {
      "nodeId": "node1",
      "shapeFamily": "basic",
      "shape": "Rectangle",
      "x": 200,
      "y": 150,
      "width": 120,
      "height": 60,
      "label": "Process Step",
      "style": { "fill": "#E0E9FD", "strokeColor": "#3E63DD", "strokeWidth": 2 },
      "annotations": [
        {
          "content": "Process Step",
          "annotationStyle": { "bold": true, "color": "#1C2024" }
        }
      ]
    }
  ]
}
```

### 3. `createDiagramConnector`
Creates straight, orthogonal, or bezier connectors between two nodes or connection ports.

```json
{
  "connectors": [
    {
      "connectorId": "connector1",
      "sourceId": "node1",
      "targetId": "node2",
      "lineType": "Orthogonal",
      "label": "Yes",
      "lineColor": "#3E63DD",
      "lineWidth": 2,
      "targetDecorator": "Arrow"
    }
  ]
}
```

### 4. `dragNode`
Translates an existing node or connector by relative coordinate offsets.

```json
{
  "nodeId": "node1",
  "tx": 50,
  "ty": -30
}
```

### 5. `manageNodeSelection`
Selects specific objects by ID, selects all objects, or clears the diagram selection.

```json
{
  "objectIds": ["node1", "node2"],
  "selectMode": "multiple"
}
```
* **Select modes:** `"single"`, `"multiple"`, `"all"`, `"none"`.

### 6. `enableEditingMode`
Toggles diagram interactivity between read-only and interactive read-write modes.

```json
{
  "mode": "readOnly"
}
```
* **Modes:** `"readOnly"`, `"readWrite"` (or boolean `"enabled": false`).

### 7. `arrangeAndAlignObjects`
Aligns, distributes, or resizes multiple selected diagram elements.

```json
{
  "objectIds": ["node1", "node2", "node3"],
  "alignMode": "Left",
  "distributeMode": "EqualSpacingHorizontal"
}
```

### 8. `deleteFromDiagram`
Deletes specified objects, currently selected elements, or all elements from the canvas.

```json
{
  "deleteMode": "selected"
}
```
* **Delete modes:** `"specified"` (requires `objectIds`), `"selected"`, `"all"`.

### 9. `manageUndoRedo`
Performs history operations on the diagram.

```json
{
  "action": "undo"
}
```
* **Actions:** `"undo"`, `"redo"`, `"startGroup"`, `"endGroup"`, `"clearHistory"`.

### 10. `manageClipboardOperations`
Executes clipboard copy, cut, or paste on selected diagram elements.

```json
{
  "operation": "copy"
}
```
* **Operations:** `"copy"`, `"cut"`, `"paste"`.

### 11. `exportAndImportDiagrams`
Exports diagram graphics, saves/loads JSON or Mermaid schema, or prints the canvas.

```json
{
  "operation": "export",
  "format": "png",
  "filename": "ArchitectureDiagram",
  "download": true,
  "fitToPage": true
}
```
* **Operations:** `"save"`, `"export"`, `"load"`, `"import"`, `"print"`.  
* **Formats:** `"json"`, `"mermaid"`, `"png"`, `"jpg"`, `"jpeg"`, `"svg"`.

### 12. `arrangeZOrder`
Changes visual stacking order for selected elements.

```json
{
  "objectIds": ["node1"],
  "order": "BringToFront"
}
```
* **Orders:** `"BringToFront"`, `"SendToBack"`, `"BringForward"`, `"SendBackward"`.

### 13. `navigateAndZoomDiagram`
Pans, zooms, resets viewport, or centers on specific objects.

```json
{
  "action": "zoom",
  "zoomFactor": 1.2
}
```
* **Actions:** `"zoom"`, `"pan"`, `"fitToPage"`, `"reset"`, `"bringIntoView"`.

### 14. `manageGroupsAndHierarchies`
Groups or ungroups elements by ID or current selection.

```json
{
  "operation": "group",
  "parentId": "group1",
  "childIds": ["node1", "node2"]
}
```
* **Operations:** `"group"`, `"ungroup"`.

### 15. `manageAnnotationsAndLabels`
Adds, updates, or deletes textual labels on nodes or connectors.

```json
{
  "objectId": "node1",
  "operation": "add",
  "labelText": "Approved",
  "fontSize": 14,
  "color": "#12B886",
  "bold": true
}
```
* **Operations:** `"add"`, `"update"`, `"delete"`.

### 16. `managePorts`
Adds or removes connection ports on nodes or connectors.

```json
{
  "nodeId": "node1",
  "action": "add",
  "portDefinitions": [
    {
      "id": "port1",
      "offset": { "x": 0.5, "y": 1.0 },
      "visibility": "Visible",
      "shape": "Circle",
      "width": 10,
      "height": 10
    }
  ]
}
```
* **Actions:** `"add"`, `"remove"`.

### 17. `applyLayoutToNodes`
Executes automatic diagram layout algorithms across nodes and connectors.

```json
{
  "layoutType": "HierarchicalTree",
  "orientation": "TopToBottom",
  "horizontalSpacing": 40,
  "verticalSpacing": 50
}
```
* **Layout types:** `"HierarchicalTree"`, `"OrganizationalChart"`, `"MindMap"`, `"ComplexHierarchicalTree"`, `"RadialTree"`, `"Flowchart"`, `"ForceDirectedTree"`.

### 18. `createUmlSequenceDiagram`
Builds a complete UML sequence diagram with lifelines, messages, and interaction fragments.

```json
{
  "participants": [
    { "id": "client", "content": "Web Client", "isActor": true },
    { "id": "api", "content": "API Gateway", "isActor": false }
  ],
  "messages": [
    {
      "id": "msg1",
      "content": "POST /order",
      "fromParticipantId": "client",
      "toParticipantId": "api",
      "messageType": "synchronous"
    }
  ],
  "fragments": [],
  "spaceBetweenParticipants": 150
}
```

---

## Retrieve Registered Tools

To query the tools currently registered with the diagram at runtime, call the `GetWebMcpToolsAsync` method on the `SfDiagramComponent` instance.

```cshtml
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent @ref="_diagram" Width="100%" Height="600px" EnableWebMcp="true">
    <DiagramWebMcpSettings Name="diagram" />
</SfDiagramComponent>

<button @onclick="EnumerateToolsAsync">List Registered Tools</button>

@code {
    private SfDiagramComponent? _diagram;

    private async Task EnumerateToolsAsync()
    {
        if (_diagram == null) return;

        // Retrieve all registered WebMCP tools
        IReadOnlyList<IDiagramWebMcpTool> tools = await _diagram.GetWebMcpToolsAsync();

        foreach (IDiagramWebMcpTool tool in tools)
        {
            Console.WriteLine($"Tool: {tool.Name}");
            Console.WriteLine($"Description: {tool.Description}");
            Console.WriteLine($"Input Schema: {tool.InputSchema}");
        }

        // Optionally filter by specific tool names
        IReadOnlyList<IDiagramWebMcpTool> filtered =
            await _diagram.GetWebMcpToolsAsync(new[] { "createDiagramNode", "deleteFromDiagram" });
    }
}
```

* **Method signature:** `Task<IReadOnlyList<IDiagramWebMcpTool>> GetWebMcpToolsAsync(IEnumerable<string>? toolNames = null)`
* **Return value:** A read-only collection of `IDiagramWebMcpTool` metadata exposing `Name`, `Description`, and `InputSchema`.
* The returned `Name` is the base tool name (e.g., `createDiagramNode`), excluding the registration namespace prefix.

---

## Create Custom Tools

You can register application-specific tools by adding a `<DiagramWebMcpTool>` with `Command="@DiagramWebMcpToolCommand.Custom"`. Custom tools require:

* **Name:** Unique tool identifier exposed to the caller.
* **Description:** Plain-text description used by AI agents to decide when to call the tool.
* **InputSchema:** Valid JSON Schema string describing the expected input parameters.

Execution of custom tools is handled inside the [OnWebMcpExecute](#handle-tool-execution) callback, where your application must assign the JSON response string to `args.Result`.

```cshtml
@using System.Text.Json
@using Syncfusion.Blazor.Diagram

<SfDiagramComponent @ref="_diagram"
                    Width="100%"
                    Height="600px"
                    EnableWebMcp="true"
                    OnWebMcpExecute="HandleWebMcpExecuteAsync">
    <DiagramWebMcpSettings Name="app">
        <DiagramWebMcpTools>
            <!-- Built-in tool -->
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.GetDiagramCatalog" />

            <!-- Custom Application Tool -->
            <DiagramWebMcpTool Command="@DiagramWebMcpToolCommand.Custom"
                               Name="getDiagramStatistics"
                               Description="Returns node and connector counts for the current diagram."
                               InputSchema="@_statsSchema" />
        </DiagramWebMcpTools>
    </DiagramWebMcpSettings>
</SfDiagramComponent>

@code {
    private SfDiagramComponent? _diagram;

    private readonly string _statsSchema = JsonSerializer.Serialize(new
    {
        type = "object",
        properties = new
        {
            includeSelectedCount = new
            {
                type = "boolean",
                description = "Whether to include count of selected objects."
            }
        }
    });

    private Task HandleWebMcpExecuteAsync(DiagramWebMcpExecuteEventArgs args)
    {
        if (args.Command == DiagramWebMcpToolCommand.Custom && args.Name == "getDiagramStatistics")
        {
            bool includeSelected = false;
            if (args.Arguments != null &&
                args.Arguments.TryGetValue("includeSelectedCount", out object? val) &&
                val is bool b)
            {
                includeSelected = b;
            }

            var response = new
            {
                totalNodes = _diagram?.Nodes.Count ?? 0,
                totalConnectors = _diagram?.Connectors.Count ?? 0,
                selectedNodes = includeSelected ? (_diagram?.SelectionSettings?.Nodes?.Count ?? 0) : 0,
                success = true
            };

            // Custom tools must assign the serialized response string to args.Result
            args.Result = JsonSerializer.Serialize(response);
        }

        return Task.CompletedTask;
    }
}
```

---

## Handle Tool Execution

The `OnWebMcpExecute` event is triggered before any tool (built-in or custom) executes. The callback receives a `DiagramWebMcpExecuteEventArgs` instance with the following properties:

| Property | Type | Direction | Description |
|---|---|---|---|
| `Command` | `DiagramWebMcpToolCommand` | Read-only | The command being executed (`Custom` or a built-in command). |
| `Name` | `string` | Read-only | Base tool name (excluding the registration prefix). |
| `Arguments` | `IReadOnlyDictionary<string, object>?` | Read-only | Parsed CLR dictionary of input arguments sent by the caller. Whole numbers are parsed as `long`, floating point as `double`, nested objects as dictionaries. |
| `Cancel` | `bool` | Read / Write | Set to `true` to cancel tool execution. |
| `ShowConfirmationDialog` | `bool` | Read / Write | Set to `true` to prompt the user with a browser confirmation dialog prior to running a built-in tool. |
| `CancellationResponse` | `string?` | Read / Write | Custom message returned to the agent when execution is cancelled or confirmation is declined. |
| `Result` | `string?` | Read / Write | JSON payload returned to the caller. **Required** for custom tools; overrides built-in tool output when set. |

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
            foreach (var kvp in args.Arguments)
            {
                Console.WriteLine($" - {kvp.Key}: {kvp.Value}");
            }
        }

        return Task.CompletedTask;
    }
}
```

---

## Cancel Tool Execution

Set `args.Cancel = true` to abort tool execution before it runs. You can also specify an optional `CancellationResponse` explaining why the action was rejected.

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
        // Prevent deletion of diagram elements based on business rules
        if (args.Command == DiagramWebMcpToolCommand.DeleteFromDiagram)
        {
            args.Cancel = true;
            args.CancellationResponse = "Deleting diagram objects is disabled in this view.";
            return Task.CompletedTask;
        }

        // Validate arguments before executing
        if (args.Command == DiagramWebMcpToolCommand.CreateDiagramNode &&
            args.Arguments != null &&
            args.Arguments.TryGetValue("shapeFamily", out object? family) &&
            family?.ToString() == "html")
        {
            args.Cancel = true;
            args.CancellationResponse = "Policy error: HTML template nodes cannot be created via AI.";
        }

        return Task.CompletedTask;
    }
}
```

Use cancellation to:
* Enforce application-level permissions and role-based access control.
* Restrict editing during read-only sessions or review modes.
* Block forbidden node shapes, dimensions, or connectors.

---

## Show Confirmation Dialog

Set `args.ShowConfirmationDialog = true` to prompt the user with a browser confirmation dialog before a built-in tool executes.

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
        // Require user confirmation before destructive actions
        if (args.Command == DiagramWebMcpToolCommand.DeleteFromDiagram ||
            args.Command == DiagramWebMcpToolCommand.ManageClipboardOperations)
        {
            args.ShowConfirmationDialog = true;
            args.CancellationResponse = "The user declined the operation prompt.";
        }

        return Task.CompletedTask;
    }
}
```

* If the user **confirms**, the tool executes normally against the diagram.
* If the user **cancels**, the tool is aborted and the agent receives the message specified in `CancellationResponse`.
* `ShowConfirmationDialog` applies to built-in tools.

---

## Override Tool Execution

You can intercept any built-in tool call and return a custom result by assigning a valid JSON string to `args.Result`. When `args.Result` is populated, the Diagram skips its internal execution switch and directly returns your result to the caller:

```cshtml
@using System.Text.Json
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
        // Intercept catalog discovery and provide custom catalog metadata
        if (args.Command == DiagramWebMcpToolCommand.GetDiagramCatalog)
        {
            var customCatalog = new
            {
                allowedShapes = new[] { "Rectangle", "Circle", "Diamond" },
                maxNodes = 50,
                customTheme = "CorporateBlue"
            };

            args.Result = JsonSerializer.Serialize(customCatalog);
        }

        return Task.CompletedTask;
    }
}
```

---

## Complete Custom Tool Example

The following complete sample demonstrates:
1. Registering both built-in and custom tools with `<DiagramWebMcpSettings>`.
2. Defining a JSON Schema for custom tool parameters.
3. Reading typed arguments in `OnWebMcpExecute`.
4. Creating a diagram node programmatically in response to a custom tool call.
5. Returning structured JSON results.

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

---

## API Reference

### `SfDiagramComponent`

| Property / Method | Type | Description | Default |
|---|---|---|---|
| `EnableWebMcp` | `bool` | Enables or disables WebMCP tool registration for this diagram instance. | `false` |
| `OnWebMcpExecute` | `EventCallback<DiagramWebMcpExecuteEventArgs>` | Event callback invoked before any WebMCP tool executes. | `default` |
| `GetWebMcpToolsAsync(IEnumerable<string>? toolNames = null)` | `Task<IReadOnlyList<IDiagramWebMcpTool>>` | Returns read-only metadata (`Name`, `Description`, `InputSchema`) of registered tools. | — |

---

### `DiagramWebMcpSettings`

| Property | Type | Description | Default |
|---|---|---|---|
| `Name` | `string` | Prefix namespace identifying the diagram instance's tools in the browser registry (e.g., `Name="diagram"` produces names like `diagram_createDiagramNode`). | `string.Empty` |
| `ChildContent` | `RenderFragment?` | Contains `<DiagramWebMcpTools>` defining the selective tool catalog. When omitted or empty, all 18 built-in tools are registered. | `null` |

---

### `DiagramWebMcpTools`

| Property | Type | Description | Default |
|---|---|---|---|
| `ChildContent` | `RenderFragment?` | Contains the collection of `<DiagramWebMcpTool>` elements. | `null` |

---

### `DiagramWebMcpTool`

| Property | Type | Description | Default |
|---|---|---|---|
| `Command` | `DiagramWebMcpToolCommand` | Selects a built-in command or specifies `Custom` for application-defined tools. *(Required)* | — |
| `Name` | `string?` | Overrides the canonical tool name. Required when `Command="DiagramWebMcpToolCommand.Custom"`. | `null` |
| `Description` | `string?` | Overrides the canonical tool description. Required when `Command="DiagramWebMcpToolCommand.Custom"`. | `null` |
| `InputSchema` | `string?` | JSON Schema string describing input arguments. Required for `Custom` tools. | Canonical schema |
| `Enabled` | `bool` | When `false`, omits the tool from browser registration while still validating configuration. | `true` |

---

### `DiagramWebMcpToolCommand`

| Member | Canonical Name | Annotations | Description |
|---|---|---|---|
| `Custom` | *(User Defined)* | User-defined | Customer-supplied tool handled through `OnWebMcpExecute`. |
| `GetDiagramCatalog` | `getDiagramCatalog` | Read-only, Idempotent | Discovers supported shapes, enums, BPMN/UML types, swimlane capabilities, and styling properties. |
| `CreateDiagramNode` | `createDiagramNode` | Standard | Creates one or more nodes with shapes, geometry, headers, styling, and annotations. |
| `CreateDiagramConnector` | `createDiagramConnector` | Standard | Creates straight, orthogonal, or bezier connectors between nodes or ports. |
| `DragNode` | `dragNode` | Standard | Translates nodes or connectors by relative offsets (`tx`, `ty`). |
| `ManageNodeSelection` | `manageNodeSelection` | Idempotent | Selects objects by ID, selects all, or clears selection (`single`, `multiple`, `all`, `none`). |
| `EnableEditingMode` | `enableEditingMode` | Destructive, Idempotent | Switches diagram between read-only and interactive read-write modes. |
| `ArrangeAndAlignObjects` | `arrangeAndAlignObjects` | Standard | Aligns, distributes, or resizes selected diagram objects. |
| `DeleteFromDiagram` | `deleteFromDiagram` | Destructive | Deletes specified objects, selected objects, or all diagram elements. |
| `ManageUndoRedo` | `manageUndoRedo` | Destructive | Triggers undo, redo, starts/ends history groups, or clears history stacks. |
| `ManageClipboardOperations` | `manageClipboardOperations` | Destructive | Performs clipboard copy, cut, or paste on current selection. |
| `ExportAndImportDiagrams` | `exportAndImportDiagrams` | Destructive | Exports/imports JSON or Mermaid formats; exports PNG, JPEG, SVG, or triggers print. |
| `ArrangeZOrder` | `arrangeZOrder` | Standard | Moves objects `BringToFront`, `SendToBack`, `BringForward`, or `SendBackward`. |
| `NavigateAndZoomDiagram` | `navigateAndZoomDiagram` | Idempotent | Viewport navigation: pan, zoom, reset, fit to page, or bring into view. |
| `ManageGroupsAndHierarchies` | `manageGroupsAndHierarchies` | Standard | Groups or ungroups selected objects or explicit `parentId` / `childIds`. |
| `ManageAnnotationsAndLabels` | `manageAnnotationsAndLabels` | Standard | Adds, updates, or deletes labels/annotations on nodes or connectors. |
| `ManagePorts` | `managePorts` | Standard | Adds, removes, or positions connection ports on nodes or connectors. |
| `ApplyLayoutToNodes` | `applyLayoutToNodes` | Standard | Applies automatic layout algorithms (Hierarchical, Radial, MindMap, OrgChart, etc.). |
| `CreateUmlSequenceDiagram` | `createUmlSequenceDiagram` | Standard | Generates UML sequence diagrams with participants, messages, and fragments. |

---

### `DiagramWebMcpExecuteEventArgs`

| Property | Type | Direction | Description |
|---|---|---|---|
| `Command` | `DiagramWebMcpToolCommand` | Read-only | The command being executed (`Custom` or built-in). |
| `Name` | `string` | Read-only | Base tool name (excluding the registration prefix). |
| `Arguments` | `IReadOnlyDictionary<string, object>?` | Read-only | Deep JSON-to-native CLR mapped dictionary of input arguments supplied by the caller. |
| `Cancel` | `bool` | Read / Write | Set to `true` to cancel tool execution. Default is `false`. |
| `ShowConfirmationDialog` | `bool` | Read / Write | Set to `true` to display a native browser confirmation prompt before executing a built-in tool. Default is `false`. |
| `CancellationResponse` | `string?` | Read / Write | Custom response returned to the caller when execution is cancelled or confirmation is rejected. Default is `null`. |
| `Result` | `string?` | Read / Write | The JSON result payload returned to the WebMCP caller. **Mandatory** for `Custom` tools; optionally overrides built-in tool output. Default is `null`. |

---

### `IDiagramWebMcpTool`

| Property | Type | Description |
|---|---|---|
| `Name` | `string` | The base tool name. |
| `Description` | `string` | The human-readable description of the tool. |
| `InputSchema` | `object?` | JSON Schema object describing the tool's input parameters. |

---

## Best Practices

* **Use descriptive names and instructions**: When overriding descriptions or defining custom tools, write clear instructions so AI models understand exactly when and how to call the tool.
* **Define strict input schemas**: Always specify types, descriptions, and `required` fields in your custom JSON schemas to prevent hallucinated arguments.
* **Validate arguments defensively**: Read arguments from `args.Arguments` using safe pattern matching or `TryGetValue` before applying them to diagram elements.
* **Protect destructive operations**: Use `args.ShowConfirmationDialog = true` for irreversible operations like `DeleteFromDiagram` or `ManageClipboardOperations`.
* **Return structured JSON responses**: In custom tools, always assign a serialized JSON object to `args.Result` containing `success` and relevant metadata so agents can reason over the outcome.
* **Leverage cancellation for security**: Enforce role-based access control or diagram lock states by setting `args.Cancel = true` with a descriptive `args.CancellationResponse`.
* **Namespace multi-diagram pages**: Always supply a unique `Name` in `<DiagramWebMcpSettings>` when hosting more than one diagram on the same page.

---

## See Also

* [Getting Started with Blazor Diagram Component](https://blazor.syncfusion.com/documentation/diagram/getting-started-with-web-app)
* [Nodes in Blazor Diagram Component](https://blazor.syncfusion.com/documentation/diagram/nodes)
* [Connectors in Blazor Diagram Component](https://blazor.syncfusion.com/documentation/diagram/connectors)
* [Automatic Layouts in Blazor Diagram Component](https://blazor.syncfusion.com/documentation/diagram/layout/automatic-layout)
* [Undo and Redo Support in Blazor Diagram Component](https://blazor.syncfusion.com/documentation/diagram/undo-redo)
