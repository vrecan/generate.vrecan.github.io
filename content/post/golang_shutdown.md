+++
Categories = ["Development"]
Description = "Managing the death of go applications."
Tags = ["Development", "GoLang", "Shutdown", "Signals"]
date = "2015-03-10T20:23:05-06:00"
menu = "post"
title = "Managing Application Shutdown in Go"
socialsharing = true
+++

### Overview

I was looking around and couldn't find a simple library that would manage application shutdown using signals. I decided to build a simple library to handle application shutdown and calling Close methods on structs when shutdown was signaled. It is called [death](http://github.com/vrecan/death "Application shutdown library for golang").

### Requirements
* Should only need to pass the signals you want to use for shutdown.
* Block the application from shutting down until signal was recieved.
* Optionally pass structs with a Close method to cleanup objects when shutdown is signaled.

### How do you use it?
Import death and syscall so that you can pass the signals you want to shutdown with.
```
package main

import (
    DEATH "github.com/vrecan/death"
    SYS "syscall"
)
```

Now create a death struct with the signals we want to use for shutdown.
```
    death := DEATH.NewDeath(SYS.SIGINT, SYS.SIGTERM)
    defer death.Close()
```
Once we are doing setting up everything in our application we can then call WaitForDeath() to block our application from shutting down.
```
    death.WaitForDeath()
```

You can also have death automically close structs for you when shutdown is triggered. Create a slice to contain all the items you want to call Close().

```
objects := make([]DEATH.Closable, 0)

```

Your structs just need a Close() method to implement the Closable interface
```
type NewType struct {
}

func (n *NewType) Close() {
}

```
Once your go routine(s) have been started append them to the slice.
```
    objects = append(objects, &NewType{})

```
Now just pass them in when you are going to call WaitForDeath()
```
    death.WaitForDeath(objects...)
```

You can also pass them in one by one if you prefer
```
death.WaitForDeath(object1, object2, object3)
```

### Summary
I'm actually pretty happy with this library. It does everything I need it to do and has been really useful to just drop in and have solid shutdown management. It does currently use seelog to log a few things this could easily be removed. All of our applications use seelog for application logs but I would love to hear if there are any good ideas about managing logging inside of libraries. I've thought about having the ablilty to pass in a logger with an interface but I would like to hear how other people deal with this.

[Managing death in go](http://github.com/vrecan/death "Application shutdown library for golang").




