# mgo
## Rich MongoDB driver for Go

Git mirror of http://labix.org/mgo

For feature requests and bug report, please visit upstream http://labix.org/mgo

## Introduction
mgo (pronounced as *mango*) is a MongoDB driver for the Go language that implements a rich and well tested selection of features under a very simple API following standard Go idioms.

"mgo may well be the most advanced MongoDB driver right now." — Mathias Stearn, MongoDB Engineer at 10gen

"mgo enables us to blazingly serve more than 1.000.000 book covers a day
while reducing our servers' load."
— Benoît Larroque, R&D Engineer at Feedbooks.com

"mgo and Go are a pair made in heaven." — Brian Ketelsen, Author of GopherTimes.com

"mgo is an awesome MongoDB driver, extremely fast, easy to use, and very actively maintained." — Travis Reeder, CTO of Appoxy and Iron.io

"mgo is the best database driver I've ever used." — Patrick Crosby, Founder of StatHat

"I wish there was something almost as good as mgo, but for sql." — Someone at #go-nuts

##Highlights

### Cluster discovery and communication

mgo offers automated cluster topology discovery and maintenance. Even if provided the address to a single server in the cluster, mgo will figure out the cluster topology and communicate with any of the servers as necessary.

### Failover management

mgo will automatically failover in the event that the master server changes.

### Synchronous and concurrent

mgo offers a synchronous interface, with a concurrent backend. Concurrent operations on the same socket do not wait for the previous operation's roundtrip before being delivered. Documents may also start being processed as soon as the first document is received from the network, and will continue being received in the background so that the connection is unblocked for further requests.

### Result pre-fetching

mgo offers configurable pre-fetching of documents, so that the next batch of results are requested automatically when an established percentage of the current batch has been processed.

### Flexible serialization

mgo supports flexible marshalling and unmarshalling of documents through gobson, a brand new BSON package written specifically to support mgo.

### Trivial consistency-level management
mgo offers trivial consistency-level selection to tweak resource usage vs. read/write ordering guarantees.

Strong consistency uses a unique connection with the master so that all reads and writes are as up-to-date as possible and consistent with each other.

Monotonic consistency will start reading from a slave if possible, so that the load is better distributed, and once the first write happens the connection is switched to the master. This offers consistent reads and writes, but may not show the most up-to-date data on reads which precede the first write.

Eventual consistency offers the best resource usage, distributing reads across multiple slaves and writes across multiple connections to the master, but consistency isn't guaranteed.

Note that this mechanism works both when connecting through a mongos server and when connecting directly to a replica set.

### Authentication support with pooling integration

mgo offers authentication support, with great connection pooling integration.

This enables mgo to talk to protected servers and replica sets in a very comfortable way. Even with a straightforward API, the authentication is internally cached in a secure way to avoid constant roundtrips to the database. The use of nonces is also optimized so that logins are usually performed with a single roundtrip to the database.

### GridFS support
mgo can be used to send and receive files to MongoDB using the standard GridFS specification for file storage. This means that it may share the same database collections used for file storage with drivers for other languages, and also the command line tools provided with MongoDB itself (e.g. mongofiles).

In addition to a selection of relevant methods, the *mgo.GridFile type implements support for both io.Reader and io.Writer interfaces, which means it integrates nicely with the standard library.

### Thoroughly tested
Automated tests cover not only good situations, but also harsh scenarios such as master failover.

## API documentation

There is online documentation for both mgo, mgo/bson, and mgo/txn:

- API docs for mho
- API docs for mgo/bson
- API docs for mgo/txn

## Mailing list
Discussion related to the use and development of mgo is held in the mailing list.

## Installing
To install mgo, make sure you have the bzr command available and then run:

```
go get labix.org/v2/mgo
```

Note: mgo is maintained up-to-date with the current stable release of Go, which is Go 1 at the moment.

###Example

```go
package main

import (
        "fmt"
        "labix.org/v2/mgo"
        "labix.org/v2/mgo/bson"
)

type Person struct {
        Name string
        Phone string
}

func main() {
        session, err := mgo.Dial("server1.example.com,server2.example.com")
        if err != nil {
                panic(err)
        }
        defer session.Close()

        // Optional. Switch the session to a monotonic behavior.
        session.SetMode(mgo.Monotonic, true)

        c := session.DB("test").C("people")
        err = c.Insert(&Person{"Ale", "+55 53 8116 9639"},
	               &Person{"Cla", "+55 53 8402 8510"})
        if err != nil {
                panic(err)
        }

        result := Person{}
        err = c.Find(bson.M{"name": "Ale"}).One(&result)
        if err != nil {
                panic(err)
        }

        fmt.Println("Phone:", result.Phone)
}
```

# License

mgo is made available under the Simplified BSD License.
