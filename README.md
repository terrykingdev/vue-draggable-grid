# vue-draggable-grid

Didn't have much luck finding one of the draggable libraries that handled responsive grids (I'm sure there are, just couldn't find one). So this is a very simple means of dragging responsive columns into a new order.

Sometimes the behavior can seem weird if you have lots of columns of different widths. But it works as expected, sometimes the element can't rearrange because they won't fit into the space left so your dragged element will sometimes appear after the position you expected to be placed.

Vue2 + vuetify

To do:
* If you drag a thin element and move too fast the element gets 'left behind'. Probably something to do with event bubbling. Need to look into it.
* Add touch support

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
