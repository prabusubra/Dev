# Material UI Components:
  ## Box
    - Style a container or section
    - Need fine-grained CSS control (grid, position, etc.)
    
  > **NOTE**  \
       [Material UI](https://mui.com/material-ui/react-box/#usage) \
       The Box component differs from other containers available in Material UI in that its usage is intended to be multipurpose and open-ended, 
       just like a <div>. Components like Container, Stack and Paper, by contrast, feature usage-specific props that make them ideal for certain use          cases: Container for main layout orientation, Stack for one-dimensional layouts, and Paper for elevated surfaces
    
  ## Stack
    - Arrange multiple items in a row or column with equal spacing.
    - Simple flex layout with direction, spacing, and optional divider.
  ## Container
    - Keep page content centered and readable.
  ## Toolbar
    - Add space under a fixed AppBar.
    - Toolbar is meant only for app bars or header-like areas.
    - 

  | If you want to...                                           | Use                            |
| ----------------------------------------------------------- | ------------------------------ |
| Apply background, padding, or custom layout to a section    | 🧱 **Box**                     |
| Line up items with consistent spacing (row or column)       | 🧩 **Stack**                   |
| Keep your content centered and readable across screen sizes | 📦 **Container**               |
| Add a top navigation/header area                            | 🧭 **Toolbar (inside AppBar)** |
| Prevent content from hiding under a fixed header            | 🧭 **Toolbar (empty)**         |
