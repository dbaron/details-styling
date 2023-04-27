# Improvements to `<details>` styling

L. David Baron, <dbaron@chromium.org>, April 2023

## Goals

The HTML `details` element represents a
[disclosure](https://open-ui.org/components/disclosure.research) widget.
It can also be used as a part of an
[accordion](https://open-ui.org/components/accordion.research) widget.
However, there are many disclosure widgets on the web
that do not use the `details` element.
In many cases, the styling of these disclosure widgets
is not achievable using the `details` element.

We should instead be in a situation where stylability
is not an obstacle to the use of the `details` element
for the vast majority of disclosure widgets on the Web.

## User needs

Developers regularly use disclosure and accordion widgets on the Web.
Today, that often means that they build their own widget
rather than using a widget provided by the platform.

This proposal is suggesting that we add features that would enable
developers to use the platform's widget in more cases
(hopefully the vast majority of cases).
Doing so would help users because
it would make (or at least would enable making)
the user experience
more consistent, and generally better, in a number of areas:
* keyboard shortcuts and focus handling,
* exposure via ARIA to assistive technology, and
* integration with browser features such as find-in-page.

Reducing the need for developers to build their own widgets
should also reduce the size of pages that use such widgets,
which reduces the time and bandwidth needed
for users to load those pages.

## Current implementation behavior

Implementations in Chromium, Gecko, and WebKit all use Shadow DOM
to implement the `details` element.  However, it is not possible for
pages to use this to override the behavior of the `details` element
because:

* `Element.attachShadow()` is
  [not allowed](https://dom.spec.whatwg.org/#dom-element-attachshadow)
  on the details element.
  This appears to be interoperably implemented
  ([Chromium](https://searchfox.org/mozilla-central/search?q=symbol:_ZNK7mozilla3dom7Element18CanAttachShadowDOMEv),
  [Gecko](https://source.chromium.org/search?q=%22Element::CanAttachShadowRoot%22&ss=chromium),
  [WebKit](https://github.com/WebKit/WebKit/blob/main/Source/WebCore/dom/Element.cpp#L2844)).
* The user-agent shadow DOM used to implement `<details>` in
  all of Chromium, Gecko, and WebKit
  is closed (`Element.shadowRoot` is `null`)
  (see [test](tests-of-existing-behavior/shadow-mode-open.html)).

In all three implementations,
it is possible to completely remove the disclosure triangle
as long as there is a `summary` element.
However, it currently requires
[using `display: block` on the summary in Chromium and Gecko](tests-of-existing-behavior/no-triangle-display-block.html)
and
[using `display: none` on a pseudo-element](tests-of-existing-behavior/no-triangle-webkit-details-marker.html)
in WebKit.
Neither technique works for the summary
that is generated when no `summary` element is present.

In all three implementations,
attribute selectors (such as with `details[open]`)
can be used to change styles based on whether a `details` is open
(see [test](tests-of-existing-behavior/styling-attribute-selector-open.html)).

### Chromium and Gecko (common)

Chromium ([source](https://source.chromium.org/search?q=%22details%20%3E%20summary:first-of-type%22&ss=chromium)) and
Gecko ([source](https://searchfox.org/mozilla-central/search?q=details+%3E+summary%3Afirst-of-type&path=&case=false&regexp=false))
treat the `summary` as a `list-item`,
with the following styles:

```css
details > summary:first-of-type {
  display: list-item;
  counter-increment: list-item 0;
  list-style: disclosure-closed inside;
}
details[open] > summary:first-of-type {
  list-style-type: disclosure-open;
}
```

Pages can override these styles (except for the internal default summary,
which in both engines has the same styles duplicated in the shadow DOM;
Gecko using a linked stylesheet and Chromium using a style attribute).

### Chromium

The `details` element [uses shadow DOM internally](https://source.chromium.org/search?q=%22HTMLDetailsElement::DidAddUserAgentShadowRoot%22&ss=chromium),
containing a slot for the main summary (with a default summary child),
and a slot for the remaining content.
The assignment of the main summary to the slot for the main summary
uses [custom C++ code](https://source.chromium.org/search?q=%22HTMLDetailsElement::ManuallyAssignSlots%22&ss=chromium) that uses a generic manual slotting mechanism.

TODO: Check how usable `::marker` is

### Gecko

The [`details` element](https://searchfox.org/mozilla-central/source/dom/html/HTMLDetailsElement.cpp) uses shadow DOM internally,
containing a style sheet,
a slot for the main summary (with a default summary child),
and a slot for the remaining content.
The assignment of the main summary to the slot for the main summary
uses [custom C++ code](https://searchfox.org/mozilla-central/search?q=ShadowRoot%3A%3AGetSlotNameFor&path=&case=false&regexp=false) that handles details specially,
and assigns the main summary to the magic `internal-main-summary` slot name.

TODO: Check how usable `::marker` is

### WebKit

WebKit uses the [following styles](https://github.com/WebKit/WebKit/blob/main/Source/WebCore/css/html.css):

```css
summary::-webkit-details-marker {
    display: inline-block;
    width: 0.66em;
    height: 0.66em;
    margin-inline-end: 0.4em;
}
```

The [`details` element](https://github.com/WebKit/WebKit/blob/main/Source/WebCore/html/HTMLDetailsElement.cpp) uses shadow DOM internally,
containing a slot for the main summary (with a default summary child),
and a slot for the remaining content.
The assignment of the main summary to the slot for the main summary
uses custom C++ code that overrides the named slotting mechanism,
slotting the main summary into a slot named `summarySlot`.

TODO: Check how usable `::-webkit-details-marker` is

## Requirements

### Replacement of disclosure triangle

### Layout of parts of disclosure widget

## Options

The following are options we have for solving various parts of the
above requirements.
Many of them do not address the entire problem,
so more than one of them is likely to be needed.

### Proposal #1: Hiding and replacing disclosure widget

Hiding and rebuilding parts of a widget is a common approach
for solving widget styling issues on the Web.
It is [doable today](https://www.scottohara.me/blog/2022/09/12/details-summary.html#:~:text=Styling%20a%20disclosure%20widget)
given the default styles for these elements.
(Having standard styles across browser engines
would slightly reduce the complexity of doing this.)

This has the advantage of being a rather well-worn path,
and also of allowing developers to replace the marker
with whatever they want.
It has the disadvantage that
any keyboard behavior, focus behavior, and ARIA
associated with the default marker is lost.

### Proposal #2: Pseudo-elements (new ones or `::part()`)

### Proposal #3: Improved `::marker` styling

### Proposal #4: Shadow tree replacement
