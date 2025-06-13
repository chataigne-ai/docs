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
- **Backend Client** - Client TypeScript type-safe qui partage les DTOs avec le backend
- **Better-Fetch** - Client HTTP performant pour le backend client
- **Zod** - Validation de schémas runtime
- **React Hook Form** - Gestion des formulaires performante
- **Sentry** - Monitoring et error tracking

## Principes fondamentaux

### 1. Séparation des responsabilités

- **Views** : Uniquement la composition visuelle, aucune logique
- **Hooks** : Toute la logique métier, orchestration et utilisation de TanStack Query
- **Services** : Communication avec le backend via le client TypeScript type-safe
- **Backend Client** : Client TypeScript qui partage les DTOs avec le backend
- **Components** : Éléments UI réutilisables et stupides
- **Store** : État local de la feature (UI state, form state temporaire)
- **Utils** : Fonctions pures et helpers

### 2. Gestion d'état hybride avec Backend Client

- **Backend Client TypeScript** : Couche d'abstraction type-safe avec DTOs partagés entre frontend et backend
- **TanStack Query** : Pour tout l'état serveur (data fetching, cache, synchronisation) via le backend client
- **Zustand** : Pour l'état client uniquement (UI state, préférences utilisateur, état global de l'app)
- **Services** : Orchestrent les appels au backend client et convertissent les données si nécessaire
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
│   │   ├── services/       # Services utilisant le backend client
│   │   │   └── auth.service.ts
│   │   ├── store/          # État Zustand
│   │   │   └── auth.store.ts
│   │   ├── validations/    # Schémas Zod
│   │   │   └── auth.schema.ts
│   │   ├── utils/          # Helpers
│   │   │   └── token.util.ts
│   │   └── types/          # Types TypeScript (complémentaires aux DTOs)
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
├── backend-client/          # Client TypeScript partagé avec le backend
│   ├── index.ts            # ChataigneClient principal
│   ├── auth/               # Client et DTOs d'authentification
│   │   ├── auth.client.ts
│   │   └── auth.dto.ts
│   ├── location/           # Client et DTOs de localisation
│   │   ├── location.client.ts
│   │   ├── location.dto.ts
│   │   └── catalog/
│   │       ├── catalog.client.ts
│   │       └── catalog.dto.ts
│   ├── user/
│   │   ├── user.client.ts
│   │   └── user.dto.ts
│   └── ...                 # Autres modules
│
├── shared/                  # Ressources partagées
│   ├── components/         # Composants réutilisables
│   │   ├── ui/            # shadcn/ui components
│   │   └── common/        # Composants métier partagés
│   ├── hooks/             # Hooks partagés
│   │   ├── queries/       # Hooks TanStack Query partagés
│   │   └── mutations/     # Mutations TanStack Query partagées
│   ├── services/          # Services communs
│   │   └── backend-client.service.ts  # Instance du client backend
│   ├── store/             # État global
│   ├── utils/             # Utilitaires communs
│   ├── validations/       # Schémas partagés
│   ├── types/             # Types globaux complémentaires
│   └── constants/         # Constantes
│
├── styles/                 # Styles globaux
└── config/                # Configuration
    ├── sentry.config.ts
    ├── query.config.ts    # Configuration TanStack Query
    ├── backend-client.config.ts  # Configuration du client backend
    └── env.config.ts

```

## Architecture détaillée

### 1. Backend Client TypeScript

Le backend client est un module TypeScript type-safe qui partage les DTOs avec le backend, garantissant une cohérence parfaite entre frontend et backend.

### Configuration et instance globale

```tsx
// config/backend-client.config.ts
import { BetterFetch } from "@better-fetch/fetch";

export const backendFetch = new BetterFetch({
  baseURL: process.env.NEXT_PUBLIC_BACKEND_URL,
  headers: {
    "Content-Type": "application/json",
  },
});

// shared/services/backend-client.service.ts
import { ChataigneClient } from "@/backend-client";
import { backendFetch } from "@/config/backend-client.config";

export const chataigneClient = new ChataigneClient(backendFetch);
```

### Structure du Backend Client

```tsx
// backend-client/index.ts
import { BetterFetch } from "@better-fetch/fetch";
import { UserClient } from "./user";
import { LocationClient } from "./location";
import { AuthClient } from "./auth";

export * from "./user";
export * from "./location";
export * from "./auth";

export class ChataigneClient {
  constructor(private readonly fetch: BetterFetch<any>) {}

