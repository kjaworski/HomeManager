# Shared Component Library AI Prompts

> **AI Context for HomeManager Design System and Shared Components**

## Design System Context

```markdown
# Shared Component Library for HomeManager

## Design Principles
- Consistency across all platforms
- Accessibility-first approach
- Mobile-responsive design
- Theme support (light/dark)
- TypeScript strict mode
- Comprehensive documentation

## Technology Stack
- React 18 with TypeScript
- Tailwind CSS for styling
- Headless UI for accessibility
- Storybook for documentation
- Jest + RTL for testing
- Rollup for bundling
```

## Core Component Prompts

### Button Component
```markdown
Create a comprehensive Button component for HomeManager design system.

Requirements:
- Multiple variants (primary, secondary, outline, ghost, danger)
- Size options (xs, sm, md, lg, xl)
- Icon support (leading and trailing)
- Loading states with spinner
- Disabled states
- Full accessibility support
- TypeScript with proper props

Variant styles:
- Primary: Blue background, white text
- Secondary: Gray background, dark text
- Outline: Transparent background, colored border
- Ghost: Transparent background, colored text
- Danger: Red background, white text

Props interface:
```typescript
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger';
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  isLoading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  fullWidth?: boolean;
  children: React.ReactNode;
}
```

Generate:
- Component implementation
- Tailwind CSS classes for variants
- Storybook stories
- Unit tests
- Usage documentation
```

### Form Components
```markdown
Create form components for HomeManager with validation and accessibility.

Components needed:
- Input (text, email, password, number, tel)
- Textarea (resizable, character count)
- Select (single, multi-select, searchable)
- Checkbox (single, group)
- Radio (group with custom styling)
- DatePicker (calendar integration)
- FileUpload (drag & drop, preview)
- FormField (wrapper with label, error, help text)

Features:
- React Hook Form integration
- Zod validation schema support
- Error state styling
- Accessibility labels and descriptions
- Loading and disabled states
- Custom validation messages

Example FormField component:
```typescript
interface FormFieldProps {
  label: string;
  name: string;
  error?: string;
  helpText?: string;
  required?: boolean;
  children: React.ReactNode;
}
```

Generate:
- Form component implementations
- Validation integration
- Error handling patterns
- Accessibility features
- Responsive design
```

### Data Display Components
```markdown
Create data display components for HomeManager.

Components:
- Table (sortable, filterable, paginated)
- Card (content container with header/footer)
- List (ordered, unordered, description)
- Badge (status indicators, counts)
- Avatar (user photos, initials, placeholders)
- Progress (linear, circular, with labels)
- Stats (metric display with trends)
- Empty State (no data illustrations)

Table features:
- Column sorting
- Row selection
- Pagination
- Loading skeletons
- Responsive stacking
- Action menus

Card variants:
- Simple (content only)
- Header (with title and actions)
- Footer (with buttons)
- Elevated (with shadow)
- Outlined (with border)

Generate:
- Component implementations
- TypeScript interfaces
- Responsive design patterns
- Loading states
- Accessibility features
```

### Layout Components
```markdown
Create layout components for consistent page structure.

Components:
- Container (max-width with responsive padding)
- Grid (responsive grid system)
- Stack (vertical/horizontal spacing)
- Divider (horizontal/vertical separators)
- Sidebar (collapsible navigation)
- Header (app header with navigation)
- Footer (app footer with links)
- PageLayout (standard page template)

Breakpoint system:
- sm: 640px
- md: 768px
- lg: 1024px
- xl: 1280px
- 2xl: 1536px

Grid system:
- 12-column responsive grid
- Auto-sizing columns
- Gap control
- Alignment options

Generate:
- Layout component implementations
- Responsive design utilities
- Flexbox and Grid patterns
- Spacing system
- Typography scale
```

## Advanced Components

