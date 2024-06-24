# Mutation Observers

Author(s): ortiza5 (nomadic)

There are many different elements to Vivaldi's interface that can be toggled on and off with different settings or keyboard shortcuts. While this is great for customizability, JavaScript mods that affect these UI elements can easily be inadvertently broken by simple actions such as hiding a toolbar, entering fullscreen, or switching over to the built in email experiment.

You can always tell the users that they aren't allowed to use various settings that you know are incompatible with your mod, but in order to make a more robust mod that fits with a wider range of users with diverse browser workflows, you should make your mod handle changes in the UI gracefully. One way to accomplish this is with mutation observers.

## Contents
- [Basics of mutation observers](#basics-of-mutation-observers)
  - [Target element](#target-element-targetelement)
  - [Configuration options](#configuration-options-configurationoptions)
  - [Handler function for mutations](#handler-function-for-mutations-callback)
- [Code examples](#code-examples)
  - [Fullscreen observer](#fullscreen-observer)
  - [Address bar / mail bar observer](#address-bar--mail-bar-observer)
  - [Toggling UI element (status bar) observer](#toggling-ui-element-status-bar-observer)

## Basics of mutation observers

The main purpose of mutation observers is, as the name suggests, to observe for mutations. These mutations, or changes, can be for things like adding or removing an element and changing attributes like classes on an element.

The basic structure of a mutation observer should look something like this:

```JavaScript
// Fill in "..." with appropriate values

const targetElement = document.querySelector(" ... ");
const configurationOptions = { ... : true};

function callback(mutations) {
  mutations.forEach(function (mutation) {
    if (mutation.type === ... ) {
      // act on mutation here
    }
  }
}

const observer = new MutationObserver(callback);
observer.observe(targetElement, configurationOptions);
```

The actual creation of the observer is only contained in the last two lines, but in order to successfully get it to do what you want, you need to set up certain values prior to creating the observer (you can define them in place, but it is better practice to define the values beforehand). The three parts you need to provide are as follows:

- The target element
- The configuration options
- The handler function for mutations

### Target element (`targetElement`)

The target element is the part of the HTML structure that the observer will watch. The element can be selected with methods like: `document.querySelector`, `document.getElementById`, or `document.getElementsByClassname` (when paired with selecting the first result with `[0]`).

### Configuration options (`configurationOptions`)

The configuration options are a JavaScript `object` that sets up the observer to watch for the appropriate types of mutations on the target element. There are many different options, but the 2 most important to mods are `attributes` and `childList`. A more complete explanation of all the options can be found [here](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserverInit).

- Setting `configurationOptions = {attributes: true}` will set the observer to watch for changes to the attributes of the `targetElement`.

  - This is particularly useful for detecting changes in state represented by adding or removing a class on an element. Like with classes such as `.active` or `.hidden`.
  - The `#browser` div in Vivaldi contains several state based classes that reflect settings the user has selected.

- Setting `configurationOptions = {childList: true}` will set the observer to watch for child elements of the `targetElement` that are added or removed.

  - Sometimes elements are not hidden or transformed into something else when a change to the UI is made. In these cases, new elements are added, so an observer of this type is useful to detect those changes.
  - Adding `subtree: true` as well, like so `{childList: true, subtree: true}`, will extend the observer to all decedent elements of the `targetElement` rather than only the first layer of children.
    - So all the elements below the `.parent` will be observed with this configuration (denoted by the class `observed`):

      ```HTML
      <div class="parent targetElement">
        <div class="child observed">
          <div class="grandChild observed"> ... </div>
        </div>
        <div class="child observed"> ... </div>
      </div>

      <div class="notObserved"> ... </div>
      ```

### Handler function for mutations (`callback()`)

The handler function is run whenever a mutation of the types you configured is detected.

The list of mutations should be passed in as a parameter to allow the function to perform different actions depending on the contents of the mutations. In the example, this parameter is named `mutations` and included like so:

```JavaScript
function callback(mutations) {
  ...
```

There is often more than one mutation included in the list, so it is important to check each one individually. This is accomplished with a `forEach` loop in the example (usage information can be found [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)).

Once an individual mutation is isolated, you can check what type of mutation it is by looking at the `type` attribute of the mutation object ( `if (mutation.type === ... ) { ...` ). The types that are possible are determined by the configuration of the observer. The `type` could be “`attribute`” or “`childList`” based on the configuration options outlined above.

When it is determined that the mutation is the correct type that you are looking for, you can then do any checks on it that you wish and run any actions accordingly. See the code examples below to see various implementations that describe what you can do. The [fullscreen observer](#fullscreen-observer) shows an example of acting on an attribute change and the [address bar / mail bar observer](#address-bar--mail-bar-observer) shows checking `addedNodes` with a childList observer.

## Code examples

The following are some examples of mutation observers that could be useful in the creation of mods. They start with the full code and then include an explanation of how they work.

### Fullscreen observer

// TODO: write intro

// TODO: generalize and comment code

```JavaScript
let browser = document.getElementById("browser");
let oldState =
  browser.classList.contains("fullscreen") ||
  browser.classList.contains("minimal-ui");
let fullscreenObserver = new MutationObserver(function (mutations) {
  mutations.forEach(function (mutation) {
    if (mutation.attributeName == "class") {
      let isFullscreen =
        mutation.target.classList.contains("fullscreen") ||
        mutation.target.classList.contains("minimal-ui");
      if (oldState !== isFullscreen) {
        oldState = isFullscreen;
        if (!isFullscreen) {
          // Run all changes that that are affected by fullscreen changes
          ...
        }
      }
    }
  });
});

fullscreenObserver.observe(browser, { attributes: true });
```

// TODO: write explanation

---

### Address bar / mail bar observer

// TODO: write intro

// TODO: generalize and comment code

```JavaScript
let main = document.getElementById("main");
// get the initial state of the addressbar as either urlbar or mailbar
let oldIsMailBarActive = main.firstChild.classList.contains("toolbar-mailbar");
let addressBarObserver = new MutationObserver(function (mutations) {
  mutations.forEach(function (mutation) {
    // only re-add on new nodes added. The list addedNodes will only have a
    // length attribute when it contains added nodes
    if (mutation.addedNodes.length) {
      // get the new state of the addressbar
      let isMailBarActive =
        mutation.addedNodes[0].classList.contains("toolbar-mailbar");

      // if it is different from the previous state, we need to act on it
      if (oldIsMailBarActive !== isMailBarActive) {
        // update the old value for comparisons on future mutations
        oldIsMailBarActive = isMailBarActive;
        // if the addressbar isn't the mailbar, we can re-add the button
        if (!isMailBarActive) {
          // Run all changes that are only in the url bar and not the mail bar
          ...
        }
      }
    }
  });
});
// only need to check childList for added nodes
addressBarObserver.observe(main, { childList: true });
```

// TODO: write explanation

---

### Toggling UI element (status bar) observer

// TODO: write intro

// TODO: generalize and comment code

```JavaScript
// Think new Snapshot probably broke this, so need to update
let browser = document.getElementById("browser");
let browserObserver = new MutationObserver(function (mutations) {
  mutations.forEach(function (mutation) {
    if (
      Array.from(mutation.addedNodes).find((element) => {
        element.classList.contains("toolbar-statusbar");
      })
    ) {
      ...
    }
  });
});

browserObserver.observe(browser, { childList: true });
```

// TODO: write explanation
