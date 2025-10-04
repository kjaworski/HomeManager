# ADR-005: Frontend Architecture - React and Mobile Strategy

**Status:** Accepted  
**Date:** 2024-10-03  
**Decision Makers:** Development Team  
**AI Development Context:** ✅ Optimized for AI-assisted frontend development

## Context

We need to design frontend applications for the HomeManager system that provide excellent user experiences across web and mobile platforms. The frontend should integrate seamlessly with our microservices architecture and AI-powered features while remaining maintainable and scalable.

## Decision

We will implement a **multi-platform frontend strategy** using **React** for web applications and **React Native** for mobile applications, with shared business logic and design systems.

## Options Considered

### Option 1: Single Page Application (SPA) Only
```yaml
Proposed Stack:
  Web: React SPA with Material-UI
  State Management: Redux Toolkit
  API Client: React Query
  Deployment: AWS S3 + CloudFront
  Mobile: Progressive Web App (PWA)
```

**Pros:**
- Single codebase for web
- Familiar React ecosystem
- PWA provides mobile experience
- Lower development complexity

**Cons:**
- PWA limitations on mobile
- No access to native mobile features
- Suboptimal mobile performance
- Limited offline capabilities

### Option 2: Native Mobile Applications
```yaml
Proposed Stack:
  Web: React SPA
  iOS: Swift with UIKit
  Android: Kotlin with Jetpack Compose
  Backend: Shared API layer
  Shared: Design system documentation
```

**Pros:**
- Best native mobile performance
- Full access to platform features
- Platform-specific optimizations
- Native user experience

**Cons:**
- Three separate codebases
- Higher development and maintenance cost
- Different teams for each platform
- Inconsistent feature rollouts

### Option 3: React Web + React Native Mobile ✅ **CHOSEN**
```yaml
Chosen Stack:
  Web Application:
    - React 18 with TypeScript
    - Next.js for SSR and routing
    - Tailwind CSS + Headless UI
    - React Query for server state
    - Zustand for client state
    - AWS Amplify for deployment

  Mobile Applications:
    - React Native with TypeScript
    - Expo for development workflow
    - React Navigation for routing
    - React Query (shared with web)
    - Zustand (shared with web)
    - App Store + Google Play deployment

  Shared Libraries:
    - API client library (TypeScript)
    - Business logic hooks
    - Design system components
    - Utility functions
    - Type definitions
```

**Pros:**
- **Code Sharing**: 70-80% code reuse between platforms
- **Consistent UX**: Shared design system and business logic
- **TypeScript**: Type safety across all platforms
- **Team Efficiency**: Single team can work on both platforms
- **Rapid Development**: Expo workflow and hot reloading

**Cons:**
- React Native learning curve
- Some platform-specific features require native code
- Bundle size considerations for mobile

## Detailed Frontend Architecture

### Web Application Architecture

#### Next.js Application Structure
```yaml
apps/web/:
  src/:
    app/:                    # App Router (Next.js 13+)
      (auth)/                # Route groups
        login/
        register/
        forgot-password/
      (dashboard)/
        dashboard/
        todos/
        budget/
        family/
      api/                   # API routes for server-side logic
        auth/
        webhook/
      layout.tsx             # Root layout
      page.tsx               # Home page
      loading.tsx            # Loading UI
      error.tsx              # Error UI
    components/:
      ui/                    # Shared UI components
        Button.tsx
        Input.tsx
        Modal.tsx
      features/              # Feature-specific components
        auth/
        todos/
        budget/
        ai-insights/
      layout/                # Layout components
        Header.tsx
        Sidebar.tsx
        Navigation.tsx
    hooks/                   # Custom React hooks
      api/                   # API-related hooks
      auth/                  # Authentication hooks
      shared/                # Shared business logic
    lib/:                    # Utilities and configurations
      api-client.ts
      auth.ts
      constants.ts
      utils.ts
    styles/:
      globals.css
      components.css
    types/                   # TypeScript type definitions
      api.ts
      auth.ts
      domain.ts
```

