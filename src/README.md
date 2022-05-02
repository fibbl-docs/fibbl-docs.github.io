# Custom Elements API

This document describes all the custom HTML elements provided by Fibbl to its clients. The elements are supposed to be
used on any third party webpage. You can quickly try out all the code examples
on [https://jsfiddle.net/](https://jsfiddle.net/) or any similar website (but you will have to add `token` and `secret`,
read further for more details).

All elements are used the same way:

1. You should include (or add into your js bundle) a script for a certain element with `type="module"` (it's important).
2. It may be rather useful to include the following style tag along with the script:

    ```html
    <style>:not(:defined){opacity:0;}</style>
    ```
   It can be especially handy when you use an element with custom markup (like the `fibbl-bar`). Before a script is
   loaded, that markup is shown with default styles, and the rule above hides it until a custom element is defined.

3. Some elements include others, e.g. `fibbl-bar` includes `model-viewer`, `carousel` and `qr-code`, so you don't need
   to include them separately, if you use `fibbl-bar`.

4. Use the element as any other HTML element - you can add styles, classes, event handlers and other valid html
   attributes to it. But do not forget that custom elements cannot be self-closed, that is, instead
   of `<fibbl-element />` you should always write `<fibbl-element></fibbl-element>`.

## Adding `token` and `secret` for development

With no additional configuration scripts are allowed to fetch product data only from the company website. If a request
is made from a different origin, it will be blocked by the back-end, and you will get an error.

To enable scripts on localhost (or any other domain), you should add the following configuration:

```html
<script
    src="https://cdn.fibbl.com/fibbl-model-viewer.js" type="module"
    data-fibbl-config
    data-token="token_given_by_Fibbl"
    data-secret="secret_given_by_Fibbl"
></script>
<script src="https://cdn.fibbl.com/fibbl-carousel.js" type="module"></script>
<!-- any other scripts, the config should be only in the first one -->
```

The configuration must comply with the following rules:

1. It must be provided in the first `script` tag used to include a Fibbl script.
2. If you use several scripts it must be added only to the first one.
3. It has to contain `data-fibbl-config` attribute with no value as a unique mark of the single source of global config.
4. It may contain other `data-` attributes with the payload, currently only `data-secret` and `data-token`.

The `secret` and `token` should be sent to you by Fibbl managers.

You must **NEVER** use `token` and `secret` in production on a publicly available website. They are to be used **only
for development** or on the back-end.

In all subsequent code examples we don't use `token` and `secret` as it must be on a production website.

## Shared behavior

All Fibbl components work in the following way:

1. Initially a component renders itself completely transparent.
2. Then it checks availability of the product with the provided product id.
3. If the product is available, the component makes itself visible.
4. If not, the component hides itself. Currently, it's hidden with `display: none`, that is, not removed from the DOM.
   But this way of hiding can be changed in the future, so you must rely on it (e.g. using CSS sibling selectors).

## fibbl-model-viewer

An element to display a 3D model of product in a special viewer.

Example:

```html
<script src="https://cdn.fibbl.com/fibbl-model-viewer.js" type="module"></script>
<fibbl-model-viewer
    data-product-id="123456"
    style="width: 300px; height: 300px"
></fibbl-model-viewer>
```

Supported custom attributes:

- `data-product-id` - a unique id of product in Fibbl's databases.
- `data-controls` - a list of controls (e.g. expand button). Possible values `expand`, `none`. Default is `expand`.

## fibbl-carousel

A carousel element showing images of product.

Example:

```html
<script src="https://cdn.fibbl.com/fibbl-carousel.js" type="module"></script>
<fibbl-carousel
    data-product-id="123456"
    style="width: 300px; height: 300px"
></fibbl-carousel>
```

Supported custom attributes:

- `data-product-id` - a unique id of product in Fibbl's databases.
- `data-controls` - a list of controls (e.g. expand button). Possible values `expand`, `none`. The default is `expand`.
- `data-mode` - a current mode of the element, either `full` or `poster` (render only one image). The default
  is `poster`.

## fibbl-qr-code

A customized element showing a QR code with a link to augmented reality or virtual try on pages on the main Fibbl
website.

Example:

```html
<script src="https://cdn.fibbl.com/fibbl-qr-code.js" type="module"></script>
<fibbl-qr-code
    data-product-id="123456"
    data-type="ar"
    style="width: 300px; height: 300px"
></fibbl-qr-code>
```

Supported custom attributes:

- `data-product-id` - a unique id of product in Fibbl's databases.
- `data-type` - either `ar` or `vto`. The parameter defines whether the link leads to an augmented reality or virtual
  try on page.

## fibbl-bar

A complex element. It must be provided with buttons as children (by a button we imply any element with `data-element`
attribute). Each button corresponds to either `model-viewer`, `carousel` or `qr-code`, which are rendered inside a user
provided container or in a pop-up.

```html
<script src="https://cdn.fibbl.com/fibbl-bar.js" type="module"></script>
<style>
    :not(:defined) {
        opacity: 0;
    }

    #container {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 300px;
        max-height: 300px;
        border: 3px solid green;
    }

    .fibbl-bar-content {
        height: 300px;
        width: 300px;
    }
</style>

<div id="container">
    <img id="picture" src="http://placekitten.com/300/300" alt="cat" />
</div>

<fibbl-bar
    data-container-selector="#container"
    data-default-content-selector="#picture"
    data-product-id="123456"
>
    <button data-element="default">Gallery</button>
    <button data-element="fibbl-model-viewer">Model Viewer</button>
    <button data-element="fibbl-carousel">Carousel</button>
    <button data-element="fibbl-qr-code" data-type="ar">AR</button>
    <button data-element="fibbl-qr-code" data-type="vto">VTO</button>
</fibbl-bar>
```

Initially, the bar hides all the buttons. Then it makes a request to the Fibbl's databases and analyzes what options are
enabled for the product with the provided `product-id` (e.g. `3D model` can be enabled, but `Virtual Try On` disabled or
not supported by the product at all). Then only buttons corresponding to the enabled options are shown, all others are
hidden.

Each button should be provided with the `data-element` attribute, which is either a name of a Fibbl element or `default`
. All other `data-` attributes are passed to the corresponding element, when it's created, so you can configure elements
this way.

`default` element is a special case - no element is rendered, the default content is used. It should be used if you have
a custom gallery or something and provide `data-default-content-selector` (read about it to know more).

Custom attributes:

- `data-product-id` - required. A unique id of product in Fibbl's databases.
- `data-container-selector` - optional. A CSS selector of **one** element wherein the bar renders its elements. If not
  provided, the bar will render elements into a pop-up.
- `data-default-content-selector` - optional. A CSS selector of **one** element that already exists (usually inside the
  container). It's hidden by the bar when any Fibbl element is rendered, and shown back once the `default` button is
  clicked. A typical use case to hide/show a gallery or a poster image, as it's in the example above.
- `data-insert-mode` - either `append` (default) or `prepend`. This option determines how the bar inserts its content
  into the container. If the bar itself is located in the same container, and the content is needed to be inserted above
  the bar, then the `prepend` option may be used to avoid adding one more wrapper element.

CSS classes:

- `.fibbl-active` - this class is added to a button inside the bar, once it's selected and the corresponding element is
  shown. It's provided to allow users to override standard bar's styles.
- `.fibbl-bar-content` - a class of the component which is inserted into client defined container. Using this class you
  can style the element. Usually only `height` and `width` required.

Custom Events:

- `fibbl-bar-content-change` - [CustomEvent][1] with `{detail: {button: HTMLElement}}`, where `button` is the clicked
  HTML element with `data-element` attribute. Currently, the event is fired only when the bar has a container and the
  content has been updated after the click. The event may be useful on mobile devices, where a click on "Augmented
  Reality" or "Virtual Try On" buttons doesn't cause the content change, but the AR or VTO are launched at once and the
  event isn't fired.

### Advanced usage of `fibbl-bar`

If you want to use the `fibbl-bar` to create a fancy menu with significant customizations, first of all, you should
**consider implementing it from scratch without `fibbl-bar` at all**, using standalone components. It will give you
more freedom.

If you still want to use the `fibbl-bar` (the only probable reason to use it is that it hides buttons for unsupported
features), here is a list of what you can do with it (also read about the styles reset in the next section):
1. The component listens for clicks on itself not on buttons inside. Also, it traverses the event path looking for an
   element with the `data-element` attribute. It means that you can use something like this
   ```html
   <div data-element="fibbl-model-viewer">
      <img src="https://icon_url" alt="button icon" />
      <span>3D model</span>
   </div>
   ```
   as a button, and if a user clicks on the `img` icon, the click still will be processed correctly, because it has a
   parent with `data-element` attribute.
2. The bar hides unsupported buttons using a global style. But it's applied only if there is at least one unsupported
   button inside, when the bar is added to the DOM. Again, you must not rely on the presence of hidden buttons in the
   DOM - the way of hiding can be changed in the future (e.g. don't use CSS sibling selectors).
3. Given the first two points, you can change the markup dynamically inside the bar - it will handle clicks on new
   markup correctly and still will hide unsupported buttons even if you inject them anew.
4. Correspondingly if you stop a click propagation, the bar will not react.
5. The bar requires you to initially provide at least 1 button for a supported feature. If there are no buttons for
   supported features, the bar will hide itself completely as any other Fibbl component.
6. You can nest `[data-element]` buttons as you wish, that is, it's allowed using wrappers around your buttons.
7. But note that the bar will hide only `[data-element]`s, not wrappers around them. So probably you will want to add
   `data-element` attribute to the first wrapper and in most cases it will be a direct child of the bar component.

## Element styles

All elements render their content inside Shadow DOM, so there will be no conflicts between Fibbl's styles and client's
styles inside the elements.

But the elements themselves can be styled by clients. The elements have default styles, e.g.

```css
:host {
    display: block;
    box-sizing: border-box;
    width: 100%;
    height: 100%;
}
```

You can reset the styles completely and maintain them by yourself, e.g.

```css
fibbl-model-viewer {
    all: unset;
    /* display, width and height are the most important */
    display: block;
    box-sizing: border-box;
    width: 300px;
    height: 50vh;
    /* Anything else you font to add */
}
```

All elements are responsive and usually don't have predefined sizes. To provide freedom in styling, elements' content
don't rely on `:host` styles, but try to get all the space via `width: 100%; height: 100%`. **So it's required to
provide elements with explicit `width` and `height`**. How you do it is up to you.

However, there are some pitfalls. E.g. you can scale your element via flex inside a container with `min-height` value
and no explicit `height`. In this case the element itself will take some space, but its content won't be visible (
because it will be of 0 size). If possible provide the element with explicit sizes or
use `display: grid; grid-template: 1fr / 1fr;` as an escape hatch. Inside a grid inner elements will take all space.
However, explicit sizes are preferable.

As for other properties like `border`, `background`, `margin`, `width`, `height` etc., which cannot affect inner layout,
you can change them as you want.

### Client's markup

In case of elements which accept children elements, e.g. `fibbl-bar` with buttons, the default styles for the buttons
are provided, but in most cases they will have to be changed. In this case it's better to always reset the styles
completely, if you want to customize your own markup, e.g.:

```css
fibbl-bar {
    all: unset; /* reset all the styles */
    display: flex;
    /* your rules here */
}

fibbl-bar [data-element] {
    all: revert; /* reset all the styles */
    /* your rules here */
}
```

The reset of styles isn't mandatory, but the default styles can change in the future, e.g. `border` can be replaced
with `box-shadow` and you will end up with your custom `border` and default `box-shadow`, if you haven't reset it. Fibbl
guarantees that default styles will be reasonable and relatively good-looking, but not that they will not change in the
future.

## Dynamic attribute changes

Right now all elements don't support dynamic attribute changes. That is once an element is attached to the DOM the first
time, its attributes must not be changed - it will not react. If you need to change attributes, recreate an element. In
frameworks like React you should provide a new `key` to an element each time you need to update it.

## API limitations

If you find the current API not enough to implement what you want, please, don't try to bypass the limitations - ask
Fibbl managers to enhance it. Otherwise, after an API update your workarounds can be broken.

[1]: https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent/CustomEvent