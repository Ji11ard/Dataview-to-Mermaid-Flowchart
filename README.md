# ReadMe
This dataviewjs script **dynamically renders DQL queries into a Mermaid flow diagram** based on queries and and formatting instructions contained in the file's YAML.

The code **can show either the diagram OR the raw Mermaid code**, which could then be copied/embedded/shared without needing dataview or the underlying source data. 

It has been designed to **require no JS expertise** - with all standard functions available through DQL and the YAML parameters. 
## TLDR
  - All setup is in YAML dataview fields, so no JS expertise required.
  - Handles internal obsidian links - **diagrams are clickable**.
  - Fully customisable names / colours / node shapes / link style / link labels
### Basic Process
  - Save file to your vault containing YAML + dataviewJS script. 
  - Write several DQL queries to list the nodes you want drawn.
  - Write several more to list the relationships between nodes.
  - The underlying dataviewJS will run your queries, process the results, and convert everything into mermaid syntax. 

**Your options are mostly limited by your DQL creativity.**
 
## Example Output
![Image](https://user-images.githubusercontent.com/110291999/183233644-ee6a6318-f085-4077-a735-d8be135d9462.png)

### Note about Sorting
Nodes and links are added to the Mermaid code in the order that they are imported - nodes first, then links. 

Ordering the DQL queries themselves (and SORTing the results within) can be very important for getting a clean layout in your final diagram. 

I suggest playing with these orderings until you are happy with the results - and then moving the raw Mermaid code to a separate file to massage the order of key nodes and links if required. 

## Example YAML Settings
The following parameters can be included in the YAML of the charting page. Alternatively, these could be hard coded into the JS code itself - and the corresponding YAML variables have been listed at the top of the JS code to make this easier.  

However, it may be easier to leave them in the YAML while you're still playing with the DQL queries and their ordering. 

```
---
Direction: "LR"
ShowCode: false
SubGroupNames: ["Authors", "Fiction", "Non-Fiction"]
RemoveOrphans: false
KeepLinksWithoutSource: true
KeepLinksWithoutDest: true

Nodes:
 - 'TABLE WITHOUT ID Sources, "", "{{", "}}", "red" FROM #T/📚_Book WHERE Sources FLATTEN Sources SORT Kind, Topics, Sources'
 - 'TABLE file.name, "([", "])", "yellow" FROM #T/📚_Book WHERE Kind = "Fiction" SORT Topics, Sources'
 - 'TABLE file.name, "([", "])", "aqua" FROM #T/📚_Book WHERE Kind = "Non-Fiction" SORT Topics, Sources'

Links:  
 - 'TABLE WITHOUT ID Sources, file.name, "-- " + dateformat(Published, "dd MMM yyyy") + " -->" FROM #T/📚_Book WHERE Sources FLATTEN Sources'

Styles: 
 - 'classDef Custom1 fill:#DDEEFF,color:#000,stroke:#000,stroke-width:1px'
---
```

## YAML Basics
```
Direction: "LR"
ShowCode: false
SubGroupNames: ["Authors", "Fiction", "Non-Fiction"]
RemoveOrphans: false
KeepLinksWithoutSource: true
KeepLinksWithoutDest: true
```

| Field | Default | Description | 
| - | - | - |
| **Direction** | *"LR"* | The Layout direction for the chart - either "LR" or "TD".|
| **ShowCode** | *false* | Whether to display results as a chart or as raw mermaid code.|
| **SubGroupNames** | *[ ]* | An optional array of subgroup name for each node query. Use "" to hide, or " " for blank.|
| **RemoveOrphans** | *false* | If true, only nodes that participate in relationships (i.e. nodes that ultimately show up in the [[#Link Queries]] will be drawn. If false, all nodes imported by the [[#Node Queries]] will be drawn, regardless of whether they connect to anything else.|
| **KeepLinksWithoutSource** | *true* | If true, unformatted nodes will be created to represent the source of a link when there isn't a real node imported explicitly by [[#Node Queries]]. If false, such 'sourceless' links will be discarded.  Mostly useful for cleaning up when you don't want to add too many filter criteria to your link queries.|
| **KeepLinksWithoutDest** | *true* | If true, unformatted nodes will be created to represent the destination of a link when there isn't a real node imported explicitly by [[#Node Queries]]. If false, such links will be discarded instead.  Mostly useful for cleaning up when you don't want to add too many filter criteria to your link queries.|


## YAML Node Queries
```
Nodes:
 - 'TABLE WITHOUT ID Sources, "", "{{", "}}", "red" FROM #T/📚_Book WHERE Sources FLATTEN Sources SORT Kind, Topics, Sources'
 - 'TABLE file.name, "([", "])", "yellow" FROM #T/📚_Book WHERE Kind = "Fiction" SORT Topics, Sources'
 - 'TABLE file.name, "([", "])", "aqua" FROM #T/📚_Book WHERE Kind = "Non-Fiction" SORT Topics, Sources'
```

Defined in the YAML by a **Nodes:** field, and structured as an array of DQL query strings that each return a set of nodes and formatting information to draw them on the Mermaid diagram. 

Queries should return at least 5 columns as follows (column names, and any extra columns will be ignored):
|   Page   | DisplayName | OpenBracket | CloseBracket | StyleClass |
|-|-|-|-|-|
| [[Test]] |  "😀 Test"  |    "(["     |      "])"    |    "red"    |

- *Column 1* should be a *page object* (though an unlinked reference or string will also work).
- *Column 2* should be a *display name* string. An empty string can be provided and the code will infer a display name from object in Column 1. 
- *Column 3* should be a string containing Mermaid open [bracket syntax](https://mermaid-js.github.io/mermaid/#/flowchart?id=node-shapes).
- *Column 4* should be a string containing the corresponding closing bracket syntax.
- *Column 5* should be the string name of a Style class. The code includes a set of standard colour options to choose from: *red, orange, yellow, green, mint, aqua, blue, purple, pink, grey*.  However [[#Custom Styles]] can also be created.

## YAML Link Queries
```
Links:  
 - 'TABLE WITHOUT ID Sources, file.name, "-- " + dateformat(Published, "dd MMM yyyy") + " -->" FROM #T/📚_Book WHERE Sources FLATTEN Sources'
```

Defined in the YAML by a **Links:** field, and structured as an array of DQL query strings that each return a set of relationships between two nodes to represent the arrows/links on a Mermaid diagram. 

Queries should return at least 3 columns as follows (column names, and any extra columns will be ignored):
|  Source  |   Dest    |    link     |
|-|-|-|
| [[Test]] | [[Test2]] | "-- By -->" |

- *Column 1* should be a *page object* representing the source of the relationship (however an unlinked reference or string will also work).
- *Column 2* should be a *page object* representing the destination of the relationship (however an unlinked reference or string will also work).
- *Column 3* should be a string representation of Mermaid [link syntax](https://mermaid-js.github.io/mermaid/#/flowchart?id=links-between-nodes).

## YAML Custom Styles
```
Styles: 
 - 'classDef Custom1 fill:#DDEEFF,color:#000,stroke:#000,stroke-width:1px'
```

Defined in the YAML by a **Styles:** field, and structured as an array of Mermaid Class format strings. These should include a unique identifier (Custom1 in the example below) that can then be used in any [[#Node Queries]] to style the node.  

- *e.g:* "classDef ==Custom1== fill:#DDEEFF,color:#000,stroke:#000,stroke-width:1px"
- Note that the included dataviewJS code also defines a set of standard colour options to choose from: *red, orange, yellow, green, mint, aqua, blue, purple, pink, grey*; so it is not necessary to define any custom styles unless the defaults aren't suitable. 

## Other Defaults (defined in the dataviewJS itself)
- **def** - *Default = [ open: "[", close: "]", style: "default"]* - an object containing the default open, close, and style parameters used when creating placeholder nodes (Nodes that exist in [[#Link Queries]], but don't exists explicitly in [[#Node Queries]]). 
- **styles.push** - A collection of default style settings for the standard colours: *red, orange, yellow, green, mint, aqua, blue, purple, pink, grey*.