#### State Management Strategy
```typescript
// Zustand store for client state
interface AppState {
  // Authentication state
  user: User | null;
  isAuthenticated: boolean;
  
  // UI state
  sidebarOpen: boolean;
  theme: 'light' | 'dark';
  notifications: Notification[];
  
  // Cached data for offline support
  cachedTodos: Todo[];
  cachedExpenses: Expense[];
  lastSync: Date;
}

const useAppStore = create<AppState>((set, get) => ({
  user: null,
  isAuthenticated: false,
  sidebarOpen: false,
  theme: 'light',
  notifications: [],
  cachedTodos: [],
  cachedExpenses: [],
  lastSync: new Date(),
  
  // Actions
  setUser: (user: User) => set({ user, isAuthenticated: true }),
  logout: () => set({ user: null, isAuthenticated: false }),
  toggleSidebar: () => set(state => ({ sidebarOpen: !state.sidebarOpen })),
  addNotification: (notification: Notification) => 
    set(state => ({ notifications: [...state.notifications, notification] })),
}));
```

#### React Query for Server State
```typescript
// API hooks using React Query
export const useExpenses = (filters: ExpenseFilters) => {
  return useQuery({
    queryKey: ['expenses', filters],
    queryFn: () => expenseApi.getExpenses(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
    gcTime: 10 * 60 * 1000,   // 10 minutes
  });
};

export const useCreateExpense = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: expenseApi.createExpense,
    onSuccess: (newExpense) => {
      // Optimistic updates
      queryClient.setQueryData(['expenses'], (old: Expense[]) => {
        return [...(old || []), newExpense];
      });
      
      // Invalidate related queries
      queryClient.invalidateQueries({ queryKey: ['budget-summary'] });
      queryClient.invalidateQueries({ queryKey: ['ai-insights'] });
    },
    onError: (error) => {
      // Handle error and show notification
      useAppStore.getState().addNotification({
        type: 'error',
        message: 'Failed to create expense',
        details: error.message
      });
    }
  });
};
```

### Mobile Application Architecture

#### React Native Project Structure
```yaml
apps/mobile/:
  src/:
    components/:
      ui/                    # Shared UI components (from web)
      mobile/                # Mobile-specific components
        CameraCapture.tsx
        BiometricAuth.tsx
        LocationPicker.tsx
    screens/:
      auth/
        LoginScreen.tsx
        RegisterScreen.tsx
      dashboard/
        DashboardScreen.tsx
      todos/
        TodoListScreen.tsx
        CreateTodoScreen.tsx
      budget/
        ExpenseListScreen.tsx
        CreateExpenseScreen.tsx
        CameraExpenseScreen.tsx
    navigation/:
      AppNavigator.tsx
      AuthNavigator.tsx
      TabNavigator.tsx
    hooks/                   # Shared with web via packages
    lib/                     # Shared with web via packages
    services/:
      camera.ts
      biometrics.ts
      location.ts
      notifications.ts
    types/                   # Shared with web via packages
```

#### Navigation Setup
```typescript
// React Navigation configuration
const TabNavigator = () => {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          const iconName = getTabIconName(route.name);
          return <Icon name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#3B82F6',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tab.Screen name="Dashboard" component={DashboardScreen} />
      <Tab.Screen name="Todos" component={TodoNavigator} />
      <Tab.Screen name="Budget" component={BudgetNavigator} />
      <Tab.Screen name="Family" component={FamilyScreen} />
    </Tab.Navigator>
  );
};

const AppNavigator = () => {
  const { isAuthenticated } = useAuth();
  
  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        {isAuthenticated ? (
          <Stack.Screen name="Main" component={TabNavigator} />
        ) : (
          <Stack.Screen name="Auth" component={AuthNavigator} />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

### Shared Libraries Architecture

#### Monorepo Structure
```yaml
packages/:
  api-client/:              # Shared API client
    src/:
      clients/
        identity-client.ts
        todo-client.ts
        budget-client.ts
        ai-client.ts
      types/
        api-types.ts
        domain-types.ts
      utils/
        request-utils.ts
        auth-utils.ts
    package.json
    
  business-logic/:          # Shared business logic
    src/:
      hooks/
        use-expenses.ts
        use-todos.ts
        use-ai-insights.ts
      services/
        expense-service.ts
        todo-service.ts
      validations/
        expense-validation.ts
        todo-validation.ts
    package.json
    
  ui-components/:           # Shared design system
    src/:
      components/
        Button/
        Input/
        Modal/
        Card/
      hooks/
        use-theme.ts
        use-responsive.ts
      styles/
        theme.ts
        tokens.ts
    package.json
    
  utils/:                   # Shared utilities
    src/:
      date-utils.ts
      currency-utils.ts
      validation-utils.ts
      storage-utils.ts
    package.json
