---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
thumbnail: "images/thumbnail.jpg" # Optional, referenced at `$HUGO_ROOT/static/images/thumbnail.jpg`
toc: true # Optional
---