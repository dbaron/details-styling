# Improvements to `<details>` styling

L. David Baron, <dbaron@chromium.org>, April-May 2023

* Feedback: https://github.com/openui/open-ui/issues/744
* [Explainer for work happening in phase 1](phase-1.md)

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
uses [custom C++ code](https://source.chromium.org/search?q=%22HTMLDetailsElement::ManuallyAssignSlots%22&ss=chromium)
that calls the
[imperative slotting API](https://html.spec.whatwg.org/multipage/scripting.html#dom-slot-assign).

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
uses [custom C++ code](https://github.com/WebKit/WebKit/blob/82faeb56882ce53ee0f979087645c0cbfa5db2bf/Source/WebCore/html/HTMLDetailsElement.cpp#L70)
that overrides the named slotting mechanism,
slotting the main summary into a slot named `summarySlot`.

TODO: Check how usable `::-webkit-details-marker` is

## Requirements

### Replacement of disclosure triangle

Most disclosure widgets want their own appearance for the open/closed indicator
which by default is a disclosure triangle.
It should be possible to customize its appearance based on open/closed state
and also pseudo-classes like `:hover` and `:active`.

### Layout of parts of disclosure widget

It should be possible to make arbitrary or nearly-arbitrary layouts
of the disclosure triangle (or open/closed indicator), the summary,
and the contents of the collapsable area of the widget.
For example, it should be possible to position the open/closed indicator
in different positions relative to the other parts (such as on the right side),
and it should be possible to use these pieces
as part of a flex or a grid layout.

### Animation

It should be possible
to have the opening/closing of the details widget animate,
and for that animation to be styled with CSS transitions or animations.
This includes both animation of the height (with appropriate clipping
and maybe opacity) of the contents of the details, and possibly also
animation of the marker (e.g., rotation of a marker triangle).

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
However, if that behavior is common to the marker and the `summary` element
then this less of a problem.

### Proposal #2: Pseudo-elements (new ones or `::part()`)

We could have pseudo-elements to address
the three pieces of the disclosure widget:
the summary's container, the open/closed marker,
and the container for the collapsable contents.
These pseudo-elements could potentially address pieces that have
a defined relationship with each other in user-agent shadow DOM content.

The pseudo-elements themselves could be newly-defined CSS pseudo-elements.
However, if this is defined in terms of user-agent shadow DOM,
we also have the option of exposing `::part()` pseudo-elements
from the defined user-agent shadow DOM,
rather than minting new pseudo-elements.
(This hasn't been done before,
but does seem like something we could reasonably do.
However, it was rejected in
[CSS Working Group discussion](https://logs.csswg.org/irc.w3.org/css/2023-07-20/#e1555355)
and in https://github.com/openui/open-ui/issues/702 .)

The pseudo-element for the open/closed marker should probably be `::marker`,
given the existing use of that pseudo-element,
and that the main summary is currently specified as a `list-item`.

Note that if we take this approach, the rules that
[are currently specified in HTML as a `style` attribute](https://html.spec.whatwg.org/multipage/rendering.html#the-details-and-summary-elements)
for the container of the collapsible contents
should be respecified as UA style sheet rules using this pseudo-element,
so that is at the UA sheet level of the cascade
rather than the style attribute level.

### Proposal #3: Improved `::marker` styling

Currently a
[limited set](https://drafts.csswg.org/css-lists-3/#marker-properties)
of properties apply to the `::marker` pseudo-element.

It probably makes sense that many properties like position and float are
not supported on `::marker`
(see [test](tests-of-existing-behavior/marker-movement.html)).
However, in the context of `details`/`summary`,
it probably makes more sense than it does for lists
to allow for repositioning of the marker.

(The [CSS2.0 `marker-offset` property](https://www.w3.org/TR/2008/REC-CSS2-20080411/generate.html#markers)
does not appear to be implemented
[in Chromium](https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/renderer/core/css/css_properties.json5)
or [in Gecko](https://searchfox.org/mozilla-central/search?q=marker-offset&path=&case=false&regexp=false)
or [in WebKit](https://github.com/WebKit/WebKit/blob/main/Source/WebCore/css/CSSProperties.json).)

A number of use cases probably could be addressed by supporting
additional properties on `::marker`, such as:
* box model properties,
* a property to say which side the marker goes on (though
  this would have similar questions regarding AT and focus order
  as those that exist for
  flex and grid properties that do visual reordering),

### Proposal #4: Shadow tree replacement

[ This option was rejected during
[CSS Working Group discussion](https://logs.csswg.org/irc.w3.org/css/2023-07-20/#e1555355).
]

Given that major browser engine implementations all
implement `details` using user-agent shadow DOM,
it seems like one option for improving stylability of `details`
would be to allow the replacement of the shadow DOM
so that developers could arrange the relationship of the parts as they need.

This faces the obstacle that of the DOM spec's restrictions
on replacing the shadow DOM of the details element,
plus the (new, I think) concept of allowing developers
to replace a user-agent provided shadow DOM.
These seem like somewhat major obstacles.

On the positive side, it would do a good job of ensuring that developers
can style the `details` element as they want.
But developers who replace the shadow DOM
would then be responsible for rebuilding the necessary
key handling, focus behavior, and ARIA integration.

### Proposal #5: Support `flex` and `grid` insides on `list-item`s

Currently the
[specification](https://drafts.csswg.org/css-display-3/#typedef-display-listitem)
allows only `flow` and `flow-root` as the inner display types for the
inside of a `list-item`; `flex` and `grid` are not allowed.  This means
it is not possible to style the inside of a `summary` element as a flex
or grid layout.

This probably isn't a major problem, but it's worth noting here that we
could adjust this if it were necessary to do so.

## See also
* [Chromium bug 1469418](https://bugs.chromium.org/p/chromium/issues/detail?id=1469418)
* [Mozilla bug 1856374](https://bugzilla.mozilla.org/show_bug.cgi?id=1856374) (display setting)
* [CSS Working Group Minutes, 2023-07-20](https://lists.w3.org/Archives/Public/www-style/2023Sep/0019.html)