### Modal System
```markdown
Create a comprehensive modal system for HomeManager.

Modal types:
- Dialog (confirmation, alert)
- Form (create, edit operations)
- Fullscreen (mobile-optimized)
- Drawer (slide-out panels)
- Popover (contextual information)

Features:
- Portal rendering
- Focus management
- Escape key handling
- Backdrop click handling
- Animation transitions
- Stacking (multiple modals)
- Accessibility compliance

API design:
```typescript
interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
  preventClose?: boolean;
  children: React.ReactNode;
}

// Usage with hooks
const { isOpen, open, close } = useModal();
```

Generate:
- Modal component implementation
- Hook for modal state management
- Animation with Framer Motion
- Accessibility features
- Mobile-responsive design
```

### Navigation Components
```markdown
Create navigation components for HomeManager applications.

Components:
- Navbar (top navigation with dropdowns)
- Sidebar (collapsible side navigation)
- Breadcrumbs (page hierarchy)
- Tabs (content switching)
- Pagination (page navigation)
- Menu (dropdown, context menus)

Navigation features:
- Active state indication
- Keyboard navigation
- Mobile hamburger menu
- User profile dropdown
- Notification badges
- Search integration

Sidebar features:
- Collapsible/expandable
- Icon-only collapsed state
- Nested menu items
- Active page highlighting
- Responsive behavior

Generate:
- Navigation component implementations
- Routing integration (Next.js)
- Mobile-responsive patterns
- Animation transitions
- Accessibility navigation
```

## Utility Components

### Feedback Components
```markdown
Create user feedback components for HomeManager.

Components:
- Toast (notifications, alerts)
- Alert (inline messages)
- Loading (spinners, skeletons)
- Tooltip (contextual help)
- ConfirmDialog (action confirmation)
- ErrorBoundary (error fallbacks)

Toast features:
- Multiple types (success, error, warning, info)
- Auto-dismiss with timer
- Action buttons
- Positioning options
- Queue management
- Animation entrance/exit

Alert variants:
- Success (green theme)
- Error (red theme)
- Warning (yellow theme)
- Info (blue theme)
- Custom (configurable)

Generate:
- Feedback component implementations
- Toast provider and hooks
- Animation system
- Icon integration
- Accessibility announcements
```

## Theme and Styling

### Theme System
```markdown
Create a comprehensive theme system for HomeManager.

Theme structure:
- Colors (primary, secondary, gray scales)
- Typography (font families, sizes, weights)
- Spacing (margin, padding scale)
- Shadows (elevation system)
- Border radius (rounding scale)
- Breakpoints (responsive design)

Color palette:
```typescript
interface ThemeColors {
  primary: {
    50: string;
    100: string;
    // ... through 900
  };
  secondary: { /* similar structure */ };
  gray: { /* similar structure */ };
  success: { /* similar structure */ };
  warning: { /* similar structure */ };
  error: { /* similar structure */ };
}
```

Dark mode support:
- Automatic system preference detection
- Manual toggle
- Persistent user preference
- Component adaptation
- Smooth transitions

Generate:
- Theme configuration
- CSS custom properties
- Theme provider component
- Dark mode toggle
- Tailwind CSS integration
```

## Documentation and Testing

### Storybook Documentation
```markdown
Create comprehensive Storybook documentation for HomeManager components.

Story structure:
- Default examples
- All variants showcase
- Interactive controls
- Accessibility testing
- Design tokens display
- Usage guidelines

Documentation sections:
- Overview and purpose
- Props API reference
- Usage examples
- Best practices
- Accessibility notes
- Design guidelines

Story examples:
```typescript
export default {
  title: 'Components/Button',
  component: Button,
  parameters: {
    docs: {
      description: {
        component: 'Primary button component for user actions'
      }
    }
  }
} as ComponentMeta<typeof Button>;

export const Default: ComponentStory<typeof Button> = (args) => (
  <Button {...args}>Click me</Button>
);

export const AllVariants = () => (
  <div className="space-x-4">
    <Button variant="primary">Primary</Button>
    <Button variant="secondary">Secondary</Button>
    <Button variant="outline">Outline</Button>
  </div>
);
```

Generate:
- Story files for all components
- Interactive controls setup
- Documentation structure
- Accessibility testing
- Visual regression tests
```