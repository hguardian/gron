# gron
[![Build Status](https://semaphoreci.com/api/v1/roylee0704/gron/branches/master/badge.svg)](https://semaphoreci.com/roylee0704/gron)
[![Go Report Card](https://goreportcard.com/badge/github.com/roylee0704/gron)](https://goreportcard.com/report/github.com/roylee0704/gron)
[![GoDoc](https://godoc.org/github.com/roylee0704/gron?status.svg)](https://godoc.org/github.com/roylee0704/gron)

Gron provides a clear syntax for writing and deploying cron jobs.

## Goals

- Minimalist APIs for scheduling jobs.
- Thread safety.
- Ability to define own job type.

## Installation

```sh
$ go get github.com/roylee0704/gron
```

## Usage
Create `schedule.go`

```go
package main

import (
	"fmt"
	"time"
	"github.com/roylee0704/gron"
)

func main() {
	c := gron.New()
	c.AddFunc(gron.Every(30*time.Minute), func() { fmt.Println("Every 30 minutes") })
	c.Start()
}
```

### Schedule Parameters

Configure basic periodic schedule with `gron.Every()`
```go
gron.Every(1*time.Second)
gron.Every(1*time.Minute)
gron.Every(1*time.Hour)
```

Also support `Day`, `Week` by importing `gron/xtime`:
```go
import "github.com/roylee0704/gron/xtime"

gron.Every(1*xtime.Day)
gron.Every(1*xtime.Week)
```

Schedule at specific time with `.At(hh:mm)`
```go
gron.Every(30*time.Day).At("00:00")
gron.Every(1*time.Week).At("23:59")
```

### Custom Job Type
You may define custom job types by implementing `gron.Job` interface: `Run()`.

For example:

```go
type Reminder struct {
	Msg string
}

func (r Reminder) Run() {
  fmt.Println(r.Msg)
}
```

After job has defined, instantiate it and schedule to run in Gron.
```go
c := gron.New()
r := Reminder{ "Feed the baby!" }
c.Add(gron.Every(1*time.Hour), r)
c.Start()
```

### Custom Job Func
You may register `Funcs` to be executed on a given schedule. Gron will run them in their own goroutines, asynchronously.

```go
c := gron.New()
c.AddFunc(gron.Every(1*time.Second), func() { fmt.Println("Every 1 second") })
c.Start()
```

### Jobs may be added to running cron.
