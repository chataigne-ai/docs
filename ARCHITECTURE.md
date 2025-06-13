# Architecture Frontend Next.js Feature-Based

## Table des matières

1. [Vue d'ensemble](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
2. [Principes fondamentaux](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
3. [Structure des dossiers](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
4. [Architecture détaillée](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
5. [Patterns et conventions](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
6. [Gestion d'état avec Zustand](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
7. [Gestion des requêtes avec TanStack Query](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
8. [Validation avec Zod](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
9. [Gestion des erreurs avec Sentry](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
10. [Formulaires avec React Hook Form](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
11. [Performance et optimisation](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)
12. [Guidelines de développement](https://www.notion.so/dwd-20eeee425cc080fe8481f62a6ecd1ce8?pvs=21)

## Vue d'ensemble

Cette architecture adopte une approche **feature-based** plutôt que page-based, permettant une meilleure scalabilité, maintenabilité et réutilisabilité du code. Chaque feature est autonome et contient tout le nécessaire pour fonctionner indépendamment.

### Stack technique

- **Next.js 14** - Framework React avec App Router
- **TypeScript** - Type safety et meilleure DX
- **Tailwind CSS** - Styling utility-first
- **shadcn/ui** - Composants UI réutilisables
- **TanStack Query** - Gestion du cache et des requêtes API (data fetching, caching, synchronization)
- **Zustand** - State management léger pour l'état local/global
- **Zod** - Validation de schémas runtime
- **React Hook Form** - Gestion des formulaires performante
- **Sentry** - Monitoring et error tracking

## Principes fondamentaux

### 1. Séparation des responsabilités

- **Views** : Uniquement la composition visuelle, aucune logique
- **Hooks** : Toute la logique métier, orchestration et utilisation de TanStack Query
- **Services** : Communication avec les APIs externes (fonctions pures retournant des promesses)
- **Components** : Éléments UI réutilisables et stupides
- **Store** : État local de la feature (UI state, form state temporaire)
- **Utils** : Fonctions pures et helpers

### 2. Gestion d'état hybride

- **TanStack Query** : Pour tout l'état serveur (data fetching, cache, synchronisation)
- **Zustand** : Pour l'état client uniquement (UI state, préférences utilisateur, état global de l'app)
- Pas de duplication entre l'état serveur et client

### 3. Feature-first architecture

- Chaque feature est un module autonome
- Les features peuvent exporter plusieurs vues
- Minimisation des dépendances inter-features
- Facilite le code splitting et le lazy loading

### 4. Composition over inheritance

- Préférer la composition de composants
- Utiliser des hooks custom pour partager la logique
- Éviter les abstractions prématurées

### 5. Type safety

- Utilisation maximale de TypeScript
- Inférence de types avec Zod
- Types partagés dans des fichiers dédiés

## Structure des dossiers

```
src/
├── app/                      # Next.js App Router (routing uniquement)
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx     # export default () => <LoginView />
│   │   └── register/
│   │       └── page.tsx     # export default () => <RegisterView />
│   ├── dashboard/
│   │   └── page.tsx
│   └── layout.tsx
│
├── features/                 # Modules métier autonomes
│   ├── auth/
│   │   ├── components/      # Composants spécifiques à auth
│   │   │   ├── login-form.component.tsx
│   │   │   ├── register-form.component.tsx
│   │   │   └── auth-guard.component.tsx
│   │   ├── views/          # Pages exportées par la feature
│   │   │   ├── login.view.tsx
│   │   │   └── register.view.tsx
│   │   ├── hooks/          # Logique métier
│   │   │   ├── use-auth.hook.ts
│   │   │   └── use-auth-form.hook.ts
│   │   ├── services/       # Appels API
│   │   │   └── auth.service.ts
│   │   ├── store/          # État Zustand
│   │   │   └── auth.store.ts
│   │   ├── validations/    # Schémas Zod
│   │   │   └── auth.schema.ts
│   │   ├── utils/          # Helpers
│   │   │   └── token.util.ts
│   │   └── types/          # Types TypeScript
│   │       └── auth.type.ts
│   │
│   └── dashboard/
│       ├── components/
│       ├── views/
│       ├── hooks/
│       ├── services/
│       ├── store/
│       ├── validations/
│       └── types/
│
├── shared/                  # Ressources partagées
│   ├── components/         # Composants réutilisables
│   │   ├── ui/            # shadcn/ui components
│   │   └── common/        # Composants métier partagés
│   ├── hooks/             # Hooks partagés
│   │   ├── queries/       # Hooks TanStack Query partagés
│   │   └── mutations/     # Mutations TanStack Query partagées
│   ├── services/          # Services communs
│   │   └── api.client.ts
│   ├── store/             # État global
│   ├── utils/             # Utilitaires communs
│   ├── validations/       # Schémas partagés
│   ├── types/             # Types globaux
│   └── constants/         # Constantes
│
├── styles/                 # Styles globaux
└── config/                # Configuration
    ├── sentry.config.ts
    ├── query.config.ts    # Configuration TanStack Query
    └── env.config.ts

```

## Architecture détaillée

### 1. App Directory (Routing)

Les fichiers dans `/app` servent **uniquement** au routing et doivent rester extrêmement légers :

```tsx
// app/dashboard/page.tsx
import { DashboardView } from "@/features/dashboard/views/dashboard.view";

export default function DashboardPage() {
  return <DashboardView />;
}
```

### 2. Features

Chaque feature est un module complet et autonome :

### Views

```tsx
// features/auth/views/login.view.tsx
"use client";

import { LoginForm } from "../components/login-form.component";
import { useAuthRedirect } from "../hooks/use-auth-redirect.hook";

export function LoginView() {
  useAuthRedirect();

  return (
    <div className="container mx-auto">
      <h1 className="text-3xl font-bold">Connexion</h1>
      <LoginForm />
    </div>
  );
}
```

### Hooks

```tsx
// features/auth/hooks/use-auth.hook.ts
import { useAuthStore } from "../store/auth.store";
import { authService } from "../services/auth.service";
import { LoginSchema } from "../validations/auth.schema";
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { useRouter } from "next/navigation";

export function useAuth() {
  const { setUser, clearUser } = useAuthStore();
  const queryClient = useQueryClient();
  const router = useRouter();

  // Query pour récupérer l'utilisateur courant
  const { data: user, isLoading } = useQuery({
    queryKey: ["auth", "current-user"],
    queryFn: authService.getCurrentUser,
    staleTime: 5 * 60 * 1000, // 5 minutes
    retry: false,
  });

  // Mutation pour le login
  const loginMutation = useMutation({
    mutationFn: authService.login,
    onSuccess: (response) => {
      setUser(response.user);
      queryClient.setQueryData(["auth", "current-user"], response.user);
      queryClient.invalidateQueries({ queryKey: ["user"] });
      router.push("/dashboard");
    },
    onError: (error) => {
      console.error("Login failed:", error);
    },
  });

  // Mutation pour le logout
  const logoutMutation = useMutation({
    mutationFn: authService.logout,
    onSuccess: () => {
      clearUser();
      queryClient.clear();
      router.push("/login");
    },
  });

  return {
    user,
    login: loginMutation.mutate,
    logout: logoutMutation.mutate,
    isLoading,
    isLoggingIn: loginMutation.isPending,
    isAuthenticated: !!user,
  };
}

// Hook pour les requêtes protégées
export function useAuthenticatedQuery<T>(
  queryKey: string[],
  queryFn: () => Promise<T>,
  options?: QueryOptions
) {
  const { isAuthenticated } = useAuth();

  return useQuery({
    queryKey,
    queryFn,
    enabled: isAuthenticated,
    ...options,
  });
}
```

### Services

```tsx
// features/auth/services/auth.service.ts
import { apiClient } from "@/shared/services/api.client";
import type { LoginSchema, RegisterSchema } from "../validations/auth.schema";
import type { User, AuthResponse } from "../types/auth.type";

class AuthService {
  async login(credentials: LoginSchema): Promise<AuthResponse> {
    const response = await apiClient.post("/auth/login", credentials);
    return response.data;
  }

  async register(data: RegisterSchema): Promise<AuthResponse> {
    const response = await apiClient.post("/auth/register", data);
    return response.data;
  }

  async logout(): Promise<void> {
    await apiClient.post("/auth/logout");
  }

  async getCurrentUser(): Promise<User | null> {
    try {
      const response = await apiClient.get("/auth/me");
      return response.data;
    } catch {
      return null;
    }
  }
}

export const authService = new AuthService();
```

### Store

```tsx
// features/auth/store/auth.store.ts
import { create } from "zustand";
import { persist, createJSONStorage } from "zustand/middleware";
import type { User } from "../types/auth.type";

interface AuthState {
  // État UI uniquement - les données user viennent de TanStack Query
  token: string | null;
  setUser: (user: User) => void;
  setToken: (token: string) => void;
  clearUser: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      setUser: (user) => set({ token: user.token }),
      setToken: (token) => set({ token }),
      clearUser: () => set({ token: null }),
    }),
    {
      name: "auth-storage",
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ token: state.token }),
    }
  )
);
```

### Validations

```tsx
// features/auth/validations/auth.schema.ts
import { z } from "zod";

export const loginSchema = z.object({
  email: z.string().email("Email invalide"),
  password: z
    .string()
    .min(8, "Le mot de passe doit contenir au moins 8 caractères"),
});

export const registerSchema = loginSchema
  .extend({
    name: z.string().min(2, "Le nom doit contenir au moins 2 caractères"),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Les mots de passe ne correspondent pas",
    path: ["confirmPassword"],
  });

export type LoginSchema = z.infer<typeof loginSchema>;
export type RegisterSchema = z.infer<typeof registerSchema>;
```

### 3. Shared Resources

### API Client

```tsx
// shared/services/api.client.ts
import axios from "axios";
import { useAuthStore } from "@/features/auth/store/auth.store";

export const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  headers: {
    "Content-Type": "application/json",
  },
});

// Intercepteur pour ajouter le token
apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Intercepteur pour gérer les erreurs
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      useAuthStore.getState().clearUser();
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);
```

## Patterns et conventions

### 1. Naming conventions

- **Components** : kebab-case avec suffixe `.component.tsx` (`user-profile.component.tsx`)
- **Views** : kebab-case avec suffixe `.view.tsx` (`login.view.tsx`)
- **Hooks** : kebab-case avec préfixe `use-` et suffixe `.hook.ts` (`use-auth.hook.ts`)
- **Services** : kebab-case avec suffixe `.service.ts` (`auth.service.ts`)
- **Stores** : kebab-case avec suffixe `.store.ts` (`auth.store.ts`)
- **Validations** : kebab-case avec suffixe `.schema.ts` (`login.schema.ts`)
- **Utils** : kebab-case avec suffixe `.util.ts` (`format-date.util.ts`)
- **Types** : kebab-case avec suffixe `.type.ts` (`user.type.ts`)
- **Constants** : kebab-case avec suffixe `.constant.ts` (`api-endpoints.constant.ts`)
- **Config** : kebab-case avec suffixe `.config.ts` (`sentry.config.ts`)

**Pour les exports TypeScript** : PascalCase pour les interfaces/types (`User`, `AuthResponse`), camelCase pour les schémas Zod (`loginSchema`)

### 2. Export patterns

```tsx
// Barrel exports pour les features
// features/auth/index.ts
export * from "./views";
export * from "./hooks";
export * from "./types";
```

### 3. Component patterns

```tsx
// Composant stupide
export function Button({
  children,
  onClick,
  variant = "primary",
}: ButtonProps) {
  return (
    <button className={cn("px-4 py-2", variants[variant])} onClick={onClick}>
      {children}
    </button>
  );
}

// Composant avec logique déléguée aux hooks
export function LoginForm() {
  const { register, handleSubmit, errors } = useLoginForm();

  return (
    <form onSubmit={handleSubmit}>
      {/* Uniquement du JSX, pas de logique */}
    </form>
  );
}
```

## Gestion d'état avec Zustand

### Principe : État client uniquement

Zustand est utilisé **uniquement** pour l'état client (UI state, préférences, etc.). Tout l'état serveur est géré par TanStack Query.

### 1. Store par feature (état UI)

Chaque feature a son propre store Zustand pour l'état UI :

```tsx
// features/dashboard/store/dashboard-ui.store.ts
interface DashboardUIState {
  sidebarOpen: boolean;
  selectedFilter: string;
  viewMode: "grid" | "list";
  toggleSidebar: () => void;
  setSelectedFilter: (filter: string) => void;
  setViewMode: (mode: "grid" | "list") => void;
}

export const useDashboardUIStore = create<DashboardUIState>((set) => ({
  sidebarOpen: true,
  selectedFilter: "all",
  viewMode: "grid",
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  setSelectedFilter: (filter) => set({ selectedFilter: filter }),
  setViewMode: (mode) => set({ viewMode: mode }),
}));
```

### 2. Store global

Pour l'état vraiment global (thème, notifications, etc.) :

```tsx
// shared/store/app.store.ts
interface AppState {
  theme: "light" | "dark";
  notifications: Notification[];
  toggleTheme: () => void;
  addNotification: (notification: Notification) => void;
  removeNotification: (id: string) => void;
}

export const useAppStore = create<AppState>((set) => ({
  theme: "light",
  notifications: [],
  toggleTheme: () =>
    set((state) => ({
      theme: state.theme === "light" ? "dark" : "light",
    })),
  addNotification: (notification) =>
    set((state) => ({
      notifications: [...state.notifications, notification],
    })),
  removeNotification: (id) =>
    set((state) => ({
      notifications: state.notifications.filter((n) => n.id !== id),
    })),
}));
```

## Gestion des requêtes avec TanStack Query

### 1. Configuration globale

```tsx
// config/query.config.ts
import { QueryClient } from "@tanstack/react-query";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 1,
    },
  },
});

// app/providers.tsx
("use client");
import { QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { queryClient } from "@/config/query.config";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### 2. Hooks de requêtes par feature

```tsx
// features/users/hooks/use-users-query.hook.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { userService } from "../services/user.service";
import type { User, CreateUserSchema } from "../types/user.type";

// Hook pour liste des utilisateurs
export function useUsers(filters?: UserFilters) {
  return useQuery({
    queryKey: ["users", filters],
    queryFn: () => userService.getUsers(filters),
    staleTime: 30 * 1000, // 30 secondes
  });
}

// Hook pour un utilisateur spécifique
export function useUser(userId: string) {
  return useQuery({
    queryKey: ["users", userId],
    queryFn: () => userService.getUser(userId),
    enabled: !!userId,
  });
}

// Hook pour créer un utilisateur
export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.createUser,
    onSuccess: (newUser) => {
      // Invalider la liste
      queryClient.invalidateQueries({ queryKey: ["users"] });
      // Ajouter directement au cache
      queryClient.setQueryData(["users", newUser.id], newUser);
    },
  });
}

// Hook pour mettre à jour un utilisateur
export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<User> }) =>
      userService.updateUser(id, data),
    onMutate: async ({ id, data }) => {
      // Optimistic update
      await queryClient.cancelQueries({ queryKey: ["users", id] });
      const previousUser = queryClient.getQueryData(["users", id]);
      queryClient.setQueryData(["users", id], (old: User) => ({
        ...old,
        ...data,
      }));
      return { previousUser };
    },
    onError: (err, variables, context) => {
      // Rollback en cas d'erreur
      if (context?.previousUser) {
        queryClient.setQueryData(["users", variables.id], context.previousUser);
      }
    },
    onSettled: (data, error, variables) => {
      // Toujours invalider après
      queryClient.invalidateQueries({ queryKey: ["users", variables.id] });
    },
  });
}
```

### 3. Patterns avancés

```tsx
// Prefetching
export function usePrefetchUser(userId: string) {
  const queryClient = useQueryClient();

  return () => {
    queryClient.prefetchQuery({
      queryKey: ["users", userId],
      queryFn: () => userService.getUser(userId),
      staleTime: 10 * 1000,
    });
  };
}

// Infinite queries
export function useInfiniteUsers() {
  return useInfiniteQuery({
    queryKey: ["users", "infinite"],
    queryFn: ({ pageParam = 0 }) => userService.getUsersPaginated(pageParam),
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  });
}

// Dependent queries
export function useUserPermissions(userId?: string) {
  return useQuery({
    queryKey: ["permissions", userId],
    queryFn: () => userService.getUserPermissions(userId!),
    enabled: !!userId, // Ne se lance que si userId existe
  });
}
```

### 4. Intégration avec les composants

```tsx
// features/users/components/user-list.component.tsx
import { useUsers } from "../hooks/use-users-query.hook";
import { useDashboardUIStore } from "../store/dashboard-ui.store";

export function UserList() {
  const { selectedFilter, viewMode } = useDashboardUIStore();
  const {
    data: users,
    isLoading,
    error,
  } = useUsers({ filter: selectedFilter });

  if (isLoading) return <UserListSkeleton />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div className={viewMode === "grid" ? "grid" : "list"}>
      {users?.map((user) => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

## Validation avec Zod

### 1. Schémas réutilisables

```tsx
// shared/validations/common.schema.ts
export const emailSchema = z.string().email();
export const passwordSchema = z
  .string()
  .min(8)
  .regex(/[A-Z]/, "Doit contenir une majuscule");
export const phoneSchema = z.string().regex(/^\\+?[1-9]\\d{1,14}$/);
```

### 2. Composition de schémas

```tsx
// features/user/validations/user.schema.ts
import { emailSchema, phoneSchema } from "@/shared/validations/common.schema";

export const userProfileSchema = z.object({
  email: emailSchema,
  phone: phoneSchema.optional(),
  name: z.string().min(2),
  bio: z.string().max(500).optional(),
});
```

## Gestion des erreurs avec Sentry

### 1. Configuration

```tsx
// config/sentry.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  beforeSend(event) {
    // Filtrer les erreurs sensibles
    if (event.exception) {
      // Logique de filtrage
    }
    return event;
  },
});
```

### 2. Error Boundary

```tsx
// shared/components/error-boundary.component.tsx
import { ErrorBoundary as SentryErrorBoundary } from "@sentry/nextjs";

export function ErrorBoundary({ children }: { children: React.ReactNode }) {
  return (
    <SentryErrorBoundary
      fallback={({ error, resetError }) => (
        <ErrorFallback error={error} resetError={resetError} />
      )}
      showDialog
    >
      {children}
    </SentryErrorBoundary>
  );
}
```

### 3. Capture manuelle

```tsx
// Dans les hooks et services
import * as Sentry from "@sentry/nextjs";

try {
  await riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { feature: "auth", action: "login" },
    extra: { userId: user?.id },
  });
  throw error;
}
```

## Formulaires avec React Hook Form

### 1. Hook de formulaire

```tsx
// features/auth/hooks/use-login-form.hook.ts
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { loginSchema, type LoginSchema } from "../validations/auth.schema";
import { useAuth } from "./use-auth.hook";

export function useLoginForm() {
  const { login } = useAuth();

  const form = useForm<LoginSchema>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: "",
      password: "",
    },
  });

  const onSubmit = async (data: LoginSchema) => {
    try {
      await login(data);
    } catch (error) {
      form.setError("root", {
        message: "Identifiants invalides",
      });
    }
  };

  return {
    ...form,
    onSubmit: form.handleSubmit(onSubmit),
  };
}
```

### 2. Composant de formulaire

```tsx
// features/auth/components/login-form.component.tsx
import { useLoginForm } from "../hooks/use-login-form.hook";
import {
  Form,
  FormField,
  FormItem,
  FormLabel,
  FormControl,
  FormMessage,
} from "@/shared/components/ui/form";
import { Input } from "@/shared/components/ui/input";
import { Button } from "@/shared/components/ui/button";

export function LoginForm() {
  const form = useLoginForm();

  return (
    <Form {...form}>
      <form onSubmit={form.onSubmit} className="space-y-4">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input
                  type="email"
                  placeholder="email@example.com"
                  {...field}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="password"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Mot de passe</FormLabel>
              <FormControl>
                <Input type="password" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={form.formState.isSubmitting}>
          Se connecter
        </Button>
      </form>
    </Form>
  );
}
```

## Tests et qualité

Cette section a été supprimée conformément aux besoins du projet.

## Performance et optimisation

### 1. Code splitting par feature

```tsx
// Lazy loading des features
const DashboardView = lazy(
  () => import("@/features/dashboard/views/dashboard.view")
);
```

### 2. Optimisation des re-renders

```tsx
// Utilisation de useMemo et useCallback
export function useExpensiveCalculation(data: Data[]) {
  return useMemo(() => {
    return data.reduce((acc, item) => {
      // Calcul coûteux
      return acc + complexCalculation(item);
    }, 0);
  }, [data]);
}
```

### 3. Optimisation des requêtes avec TanStack Query

```tsx
// Utilisation des capacités de cache de TanStack Query
export function useUserProfile(userId: string) {
  return useQuery({
    queryKey: ["user", userId],
    queryFn: () => userService.getProfile(userId),
    staleTime: 5 * 60 * 1000, // Les données restent "fresh" pendant 5 minutes
    cacheTime: 10 * 60 * 1000, // Garde en cache pendant 10 minutes après être "stale"
    refetchOnMount: false, // Ne pas refetch automatiquement au mount si les données sont fresh
    refetchOnReconnect: "always", // Toujours refetch à la reconnexion
  });
}

// Optimistic updates avec TanStack Query
export function useUpdateProfile() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.updateProfile,
    onMutate: async (newData) => {
      // Cancel les requêtes en cours
      await queryClient.cancelQueries({ queryKey: ["user", newData.id] });

      // Snapshot de la valeur précédente
      const previousData = queryClient.getQueryData(["user", newData.id]);

      // Mise à jour optimiste
      queryClient.setQueryData(["user", newData.id], newData);

      return { previousData };
    },
    onError: (err, newData, context) => {
      // Rollback en cas d'erreur
      queryClient.setQueryData(["user", newData.id], context.previousData);
    },
    onSettled: () => {
      // Toujours invalider pour s'assurer de la synchronisation
      queryClient.invalidateQueries({ queryKey: ["user"] });
    },
  });
}
```

## Guidelines de développement

### 1. Checklist pour une nouvelle feature

- [ ] Créer la structure de dossiers complète
- [ ] Définir les types TypeScript dans `.type.ts`
- [ ] Créer les schémas de validation Zod dans `.schema.ts`
- [ ] Implémenter les services API dans `.service.ts`
- [ ] Créer les hooks TanStack Query pour les requêtes dans `use-*.hook.ts`
- [ ] Créer le store Zustand dans `.store.ts` pour l'état UI uniquement
- [ ] Développer les hooks custom dans `.hook.ts`
- [ ] Créer les composants dans `.component.tsx`
- [ ] Assembler les views dans `.view.tsx`
- [ ] Ajouter les routes dans `/app`
- [ ] Configurer les query keys pour TanStack Query
- [ ] Documenter l'API publique

### 2. Code review checklist

- [ ] Pas de logique dans les composants/views
- [ ] Hooks utilisés pour toute la logique
- [ ] Services pour tous les appels API
- [ ] TanStack Query pour tout l'état serveur
- [ ] Zustand uniquement pour l'état UI
- [ ] Pas de duplication état serveur/client
- [ ] Validation Zod pour toutes les entrées utilisateur
- [ ] Types TypeScript corrects et complets
- [ ] Gestion d'erreur appropriée
- [ ] Conventions de nommage respectées (kebab-case + suffixes)
- [ ] Pas de dépendances circulaires
- [ ] Performance optimisée (memo, callback, etc.)
- [ ] Query keys bien structurées et cohérentes

### 3. Best practices

1. **État serveur vs client** : Utiliser TanStack Query pour tout l'état serveur, Zustand uniquement pour l'UI
2. **Query keys cohérentes** : Suivre un pattern uniforme comme `['resource', id, ...params]`
3. **Optimistic updates** : Utiliser les capacités de TanStack Query pour une UX fluide
4. **Immutabilité** : Toujours créer de nouvelles références pour les objets/arrays
5. **Pure functions** : Les utils et services doivent être des fonctions pures
6. **Error boundaries** : Wrapper les features critiques
7. **Loading states** : Utiliser les états de TanStack Query (isLoading, isFetching, etc.)
8. **Invalidation intelligente** : Invalider uniquement les queries nécessaires
9. **Prefetching** : Anticiper les besoins de données avec prefetchQuery
10. **Accessibility** : Utiliser les composants shadcn/ui qui sont accessibles par défaut
11. **Responsive** : Mobile-first avec Tailwind
12. **SEO** : Utiliser les metadata Next.js appropriées
13. **i18n** : Préparer l'internationalisation dès le début

### 4. Anti-patterns à éviter

- ❌ Logique métier dans les composants
- ❌ Appels API directs dans les composants (toujours passer par TanStack Query)
- ❌ Dupliquer l'état serveur dans Zustand
- ❌ Utiliser useEffect pour synchroniser l'état (utiliser TanStack Query)
- ❌ État local quand TanStack Query ou Zustand serait plus approprié
- ❌ Props drilling excessif
- ❌ Composants trop complexes (> 200 lignes)
- ❌ Duplication de code entre features
- ❌ Validation côté client uniquement
- ❌ Ignorer les erreurs TypeScript
- ❌ Query keys incohérentes ou mal structurées
- ❌ Oublier la gestion des états de chargement et d'erreur

## Conclusion

Cette architecture feature-based avec TanStack Query et Zustand permet de :

- ✅ **Séparation claire** : État serveur (TanStack Query) vs état client (Zustand)
- ✅ **Performance optimale** : Cache intelligent et optimistic updates
- ✅ **Scalabilité** : Ajout facile de nouvelles features
- ✅ **Maintenabilité** : Code organisé et prévisible
- ✅ **Réutilisabilité** : Composants et logique partagés
- ✅ **Testabilité** : Isolation des responsabilités
- ✅ **Code splitting** : Naturel par feature
- ✅ **DX excellente** : Structure claire et conventions établies
- ✅ **Synchronisation automatique** : Les données restent toujours à jour
- ✅ **Gestion d'erreur robuste** : Built-in avec TanStack Query

L'objectif est de maintenir cette architecture tout au long du projet, en résistant à la tentation de prendre des raccourcis qui compromettraient la qualité à long terme.
