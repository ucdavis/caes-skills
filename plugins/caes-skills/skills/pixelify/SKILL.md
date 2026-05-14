---
name: pixelify
description: Show a caes-cru team member's pixelated team image by name
---

# Pixelify

Given a team member's name, show their pixelated team image from `https://computing.caes.ucdavis.edu/team/<slug>.png`.

## Name → Slug Mapping

| Name | Slug |
|------|------|
| Adam Getchell | adam |
| Arista Chauhan | arista |
| Cal Doval | cal |
| Darrell Joe | dj |
| Jackie Emerson | jackie |
| Jason Sylvestre | jason |
| Odin Jasonson | odin |
| Rob Martinsen | rob |
| Scott Kirkland | scott |
| Shuka Smith | shuka |
| Spruce Weber-Milne | spruce |
| Steven Barkey | steven |

## Steps

1. Match the requested name to a slug from the table above (first name is usually enough; Darrell goes by `dj`)
2. Construct the URL: `https://computing.caes.ucdavis.edu/team/<slug>.png`
3. Display the image inline using standard Markdown image syntax:
   ```markdown
   ![<Name>](https://computing.caes.ucdavis.edu/team/<slug>.png)
   ```
4. If the current agent UI does not render Markdown images, present the URL directly or navigate to it with the available browser tool