  public auth = new AuthClient(this.fetch);
  public user = new UserClient(this.fetch);
  public location = new LocationClient(this.fetch);
}
```

### Exemple de Client avec DTOs

```tsx
// backend-client/location/location.client.ts
import { BetterFetch } from "@better-fetch/fetch";
import { CatalogClient } from "./catalog";
import {
  CreateLocationDTO,
  UpdateLocationDTO,
  LocationDTO,
  DeliverySettingsDTO,
} from "./location.dto";

export class LocationClient {
  readonly catalog: CatalogClient;

  constructor(private readonly fetch: BetterFetch<any>) {
    this.catalog = new CatalogClient(this.fetch);
  }

  async create(createLocationDTO: CreateLocationDTO): Promise<LocationDTO> {
    const response = await this.fetch(
      `${process.env.NEXT_PUBLIC_BACKEND_URL}/location`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(createLocationDTO),
      }
    );

    if (response.error) {
      throw new Error(response.error.message);
    }

    return response.data;
  }

  async update(
    locationId: string,
    updateLocationDTO: UpdateLocationDTO
  ): Promise<LocationDTO> {
    const response = await this.fetch<LocationDTO>(
      `${process.env.NEXT_PUBLIC_BACKEND_URL}/location/${locationId}`,
      {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(updateLocationDTO),
      }
    );

    if (response.error) {
      throw new Error(response.error.message);
    }

    return response.data;
  }

  async updateDeliverySettings(
    locationId: string,
    deliverySettingsDTO: DeliverySettingsDTO
  ): Promise<void> {
    const response = await this.fetch(
      `${process.env.NEXT_PUBLIC_BACKEND_URL}/location/${locationId}/delivery-settings`,
      {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(deliverySettingsDTO),
      }
    );

    if (response.error) {
      throw new Error(response.error.message);
    }
  }
}
```

### DTOs partagés

```tsx
// backend-client/location/location.dto.ts
export interface LocationDTO {
  id: string;
  name: string;
  address: string;
  phone?: string;
  email?: string;
  createdAt: string;
  updatedAt: string;
}

export interface CreateLocationDTO {
  name: string;
  address: string;
  phone?: string;
  email?: string;
}

export interface UpdateLocationDTO {
  name?: string;
  address?: string;
  phone?: string;
  email?: string;
}

export interface DeliverySettingsDTO {
  enabled: boolean;
  radius: number;
  fee: number;
  freeDeliveryThreshold?: number;
}
```

### 2. App Directory (Routing)

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

### Services utilisant le Backend Client

```tsx
// features/auth/services/auth.service.ts
import { chataigneClient } from "@/shared/services/backend-client.service";
import type { LoginSchema, RegisterSchema } from "../validations/auth.schema";
import type {
  LoginDTO,
  RegisterDTO,
  AuthResponseDTO,
  UserDTO,
} from "@/backend-client/auth";

class AuthService {
  async login(credentials: LoginSchema): Promise<AuthResponseDTO> {
    // Conversion du schéma Zod vers DTO si nécessaire
    const loginDTO: LoginDTO = {
      email: credentials.email,
      password: credentials.password,
    };

    return await chataigneClient.auth.login(loginDTO);
  }

  async register(data: RegisterSchema): Promise<AuthResponseDTO> {
    const registerDTO: RegisterDTO = {
      name: data.name,
      email: data.email,
      password: data.password,
    };

    return await chataigneClient.auth.register(registerDTO);
  }

  async logout(): Promise<void> {
    await chataigneClient.auth.logout();
  }

