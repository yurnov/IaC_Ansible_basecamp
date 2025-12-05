## What is YAML

YAML (a recursive acronym for "YAML Ain't Markup Language") is a human-readable data-serialization language. It is commonly used for configuration files and in applications where data is being stored or transmitted. 

YAML uses both Python-style indentation to indicate nesting, and a more compact format that uses `[...]` for lists and `{...}` for maps.

## YAML Syntax (basic)

Three hypen (`---`) used for separation multiple document in same stream, so it commonly to put three hypen in beginning of YAML file.

Comments begins with the number sing (`#`), it can start enywhere and continue till end of the line.

## Example of YAML

Simple list

```
--- # Favorite movies
- Casablanca
- North by Northwest
- The Man Who Wasn't There
```

Examples of keys in two forms:

```
--- # Indented Block
  name: John Smith
  age: 33
--- # Inline Block
{name: John Smith, age: 33}
```

Examples of nesting:

```
--- #List of Students
- student1:
    name: Delilah Wisneski
    age: 17
    sex: Female
- student2:
    name: Colin Bonds
    age: 43
    sex: Male
- student3:
    name: Douglass Burleigh
    age: 27
    sex: Male
    hobby: >
      Furniture building, Gaming (tabletop games and
      role-playing games), Genealogy, Gingerbread house
      making, Glassblowing
- student4:
    name: Loriann Willman
    age: 18
    sex: Female
```

## Future reading

- [YAML Essentials Udemy free course](https://www.udemy.com/course/yaml-essentials/)
- [YAML official site](https://yaml.org/)
- Google it!
