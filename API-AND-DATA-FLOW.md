# API Calls and Data Binding in Grid 25

This document explains how Grid 25 gets its data, when it makes network
requests, and how Vue binds that data to the interface. The project is a Vue 3
single-page application built with Vite.

## 1. Where the data comes from

The main collection is **not loaded from an API**. It is defined as a
JavaScript array in [`src/data/cars.js`](./src/data/cars.js), exported as
`carsData`, and imported by [`src/App.vue`](./src/App.vue). Each car is a
plain object containing fields such as:

- `id`: stable identifier used for Vue's list key and displayed collection
  number.
- `name`, `year`, `color`, `series`, and `rarity`: descriptive fields used in
  cards, filters, and the detail modal.
- `isTreasureHunt`: a boolean used by the Treasure Hunt filter and count.
- `image`: URL of the image displayed by the browser.
- `imageSource`: source reference recorded in the data; it is not currently
  used by the Vue components.
- `detailsSourceUrl`: URL requested if a visitor chooses to load more details.
- `notes`: locally stored description displayed in the modal.

At startup, `App.vue` initializes `cars` with `ref(carsData)`. There is no
initial collection request, loading spinner, API client, or backend service in
this project. The car list and the initial statistics are available as soon as
the JavaScript application has loaded.

## 2. Network requests in this project

There are three distinct kinds of browser network activity. Only one is an
explicit JavaScript API-style request.

### 2.1 Optional detail request: `fetch()`

[`src/components/CarDetailModal.vue`](./src/components/CarDetailModal.vue)
uses the browser's built-in `fetch()` function to request a car's
`detailsSourceUrl`. This happens only after a visitor opens a car and presses
**See more**. The app does not send this request when the page first loads or
when it renders the car cards.

The request is a default `GET`: the code does not specify a method, request
body, custom headers, or authentication. In the current data, the detail URLs
point to external product pages, often through `r.jina.ai`; they are not
endpoints belonging to a Grid 25 backend.

### 2.2 Images: browser-managed asset requests

The templates bind `car.image` to `<img src="...">`. The browser downloads
those images as image resources; Vue does not call `fetch()` for them. Car
images use `loading="lazy"` in the collection grid, so the browser can defer
loading images that are offscreen. The featured image and modal image are
rendered as normal image elements when their components are displayed.

The header logo and favicon are separate static assets under `public/`.

### 2.3 Third-party page resources

[`index.html`](./index.html) loads Tailwind CSS and a font from external
providers. These are browser resource loads, not requests made by Vue
components.

## 3. Detail request lifecycle

The detail request follows this sequence:

1. **Select a car.** A card emits `select-car` with the car object. The root
   component saves that object as `selectedCar`, which makes the modal appear.
2. **Request on demand.** The modal's **See more** button calls
   `toggleProductDetails()`. If a request is already loading, or there is no
   `detailsSourceUrl`, the function returns without starting another request.
3. **Check the URL scheme.** `new URL(...)` parses the source URL, and the code
   allows only `http:` and `https:`. Invalid URLs or other protocols set the
   status to `error`.
4. **Start the request.** The component creates an `AbortController`, stores
   the `loading` status, and calls:

   ```js
   const response = await fetch(props.car.detailsSourceUrl, {
     signal: requestController.signal,
   })
   ```

   Passing the abort signal allows a pending request to be cancelled.
5. **Check the HTTP response.** A `404` is treated as unavailable. Other
   non-success HTTP statuses throw an error. A successful response is read as
   text with `response.text()`.
6. **Interpret the response.** The component first tries to parse the text as
   JSON. If it is JSON with a string `description`, that description is
   converted to plain text using `DOMParser` and `textContent`. Otherwise, the
   component attempts to extract a description from reader-style text or
   Markdown. The extracted text is trimmed and limited to 2,000 characters.
7. **Update the interface.** A non-empty description sets the status to
   `loaded` and expands the details region. An empty or unrecognized
   description sets the status to `unavailable`.

The template reflects the status reactively:

| `detailsStatus` | What the user sees |
|---|---|
| `idle` | The **See more** button, if the car has a details URL. |
| `loading` | A disabled button and “Fetching casting details…” status message. |
| `loaded` | The fetched description, initially expanded; the button toggles between **See less** and **See more**. |
| `unavailable` | A message that additional details are not available. |
| `error` | An error message containing the failure detail available to the browser. |

Clicking **See less** after a successful request only collapses the displayed
description; it does not make another request. If a request ended in
`unavailable` or `error`, pressing **See more** again can retry it.

### Cancellation and browser constraints

A `watch()` observes the selected car's `id`. When the selection changes, it
aborts a previous request and resets the modal's local request state. Closing
the modal changes the selected car to `null`, which also triggers that reset.
The component additionally aborts the request in `onBeforeUnmount()`.

Because these URLs are on other domains, successful access depends on browser
network rules and the remote server. In particular, the remote server must
allow the page's origin through its CORS policy for JavaScript to read the
response. An image may be displayable as an `<img>` while a JavaScript
`fetch()` of a related page is blocked by CORS; those are different browser
operations. The code reports request failures in the modal rather than
silently presenting them as successful descriptions.

