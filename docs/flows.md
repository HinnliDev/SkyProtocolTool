<div align="center">

# Flows guide

[![Русский](https://img.shields.io/badge/lang-Русский-blue)](flows_ru.md)

Flows are scenarios built from connected nodes. Saved JSON may contain URLs,
request bodies and custom headers. Do not include secret values in flows you
intend to share; use separately stored named secrets instead.

</div>

---

## Quick start

1. Open Flows. With no library card selected, the canvas is an editable draft.
2. Tap empty canvas space and choose a type in Add node.
3. Select the node and configure the lower Node settings panel. If hidden,
   enable it using the bottom-panel button.
4. Add Notification. Connect the first node's Flow output beside its title
   to Notification's input beside its title.
5. Enter `Status: {{status}} | {{response.msg}}` in Notification text.
6. Press Run to start the flow.
7. Save creates a Saved flows card; subsequent saves update its file.

Save, Run and Repeat are unavailable on an empty canvas. One node is enough
for the first save. Saving does not run a scenario.

Ready-to-use scenarios are available in [Example flows](#example-flows).

## Canvas and controls

- Drag empty space to pan. Hold, then drag to create the red selection rectangle.
  Another rectangle adds nodes to the existing selection.
- One selected node opens Node settings; multiple selection disables the panel.
  Dragging a selected node moves the selected group.
- A blank-space tap first clears selection; another tap opens Add node.
- `+`, `−` and Center control zoom. The percentage shows the position between
  minimum and maximum zoom. At 25% or below, text and icons are hidden.
- The minimap shows the graph and viewport. Moving nodes, the selection rectangle
  or a connection near the canvas edge pans automatically.
- Delete removes selected nodes; while dragging, dropping on Delete also removes them.
  Clear empties the canvas immediately and can be undone.
- Copy copies the selection and its internal connections. Paste preserves relative positions,
  assigns new IDs and updates qualified references to the copied sources. A group has a white
  paste icon; a single node keeps its type color. External connections are not copied.
  Paste places the copy at the current viewport center, allowing overlap. Hold Paste to clear it.
- Adding a new node finds free world space. Capacity is calculated from the whole world and
  the largest card, including menu font scale (up to 4,464 default-sized nodes).
  When capacity or free space is exhausted, addition is unavailable; groups paste atomically.
- Undo/Redo restore nodes, settings, positions and connections. The top Reset
  restores the last saved graph.

Selecting another library card or deselecting the current card discards its
unsaved changes and Undo/Redo history. The file does not change until Save.
Cards support rename, favorites, reorder and delete. Refresh rereads the folder;
Import flow adds JSON.

## Connections

One right-side output can feed multiple inputs. Compatible input circles fill
only while dragging a connection. Dropping on empty
space opens Add node and automatically connects the new node.

The circles beside the title are Flow ports for execution order.
Other ports expose Method, URL/Endpoint, Headers, Body, Response and Status.
Every connection creates a dependency: a node waits for **all** incoming sources.
Condition skips the inactive branch. Collect also accepts skipped branches and
joins the results of the active ones; other nodes require all inputs to be active.
Nodes without inputs start independently. Cycles are forbidden; Repeat reruns
the entire graph instead.

Direct Method, URL/Endpoint and Headers connections take their values from
the source and disable manual settings. Body, Notification text, Request contains
and Response contains stay editable: connecting adds a `{{variable}}` reference
that you can combine with your own text. They accept multiple connected ports.
Condition values and Save/Read file names are editable templates too. Collect's
Data input collects the connected values directly and needs no text field.

Grab a connected input circle to move an existing edge. Repeated grabs cycle
through multiple edges. Drop the edge on the bottom Cancel button to delete it
without opening Add node.

## Node types

| Node | Behavior and settings |
| --- | --- |
| Intercept | Waits for a completed HTTP exchange or a matching WS message. In HTTP mode, ANY accepts all methods; an empty Endpoint accepts all endpoints. Request contains/Response contains are substring filters, not regular expressions. Skip first ignores N matches; Delay postpones completion after a match. |
| Rewrite | Delay postpones arming the node without blocking the network thread. Matches Method, URL/Endpoint and Request contains. After Skip first, edits the selected outgoing request or incoming response body, without sending a duplicate. Completes after the exchange. |
| Request | Sends Method, URL/Endpoint, Headers and Body. Delay is before sending; Timeout limits response waiting; Retry count means additional attempts. An absolute URL chooses a server; an endpoint beginning with `/` uses the current client server. |
| Socket | Sends UTF-8 text or JSON through an existing WebSocket/MQTT connection. Set URL/Endpoint, Topic, Body, Delay and Timeout. Topic is required for MQTT and empty for ordinary WS. URL, Topic and Body support templates. Does not open a new connection. |
| Action | Invokes a menu operation: Switch account (choose a saved account), Relog account, Play route, Stop route, Play animation or Stop animation. Playback starts without waiting for it to finish. Stopping already stopped playback has no effect, but the node still succeeds. |
| Condition | Compares Value with Compare with: equal, not equal, greater, less, contains, exists. Execution follows True or False. A missing JSON path evaluates to False for Exists; invalid sources/JSON are errors. |
| Collect | Aggregates active sources under keys like `request`, `request#2` or `request.body` (for a specific port). As array collects values in connection order. Inactive Condition branches are omitted. |
| Read | Reads JSON from File after Delay. If the file is missing, uses Default value. If empty, an error occurs. Invalid JSON and other read errors are not ignored. The result is available through Response and downstream variables. |
| Save | Writes Body as JSON to File after Delay. Adds `.json` when needed. Invalid JSON cannot overwrite the existing file. |
| Notification | Displays Notification text after Delay. Duration controls display time, default 5000 ms. The node completes immediately after displaying, without waiting for the toast to disappear. |

### HTTP, WebSocket and MQTT

Request is HTTP-only. Intercept and Rewrite have HTTP/WS transport selection.
In WS mode each frame is independent: Request contains filters outgoing frames,
Response contains filters incoming frames. Rewrite's target chooses the direction;
Replace body, Merge fields and Patch operations work on UTF-8 text bodies. Incoming WS replacements must fit
the native 2048-byte buffer. A rule is one-shot per iteration.

Socket uses an existing connection; an inactive or disconnected connection times
out. Sending does not require a new incoming frame. Game WS URLs copy as endpoints;
external origins retain the full URL. A copied textual MQTT PUBLISH keeps its
Topic and sends a new QoS 0 publication without replaying packet IDs. MQTT still
uses binary WS framing, but Socket's Body must be UTF-8 text or JSON. Incoming
messages and opaque non-text frames cannot be copied into Socket.

### Partial Rewrite updates

Select **Request body** (default) or **Response body** under URL/Endpoint.
Response mode leaves the outgoing request unchanged and edits the completed body
before the native caller consumes it. Matching/Skip first still use the original
request. Replacement is in-place and must fit the native response buffer; an
oversized replacement fails the node and leaves the original response unchanged.

Start Rewrite **before** the client sends its matching request. Intercept waits
for a completed exchange, so Intercept → Rewrite modifies the next matching
request, never an already-sent request. Loop rearms after completion.

- Replace body replaces the entire body, including plain text.
- Merge fields applies nested object changes. `null` removes a field; unmentioned
  fields remain. Arrays are replaced in full (JSON Merge Patch).
- Patch operations supports `add`, `replace`, `remove`, `test` and JSON Pointers.
  `/users/1/name` addresses the second array item, `/-` appends;
  `~1` escapes `/` in a key, `~0` escapes `~`.

Merge fields example:

```json
{"name":"New name","extra":true,"obsolete":null}
```

Patch operations example:

```json
[{"op":"replace","path":"/name","value":"New name"},
 {"op":"add","path":"/extra","value":true},
 {"op":"remove","path":"/obsolete"}]
```

Invalid operations/paths or oversized results leave the original request intact
and fail the node. Absolute URL filters also match the origin; endpoint matching
ignores query parameters.

Request's Client headers enables the current client header set. Individual
headers can be disabled; their inputs are read-only. Custom headers can be
edited, disabled or removed. Header names are case-insensitive: an enabled
custom value overrides the matching client value. Client headers are automatically
omitted for external services.

Received HTTP 4xx/5xx responses remain available downstream through Status/Response.
Transport failures, timeouts and variable errors stop the flow after retries
are exhausted. Stop cancels scheduling and waiting; it cannot undo a request
already sent to a server.

## Secrets

Disabling Client secrets only changes the visibility of client data in the interface;
transmitted values remain unchanged. Custom secret values are visually masked
with `***` and can be revealed while editing.

Add a custom name/value, for example `telegram_bot` and a bot key, then use
`https://example.test/bot{{secret.telegram_bot}}/send` in a URL. The name field has a fixed `secret.` prefix; enter only `telegram_bot`. The same expressions work
in headers, filters, bodies and Notification text. Only actual hidden values
appear as gray `***` outside active editing, including URLs in Traffic stream.
References such as `{{secret.telegram_bot}}` and `{{client.session}}` stay visible.
Named secrets are not automatically renamed when connecting another source.
The Network Inspector shows the original request and response, including secrets;
masking affects the Traffic stream preview, not transmitted data.

Custom secrets are saved locally in the general cache, separately from flow JSON,
in plain text (masking is not encryption). Do not share that cache. Literal keys
typed directly into a flow are still stored in its JSON; prefer named references.

## Variables

`{{client.user}}`, `{{client.user_id}}` and `{{client.session}}` use current client data.
Other variables come from **directly connected** previous nodes.
Display names do not affect references and can contain spaces.

| Sources | Example |
| --- | --- |
| One | `{{users[1].age[1]}}` or `{{status}}` |
| Multiple Request nodes | `{{request.users[1].age[1]}}`, `{{request#2.status}}` |
| Different types | `{{request.status}}`, `{{intercept.response.msg}}`, `{{rewrite.response}}` |

`#2` identifies the second node of that type in the graph, regardless of its name.
Array indices are zero-based: `[1]` selects the second element.
The `response.` prefix is optional for response JSON properties:
`{{msg}}` equals `{{response.msg}}`. Use the prefix to disambiguate a response
property named `method` or `status`; bare names refer to HTTP metadata.

Metadata: `method`, `url`, `endpoint` (path including query), `headers`, `body`
(outgoing body), `response` (whole response), `status` and Notification's `text`.
Use `{{body.lvl}}`, `{{request.body.users[0].name}}` for fields of request JSON,
and `{{headers.Content-Type}}` for a captured header. Bare fields read the response.
Network node copying replaces JSON `user`, `user_id` and `session` values with
`{{client.user}}`, `{{client.user_id}}` and `{{client.session}}`.
Objects/arrays are substituted as JSON.

### Expressions

Expressions preserve JSON types. `exists` and `??` treat only missing/null as
absent: `0`, `false` and `""` exist. An ordinary missing variable still stops the
flow. Use an optional expression or fallback when absence is expected.

| Purpose | Example |
| --- | --- |
| Presence / absence | `{{exists(request.user.points)}}`, `{{!exists(request.user.points)}}` |
| Default | `{{request.user.points ?? 0}}` |
| Comparisons | `{{request.user.level >= 3}}` (`==`, `!=`, `>`, `<`, `>=`, `<=`) |
| Logic | `{{request.user.points > 0 && request.user.level > 0}}` (`&&`, `\|\|`, `!`) |
| Choice | `{{request.user.active ? "Active" : "Inactive"}}` |
| Prefix | `{{"Name: "?request.user.name}}` |
| Suffix | `{{request.user.name?" (user)"}}` |
| Both | `{{"User: "?request.user.name?" (active)"}}` |
| Text when absent | `{{!request.user.name?"Unknown user"}}` |
| Optional field | `{{optional("Name: ", request.user.name)}}` |
| Existing fields only | `{{join(" \| ", optional("Name: ", request.user.name), optional("Level: ", request.user.level))}}` |
| Concatenation | `{{"Name: " + request.user.name}}` |
| Substring | `{{contains(request.user.name, "Ali")}}` |
| Array to text | `{{join(", ", request.items)}}` |
| Size | `{{length(request.items)}}` |
| Case | `{{upper(request.user.name)}}`, `{{lower(request.user.name)}}` |
| Trim / replace | `{{trim(request.user.name)}}`, `{{replace(request.user.name, "A", "B")}}` |
| Split | `{{split(request.message, ",")}}` |
| Format | `{{format("Name: {}", request.user.name)}}` |
| Aggregates | `{{sum(request.numbers)}}`, `{{min(request.numbers)}}`, `{{max(request.numbers)}}` |
| Round | `{{round(request.total, 2)}}` |
| Conversion / type | `{{number(request.user.points)}}`, `{{string(request.user.points)}}`, `{{type(request.user.points)}}` |

Precedence (high to low): parentheses/functions, `!`, `+`, relational operators,
equality, `&&`, `||`, `??`, then `?:` and short `?` affixes. Use parentheses when
mixing affixes with a ternary. `&&`, `||`, `??` and `?:` evaluate only the needed
branch. The short `!field?"text"` tests absence, not whether the field is false.
Equality is typed (`1 != "1"`); relational operators require numbers.
Two numeric operands to `+` add; otherwise it joins text.

`join` omits missing/null/empty parts, without extra separators. `optional` also
accepts a third suffix argument. `length` counts UTF-8 characters, array elements
or object members. `upper`/`lower` cover ASCII, Latin-1 and Russian. `split` returns
a JSON array; an empty separator splits characters. `format` uses `{}` and
escapes literal braces as `{{`/`}}`. `round` accepts 0–12 decimal places.
For an empty array, `sum` returns 0 and `min`/`max` return null. Aggregates require numeric elements.
`number` accepts a JSON numeric string or boolean; invalid conversion is an error.
`type` returns `missing`, `null`, `boolean`, `number`, `string`, `array` or `object`.
Expressions cannot execute code, access files or make requests. Depth, work,
array creation and output size are bounded.

## Notification formatting

The round heading icon cycles through the badge variants. The palette button
before Rename sets an opaque RGB color; the canvas, copied node and toast use
the same appearance. Both values are saved with the node and support Undo/Redo.

Notification supports simple formatting: `**text**` for bold text and
`` `text` `` for gray text.

Use an actual newline or `\n` to break lines.
Only literal Notification text expands `\n`. Escapes received through `{{body}}`,
`{{response}}` or other substitutions are not interpreted a second time.

```text
**Status:** {{status}}\nResult: {{response.msg}}
```

## Execution and files

Yellow node/port outlines mean working or waiting; green means completion,
red means failure. Results stay visible for 3 seconds. A thin highlight travels
only when a result is passed along an edge, not continuously while waiting.
Interface animations controls animation, not execution.

Action's Switch account waits for account preparation/apply, not server
authentication. Connect an Intercept afterwards to wait for the login response.
Play route/animation starts playback and immediately continues the flow; Play
animation also selects the clip in the Animations library. Stopping a flow
cancels pending nodes, not an account switch or playback already started.
Use explicit Stop route/Stop animation actions to stop playback.

Saved scenarios live in `SPT/Flows/Scenarios`; Save/Read data is in `SPT/Flows/Data`.
Save/Read accept a file name, not a path; `.json` is added if missing.
File operations run in the background and accept at most 256 KiB of valid JSON.
The schema contains `schema: "spt_node_v1"`, name, the repeat flag `loop` and `nodes`.
Node IDs are numbers starting at 1; positions use `pos: {x, y}`. Each node stores
outgoing edges in `connections`:
a number is a Flow edge; `{node, output, input}` connects named ports.
`custom_headers` is one name → value object with disabled names in a separate list.
`client_headers` is boolean. Notification stores `delay_ms` and `duration_ms`.

Review imported scenarios before running; the import warning precedes file selection.

## Example flows

### Wax collection and candle forging

A ready-to-use scenario that automates wax collection and candle forging.
Saves progress, skips completed steps, and resumes execution after interruptions.

[Download scenario](../examples/flows/Sky_Wax_EN.json) and import it using **Import flow** in the Flows tab.
