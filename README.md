

## AVPerson Component (slightly overengineered)

[Component page](https://aavainshtein.github.io/front-task/#/person/1)

`AVPerson` is a numeric input component with an avatar, label, and caption.

### Props

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `avatarSrc` | `string` | ✅ | Avatar image URL |
| `label` | `string` | — | Label text displayed above the input |
| `caption` | `string` | — | Text after the input (defaults to `"hours old"`) |
| `altText` | `string` | — | Alt text for the avatar image |
| `ui` | `AVPersonUI` | — | Object to override Tailwind classes for individual parts of the component |

### Model

`v-model` accepts `number | null`. `null` represents an empty input.

### Usage

```vue
<AVPerson
  v-model="person.ageInHours"
  :avatar-src="'/cat.jpg'"
  :alt-text="person.name"
  :label="`${person.name.toUpperCase()} IS`"
  caption="hours old"
/>
```

### Design decisions

**Input mask (`maska`).** The [maska](https://github.com/beholdr/maska) library handles digit formatting with space separators (1 442) and non-numeric character filtering. It's lightweight, supports Vue 3 directives, and provides reversed masks — essential for right-to-left digit grouping.

**Auto-width via mirror span.** The input and an invisible `<span>` share the same CSS Grid cell (`col-start-1 row-start-1`). The span stretches the grid cell to match its text width, while the input (with `size="1"`) fills that same cell. This produces content-adaptive width starting from `min-width: 72px`.

**Three visual states.** The component tracks `isFocused` and `isHovered` to compute `visualState` (`default` / `hover` / `active`), which drives border, text, placeholder, and label colors. The ring around the avatar only appears in `active` (focused) state.

**15-digit limit.** `Number.MAX_SAFE_INTEGER` has 16 digits. Capping at 15 digits guarantees no precision loss when converting via `Number()`.

**`ui` prop for customization.** Inspired by NuxtUI. Allows overriding Tailwind classes for each part of the component (root, avatar, ring, label, inputWrapper, input, caption) without touching internal structure.


## Goal: Create a numeric input component.

![figma Preview](./public/img.png)

[Figma](https://www.figma.com/file/OcyCt22I1Ha3fgLzGi0ZZy/Front-end-UI-Task?type=design&node-id=1-4&mode=design&t=ZzZ3vo84xwZ6uxJF-0)


### Functional requirements:
1. The user should only be able to enter digits.
2. Groups of 3 digits should be separated by spaces (“1442” → “1 442”).
3. Starting at a width of 72 px, the input should adapt to the size of the entered value.

### Design requirements:
1. Should match the provided [Figma](https://www.figma.com/file/OcyCt22I1Ha3fgLzGi0ZZy/Front-end-UI-Task?type=design&node-id=1-4&mode=design&t=ZzZ3vo84xwZ6uxJF-0).

### Code requirements:
1. The input component should be usable as-is in other parts of the project.
2. You can modify this project as you see fit to match a “production-ready” state. (optional)
3. You can use any library/component that you deem necessary and would use in a real application. (optional)
---