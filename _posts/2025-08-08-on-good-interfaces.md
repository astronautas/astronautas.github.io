---
layout: post
title: On good interfaces
---
One of the ideas from A Philosophy of Software Design that struck me was that good method interfaces should be narrow. Fewer parameters make it more obvious how to use a method, which improves reusability. Fewer parameters mean less things to go wrong using it. Fewer parameters reduce coupling, making changes easier.

Resist the urge of adding yet another parameter to your API. Your users will thank you.