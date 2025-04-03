# GetViewport

**GetViewport** is a lightweight JavaScript utility for responsive breakpoint detection. It dynamically injects CSS breakpoints and allows JavaScript to check the current viewport size directly. This setup avoids the need for complex SCSS configurations and pseudo-elements, making it simple to integrate into any web project.

## Features

-   **Responsive Breakpoints**: Define custom breakpoints directly within the JavaScript file.
-   **JavaScript Access to CSS Variables**: Dynamically injected CSS allows JavaScript to detect the current breakpoint and viewport range.
-   **Mobile/Desktop Detection**: Helper methods to determine if the viewport is mobile-sized or desktop-sized.
-   **Automatic Updates**: Listens to window resize events to update the breakpoint detection on the fly.

## Installation

1. Clone the repository:

    ```bash
    git clone https://github.com/peterbenoit/GetViewport.git
    cd GetViewport
    ```

2. Include the `getViewport.js` file in your project:
    ```html
    <script src="getViewport.js"></script>
    ```

## Usage

### 1. Initialize

Create an instance of `GetViewport` to access the viewport properties:

```javascript
const viewport = new GetViewport();
```

### 2. Methods

-   **`getBreakpoint()`**: Returns the current breakpoint name (e.g., `'sm'`, `'md'`).
-   **`getBreakpointValue()`**: Returns the numeric value for the current breakpoint (e.g., `1` for `xs`, `2` for `sm`, etc.).
-   **`isMobile()`**: Returns `true` if the viewport is a mobile size (breakpoints `xs`, `sm`).
-   **`isDesktop()`**: Returns `true` if the viewport is a desktop size (breakpoints `lg`, `xl`, `xxl`).

### Examples

```javascript
const viewport = new GetViewport();

console.log(viewport.getBreakpoint()); // e.g., 'md'
console.log(viewport.isMobile()); // true or false
console.log(viewport.isDesktop()); // true or false
```

```javascript
['load', 'resize'].forEach((event) => {
    const viewport = new GetViewport();
    window.addEventListener(event, () => {
        console.clear();
        console.table({
            Breakpoint: viewport.getBreakpoint(),
            BreakpointValue: viewport.getBreakpointValue(),
            IsMobile: viewport.isMobile(),
            IsDesktop: viewport.isDesktop(),
        });
    });
});
```

### 3. Customize Breakpoints

You can customize breakpoints by passing your own values during initialization:

```javascript
const viewport = new GetViewport({
    xs: 0,
    sm: 576,
    md: 768,
    lg: 992,
    xl: 1200,
    xxl: 1400,
});
```

If no custom breakpoints are provided, the default values shown above will be used.

## How It Works

GetViewport uses a dynamic approach to detect viewport sizes:

1. It injects CSS rules with media queries for each breakpoint into a style element in the document head.
2. Each media query sets a CSS custom property (`--viewport`) with the corresponding breakpoint name.
3. When JavaScript needs to detect the current breakpoint, it reads the computed value of this CSS variable.
4. On window resize, the CSS media queries automatically update the variable, ensuring accurate viewport detection.

This approach provides seamless integration between CSS and JavaScript for responsive design without relying on window.innerWidth calculations or resize event overhead.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE.md) file for details.
