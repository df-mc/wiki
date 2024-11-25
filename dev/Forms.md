[![Forms](https://badges.fyi/static/go.dev/Documentation/29BEB0)](https://pkg.go.dev/github.com/df-mc/dragonfly/dragonfly/player/form?tab=doc)

## Index
* [Overview](#overview)
* [Custom forms](#custom-forms)
* [Menu forms](#menu-forms)
* [Modal forms](#modal-forms)

## Overview
Forms in Dragonfly may be sent using the `(*Player).SendForm()` method, which will send a form to a player and 
make the form run its `Submit` method when the player submits it. If the player does not submit it, this 
method will not be called. Form implementations should therefore never rely on their form being submitted.

Dragonfly distinguishes three types of forms. The first being [custom forms](#custom-forms), which allow
adding buttons of multiple types to record data, the second being [menu forms](#menu-forms), which allow
adding multiple buttons with images of which the player can select one, and the last being 
[modal forms](#modal-forms), which allow a body with only a 'yes' or 'no' response.

## Custom forms
Custom forms may have any amount of buttons which the player can fill out in their respective ways. Available
buttons are listed below.
* [Label](https://github.com/df-mc/dragonfly/blob/master/server/player/form/element.go#L11) (Text element)
* [Input](https://github.com/df-mc/dragonfly/blob/master/server/player/form/element.go#L18)
* [Toggle](https://github.com/df-mc/dragonfly/blob/master/server/player/form/element.go#L38)
* [Slider](https://github.com/df-mc/dragonfly/blob/master/server/player/form/element.go#L55)
* [Dropdown](https://github.com/df-mc/dragonfly/blob/master/server/player/form/element.go#L77)
* [Step slider](https://github.com/df-mc/dragonfly/blob/master/server/player/form/element.go#L98)

Custom forms may be created by building a structure composed of fields that have the types above. 
Additionally, custom form structures must implement the 
[form.Submittable](https://github.com/df-mc/dragonfly/blob/master/dragonfly/player/form/submit.go#L8)
interface.

Example `form.Submittable` implementation:
```go
package main

import (
	"fmt"
	"github.com/df-mc/dragonfly/server/player"
	"github.com/df-mc/dragonfly/server/player/form"
	"github.com/df-mc/dragonfly/server/world"
)

// SomeForm holds the structure of our form. The order in which elements appear is defined by the order of
// the struct fields.
type SomeForm struct {
	// The names of the fields can be anything and do not influence the text displayed in the form sent to the
	// player.
	Names form.Dropdown
	Input form.Input
	Amount form.Slider
}

// Submit is called when the form is submitted. It must have this signature in order to implement the
// form.Submittable interface.
func (f SomeForm) Submit(submitter form.Submitter, tx *world.Tx) {
	// A form.Submitter is guaranteed to be a *player.Player when sent to a player, so we can assert it to a
	// player right away.
	p := submitter.(*player.Player)

	// The Value() method may be called on each element to retrieve the value that the user selected.
	selectedName := f.Names.Options[f.Names.Value()]
	text := f.Input.Value()
	amount := f.Amount.Value()

	fmt.Printf("Player %v selected name %v with amount %v: %v\n", p.Name(), selectedName, amount, text)
}
```
In order to send the form to a player, it will first have to be initialised. The following snippet shows how
the form is finally initialised and sent to the player.
```go
package main

import (
	"github.com/df-mc/dragonfly/server/player"
	"github.com/df-mc/dragonfly/server/player/form"
)

// ...

// send shows SomeForm to a *player.Player.
func send(p *player.Player) {
	f := form.New(
		SomeForm{
			Names:  form.Dropdown{
				Text:         "Dropdown text",
				Options:      []string{"AAA", "BBB"},
				DefaultIndex: 0, // This field may be omitted as it has the default value.
			},
			Input:  form.Input{
				Text:        "Input text",
				Default:     "", // This field may be omitted as it has the default value.
				Placeholder: "Hello, World!",
			},
			Amount: form.Slider{
				Text:     "Slider text",
				Min:      0, // This field may be omitted as it has the default value.
				Max:      10,
				StepSize: 1,
				Default:  5,
			},
		},
		"SomeForm's title",
	)
	p.SendForm(f)
}
```

## Menu forms

## Modal forms