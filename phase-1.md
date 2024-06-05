# Improvements to `<details>` styling, phase 1

L. David Baron, <dbaron@chromium.org>, May 2024

* CSS standardization issue: [w3c/csswg-drafts#9879](https://github.com/w3c/csswg-drafts/issues/9879)
* HTML standardization PR: [whatwg/html#10265](https://github.com/whatwg/html/pull/10265)

This explainer describes the initial phase of changes proposed
to improve styling of the `<details>` and `<summary>` elements.
For more detail on the overall goals, see
the [overall project explainer](README.md),
which includes some alternatives that are not being done
and some alternatives (particularly improvements to marker styling)
that may still be done in the future.

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

## State prior to this proposal

Prior to this proposal
the specification text describing the rendering of the `<details>`
and `<summary>` elements is
the following [section](https://html.spec.whatwg.org/multipage/rendering.html#the-details-and-summary-elements) of HTML
reproduced here in its entirety:

> ```css
> @namespace "http://www.w3.org/1999/xhtml";
> 
> details > summary:first-of-type {
>   display: list-item;
>   counter-increment: list-item 0;
>   list-style: disclosure-closed inside;
> }
> details[open] > summary:first-of-type {
>   list-style-type: disclosure-open;
> }
> ```
> 
> The details element is expected to render as a block box. The element is also expected to have an internal shadow tree with two slots. The first slot is expected to take the details element's first summary element child, if any. The second slot is expected to take the details element's remaining descendants, if any.
> 
> The details element's first summary element child, if any, is expected to allow the user to request the details be shown or hidden.
> 
> The details element's second slot is expected to have its style attribute set to "display: block; content-visibility: hidden;" when the details element does not have an open attribute. When it does have the open attribute, the style attribute is expected to be removed from the second slot.
> 
> > [!NOTE]
> > Because the slots are hidden inside a shadow tree, this style attribute is not directly visible to author code. Its impacts, however, are visible. Notably, the choice of content-visibility: hidden instead of, e.g., display: none, impacts the results of various APIs that query layout information.

The structure of the shadow tree was implemented as described in Chromium, Gecko, and WebKit.

That the disclosure triangle exists because it is a list item marker is implemented only in Chromium and Gecko;
WebKit has a separate `::-webkit-details-marker` mechanism.

All three of those engines
interpret the prose above to restrict changes to the `display` property on the `<details>` element,
and some also restrict changes on the `<summary>` element.

## Proposed changes

This proposes a set of changes that improve the ability of authors to style these elements:
* remove restrictions on the `display` property of these elements so that they can be styled with other display types (such as flexbox or grid)
* specify the structure of the shadow tree more clearly (so that it is clear that the slots are siblings of each other, and their order) so that display types (such as flexbox and grid) that depend on the tree structure (and particularly on the lack of other intermediate containers) will produce interoperable results
* add a `::details-content` pseudo-element to address the second slot so that a container for the "additional information" in the `<details>` element can be styled.

For details, see:
* CSS standardization issue: [w3c/csswg-drafts#9879](https://github.com/w3c/csswg-drafts/issues/9879)
* HTML standardization PR: [whatwg/html#10265](https://github.com/whatwg/html/pull/10265)

## Related changes

There are also a set of related changes that help to improve the ability to animate these elements,
although these are not strictly part of this proposal:

* [The `transition-behavior` property](https://drafts.csswg.org/css-transitions-2/#transition-behavior-property)
  (part of the `@starting-style` and `transition-behavior` focus area in [Interop 2024](https://wpt.fyi/interop-2024))
* [The `@starting-style` rule](https://drafts.csswg.org/css-transitions-2/#defining-before-change-style)
  (part of the `@starting-style` and `transition-behavior` focus area in [Interop 2024](https://wpt.fyi/interop-2024))
* [The ability to transition to/from auto heights](https://github.com/w3c/csswg-drafts/blob/main/css-values-5/calc-size-explainer.md)

## Examples

The following [example](examples/opacity-animation.html) code causes the opacity and the height
of a `<details>` element to animate when it opens and closes.

```css
:root {
  /* This is tentative syntax pending resolution of
   * https://github.com/w3c/csswg-drafts/issues/10294 .
   * Until this syntax is finalized and implemented, the height: auto
   * below can be replaced with height: calc-size(auto).  */
  size-keyword-interpolation: auto;
}

details {
  --open-close-duration: 500ms;
}

details::details-content {
  opacity: 0;
  height: 0;
  overflow-y: hidden; /* clip content when height is animating */
  transition: content-visibility var(--open-close-duration) allow-discrete step-end,
              opacity var(--open-close-duration),
              height var(--open-close-duration);
}

details[open]::details-content {
  opacity: 1;
  height: auto;
  transition: content-visibility var(--open-close-duration) allow-discrete step-start,
              opacity var(--open-close-duration),
              height var(--open-close-duration);
}
```
