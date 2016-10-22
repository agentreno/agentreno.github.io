---
layout: post
title: "A basic course in web application security"
date: 2016-10-22
---

Generally, the best way to learn is by doing. A little while back I wrote a 
small web application security course to try and teach first myself, and then 
some of my colleagues, about common web vulnerabilities.

The idea was to make a really small application, a bulletin board, and leave 
three security flaws in it. The test is to find them, exploit them, and then 
patch them. The application is pretty small, it's a one-file Python Flask app.

There's a lot to be said for hands-on learning. On one level, there's the 
knowledge that cross-site scripting attacks are about injecting scripts 
into an app. Then there's the understanding you get from finding XSS in code, 
taking advantage of it, fixing it and adding monitoring for it.

It can be tempting to take shortcuts. For example, most of us will be aware 
that running in debug mode in production is A Bad Thing. So when you look at 
some code like this...

```python
if __name__ == "__main__":
   app.run(debug=True)
```

You might just say "debug in prod is bad, turn it off, job done". This is fine 
but why is it bad? What can you do to this application because it's in debug? 
Try running the app and find out.

So whether you build web apps, or offer advice to those who do, maybe this 
course can be useful to you. On the other hand, it is fairly basic, so if you 
know all of this stuff already, maybe you could build on it and add more 
complex vulnerabilities?

Feel free to [*clone the repository*](https://github.com/agentreno/aas-demo)
and try it out. If you have any suggestions for improving it, just open an
issue or make a pull request. Also, if you're finding it tough or just want to
check your answers, there's a branch called `secured` with commented fixes.
