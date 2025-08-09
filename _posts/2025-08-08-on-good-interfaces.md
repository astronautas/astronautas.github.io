---
layout: post
title: On good interfaces
---
One of the ideas from [A Philosophy of Software Design](https://www.goodreads.com/book/show/39996759-a-philosophy-of-software-design) that struck me was that good method interfaces should be narrow. Fewer parameters make it more obvious how to use a method, which improves reusability. Fewer parameters mean less levers, so lower chance of misuse. Fewer parameters reduce coupling, making changes easier.

Resist the urge to add yet another parameter to your API. Your users will thank you.