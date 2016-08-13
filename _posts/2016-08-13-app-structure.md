---
layout: post
title: How do you structure your apps?
description: "My notes"
category:
tags: [clojure]
---

{{ page.title }}
================

<p class="meta">13 Aug 2016</p>

In clojure gazette (https://clojuregazette.com/) Eric Normand once talked about code structure. According to him
there's no such value in organizing code in FP upfront. You should just write the code
and then organize it into logical units:

> In Clojure, I literally don't think about organizing my code until it gets longer than 100 lines. At that point, I start looking for logical ways to divide it in two. In an OO language, I often see people doing this organization up-front. To me, organizing code is no more interesting than organizing a sock drawer. You don't really need to organize socks until you get a lot of them. But this answer does not satisfy them either.

He also describes universal process for writing software:

> First, understand the problem to some degree, which includes concepts and their relationships. Second, map those concepts and relationships to constructs in the language. Third, write a solution using those constructs, which will deepen your understanding. Then repeat all the steps as many times as needed.