## 4. How Vue binds data to the interface

Vue templates connect JavaScript values to rendered HTML. In this project,
the important binding forms are:

- **Text interpolation**, `{{ car.name }}`: displays a value as text.
- **Attribute binding**, `:src="car.image"` or `:value="searchQuery"`:
  updates an HTML attribute/property from a JavaScript expression. The `:`
  is shorthand for `v-bind:`.
- **Event binding**, `@click="..."` or `@input="..."`: runs a handler when a
  browser event occurs. The `@` is shorthand for `v-on:`.
- **Conditional rendering**, `v-if` / `v-else`: adds or removes a piece of
  interface depending on a condition.
- **List rendering**, `v-for`: creates one repeated element for each item.
- **Class binding**, `:class="{ 'dark-theme': darkTheme }"`: conditionally
  adds or removes a CSS class.

For example, `CarCollection.vue` renders a card for each filtered car:

```vue
<article v-for="car in filteredCars" :key="car.id">
  <img :src="car.image" :alt="car.name" />
  <h3>{{ car.name }}</h3>
</article>
```

`v-for` repeats the card markup. The unique `car.id` key lets Vue associate
each rendered element with the correct car as the list changes. The bindings
then provide each element's image and text.

## 5. Reactive state and derived values

The root component in [`src/App.vue`](./src/App.vue) owns values shared across
multiple parts of the page:

```js
const cars = ref(carsData)
const darkTheme = ref(true)
const filterOnlyTH = ref(false)
const searchQuery = ref('')
const selectedSeries = ref('')
const selectedCar = ref(null)
```

`ref()` creates a reactive value. In JavaScript, its current value is accessed
using `.value`; in Vue templates, refs are automatically unwrapped. When one of
these values changes, Vue updates the portions of the DOM that depend on it.

The component also defines computed values:

- `uniqueSeries` derives the distinct series names from `cars`.
- `countTreasures` and `countPremiums` derive the banner counts.
- `filteredCars` derives the visible list from `cars`, `searchQuery`,
  `selectedSeries`, and `filterOnlyTH`.

A **computed property** is a cached value derived from reactive dependencies.
For example, the displayed list changes when a filter changes, while the
original `cars` array remains the source collection. The visible count uses
`filteredCars.length`; the total uses `cars.length`.

## 6. Parent-child data binding: props down, events up

The application follows a **props down, events up** pattern:

- A parent passes values into a child as **props**.
- A child reports user actions to its parent using **emitted events**.
- The parent updates its state; Vue then passes updated props back down.

For example, `App.vue` passes the filtered list and current filter values to
`CarCollection.vue`. The collection component declares those props with
`defineProps()` and declares its events with `defineEmits()`.

The search input illustrates a controlled binding:

```vue
<input
  :value="searchQuery"
  @input="$emit('update:search-query', $event.target.value)"
/>
```

The input's displayed value comes from the `searchQuery` prop. When it changes,
the child emits the new string. The parent listens for that event and updates
its `searchQuery` ref:

```vue
@update:search-query="searchQuery = $event"
```

This is functionally similar to two-way binding, but the project spells out
both directions rather than using `v-model`. The series `<select>` uses the
same pattern with `update:selected-series`.

The same event direction is used for other actions:

| Child action | Event | Parent update |
|---|---|---|
| Click a car card | `select-car` with the car object | Sets `selectedCar`. |
| Click reset | `reset-filters` | Clears search, series, and Treasure Hunt filters. |
| Click the theme button | `toggle-theme` | Flips `darkTheme`. |
| Choose “Browse Hot Trucks” | `browse-hot-trucks` | Sets the series filter to `HW Hot Trucks`. |
| Close the modal | `close` | Sets `selectedCar` to `null`. |

The event name is written in kebab case in the templates (for example,
`@toggle-theme`), while Vue component props declared in camel case are passed
in kebab case (for example, `filterOnlyTh` is supplied as `:filter-only-th`).

## 7. Full interaction example: searching for a car

1. The visitor types into the search input rendered by
   `CarCollection.vue`.
2. The input's `@input` handler emits `update:search-query` with the typed
   string.
3. `App.vue` assigns `$event` to `searchQuery`.
4. Vue observes that `searchQuery` is reactive and invalidates the
   `filteredCars` computed value.
5. `filteredCars` checks whether each car's name or color contains the
   lowercased query, or its year string contains the query. It also applies
   the series and Treasure Hunt conditions.
6. Vue updates the `filtered-cars` prop on `CarCollection.vue`.
7. The `v-for` list and “Showing _ of _ cars” count update. If there are no
   matches, `v-if` displays the empty state instead.

No network request is involved in this flow; all filtering happens in the
browser against the local array.

## 8. Error and state boundaries

The app's main list state and the modal's optional request state are separate.
Failure to fetch extra product text does not prevent the locally defined car
cards, notes, or collection filters from working.

This is a small front-end-only project rather than a full API-backed
application. It has no API base URL configuration, shared request layer,
server-side persistence, or user-specific data. The optional details request
is made directly by the user's browser to the external URL stored on that
car.
