# Tripwire

Tripwire is a dead simple package for managing and dispatching events in your Go application.

## Installation

Install Tripwire using the go get command:

```
$ go get github.com/FurqanSoftware/tripwire/cmd/tripwire
```

The package requires no additional dependencies other than Go itself.

## Example

Running `go generate` with a file like the following:

```
package hook

import "github.com/globalsign/mgo/bson"

//go:generate $GOPATH/bin/tripwire -type=ClarificationCreated,ClarificationUpdated,AccountCreated

type ClarificationCreated struct {
	ClarificationID bson.ObjectId
	IsRequest       bool
}

type ClarificationUpdated struct {
	ClarificationID bson.ObjectId
	IsResponse      bool
	UpdaterID       bson.ObjectId
}

type AccountCreated struct {
	AccountID bson.ObjectId
}
```

... will produce:

```
// Code generated by "tripwire -type=ClarificationCreated,ClarificationUpdated,AccountCreated"; DO NOT EDIT

package hook

type EventType int

const (
	EventClarificationCreated EventType = iota
	EventClarificationUpdated
	EventAccountCreated
)

type Event interface {
	Type() EventType
	Trigger()
}

func (e ClarificationCreated) Type() EventType {
	return EventClarificationCreated
}

func (e ClarificationCreated) Trigger() {
	EmitterClarificationCreated.Trigger(e)
}

type ClarificationCreatedHandler interface {
	Handle(ClarificationCreated)
}

type ClarificationCreatedHandlerFunc func(ClarificationCreated)

func (f ClarificationCreatedHandlerFunc) Handle(e ClarificationCreated) {
	f(e)
}

type ClarificationCreatedEmitter struct {
	handlers []ClarificationCreatedHandler
}

func (m *ClarificationCreatedEmitter) Trigger(e ClarificationCreated) {
	for _, h := range m.handlers {
		h.Handle(e)
	}
}

func (m *ClarificationCreatedEmitter) Handle(h ClarificationCreatedHandler) {
	m.handlers = append(m.handlers, h)
}

func (m *ClarificationCreatedEmitter) HandleFunc(f func(ClarificationCreated)) {
	m.Handle(ClarificationCreatedHandlerFunc(f))
}

func (e ClarificationUpdated) Type() EventType { ... }

func (e ClarificationUpdated) Trigger() { ... }

type ClarificationUpdatedHandler interface { ... }

type ClarificationUpdatedHandlerFunc func(ClarificationUpdated)

func (f ClarificationUpdatedHandlerFunc) Handle(e ClarificationUpdated) { ... }

type ClarificationUpdatedEmitter struct { ... }

func (m *ClarificationUpdatedEmitter) Trigger(e ClarificationUpdated) { ... }

func (m *ClarificationUpdatedEmitter) Handle(h ClarificationUpdatedHandler) { ... }

func (m *ClarificationUpdatedEmitter) HandleFunc(f func(ClarificationUpdated)) { ... }

func (e AccountCreated) Type() EventType { ... }

func (e AccountCreated) Trigger() { ... }

type AccountCreatedHandler interface { ... }

type AccountCreatedHandlerFunc func(AccountCreated)

func (f AccountCreatedHandlerFunc) Handle(e AccountCreated) { ... }

type AccountCreatedEmitter struct { ... }

func (m *AccountCreatedEmitter) Trigger(e AccountCreated) { ... }

func (m *AccountCreatedEmitter) Handle(h AccountCreatedHandler) { ... }

func (m *AccountCreatedEmitter) HandleFunc(f func(AccountCreated)) { ... }

var (
	EmitterClarificationCreated = ClarificationCreatedEmitter{}
	EmitterClarificationUpdated = ClarificationUpdatedEmitter{}
	EmitterAccountCreated       = AccountCreatedEmitter{}
)

func Trigger(e Event) {
	e.Trigger()
}
```