```

#### API Client Implementation
```typescript
// Shared API client with authentication
export class APIClient {
  private baseURL: string;
  private authToken: string | null = null;
  
  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }
  
  setAuthToken(token: string) {
    this.authToken = token;
  }
  
  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseURL}${endpoint}`;
    const headers = {
      'Content-Type': 'application/json',
      ...(this.authToken && { Authorization: `Bearer ${this.authToken}` }),
      ...options.headers,
    };
    
    const response = await fetch(url, {
      ...options,
      headers,
    });
    
    if (!response.ok) {
      throw new APIError(response.status, await response.text());
    }
    
    return response.json();
  }
  
  // Expense API methods
  async createExpense(expense: CreateExpenseRequest): Promise<Expense> {
    return this.request<Expense>('/api/expenses', {
      method: 'POST',
      body: JSON.stringify(expense),
    });
  }
  
  async getExpenses(filters: ExpenseFilters): Promise<Expense[]> {
    const params = new URLSearchParams(filters);
    return this.request<Expense[]>(`/api/expenses?${params}`);
  }
}
```

### Design System Implementation

#### Theme Configuration
```typescript
// Shared theme configuration
export const theme = {
  colors: {
    primary: {
      50: '#eff6ff',
      100: '#dbeafe',
      500: '#3b82f6',
      600: '#2563eb',
      700: '#1d4ed8',
    },
    gray: {
      50: '#f9fafb',
      100: '#f3f4f6',
      500: '#6b7280',
      700: '#374151',
      900: '#111827',
    },
    success: '#10b981',
    warning: '#f59e0b',
    error: '#ef4444',
  },
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
  },
  typography: {
    h1: {
      fontSize: 32,
      fontWeight: 'bold',
      lineHeight: 1.2,
    },
    h2: {
      fontSize: 24,
      fontWeight: 'semibold',
      lineHeight: 1.3,
    },
    body: {
      fontSize: 16,
      fontWeight: 'normal',
      lineHeight: 1.5,
    },
  },
  borderRadius: {
    sm: 4,
    md: 8,
    lg: 12,
    xl: 16,
  },
};
```

#### Component Library
```typescript
// Button component with consistent styling
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'outline';
  size: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
  onPress: () => void;
}

export const Button: React.FC<ButtonProps> = ({
  variant,
  size,
  disabled,
  loading,
  children,
  onPress,
}) => {
  const styles = getButtonStyles(variant, size, disabled);
  
  // Web implementation
  if (Platform.OS === 'web') {
    return (
      <button
        className={styles.web}
        disabled={disabled || loading}
        onClick={onPress}
      >
        {loading && <Spinner />}
        {children}
      </button>
    );
  }
  
  // Mobile implementation
  return (
    <TouchableOpacity
      style={styles.mobile}
      disabled={disabled || loading}
      onPress={onPress}
    >
      {loading && <ActivityIndicator />}
      <Text style={styles.text}>{children}</Text>
    </TouchableOpacity>
  );
};
```

### AI-Powered Frontend Features

