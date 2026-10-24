# Grid 25

Grid 25 is a responsive Hot Wheels collection showcase built with Vue 3 and Vite.

## Project structure

- `src/App.vue` owns collection state, filtering, and theme behavior.
- `src/components/` contains the header, welcome banner, collection grid, and car detail modal.
- `src/data/cars.js` contains the collection data.
- `src/assets/main.css` contains the site styles.
- `public/grid25-logo.png` is used for the header brand mark and browser favicon.
- `index.html` provides the document shell and loads the Vue entry point.

## Development

From this directory, install dependencies and start the Vite server:

```sh
npm install
npm run dev
```

The workspace root also provides forwarding `dev`, `build`, and `preview` scripts.

## Production build

```sh
npm run build
npm run preview
```
