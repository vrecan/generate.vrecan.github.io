+++
date = "2015-03-01T20:38:08-07:00"
Categories = ["Development"]
draft = false
title = "Go Dependency Management"
section = "post"
tags  = [ "Development", "GoLang", "Godeps", "Dependency Management" ]
+++

<p>Godeps is a nice simple tool that allows you to manage dependencies in a very easy way. One major drawback though is that it requires you to commit your dependencies into your repository. This is great for simplicity but horrible for code review. Seeing a commit with 26k files changed really makes it hard to review.</p>

I've found a simple solution to the problem if you use the common go import layout
```
+-- github.com
|   +-- company
|   |   +-- project
```

<p>If you follow this layout then you can easily extract all your deps into their own repo like so.</p>
```
cd github.com/company
godep save ./...
```
<p>This will Recursively grab your dependencies that are not in this path and put them in a godeps folder at the company depth. Now add the contents of the new Godeps folder into a new repo with the name Godeps. </p>

## Building with new dependencies ##

Now that you have dependencies in their own repo you can add Godeps to your .gitignore for all your other repos and pull them down locally.

```
cd github.com/company/project
git clone github.com/company/Godeps
godep go buld
godep go test ./... --race
```

## Conclusion / Drawbacks ##
<p>
This works great if you are ok with taking the master branch versions of all your own software. This might not work for everyone but with my workflow this seems to give us the best of both worlds. Easy third party library management and go getable software for all of our repos. This also has the added benefit that we no longer have to remember to update godep everytime we make an internal library change, in our workflow everything in master should be buildable together.</p>