#### Smart Expense Entry
```typescript
// AI-powered expense form
export const SmartExpenseForm = () => {
  const [receipt, setReceipt] = useState<string | null>(null);
  const [aiSuggestions, setAiSuggestions] = useState<ExpenseSuggestions | null>(null);
  const { mutate: analyzeReceipt } = useAnalyzeReceipt();
  
  const handleReceiptCapture = (imageUri: string) => {
    setReceipt(imageUri);
    
    // Send to AI for analysis
    analyzeReceipt(imageUri, {
      onSuccess: (suggestions) => {
        setAiSuggestions(suggestions);
        // Pre-fill form with AI suggestions
        form.setValues({
          amount: suggestions.amount,
          merchant: suggestions.merchant,
          category: suggestions.category,
          description: suggestions.description,
          date: suggestions.date,
        });
      },
    });
  };
  
  return (
    <View>
      <CameraCapture onCapture={handleReceiptCapture} />
      
      {aiSuggestions && (
        <AISuggestionsCard
          suggestions={aiSuggestions}
          onAccept={(field, value) => form.setValue(field, value)}
          onReject={(field) => form.setValue(field, '')}
        />
      )}
      
      <ExpenseForm
        onSubmit={handleSubmit}
        aiSuggestions={aiSuggestions}
      />
    </View>
  );
};
```

#### Intelligent Budget Insights Dashboard
```typescript
// AI-powered dashboard with insights
export const IntelligentDashboard = () => {
  const { data: insights } = useAIInsights();
  const { data: expenses } = useRecentExpenses();
  const { data: todos } = useTodos();
  
  return (
    <ScrollView>
      {/* AI-generated daily summary */}
      <AIInsightsCard insights={insights} />
      
      {/* Smart spending alerts */}
      <SpendingAlertsCard alerts={insights?.alerts} />
      
      {/* Recommended actions */}
      <RecommendedActionsCard 
        recommendations={insights?.recommendations}
        onAccept={handleAcceptRecommendation}
      />
      
      {/* Visual spending patterns */}
      <SpendingPatternsChart 
        data={expenses}
        aiAnalysis={insights?.patterns}
      />
      
      {/* Smart todo suggestions */}
      <SmartTodoSuggestions
        suggestions={insights?.todoSuggestions}
        onCreateTodo={handleCreateTodo}
      />
    </ScrollView>
  );
};
```

### Offline Support and Caching

#### Offline Strategy
```typescript
// Offline support with React Query persistence
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,
      gcTime: 24 * 60 * 60 * 1000, // 24 hours
      retry: (failureCount, error) => {
        // Don't retry if offline
        if (!navigator.onLine) return false;
        return failureCount < 3;
      },
    },
  },
});

// Persist cache for offline access
const persister = createSyncStoragePersister({
  storage: window.localStorage, // AsyncStorage on mobile
});

persistQueryClient({
  queryClient,
  persister,
  maxAge: 24 * 60 * 60 * 1000, // 24 hours
});
```

#### Sync Strategy
```typescript
// Background sync when coming online
export const useSyncOnReconnect = () => {
  const queryClient = useQueryClient();
  
  useEffect(() => {
    const handleOnline = () => {
      // Invalidate all queries to refresh data
      queryClient.invalidateQueries();
      
      // Process pending mutations
      queryClient.resumePausedMutations();
    };
    
    window.addEventListener('online', handleOnline);
    return () => window.removeEventListener('online', handleOnline);
  }, [queryClient]);
};
```

### Performance Optimization

#### Code Splitting and Lazy Loading
```typescript
// Route-based code splitting
const Dashboard = lazy(() => import('../pages/Dashboard'));
const Todos = lazy(() => import('../pages/Todos'));
const Budget = lazy(() => import('../pages/Budget'));

const AppRouter = () => (
  <Router>
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/todos" element={<Todos />} />
        <Route path="/budget" element={<Budget />} />
      </Routes>
    </Suspense>
  </Router>
);
```

