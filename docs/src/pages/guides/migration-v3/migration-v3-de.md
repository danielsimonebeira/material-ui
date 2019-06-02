# Migration From v3 to v4

<p class="description">Ja, v4 wurde veröffentlicht!</p>

Looking for the v3 docs? [Find them here](https://material-ui.com/versions/).

> This document is a work in progress. Have you upgraded your site and run into something that's not covered here? [Add your changes on GitHub](https://github.com/mui-org/material-ui/blob/master/docs/src/pages/guides/migration-v3/migration-v3.md)

## Introduction

This is a reference for upgrading your site from Material-UI v3 to v4. While there's a lot covered here, you probably won't need to do everything for your site. We'll do our best to keep things easy to follow, and as sequential as possible so you can quickly get rocking on v4!

## Why you should migrate

This documentation page covers the *how* of migrating from v3 to v4. The *why* is covered in the [release blog post on Medium](https://medium.com/material-ui/material-ui-v4-is-out-4b7587d1e701).

## Updating Your Dependencies

The very first thing you will need to do is update your dependencies.

### Update Material-UI version

You need to update your `package.json` to use the latest version of Material-UI.

```json
"dependencies": {
  "@material-ui/core": "^4.0.0"
}
```

Oder führe aus

```sh
npm install @material-ui/core@next

or

yarn add @material-ui/core@next
```

### Update React version

The minimum required version of React was increased from `react@^16.3.0` to `react@^16.8.0`. This allows us to rely on [Hooks](https://reactjs.org/docs/hooks-intro.html) (we no longer use the class API).

### Update Material-UI Styles version

If you are previously using `@material-ui/styles` with v3 you need to update your `package.json` to use the latest version of Material-UI Styles

```json
"dependencies": {
  "@material-ui/styles": "^4.0.0"
}
```

Oder führe aus

```sh
npm install @material-ui/styles@next

or

yarn add @material-ui/styles@next
```

## Handling breaking changes

### Grundlegendes

- Every component forward their ref. This is implemented by using `React.forwardRef()`. This affects the internal component tree and display name and therefore might break shallow or snapshot tests. `innerRef` will no longer return a ref to the instance (or nothing if the inner component is a function component) but a ref to its root component. The corresponding API docs list the root component.

### Stile

- ⚠️ Material-UI depends on JSS v10. JSS v10 is not backward compatible with v9. Make sure JSS v9 is not installed in your environment. Removing `react-jss` from your package.json can help.
- Remove the first option argument of `withTheme()`. The first argument was a placeholder for a potential future option. We have never found a need for it. It's time to remove this argument. It matches the emotion and styled-components API.
  
  ```diff
  -const DeepChild = withTheme()(DeepChildRaw);
  +const DeepChild = withTheme(DeepChildRaw);
  ```

- Scope the [keyframes API](https://cssinjs.org/jss-syntax/#keyframes-animation). Sie sollten die folgenden Änderungen in Ihrer Codebase anwenden. Es hilft, die Animationslogik zu isolieren:
  
  ```diff
    rippleVisible: {
      opacity: 0.3,
  -   animation: 'mui-ripple-enter 100ms cubic-bezier(0.4, 0, 0.2, 1)',
  +   animation: '$mui-ripple-enter 100ms cubic-bezier(0.4, 0, 0.2, 1)',
    },
    '@keyframes mui-ripple-enter': {
      '0%': {
        opacity: 0.1,
      },
      '100%': {
        opacity: 0.3,
      },
    },
  ```

### Theme

- The `theme.palette.augmentColor()` method no longer performs a side effect on its input color. To use it correctly, you have to use the returned value.
  
  ```diff
  -const background = { main: color };
  -theme.palette.augmentColor(background);
  +const background = theme.palette.augmentColor({ main: color });
  
  console.log({ background });
  ```

- You can safely remove the next variant from the theme creation:
  
  ```diff
  typography: {
  - useNextVariants: true,
  },
  ```

- `theme.spacing.unit` usage is deprecated, you can use the new API:
  
  ```diff
  label: {
    [theme.breakpoints.up('sm')]: {
  -   paddingTop: theme.spacing.unit * 12,
  +   paddingTop: theme.spacing(12),
    },
  }
  ```
  
  You can use the `https://github.com/mui-org/material-ui/tree/master/packages/material-ui-codemod/README.md#theme-spacing-api` migration helper to make the migration process smoother.
  
  *Tip: you can provide more than 1 argument: theme.spacing(1, 2) // = '8px 16px'*
  
  You can use [the migration helper](https://github.com/mui-org/material-ui/tree/master/packages/material-ui-codemod/README.md#theme-spacing-api) on your project to make this smoother.

### Layout

- [Grid] In order to support arbitrary spacing values and to remove the need to mentally count by 8, we are changing the spacing API:
  
  ```diff
    /**
     * Defines the space between the type `item` component.
     * It can only be used on a type `container` component.
     */
  -  spacing: PropTypes.oneOf([0, 8, 16, 24, 32, 40]),
  +  spacing: PropTypes.oneOf([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]),
  ```
  
  Going forward, you can use the theme to implement [a custom Grid spacing transformation function](https://material-ui.com/system/spacing/#transformation).

- [Container] Moved from `@material-ui/lab` to `@material-ui/core`
  
  ```diff
  -import Container from '@material-ui/lab/Container';
  +import Container from '@material-ui/core/Container';
  ```

### Button

- [Button] Remove the deprecated button variants (flat, raised and fab):
  
  ```diff
  -<Button variant="raised" />
  +<Button variant="contained" />
  ```
  
  ```diff
  -<Button variant="flat" />
  +<Button variant="text" />
  ```
  
  ```diff
  -import Button from '@material-ui/core/Button';
  -<Button variant="fab" />
  +import Fab from '@material-ui/core/Fab';
  +<Fab />
  ```
  
  ```diff
  -import Button from '@material-ui/core/Button';
  -<Button variant="extendedFab" />
  +import Fab from '@material-ui/core/Fab';
  +<Fab variant="extended />
  ```

- [ButtonBase] The component passed to the `component` prop needs to be able to hold a ref. The [composition guide](/guides/composition/#caveat-with-refs) explains the migration strategy.
  
  This also applies to `BottomNavigationAction`, `Button`, `CardActionArea`, `Checkbox`, `ExpansionPanelSummary`, `Fab`, `IconButton`, `MenuItem`, `Radio`, `StepButton`, `Tab`, `TableSortLabel` as well as `ListItem` if the `button` prop is true

### Card

- [CardActions] Rename the `disableActionSpacing` prop `disableSpacing`.
- [CardActions] Remove the `disableActionSpacing` CSS class.
- [CardActions] Rename the `action` CSS class `spacing`.

### ClickAwayListener

- [ClickAwayListener] Hide react-event-listener props.

### Dialog

- [DialogActions] Rename the `disableActionSpacing` prop `disableSpacing`.
- [DialogActions] Rename the `action` CSS class `spacing`.
- [DialogContentText] Use typography variant `body1` instead of `subtitle1`.
- [Dialog] The child needs to be able to hold a ref. The [composition guide](/guides/composition/#caveat-with-refs) explains the migration strategy.

### Divider

- [Divider] Entfernen Sie die veraltete `inset` Eigenschaft.
  
  ```diff
  - <0 />
  + <1 />
  ```

### ExpansionPanel

- [ExpansionPanelActions] Rename the `action` CSS class `spacing`.
- [ExpansionPanel] Increase the CSS specificity of the `disabled` style rule.

### List

- [List] Rework the list components to match the specification:
  
  - The usage of the `ListItemAvatar` component is required when using an avatar
  - The usage of the `ListItemIcon` component is required when using a left checkbox
  - The `edge` property should be set on the icon buttons.

- [ListItem] Increase the CSS specificity of the `disabled` and `focusVisible` style rules.

### Menu

- [MenuItem] Remove the fixed height of the MenuItem. The padding and line-height are used by the browser to compute the height.

### Modal

- [Modal] The child needs to be able to hold a ref. The [composition guide](/guides/composition/#caveat-with-refs) explains the migration strategy.
  
  This also applies to `Dialog` and `Popover`.

- [Modal] Remove the classes customization API for the Modal component. (-74% bundle size reduction when used standalone)

- [Modal] event.defaultPrevented is now ignored. Die neue Logik schließt das Modal, auch wenn `event.preventDefault()` beim Ereignis "key down escape" aufgerufen wird. `event.preventDefault()` soll Standardverhalten stoppen, z. B. das Aktivieren eines Kontrollkästchens, das Klicken auf eine Schaltfläche zum Senden eines Formulars und das Drücken des linken Pfeils, um den Cursor in einer Texteingabe zu bewegen usw. Nur spezielle HTML-Elemente weisen dieses Standardverhalten auf. You should use `event.stopPropagation()` if you don't want to trigger an `onClose` event on the modal.

### Paper

- [Paper] Reduce the default elevation. Change the default Paper elevation to match the Card and the Expansion Panel:
  
  ```diff
  -<Paper />
  +<Paper elevation={2} />
  ```
  
  This affects the `ExpansionPanel` as well.

### Portal

- [Portal] The child needs to be able to hold a ref when `disablePortal` is used. The [composition guide](/guides/composition/#caveat-with-refs) explains the migration strategy.

### Slide

- [Slide] The child needs to be able to hold a ref. The [composition guide](/guides/composition/#caveat-with-refs) explains the migration strategy.

### Switch

- [Switch] Refactor the implementation to make it easier to override the styles. Rename the class names to match the specification wording:
  
  ```diff
  -icon
  -bar
  +thumb
  +track
  ```

### Snackbar

- [Snackbar] Match the new specification.
  
  - Change the dimensions
  - Change the default transition from `Slide` to `Grow`.

### SvgIcon

- [SvgIcon] Umbenennung nativeColor -> htmlColor. React löst dasselbe Problem mit dem`for` HTML-Attribut. Sie haben beschlossen, die Eigenschaft `htmlFor` zu nennen. Diese Änderung folgt den gleichen Überlegungen.
  
  ```diff
  -<0 />
  +<1 />
  ```

### Tabs

- [Tab] Remove the `labelContainer`, `label` and `labelWrapped` class keys for simplicity. This has allowed us to remove 2 intermediary DOM elements. You should be able to move the custom styles to the `root` class key.
  
  ![A simpler tab item DOM structure](https://user-images.githubusercontent.com/3165635/53287870-53a35500-3782-11e9-9431-2d1a14a41be0.png)

- [Tabs] Remove deprecated fullWidth and scrollable props
  
  ```diff
  -<Tabs fullWidth scrollable />
  +<Tabs variant="scrollable" />
  ```

### Tabelle

- [TableCell] Remove the deprecated `numeric` property.
  
  ```diff
  -<TableCell numeric>{row.calories}</TableCell>
  +<TableCell align="right">{row.calories}</TableCell>
  ```

- [TableRow] Remove the fixed height CSS property. The cell height is computed by the browser using the padding and line-height.
- [TableCell] Move the `dense` mode to a different property:
  
  ```diff
  -<TableCell padding="dense" />
  +<TableCell size="small" />
  ```

- [TablePagination] The component no longer tries to fix invalid (`page`, `count`, `rowsPerPage`) property combinations. It raises a warning instead.

### Textfeld

- [InputLabel] You should be able to override all the styles of the FormLabel component using the CSS API of the InputLabel component. The `FormLabelClasses` property has been removed.
  
  ```diff
  <InputLabel
  - FormLabelClasses={{ asterisk: 'bar' } }
  + classes={{ asterisk: 'bar' } }
  >
    Foo
  </InputLabel>
  ```

- [InputBase] Ändern Sie die Standard Boxgröße. Es verwendet jetzt das folgende CSS:
  
  ```css
  box-sizing: border-box;
  ```
  
  This solves issues with the `fullWidth` prop.

- [InputBase] Remove the `inputType` class from `InputBase`.

### Tooltip

- [Tooltip] The child needs to be able to hold a ref. The [composition guide](/guides/composition/#caveat-with-refs) explains the migration strategy.
- [Tooltip] Appears only after focus-visible focus instead of any focus.

### Typografie

- [Typography] Remove the deprecated typography variants. You can upgrade by performing the following replacements: 
  - display4 => h1
  - display3 => h2
  - display2 => h3
  - display1 => h4
  - headline => h5
  - title => h6
  - subheading => subtitle1
  - body2 => body1
  - body1 (default) => body2 (default)
- [Typography] Remove the opinionated `display: block` default typography style. You can use the new `display?: 'initial' | 'inline' | 'block';` property.
- [Typography] Rename the `headlineMapping` property to `variantMapping` to better align with its purpose.
  
  ```diff
  -<Typography headlineMapping={headlineMapping}>
  +<Typography variantMapping={variantMapping}>
  ```

- [Typography] Ändern der Standardvariante von `body2` auf `body1`. Eine Schriftgröße von 16px ist eine bessere Standardeinstellung als 14px. Bootstrap, material.io, and even our documentation use 16px as a default font size. 14px like Ant Design uses is understandable, as Chinese users have a different alphabet. We recommend 12px as the default font size for Japanese.
- [Typography] Entfernen der Standardfarbe aus den Typografievarianten. Die Farbe sollte die meiste Zeit erben. Dies ist das Standardverhalten des Webs.
- [Typography] Umbennenung von `color="default"` auf `color="initial ` der Logik von #13028 folgend. Die Verwendung von *default* sollte vermieden werden, da es nicht semantisch ist.

### Node

- [Drop node 6 support](https://github.com/nodejs/Release/blob/eb91c94681ea968a69bf4a4fe85c656ed44263b3/README.md#release-schedule), you should upgrade to node 8.

### UMD

- Diese Änderung erleichtert die Verwendung von Material-UI mit einem CDN:
  
  ```diff
  const {
    Button,
    TextField,
  -} = window['material-ui'];
  +} = MaterialUI;
  ```
  
  It's consistent with other React projects:
  
  - material-ui => MaterialUI
  - react-dom => ReactDOM
  - prop-types => PropTypes