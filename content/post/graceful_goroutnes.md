+++
date = "2015-03-29T16:38:08-07:00"
Categories = ["Development"]
draft = false
title = "Gracefull Shutdown of Goroutines"
section = "post"
tags  = [ "Development", "GoLang", "WaitGroup", "Channels" ]
socialsharing = true
+++

### Overview

A common problem people new to Go tend to run into is managing goroutines. Starting a goroutine is easy but how do you guarantee it closes gracefully when you are ready to exit? All code for this example lives [here](https://github.com/vrecan/shutdown_example).


First, lets create a struct that will manage our goroutine. In this case it's just going to be one but there are no restrictions here.

<pre>
	<code class="go-css">

//App is an example struct that will run goroutines
type App struct {
	wg   *sync.WaitGroup //allow us to wait for close
	done chan bool
	Data chan string
}

func NewApp() (app *App) {
	app = &App{
		wg:   &sync.WaitGroup{},
		done: make(chan bool, 1),
		Data: make(chan string, 100),
	}
	return app
</code>
</pre>

With this we can now add a few methods to start and run our goroutine. First, our start method which will increment our wait group and run our goroutine. It's important to increment the wait group before you spin up the goroutine otherwise you can have a race condition on close.

```
//Start go routines
func (a *App) Start() {
	a.wg.Add(1)
	go a.runSelect()
}
```

There are two main ways to handle signaling for shutdown. This example shows both for completeness. The first thing we are doing is using a channel called done in a select statement which allows us to get a signal from something external telling us it's time for shutdown. 

The other way of managing shutdown of a goroutine is to just close it when you are done sending things to the channel. The benefit of this approach is that you will always process all of the messages before you exit. Obviously this could be a negative as well if you don't want to process everything on the queue before a shutdown.

<pre>
	<code class="go-css">
//Run with select statement to manage shutdown
func (a *App) runSelect() {
	defer a.wg.Done()
loop:
	for {
		select {

		case <-a.done:
			fmt.Println("Detected close doing cleanup and exiting")
			a.wg.Add(1)
			a.runClose()
			break loop
			//do cleanup if needed
		case s, ok := <-a.Data:
			if ok { //ok will tell us if the channel has been shutdown
				fmt.Println("select: ", s)
			} else {
				//done processing all messages exit
				fmt.Println("Breaking loop")
				break loop
			}
		}
	}
}
//process all messaging on the queue until we get close message
func (a *App) runClose() {
	for s := range a.Data {
		fmt.Println("close: ", s)
	}
}
</code>
</pre>

Now it's time for our simple close method that will send our done signal and 
also close our channel. ***In a real application you want to make sure to only close a Go channel after you can guarantee all the senders are done.*** Go channels will panic if you try to send to a closed channel.


<pre>
	<code class="go-css">
//Cleanly close background routines
func (a *App) Close() {
	if a != nil {
		a.done <- true
		close(a.Data) //this lets the recv know there is no more data
		a.wg.Wait()
	}
}
</code>
</pre>

Example main loop for completeness.
```
//Simple example on managing shutdown
func main() {
	death := DEATH.NewDeath(SYS.SIGINT, SYS.SIGTERM)
	app := NewApp()
	app.Start()
	// app.Start()
	for i := 1; i <= 100; i++ {
		app.Data <- fmt.Sprintf("TESTING %d", i)
	}
	//use death
	death.WaitForDeath(app)
	//or call close manually
	//app.Close()
}
```

### Conclusion
I hope this example was useful. Every new Go developer I train seems to get stuck on managing closing goroutines and this information has been helpful in explaining the pitfalls of common Go patterns to shutdown.

[Link to code](https://github.com/vrecan/shutdown_example)


