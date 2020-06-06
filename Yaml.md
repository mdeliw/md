Yaml

[toc]

# Yaml

> Remember to use two `spaces` not `tab` .

## Basic Data Types

- Scalars (strings / numbers / boolean)
- Sequences (arrays / lists)
- Mappings (hashes / dictionaries)

Scalars don’t need quotes unless their value embeds elements that can be confused with yaml syntax. Quotes can be single or double quotes.

```yaml
integer: 25
string: "25"
float: 25.0
boolean: Yes
```

Sequences support multiple levels. Sequences are simply lists of scalars or other data structures.

```yaml
# Blocked sequence
- Cat
- Dog
- Goldfish
- Languages
  - Python
  - Java
 
# Inline sequence
myflags: [GREEN, YELLOW]
mynestedflags: [APPLe, [GREEN, YELLOW], BANANA]
```

Mappings is also simple, and note that you can mix data types. 

```yaml
animal: pets
pets:
  - Cat
  - Dog
  - Goldfish
foot: whatever
bar:
  fruit: apple
  name: steve
```

## Anchors

A yaml node can be anchored and later referenced elsewhere.

```yaml
item:
  - method: UPDATE
  where: &FREE_ITEMS
    - Portable Hole
    - Light Feather
  SellPrice: 0
  BuyPrice: 0
  
npc:
  - method: MERGE
    merge-from: {name: General Goods}
    items: *FREE_ITEMS
```

### Merging Mappings

```yaml
<<: *FREE # allows to merge one or more mappings as inline sequences
```



```yaml
where: {name: !include "itemList/pve_list.yml"}
```