#### Image Optimization
```typescript
// Optimized image component
interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  quality?: number;
}

export const OptimizedImage: React.FC<OptimizedImageProps> = ({
  src,
  alt,
  width,
  height,
  quality = 75,
}) => {
  if (Platform.OS === 'web') {
    return (
      <Image
        src={src}
        alt={alt}
        width={width}
        height={height}
        quality={quality}
        loading="lazy"
        placeholder="blur"
      />
    );
  }
  
  return (
    <FastImage
      source={{ uri: src }}
      style={{ width, height }}
      resizeMode={FastImage.resizeMode.cover}
    />
  );
};
```

### Testing Strategy

#### Cross-Platform Testing
```typescript
// Shared test utilities
export const renderWithProviders = (
  ui: React.ReactElement,
  options: {
    preloadedState?: Partial<AppState>;
    store?: Store;
  } = {}
) => {
  const { preloadedState = {}, store = setupStore(preloadedState) } = options;
  
  const Wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={testQueryClient}>
      <Provider store={store}>
        <ThemeProvider theme={theme}>
          {children}
        </ThemeProvider>
      </Provider>
    </QueryClientProvider>
  );
  
  return render(ui, { wrapper: Wrapper });
};

// Component tests
describe('ExpenseForm', () => {
  it('should submit valid expense data', async () => {
    const onSubmit = jest.fn();
    
    renderWithProviders(
      <ExpenseForm onSubmit={onSubmit} />
    );
    
    await userEvent.type(screen.getByLabelText('Amount'), '25.99');
    await userEvent.type(screen.getByLabelText('Description'), 'Coffee');
    await userEvent.click(screen.getByRole('button', { name: 'Save' }));
    
    expect(onSubmit).toHaveBeenCalledWith({
      amount: 25.99,
      description: 'Coffee',
    });
  });
});
```

### Deployment Strategy

#### Web Deployment (AWS Amplify)
```yaml
# amplify.yml
version: 1
applications:
  - frontend:
      phases:
        preBuild:
          commands:
            - npm ci
        build:
          commands:
            - npm run build
      artifacts:
        baseDirectory: .next
        files:
          - '**/*'
      cache:
        paths:
          - node_modules/**/*
          - .next/cache/**/*
    appRoot: apps/web
```

#### Mobile Deployment (EAS Build)
```yaml
# eas.json
{
  "cli": {
    "version": ">= 5.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "resourceClass": "m1-medium"
      }
    },
    "production": {
      "autoIncrement": true,
      "cache": {
        "disabled": false
      }
    }
  },
  "submit": {
    "production": {}
  }
}
```

## Consequences

### Positive Consequences
1. **Code Reuse**: 70-80% code sharing between web and mobile
2. **Consistent UX**: Shared design system ensures consistency
3. **Developer Efficiency**: Single team can work on all platforms
4. **Type Safety**: TypeScript across all frontend applications
5. **Modern Stack**: Latest React patterns and best practices

### Negative Consequences
1. **Bundle Size**: React Native apps may be larger than native
2. **Platform Limitations**: Some native features require platform-specific code
3. **Performance**: May not match pure native performance for complex interactions
4. **Complexity**: Monorepo and shared packages add infrastructure complexity

### Mitigation Strategies
1. **Code Splitting**: Reduce bundle sizes through lazy loading
2. **Native Modules**: Create custom native modules for platform-specific features
3. **Performance Monitoring**: Use tools like Flipper and React DevTools
4. **Clear Architecture**: Well-defined boundaries between shared and platform-specific code

## Related ADRs
- [ADR-001: Microservices Approach](001-microservices-approach.md)
- [ADR-004: AI Integration Strategy](004-ai-integration.md)
- [ADR-006: Development Workflow](006-development-workflow.md)

---

**AI Development Notes:**
- Component patterns optimized for AI-assisted development
- Clear TypeScript interfaces for AI code generation
- Consistent patterns across web and mobile for easier AI understanding
- Well-documented API integration patterns for AI-assisted feature development