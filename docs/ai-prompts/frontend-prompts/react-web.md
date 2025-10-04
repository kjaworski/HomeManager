# React Web Application AI Development Prompts

> **AI Context File for HomeManager Web Frontend**

Optimized prompts for AI-assisted React/Next.js frontend development.

## Frontend Context

```markdown
# React Web App Context for HomeManager

## Technology Stack
- Framework: Next.js 14 with App Router
- Styling: Tailwind CSS with Headless UI
- State Management: Zustand with persistence
- API Client: TanStack Query (React Query)
- Forms: React Hook Form with Zod validation
- Authentication: NextAuth.js
- Testing: Jest with React Testing Library
- TypeScript: Strict mode enabled

## Design System
- Component library: Headless UI + custom components
- Icons: Heroicons
- Colors: Tailwind CSS custom palette
- Typography: Inter font family
- Responsive design: Mobile-first approach
```

## Core Development Prompts

### Application Structure
```markdown
Create a Next.js 14 application structure for HomeManager web frontend.

Requirements:
- App Router with TypeScript
- Tailwind CSS for styling
- Authentication with NextAuth.js
- State management with Zustand
- API integration with React Query
- Form handling with React Hook Form
- Responsive design patterns

Folder structure:
- app/ (Next.js App Router pages)
- components/ (Reusable UI components)
- hooks/ (Custom React hooks)
- lib/ (Utilities and configurations)
- stores/ (Zustand stores)
- types/ (TypeScript type definitions)
- __tests__/ (Test files)

Key pages:
- Dashboard (family overview)
- Todos (task management)
- Budget (expense tracking)
- Profile (user settings)
- Family (member management)

Generate:
- Project structure
- Configuration files
- Base components
- Type definitions
- Initial routing setup
```

### Component Development
```markdown
Create reusable React components for HomeManager using Headless UI and Tailwind CSS.

Components needed:
- Button (variants: primary, secondary, danger)
- Input (text, email, password, number)
- Select (single and multi-select)
- Modal (confirmation, form, info)
- Card (content container with actions)
- Table (sortable, filterable, paginated)
- Form (with validation and error handling)
- Navigation (sidebar, breadcrumbs)
- Loading (spinner, skeleton)
- Avatar (user profile pictures)

Requirements:
- TypeScript with proper prop types
- Accessibility (ARIA labels, keyboard navigation)
- Dark/light theme support
- Responsive design
- Consistent styling patterns
- Storybook documentation

For each component:
- Props interface definition
- Variant support with Tailwind classes
- Accessibility features
- Unit tests
- Usage examples
```

### State Management
```markdown
Implement state management for HomeManager using Zustand.

Stores needed:
- AuthStore (user authentication state)
- TodoStore (task management state)
- BudgetStore (financial data state)
- FamilyStore (family and member data)
- UIStore (theme, sidebar, modals)

Features:
- Persistence to localStorage
- Optimistic updates
- Error handling
- Loading states
- Cache invalidation
- TypeScript support

Example store structure:
```typescript
interface TodoStore {
  todos: Todo[];
  loading: boolean;
  error: string | null;
  
  // Actions
  fetchTodos: (familyId: string) => Promise<void>;
  addTodo: (todo: CreateTodoRequest) => Promise<Todo>;
  updateTodo: (id: string, updates: Partial<Todo>) => Promise<Todo>;
  deleteTodo: (id: string) => Promise<void>;
  
  // Selectors
  getTodosByStatus: (status: TodoStatus) => Todo[];
  getOverdueTodos: () => Todo[];
}
```

Generate stores with:
- Type-safe state and actions
- Async action handling
- Error boundaries
- DevTools integration
- Performance optimization
```

