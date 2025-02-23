---
title: Hello World
description: Welcome to Hugo Theme Stack
slug: hello-world
date: 2022-03-06 00:00:00+0000
image: cover.jpg
categories:
    - Example Category
    - Test
    - Stack
tags:
    - Example Tag
---

Welcome to Hugo theme Stack. This is your first post. Edit or delete it, then start writing!

For more information about this theme, check the documentation: https://stack.jimmycai.com/

Want a site like this? Check out [hugo-theme-stack-stater](https://github.com/CaiJimmy/hugo-theme-stack-starter)

```bash
git clone
cd hugo-theme-stack-starter
hugo server
```


## GO webserver
```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World!")
}


func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServe(":8080", nil)
}
```

> Photo by [Pawel Czerwinski](https://unsplash.com/@pawel_czerwinski) on [Unsplash](https://unsplash.com/)