  async getCurrentUser(): Promise<UserDTO | null> {
    try {
      return await chataigneClient.auth.getCurrentUser();
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
- **Backend Clients** : kebab-case avec suffixe `.client.ts` (`user.client.ts`)
- **DTOs** : kebab-case avec suffixe `.dto.ts` (`user.dto.ts`)
- **Stores** : kebab-case avec suffixe `.store.ts` (`auth.store.ts`)
- **Validations** : kebab-case avec suffixe `.schema.ts` (`login.schema.ts`)
- **Utils** : kebab-case avec suffixe `.util.ts` (`format-date.util.ts`)
- **Types** : kebab-case avec suffixe `.type.ts` (`user.type.ts`) - pour les types complémentaires aux DTOs
- **Constants** : kebab-case avec suffixe `.constant.ts` (`api-endpoints.constant.ts`)
- **Config** : kebab-case avec suffixe `.config.ts` (`sentry.config.ts`)

**Pour les exports TypeScript** :

- PascalCase pour les interfaces/types (`User`, `AuthResponse`)
- PascalCase avec suffixe `DTO` pour les DTOs (`UserDTO`, `CreateUserDTO`)
- camelCase pour les schémas Zod (`loginSchema`)

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
import type {
  UserDTO,
  CreateUserDTO,
  UserFiltersDTO,
} from "@/backend-client/user";

// Hook pour liste des utilisateurs
export function useUsers(filters?: UserFiltersDTO) {
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
    mutationFn: (data: CreateUserDTO) => userService.createUser(data),
    onSuccess: (newUser: UserDTO) => {
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

### 5. Exemple d'utilisation avec sous-clients

```tsx
// features/location/hooks/use-catalog-mutations.hook.ts
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { chataigneClient } from "@/shared/services/backend-client.service";
import type { CreateCatalogDTO } from "@/backend-client/location/catalog";

export function useCreateCatalog() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (data: CreateCatalogDTO) => {
      // Utilisation du sous-client catalog
      await chataigneClient.location.catalog.create(data);

      // Récupérer le catalogue créé via le client parent
      const catalogs = await chataigneClient.location.getLocationCatalogs(
        data.locationId
      );
      const createdCatalog = catalogs.find((c) => c.name === data.name);

      if (!createdCatalog) {
        throw new Error("Le catalogue n'a pas pu être créé");
      }

      return createdCatalog;
    },
    onSuccess: (newCatalog) => {
      // Invalidation précise des requêtes
      queryClient.invalidateQueries({
        queryKey: ["location", newCatalog.locationId, "catalogs"],
      });
      queryClient.setQueryData(["catalogs", newCatalog.id], newCatalog);
    },
  });
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

## Avantages du Backend Client TypeScript

Le backend client apporte plusieurs avantages majeurs à notre architecture :

### 1. Type Safety complète

```tsx
// ✅ Erreur de compilation si le DTO change
const user: UserDTO = await chataigneClient.user.getById("123");

// ✅ Autocomplétion et vérification des types
await chataigneClient.location.catalog.create({
  name: "Menu Été",
  locationId: "loc-123",
  // TypeScript nous aide avec les propriétés requises
});
```

### 2. Synchronisation automatique des contrats API

- Les DTOs sont partagés entre frontend et backend
- Tout changement de contrat API provoque une erreur de compilation
- Élimine les erreurs runtime dues aux contrats API non synchronisés

### 3. Clients structurés et prévisibles

```tsx
// Structure cohérente et prévisible
chataigneClient.user.create(data);
chataigneClient.location.update(id, data);
chataigneClient.location.catalog.create(data);
chataigneClient.order.draft.save(data);
```

### 4. Testabilité améliorée

```tsx
// Mock simple du client pour les tests
const mockClient = {
  user: {
    getById: vi.fn().mockResolvedValue(mockUserDTO),
  },
} as jest.Mocked<ChataigneClient>;
```

### 5. Réutilisabilité

- Le client peut être utilisé dans d'autres projets (admin, mobile, etc.)
- Les DTOs garantissent la cohérence entre tous les clients
- Facilite l'intégration de nouveaux frontends

### 6. Découplage des responsabilités

- **Backend Client** : Communication et typage
- **Services** : Logique métier et conversion
- **Hooks** : Orchestration et cache (TanStack Query)
- **Components** : Affichage uniquement

## Conclusion

Cette architecture feature-based avec Backend Client, TanStack Query et Zustand permet de :

- ✅ **Type Safety complète** : DTOs partagés entre frontend et backend
- ✅ **Contrats API synchronisés** : Erreurs de compilation en cas de désynchronisation
- ✅ **Séparation claire** : État serveur (TanStack Query) vs état client (Zustand)
- ✅ **Performance optimale** : Cache intelligent et optimistic updates
- ✅ **Scalabilité** : Ajout facile de nouvelles features et clients
- ✅ **Maintenabilité** : Code organisé, prévisible et type-safe
- ✅ **Réutilisabilité** : Client backend utilisable dans d'autres projets
- ✅ **Testabilité** : Isolation des responsabilités et mocking simplifié
- ✅ **Code splitting** : Naturel par feature
- ✅ **DX excellente** : Autocomplétion, type checking et structure claire
- ✅ **Synchronisation automatique** : Les données restent toujours à jour
- ✅ **Gestion d'erreur robuste** : Built-in avec TanStack Query et type safety

L'objectif est de maintenir cette architecture tout au long du projet, en résistant à la tentation de prendre des raccourcis qui compromettraient la qualité à long terme. Le backend client TypeScript garantit une robustesse et une maintenabilité exceptionnelles grâce à la type safety partagée.