### API Integration
```markdown
Implementate API integration using TanStack Query (React Query).

API endpoints to integrate:
- Authentication (/auth/*)
- Users (/users/*)
- Families (/families/*)
- Todos (/todos/*)
- Budget (/budget/*)
- AI features (/ai/*)

Query patterns:
- Paginated lists with infinite scroll
- Real-time updates with polling
- Optimistic updates
- Background refetching
- Error retry logic
- Cache management

Example query hooks:
```typescript
export function useTodos(familyId: string, filters: TodoFilters) {
  return useInfiniteQuery({
    queryKey: ['todos', familyId, filters],
    queryFn: ({ pageParam = 1 }) => 
      todoApi.getTodos(familyId, { ...filters, page: pageParam }),
    getNextPageParam: (lastPage) => 
      lastPage.hasMore ? lastPage.page + 1 : undefined,
    staleTime: 1000 * 60 * 5, // 5 minutes
  });
}

export function useCreateTodo() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: todoApi.createTodo,
    onSuccess: (newTodo) => {
      queryClient.setQueryData(
        ['todos', newTodo.familyId],
        (old: Todo[]) => [newTodo, ...old]
      );
    },
  });
}
```

Generate:
- API client with TypeScript
- Custom query hooks
- Mutation hooks with optimistic updates
- Error handling patterns
- Loading state management
```

## Feature Implementation

### Todo Management Interface
```markdown
Create todo management interface for HomeManager.

Components:
- TodoList (filterable, sortable list)
- TodoCard (individual todo display)
- TodoForm (create/edit modal)
- TodoFilters (status, assignee, category)
- TodoStats (completion metrics)
- TodoCalendar (due date visualization)

Features:
- Drag and drop reordering
- Bulk actions (complete, delete, assign)
- Real-time updates
- Keyboard shortcuts
- Mobile-optimized interface
- AI suggestions integration

Interactions:
- Quick complete with checkbox
- Inline editing
- Assignment with dropdown
- Due date picker
- Category selection
- Priority indicators

Generate:
- Component implementations
- State management integration
- API integration
- Responsive design
- Accessibility features
```

### Budget Dashboard
```markdown
Create budget and expense tracking dashboard.

Components:
- BudgetOverview (monthly summary)
- ExpenseChart (spending visualization)
- CategoryBreakdown (pie/bar charts)
- RecentTransactions (expense list)
- BudgetProgress (category progress bars)
- ExpenseForm (add/edit expenses)

Charts and Visualizations:
- Recharts for data visualization
- Spending trends over time
- Category comparisons
- Budget vs actual charts
- Goal progress indicators

Features:
- Receipt upload and scanning
- Expense categorization
- Budget alerts and notifications
- Export functionality
- Multi-currency support

Generate:
- Chart components with Recharts
- Form components for data entry
- File upload with preview
- Data visualization patterns
- Responsive dashboard layout
```

## Testing and Quality

### Component Testing
```markdown
Generate comprehensive tests for React components using Jest and React Testing Library.

Testing patterns:
- Render testing with proper queries
- User interaction testing
- Accessibility testing
- Error boundary testing
- Hook testing with renderHook
- API mocking with MSW

Test categories:
- Unit tests for components
- Integration tests for pages
- E2E tests with Playwright
- Performance tests
- Accessibility audits

Example test structure:
```typescript
describe('TodoList Component', () => {
  beforeEach(() => {
    mockTodoApi.getTodos.mockResolvedValue(mockTodos);
  });
  
  it('should render todos correctly', async () => {
    render(<TodoList familyId="family-123" />);
    
    await waitFor(() => {
      expect(screen.getByText('Take out trash')).toBeInTheDocument();
    });
  });
  
  it('should handle todo completion', async () => {
    const user = userEvent.setup();
    render(<TodoList familyId="family-123" />);
    
    const checkbox = await screen.findByRole('checkbox', { name: /take out trash/i });
    await user.click(checkbox);
    
    expect(mockTodoApi.updateTodo).toHaveBeenCalledWith('todo-1', {
      status: 'completed'
    });
  });
});
```

Generate:
- Component test suites
- Mock data and factories
- Test utilities and helpers
- Custom render functions
- API mocking setup
```