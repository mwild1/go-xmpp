# XMPP library for Go

> **This is an unofficial fork of [FluuxIO/go-xmpp](https://github.com/FluuxIO/go-xmpp).**
> This fork is maintained for my own projects only, because of some fixes I
> needed that were not merged upstream, and because I wanted to make some
> design changes to better suit my needs.
>
> If you use this fork and find it useful, contributions are welcome.
> Issues opened without a contribution are unlikely to get fixed.

**Known differences from upstream**

- Improved reconnect logic - this library will try hard to keep connected to
  the server (while using sensible retry practices, such as backoff). In some
  cases the upstream library will not attempt to reconnect.
- This library does not send initial presence, the application must send it if
  desired. At the time of writing, the upstream library sends a default
  initial presence unconditionally.

Parts of the original README remain below.

---

## Supported specifications

### Clients

- [RFC 6120: XMPP Core](https://xmpp.org/rfcs/rfc6120.html)
- [RFC 6121: XMPP Instant Messaging and Presence](https://xmpp.org/rfcs/rfc6121.html)

### Components

  - [XEP-0114: Jabber Component Protocol](https://xmpp.org/extensions/xep-0114.html)
  - [XEP-0355: Namespace Delegation](https://xmpp.org/extensions/xep-0355.html)
  - [XEP-0356: Privileged Entity](https://xmpp.org/extensions/xep-0356.html)

### Extensions 
  - [XEP-0060: Publish-Subscribe](https://xmpp.org/extensions/xep-0060.html)  
    Note : "6.5.4 Returning Some Items" requires support for [XEP-0059: Result Set Management](https://xmpp.org/extensions/xep-0059.html), 
    and is therefore not supported yet. 
  - [XEP-0004: Data Forms](https://xmpp.org/extensions/xep-0004.html)
  - [XEP-0050: Ad-Hoc Commands](https://xmpp.org/extensions/xep-0050.html)

## Package overview

### Stanza subpackage

XMPP stanzas are basic and extensible XML elements. Stanzas (or sometimes special stanzas called 'nonzas') are used to 
leverage the XMPP protocol features. During a session, a client (or a component) and a server will be exchanging stanzas
back and forth.

At a low-level, stanzas are XML fragments. However, Fluux XMPP library provides the building blocks to interact with
stanzas at a high-level, providing a Go-friendly API.

The `stanza` subpackage provides support for XMPP stream parsing, marshalling and unmarshalling of XMPP stanza. It is a
bridge between high-level Go structure and low-level XMPP protocol.

Parsing, marshalling and unmarshalling is automatically handled by Fluux XMPP client library. As a developer, you will
generally manipulates only the high-level structs provided by the stanza package.

The XMPP protocol, as the name implies is extensible. If your application is using custom stanza extensions, you can
implement your own extensions directly in your own application.

To learn more about the stanza package, you can read more in the
[stanza package documentation](https://github.com/FluuxIO/go-xmpp/blob/master/stanza/README.md).

### Router

TODO

### Getting IQ response from server

TODO

## Examples

We have several [examples](https://github.com/FluuxIO/go-xmpp/tree/master/_examples) to help you get started using
Fluux XMPP library.

Here is the demo "echo" client:

```go
package main

import (
	"fmt"
	"log"
	"os"

	"gosrc.io/xmpp"
	"gosrc.io/xmpp/stanza"
)

func main() {
	config := xmpp.Config{
		TransportConfiguration: xmpp.TransportConfiguration{
			Address: "localhost:5222",
		},
		Jid:          "test@localhost",
		Credential:   xmpp.Password("test"),
		StreamLogger: os.Stdout,
		Insecure:     true,
		// TLSConfig: tls.Config{InsecureSkipVerify: true},
	}

	router := xmpp.NewRouter()
	router.HandleFunc("message", handleMessage)

	client, err := xmpp.NewClient(&config, router, errorHandler)
	if err != nil {
		log.Fatalf("%+v", err)
	}

	// If you pass the client to a connection manager, it will handle the reconnect policy
	// for you automatically.
	cm := xmpp.NewStreamManager(client, nil)
	log.Fatal(cm.Run())
}

func handleMessage(s xmpp.Sender, p stanza.Packet) {
	msg, ok := p.(stanza.Message)
	if !ok {
		_, _ = fmt.Fprintf(os.Stdout, "Ignoring packet: %T\n", p)
		return
	}

	_, _ = fmt.Fprintf(os.Stdout, "Body = %s - from = %s\n", msg.Body, msg.From)
	reply := stanza.Message{Attrs: stanza.Attrs{To: msg.From}, Body: msg.Body}
	_ = s.Send(reply)
}

func errorHandler(err error) {
	fmt.Println(err.Error())
}

```

## Reference documentation

The code documentation is available on GoDoc: [gosrc.io/xmpp](https://godoc.org/gosrc.io/xmpp)
