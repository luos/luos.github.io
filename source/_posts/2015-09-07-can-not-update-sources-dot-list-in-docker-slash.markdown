---
layout: post
title: "Can not update sources.list in docker?"
date: 2015-09-07 21:32:37 +0200
comments: true
categories: docker 
---

For unknown reasons to me when you try to run the Dockerfile containing this line, it will not work:

```
RUN echo "deb http://http.debian.net/debian jessie-backports main" >> /etc/apt/sources.list
```

After the machine is booted however you can insert the line succesfully. You can solve this by running the command below:

```
RUN echo "deb http://http.debian.net/debian jessie-backports main" > /etc/apt/sources.list.d/backports.list
```

This creates a new file in the sources.list.d and it gets picked up by `apt-get update`. Make sure the extension is .list.






