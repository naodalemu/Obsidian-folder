## Example “one-shot” ask you can paste
## 1. 
“Create reusable React + Tailwind components WorkflowLayout, WorkflowRow, WorkflowBox with the APIs above. Use a 5-column grid to the right of a 160px label column. Implement visual variants: step (light gray), outcome (light blue), tag (gray pill), note (rose pill), default (dark gray). Then create src/workflow.jsx and assemble these rows:

- Steps: [Initial Step, Stirring Together, Fermenting, Baking, Packaging] as step boxes across columns 1–5.
- Ingredients: tags [Sugar, Orange, Water, Lime, Corn].
- Outcome: [Dough, Soft Dough, Fermented Dough, Hot Biscuit, Packaged Biscuit] as outcome boxes across columns 1–5.
- Loss Types: one tag per column, label ‘Loss type over x,xxx | $x.xx’.
- Other rows: [Other Food Production, Research, Fuel Production, Manure, Destroyed] each containing a single pink note ‘[Input type] x,xx | $x.xx’ centered in its column.

Keep page background dark, add subtle vertical guide lines aligned to columns, and side padding on small screens.”

## 2. 
### Data-driven option (even shorter)
Provide a JSON-like spec and ask me to render it:

```
{
  "columns": 5,
  "rows": [
    { "label": "Steps", "type": "grid", "items": [
      { "col": 1, "variant": "step", "title": "Initial Step" },
      { "col": 2, "variant": "step", "title": "Stirring Together" },
      { "col": 3, "variant": "step", "title": "Fermenting" },
      { "col": 4, "variant": "step", "title": "Baking" },
      { "col": 5, "variant": "step", "title": "Packaging" }
    ]},
    { "label": "Ingredients", "type": "tags", "items": ["Sugar","Orange","Water","Lime","Corn"] },
    { "label": "Outcome", "type": "grid", "items": [
      { "col": 1, "variant": "outcome", "title": "Dough" },
      { "col": 2, "variant": "outcome", "title": "Soft Dough" },
      { "col": 3, "variant": "outcome", "title": "Fermented Dough" },
      { "col": 4, "variant": "outcome", "title": "Hot Biscuit" },
      { "col": 5, "variant": "outcome", "title": "Packaged Biscuit" }
    ]},
    { "label": "Loss Types", "type": "grid", "items": [
      { "col": 1, "variant": "tag", "title": "Loss type over x,xxx | $x.xx" },
      { "col": 2, "variant": "tag", "title": "Loss type over x,xxx | $x.xx" },
      { "col": 3, "variant": "tag", "title": "Loss type over x,xxx | $x.xx" },
      { "col": 4, "variant": "tag", "title": "Loss type over x,xxx | $x.xx" },
      { "col": 5, "variant": "tag", "title": "Loss type over x,xxx | $x.xx" }
    ]},
    { "label": "Other Food Production", "type": "note", "item": { "col": 2, "title": "[Input type] x,xx | $x.xx" } },
    { "label": "Research", "type": "note", "item": { "col": 2, "title": "[Input type] x,xx | $x.xx" } },
    { "label": "Fuel Production", "type": "note", "item": { "col": 3, "title": "[Input type] x,xx | $x.xx" } },
    { "label": "Manure", "type": "note", "item": { "col": 4, "title": "[Input type] x,xx | $x.xx" } },
    { "label": "Destroyed", "type": "note", "item": { "col": 5, "title": "[Input type] x,xx | $x.xx" } }
  ]
}
```

Then say: “Use this spec with the previously described components to render the page at src/workflow.jsx.”
This level of detail is enough for me to build the same UI without any image.

### Outcome:
![[Pasted image 20251016183522.png]]



## Alternate path instead of #2:

## 3.
### Further explanation about columns in loss types
Everything below loss types have it's own colums. Which means each loss types will have thier own multiple columns inside them. so if there are 3 loss types on initial step then all 3 should be side by side and the copy of that loss type is also going to be in one of the areas under loss types, it could be in other food production, or research or any of the avalable waste taking places. Everything under the row loss types is where those losses would go (like where the waste is gonna go) so each of them would have their own sides perpendicular to their copy at the loss types.

### Outcome:
![[Pasted image 20251016184941.png]]


### Conclusion:
I generated the page with the help of the design image and it gave me an okay design then I asked the AI itself which way would be good to ask it to make the code it just made without any image and it gave me the values above, including #1 and #2. The issue with this approach is it's greatly technical. The words and instructions have a clear technical planning in it and also the user defines the structure of the components, how they are going to be, the #2 makes it worse because that needs additional skill and eye to generate a JSON data from the mind.
### Recommendation: 
It is better to have a rough sketch, even if it's not a detailed high fidelity design a low fidelity one would be okay to give it idea about the structure. Specially when it's a complex and uncommon type of structure we are trying to build.