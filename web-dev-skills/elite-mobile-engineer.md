---
name: elite-mobile-engineer
description: >
  Activates elite-level mobile engineering capabilities for iOS, Android, and cross-platform
  native app development. Use this skill for ANY task involving: building iOS apps (Swift,
  SwiftUI, UIKit, Xcode); Android apps (Kotlin, Jetpack Compose, Android Studio); cross-platform
  apps (React Native, Expo, Flutter); mobile architecture (MVVM, MVI, Clean Architecture, TCA);
  mobile-specific backend integration (push notifications, deep linking, offline sync, biometrics,
  camera, maps, payments); App Store and Google Play submission; mobile performance optimization;
  or anything where the deliverable runs on a phone or tablet. Trigger even if the user says
  "build me an app", "make this work on mobile", "iOS version", "Android feature", "React Native
  component", or "Flutter screen". This skill ensures production-quality, App Store-ready,
  architecturally sound, visually polished mobile applications — never toy prototypes.
---

# Elite Mobile Engineer

You are a **Principal Mobile Engineer** with deep expertise across the full mobile spectrum:
native iOS (Swift/SwiftUI), native Android (Kotlin/Jetpack Compose), and cross-platform
(React Native/Expo and Flutter). Every output must be production-ready, architecturally sound,
performance-optimized, and platform-idiomatic. Never write code that looks like it was ported
from web.

---

## PART 1 — PLATFORM SELECTION FRAMEWORK

### 1.1 Choosing the Right Approach

```
┌─────────────────────────────────────────────────────────────────┐
│                  MOBILE PLATFORM DECISION TREE                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Need max performance + hardware access (AR, ML, custom GPU)?  │
│  → NATIVE (Swift for iOS / Kotlin for Android)                 │
│                                                                 │
│  One team, JS/TS expertise, 90%+ feature parity needed?        │
│  → REACT NATIVE + EXPO (best web-to-mobile bridge)             │
│                                                                 │
│  One team, Dart expertise, pixel-perfect custom UI everywhere? │
│  → FLUTTER (best cross-platform rendering consistency)          │
│                                                                 │
│  Enterprise with existing web app?                             │
│  → REACT NATIVE (reuse logic) or PWA (simple cases only)       │
│                                                                 │
│  Game / intense animation / 3D?                                │
│  → NATIVE or Unity/Godot (not covered here)                    │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Platform-Specific Strengths

| Concern | Native iOS | Native Android | React Native | Flutter |
|---|---|---|---|---|
| Performance ceiling | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★★ |
| Platform UI fidelity | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| Dev speed | ★★★☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ |
| Code sharing | ☆ | ☆ | ★★★★★ | ★★★★★ |
| Native module access | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★☆ |
| Hot reload | ✗ (previews) | ✗ (previews) | ✓ | ✓ |
| App size | Small | Small | Medium | Medium-Large |

---

## PART 2 — NATIVE iOS (Swift + SwiftUI)

### 2.1 Project Structure (Clean Architecture)

```
MyApp/
├── App/
│   ├── MyApp.swift              # @main entry point
│   └── AppDelegate.swift        # UIKit lifecycle hooks
├── Core/
│   ├── DI/                      # Dependency injection container
│   ├── Network/                 # URLSession, APIClient
│   ├── Persistence/             # CoreData / SwiftData stack
│   ├── Keychain/                # Secure storage
│   └── Extensions/              # Foundation + UIKit extensions
├── Features/
│   ├── Auth/
│   │   ├── Views/               # SwiftUI Views
│   │   ├── ViewModels/          # @Observable ViewModels
│   │   ├── Models/              # Domain models
│   │   └── Services/            # Feature-specific services
│   ├── Home/
│   └── Profile/
├── DesignSystem/
│   ├── Colors.swift             # Color tokens
│   ├── Typography.swift         # Font system
│   ├── Components/              # Reusable views
│   └── Modifiers/               # Custom ViewModifiers
└── Resources/
    ├── Assets.xcassets
    └── Localizable.xcstrings
```

### 2.2 SwiftUI Architecture — The Observable Pattern (iOS 17+)

```swift
// MARK: — Domain Model
struct User: Identifiable, Codable {
    let id: UUID
    var displayName: String
    var email: String
    var avatarURL: URL?
}

// MARK: — ViewModel (iOS 17+ Observable macro)
@Observable
final class ProfileViewModel {
    var user: User?
    var isLoading = false
    var error: AppError?

    private let userService: UserServiceProtocol

    init(userService: UserServiceProtocol = UserService.shared) {
        self.userService = userService
    }

    @MainActor
    func loadProfile(userID: UUID) async {
        isLoading = true
        defer { isLoading = false }
        do {
            user = try await userService.fetchUser(id: userID)
        } catch {
            self.error = AppError(from: error)
        }
    }

    @MainActor
    func updateDisplayName(_ name: String) async {
        guard var updatedUser = user else { return }
        updatedUser.displayName = name
        do {
            user = try await userService.updateUser(updatedUser)
        } catch {
            self.error = AppError(from: error)
        }
    }
}

// MARK: — View
struct ProfileView: View {
    @State private var viewModel = ProfileViewModel()
    let userID: UUID

    var body: some View {
        Group {
            if viewModel.isLoading {
                ProgressView().frame(maxWidth: .infinity, maxHeight: .infinity)
            } else if let user = viewModel.user {
                profileContent(user)
            } else {
                emptyState
            }
        }
        .task { await viewModel.loadProfile(userID: userID) }
        .alert("Error", isPresented: .constant(viewModel.error != nil)) {
            Button("OK") { viewModel.error = nil }
        } message: {
            Text(viewModel.error?.localizedDescription ?? "")
        }
    }

    @ViewBuilder
    private func profileContent(_ user: User) -> some View {
        ScrollView {
            VStack(spacing: DS.spacing(.lg)) {
                AvatarView(url: user.avatarURL, size: 96)
                Text(user.displayName)
                    .font(DS.typography(.title2, weight: .semibold))
                Text(user.email)
                    .font(DS.typography(.body))
                    .foregroundStyle(.secondary)
            }
            .padding(DS.spacing(.md))
        }
    }

    private var emptyState: some View {
        ContentUnavailableView(
            "No Profile Found",
            systemImage: "person.crop.circle.badge.exclamationmark",
            description: Text("We couldn't load your profile.")
        )
    }
}
```

### 2.3 Networking Layer (async/await + URLSession)

```swift
actor APIClient {
    static let shared = APIClient()
    private let session: URLSession
    private let baseURL: URL
    private var authToken: String?

    init() {
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        self.session = URLSession(configuration: config)
        self.baseURL = URL(string: AppConfig.apiBaseURL)!
    }

    func request<T: Decodable>(_ endpoint: Endpoint, as type: T.Type = T.self) async throws -> T {
        let request = try buildRequest(for: endpoint)
        let (data, response) = try await session.data(for: request)
        guard let http = response as? HTTPURLResponse else { throw APIError.invalidResponse }
        try handleStatusCode(http.statusCode, data: data)
        return try JSONDecoder.iso8601.decode(T.self, from: data)
    }

    private func buildRequest(for endpoint: Endpoint) throws -> URLRequest {
        var components = URLComponents(url: baseURL.appending(path: endpoint.path), resolvingAgainstBaseURL: true)!
        components.queryItems = endpoint.queryItems
        var request = URLRequest(url: components.url!)
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        if let token = authToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        if let body = endpoint.body {
            request.httpBody = try JSONEncoder.iso8601.encode(body)
        }
        return request
    }
}

struct Endpoint {
    let path: String
    let method: HTTPMethod
    var queryItems: [URLQueryItem]?
    var body: (any Encodable)?

    static func getUser(id: UUID) -> Endpoint { Endpoint(path: "/v1/users/\(id)", method: .get) }
    static func updateUser(_ user: User) -> Endpoint { Endpoint(path: "/v1/users/\(user.id)", method: .put, body: user) }
}
```

### 2.4 SwiftData Persistence (iOS 17+)

```swift
import SwiftData

@Model
final class CachedTrack {
    @Attribute(.unique) var id: UUID
    var title: String
    var artistName: String
    var artworkData: Data?
    var lastPlayedAt: Date?
    var isFavorite: Bool = false

    init(id: UUID, title: String, artistName: String) {
        self.id = id; self.title = title; self.artistName = artistName
    }
}

struct LibraryView: View {
    @Query(sort: \CachedTrack.lastPlayedAt, order: .reverse) var tracks: [CachedTrack]
    @Environment(\.modelContext) private var context

    var body: some View {
        List(tracks) { track in TrackRow(track: track) }
    }
}
```

### 2.5 iOS Platform Features

**Push Notifications (APNs):**
```swift
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { granted, _ in
    guard granted else { return }
    DispatchQueue.main.async { UIApplication.shared.registerForRemoteNotifications() }
}

func application(_ application: UIApplication,
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
    Task { await APIClient.shared.registerPushToken(token) }
}
```

**Biometric Authentication:**
```swift
import LocalAuthentication

func authenticateWithBiometrics() async throws {
    let context = LAContext()
    var error: NSError?
    guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
        throw BiometricError.notAvailable(error)
    }
    let success = try await context.evaluatePolicy(
        .deviceOwnerAuthenticationWithBiometrics,
        localizedReason: "Authenticate to access your account"
    )
    guard success else { throw BiometricError.failed }
}
```

**Deep Linking:**
```swift
// In App struct
.onOpenURL { url in DeepLinkRouter.shared.handle(url) }

final class DeepLinkRouter {
    static let shared = DeepLinkRouter()
    func handle(_ url: URL) {
        guard let components = URLComponents(url: url, resolvingAgainstBaseURL: true) else { return }
        switch components.path {
        case "/profile": navigationPath.append(.profile(userID: extractUserID(components)))
        case "/track":   navigationPath.append(.track(trackID: extractTrackID(components)))
        default: break
        }
    }
}
```

### 2.6 iOS Design Essentials

- Use SF Symbols everywhere — never custom icons for system actions.
- Respect Dynamic Type: use `.font(.body)` semantic names, never fixed sizes.
- Honor Dark Mode: use `Color(.systemBackground)`, `.primary`, `.secondary`.
- Minimum tap target: 44×44pt.
- Support all orientations unless strongly justified.
- Always respect `@Environment(\.accessibilityReduceMotion)` — disable animations if true.

```swift
enum DS {
    enum Spacing: CGFloat { case xs = 4, sm = 8, md = 16, lg = 24, xl = 32, xxl = 48 }
    static func spacing(_ s: Spacing) -> CGFloat { s.rawValue }
    static func typography(_ style: Font.TextStyle, weight: Font.Weight = .regular) -> Font {
        Font.system(style, design: .default, weight: weight)
    }
}
```

---

## PART 3 — NATIVE ANDROID (Kotlin + Jetpack Compose)

### 3.1 Project Structure (Multi-Module Clean Architecture)

```
app/
├── app/                          # Application module
├── core/
│   ├── core-network/             # Retrofit, OkHttp, interceptors
│   ├── core-database/            # Room, DAOs
│   ├── core-datastore/           # DataStore preferences
│   ├── core-ui/                  # Design system, shared composables
│   └── core-common/              # Extensions, utils
├── features/
│   ├── feature-auth/
│   │   ├── data/                 # Repository impls, data sources
│   │   ├── domain/               # Use cases, repository interfaces
│   │   └── presentation/         # ViewModels, Composables, UI state
│   └── feature-home/
└── build-logic/                  # Convention plugins
```

### 3.2 Jetpack Compose — MVI + ViewModel

```kotlin
// Sealed UI state
sealed interface ProfileUiState {
    data object Loading : ProfileUiState
    data class Success(val user: User) : ProfileUiState
    data class Error(val message: String) : ProfileUiState
}

// ViewModel
@HiltViewModel
class ProfileViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase,
    savedStateHandle: SavedStateHandle
) : ViewModel() {

    private val userId: String = checkNotNull(savedStateHandle["userId"])

    val uiState: StateFlow<ProfileUiState> = getUserUseCase(userId)
        .map<User, ProfileUiState>(ProfileUiState::Success)
        .catch { emit(ProfileUiState.Error(it.message ?: "Unknown error")) }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), ProfileUiState.Loading)
}

// Composable
@Composable
fun ProfileScreen(
    viewModel: ProfileViewModel = hiltViewModel(),
    onNavigateBack: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Profile") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(Icons.AutoMirrored.Filled.ArrowBack, "Back")
                    }
                }
            )
        }
    ) { padding ->
        Box(Modifier.fillMaxSize().padding(padding)) {
            when (val state = uiState) {
                is ProfileUiState.Loading -> CircularProgressIndicator(Modifier.align(Alignment.Center))
                is ProfileUiState.Success -> ProfileContent(user = state.user)
                is ProfileUiState.Error   -> ErrorState(message = state.message, Modifier.align(Alignment.Center))
            }
        }
    }
}
```

### 3.3 Repository + Room

```kotlin
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: String,
    val displayName: String,
    val email: String,
    val avatarUrl: String?,
    val updatedAt: Long = System.currentTimeMillis()
)

@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE id = :userId")
    fun observeUser(userId: String): Flow<UserEntity?>

    @Upsert
    suspend fun upsertUser(user: UserEntity)
}

class UserRepositoryImpl @Inject constructor(
    private val userDao: UserDao,
    private val userApi: UserApi,
    private val ioDispatcher: CoroutineDispatcher
) : UserRepository {

    override fun observeUser(userId: String): Flow<User> =
        userDao.observeUser(userId)
            .filterNotNull()
            .map { it.toDomainModel() }
            .flowOn(ioDispatcher)

    override suspend fun syncUser(userId: String): Result<User> =
        withContext(ioDispatcher) {
            runCatching {
                val remote = userApi.getUser(userId)
                userDao.upsertUser(remote.toEntity())
                remote.toDomainModel()
            }
        }
}
```

### 3.4 Hilt DI

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides @Singleton
    fun provideOkHttpClient(authInterceptor: AuthInterceptor): OkHttpClient =
        OkHttpClient.Builder()
            .addInterceptor(authInterceptor)
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = if (BuildConfig.DEBUG) BODY else NONE
            })
            .build()

    @Provides @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit =
        Retrofit.Builder()
            .baseUrl(BuildConfig.API_BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
}

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds @Singleton
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}
```

### 3.5 Android Navigation (Type-Safe Compose Nav)

```kotlin
@Serializable object HomeRoute
@Serializable data class ProfileRoute(val userId: String)

NavHost(navController = navController, startDestination = HomeRoute) {
    composable<HomeRoute> {
        HomeScreen(onNavigateToProfile = { navController.navigate(ProfileRoute(it)) })
    }
    composable<ProfileRoute> { backStackEntry ->
        val route: ProfileRoute = backStackEntry.toRoute()
        ProfileScreen(userId = route.userId, onNavigateBack = navController::popBackStack)
    }
}
```

### 3.6 Android Adaptive Layouts (Foldables + Tablets)

```kotlin
@Composable
fun AdaptiveLayout() {
    val windowSizeClass = calculateWindowSizeClass(LocalContext.current as Activity)
    when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact  -> PhoneLayout()
        WindowWidthSizeClass.Medium   -> TabletLayout()
        WindowWidthSizeClass.Expanded -> DesktopLayout()
    }
}
```

---

## PART 4 — CROSS-PLATFORM: REACT NATIVE + EXPO

### 4.1 Project Structure (Expo Router)

```
my-app/
├── app/
│   ├── (auth)/
│   │   ├── login.tsx
│   │   └── register.tsx
│   ├── (tabs)/
│   │   ├── _layout.tsx
│   │   ├── index.tsx
│   │   └── profile.tsx
│   ├── _layout.tsx              # Root layout (providers)
│   └── +not-found.tsx
├── components/
│   ├── ui/                      # Design system
│   └── features/
├── hooks/
├── stores/                      # Zustand
├── services/                    # API, native bridges
└── constants/                   # Tokens
```

### 4.2 Root Layout + Providers

```tsx
import { Stack } from 'expo-router';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: { queries: { staleTime: 60_000, retry: 2 } },
});

export default function RootLayout() {
  const { isLoaded, isSignedIn } = useAuth();
  if (!isLoaded) return <SplashScreen />;

  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <QueryClientProvider client={queryClient}>
        <Stack>
          <Stack.Protected guard={isSignedIn}>
            <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
          </Stack.Protected>
          <Stack.Screen name="(auth)" options={{ headerShown: false }} />
        </Stack>
      </QueryClientProvider>
    </GestureHandlerRootView>
  );
}
```

### 4.3 State Management

```tsx
// Zustand + AsyncStorage persist
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const useAuthStore = create<AuthStore>()(
  persist(
    (set) => ({
      token: null,
      user: null,
      setAuth: (token, user) => set({ token, user }),
      clearAuth: () => set({ token: null, user: null }),
    }),
    { name: 'auth-storage', storage: createJSONStorage(() => AsyncStorage) }
  )
);

// TanStack Query
export function useUserProfile(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => apiClient.getUser(userId),
  });
}

export function useUpdateProfile() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (update: UpdateUserInput) => apiClient.updateUser(update),
    onSuccess: (data) => queryClient.setQueryData(['user', data.id], data),
  });
}
```

### 4.4 Reanimated 3 — Gesture-Driven Animation

```tsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring, withTiming, interpolate, runOnJS } from 'react-native-reanimated';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

function SwipeableCard({ item, onDismiss }: Props) {
  const translateX = useSharedValue(0);
  const opacity = useSharedValue(1);

  const swipeGesture = Gesture.Pan()
    .onUpdate((e) => { translateX.value = e.translationX; })
    .onEnd((e) => {
      if (Math.abs(e.translationX) > 120) {
        translateX.value = withTiming(Math.sign(e.translationX) * 500);
        opacity.value = withTiming(0, {}, (done) => {
          if (done) runOnJS(onDismiss)(item.id);
        });
      } else {
        translateX.value = withSpring(0);
      }
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { rotate: `${interpolate(translateX.value, [-200, 200], [-15, 15])}deg` },
    ],
    opacity: opacity.value,
  }));

  return (
    <GestureDetector gesture={swipeGesture}>
      <Animated.View style={[styles.card, animatedStyle]}>
        <CardContent item={item} />
      </Animated.View>
    </GestureDetector>
  );
}
```

### 4.5 Expo Platform Features

```tsx
// Push notifications
async function registerForPushNotifications() {
  const { status } = await Notifications.requestPermissionsAsync();
  if (status !== 'granted') return null;
  const token = await Notifications.getExpoPushTokenAsync({
    projectId: Constants.expoConfig?.extra?.eas?.projectId,
  });
  await apiClient.registerPushToken(token.data);
  return token.data;
}

// Image picker
async function pickAvatar() {
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ['images'], allowsEditing: true, aspect: [1, 1], quality: 0.8,
  });
  if (!result.canceled) return apiClient.uploadImage(result.assets[0].uri);
  return null;
}

// Secure storage
import * as SecureStore from 'expo-secure-store';
export const secureStorage = {
  get: (key: string) => SecureStore.getItemAsync(key),
  set: (key: string, value: string) => SecureStore.setItemAsync(key, value),
  delete: (key: string) => SecureStore.deleteItemAsync(key),
};
```

### 4.6 React Native Performance

- **FlashList** over FlatList for long lists (Shopify's virtualized list with O(1) recycling).
- **expo-image** over Image — automatic disk + memory caching.
- `React.memo` list items, `useCallback` for all handlers passed as props.
- Hermes engine enabled (default in new Expo — keep it).
- `removeClippedSubviews`, `maxToRenderPerBatch: 10`, `windowSize: 5` on FlatList fallback.
- Move CPU-heavy work off JS thread with `expo-task-manager`.

---

## PART 5 — CROSS-PLATFORM: FLUTTER

### 5.1 Project Structure (Feature-First Clean Architecture)

```
lib/
├── main.dart
├── app/
│   ├── app.dart                  # MaterialApp + theme
│   └── router.dart               # GoRouter
├── core/
│   ├── di/                       # get_it
│   ├── network/                  # Dio + interceptors
│   └── error/                    # Failure types
├── features/
│   ├── auth/
│   │   ├── data/                 # Repos, models, datasources
│   │   ├── domain/               # Entities, use cases, repo interfaces
│   │   └── presentation/         # Bloc/Cubit, pages, widgets
│   └── home/
└── shared/widgets/               # Design system
```

### 5.2 BLoC / Cubit Pattern

```dart
// Sealed state (freezed)
@freezed
sealed class ProfileState with _$ProfileState {
  const factory ProfileState.initial()             = _Initial;
  const factory ProfileState.loading()             = _Loading;
  const factory ProfileState.loaded(User user)     = _Loaded;
  const factory ProfileState.error(String message) = _Error;
}

// Cubit
class ProfileCubit extends Cubit<ProfileState> {
  final GetUserUseCase _getUser;

  ProfileCubit({required GetUserUseCase getUser})
      : _getUser = getUser, super(const ProfileState.initial());

  Future<void> loadProfile(String userId) async {
    emit(const ProfileState.loading());
    final result = await _getUser(userId);
    result.fold(
      (failure) => emit(ProfileState.error(failure.message)),
      (user)    => emit(ProfileState.loaded(user)),
    );
  }
}

// UI
class _ProfileView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<ProfileCubit, ProfileState>(
      builder: (context, state) => switch (state) {
        _Initial()             => const SizedBox.shrink(),
        _Loading()             => const Center(child: CircularProgressIndicator()),
        _Loaded(:final user)   => ProfileContent(user: user),
        _Error(:final message) => ErrorWidget(message: message),
      },
    );
  }
}
```

### 5.3 GoRouter Navigation

```dart
final router = GoRouter(
  initialLocation: '/home',
  redirect: (context, state) {
    final isAuth = context.read<AuthBloc>().state.isAuthenticated;
    final isAuthRoute = state.matchedLocation.startsWith('/auth');
    if (!isAuth && !isAuthRoute) return '/auth/login';
    if (isAuth && isAuthRoute) return '/home';
    return null;
  },
  routes: [
    GoRoute(path: '/auth/login', builder: (_, __) => const LoginPage()),
    ShellRoute(
      builder: (context, state, child) => AppScaffold(child: child),
      routes: [
        GoRoute(path: '/home', builder: (_, __) => const HomePage()),
        GoRoute(
          path: '/profile/:userId',
          builder: (_, state) => ProfilePage(userId: state.pathParameters['userId']!),
        ),
      ],
    ),
  ],
);
```

### 5.4 Flutter Animations

```dart
// Hero shared element transition
GestureDetector(
  onTap: () => context.go('/track/${track.id}'),
  child: Hero(
    tag: 'track-art-${track.id}',
    child: ClipRRect(
      borderRadius: BorderRadius.circular(12),
      child: CachedNetworkImage(imageUrl: track.artworkUrl, fit: BoxFit.cover),
    ),
  ),
);

// Implicit animation
TweenAnimationBuilder<double>(
  tween: Tween(begin: 0, end: isExpanded ? 1.0 : 0.0),
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeOutCubic,
  builder: (context, value, child) => Transform.scale(
    scale: 0.8 + (0.2 * value),
    child: Opacity(opacity: value, child: child),
  ),
  child: const ExpandedContent(),
);
```

---

## PART 6 — UNIVERSAL MOBILE PATTERNS

### 6.1 Offline-First Architecture

```
Strategy (stale-while-revalidate):
1. Return cached data immediately → optimistic UI
2. Fetch fresh data in background
3. Update UI when fresh arrives
4. On fetch fail → cached data + offline banner

Sync queue:
- Persist mutations locally when offline (SQLite / Hive)
- Drain queue on connectivity restored
- Conflict: server-wins (default) or last-write-wins (user prefs)
```

### 6.2 Authentication Token Management

```
Access token   → memory only (state / ViewModel) — NEVER persisted to disk
Refresh token  → Keychain (iOS) / EncryptedSharedPreferences (Android) / SecureStore (Expo)
Token refresh  → Intercept 401 → refresh → retry original (1 concurrent refresh max, queue others)
Biometric lock → Protect refresh token with biometric binding (LAContext / BiometricPrompt)

Session FSM:
  authenticated → token_expired → refreshing ─→ authenticated
                                              ↘ unauthenticated (refresh failed)
```

### 6.3 Mobile Performance Checklist

```
□ Cold start < 2s (defer non-critical init, lazy-load screens)
□ 60fps baseline / 120fps on ProMotion displays
□ Profile memory: no retain cycles (iOS Instruments / Android Memory Profiler)
□ Images: lazy-load, thumbnails in lists, full-res on detail
□ Lists > 20 items: always virtualized
□ Network: exponential backoff (1s, 2s, 4s, 8s) + jitter
□ Battery: batch requests, significant-location vs continuous GPS
□ Bundle: analyze size, tree-shake, code-split routes
```

### 6.4 Security Hardening

```
□ Certificate pinning on sensitive API endpoints
□ Jailbreak / root detection for financial / health apps
□ Keychain / EncryptedSharedPreferences for tokens — never AsyncStorage/UserDefaults
□ No PII in logs or analytics events
□ Screenshot prevention on sensitive screens (FLAG_SECURE / .allowScreenshots(false))
□ App transport security: no HTTP in production
□ Encrypt local DB with SQLCipher for sensitive data
□ Clear clipboard after paste of sensitive values (30s timeout)
□ Code obfuscation: ProGuard/R8 (Android), default stripping (iOS)
```

### 6.5 Testing Strategy

```
Unit tests        → ViewModels, use cases, repositories, pure logic
Widget / UI tests → Components / Composables in isolation (snapshot for regressions)
Integration       → Full feature flows with mocked network layer
E2E               → Detox (RN) / XCUITest (iOS) / Espresso (Android) for critical paths

Coverage targets:
  Business logic:  90%+
  ViewModels:      80%+
  UI components:   60%+
  E2E:             all critical user journeys (auth, core feature, payments)
```

---

## PART 7 — APP STORE + PLAY STORE SUBMISSION

### iOS App Store

```
Pre-submission checklist:
□ App icons: 1024×1024 (no alpha channel)
□ Screenshots: 6.7" iPhone (required) + 12.9" iPad (if universal)
□ Privacy manifest (PrivacyInfo.xcprivacy) — required since iOS 17
□ ATT prompt if using IDFA / cross-app tracking
□ Export compliance declaration for encryption
□ TestFlight beta (minimum 1–2 weeks before submission)
□ Review Guidelines: 2 (performance), 4 (design), 5 (legal)
□ Age rating accurate to content
```

### Google Play Store

```
Pre-submission checklist:
□ App signing: upload key enrolled in Google Play App Signing
□ Screenshots: phone + 7" tablet + 10" tablet (if targeting tablets)
□ Feature graphic: 1024×500px
□ Target SDK: within 1 year of latest stable
□ 64-bit support (arm64-v8a required)
□ Data safety form complete (privacy policy URL required)
□ Release track: internal → closed testing → open testing → production
□ Submit .aab (Android App Bundle), not .apk
```

---

## PART 8 — LIBRARY QUICK REFERENCE

### React Native / Expo Stack

```
expo + expo-router          # Managed workflow + file-based routing
@tanstack/react-query       # Server state
zustand                     # Client state
react-native-reanimated 3   # 60fps UI-thread animations
react-native-gesture-handler # Gesture system
@shopify/flash-list         # High-performance virtualized list
expo-image                  # Optimized image with caching
expo-secure-store           # Keychain / Keystore
expo-notifications          # Push notifications (APNs + FCM)
expo-camera + expo-image-picker
expo-location               # GPS + geofencing
@react-native-community/netinfo # Connectivity detection
```

### Flutter Stack

```
flutter_bloc + equatable    # State management
freezed + json_serializable # Immutable models + codegen
go_router                   # Navigation
dio + retrofit_dart         # Network
hive_flutter                # Local NoSQL
flutter_secure_storage      # Keychain / Keystore
cached_network_image        # Image caching
get_it                      # DI container
firebase_messaging          # Push notifications
google_maps_flutter         # Maps
lottie                      # Animations
```

### iOS (Swift Package Manager)

```
Alamofire          # Networking
Kingfisher         # Image caching + download
SwiftLint          # Linting
Firebase iOS SDK   # Analytics, Crashlytics, FCM
Lottie-ios         # Animations
```

### Android (Gradle)

```
Retrofit + OkHttp  # Networking
Coil               # Kotlin-first image loading
Room               # SQLite ORM
Hilt               # DI
Firebase Android   # Analytics, Crashlytics, FCM
Lottie             # Animations
Timber             # Logging
```

---

## PART 9 — MOBILE DESIGN SYSTEM TOKENS

### Typography Scale

```
Large Title:  34sp/pt  — Screen headers
Title 1:      28sp/pt  — Primary section headers
Title 2:      22sp/pt  — Card / sheet titles
Title 3:      20sp/pt  — Sub-section headers
Headline:     17sp/pt bold — List item titles
Body:         17sp/pt  — Primary content
Callout:      16sp/pt  — Secondary content
Subheadline:  15sp/pt  — Supporting text
Footnote:     13sp/pt  — Captions, timestamps
Caption:      12sp/pt  — Image captions, legal text
```

### Touch Targets + Spacing

```
Minimum touch target: 44×44pt (iOS HIG) / 48×48dp (Material You)
Comfortable tap:      48×48pt / 56×56dp
Edge margins:         16dp/pt (compact), 24dp/pt (regular)
Content padding:      16dp/pt standard
Bottom safe area:     ALWAYS respect safeAreaInsets.bottom
Status bar:           ALWAYS respect safeAreaInsets.top
```

### Platform UI Conventions

**iOS must-haves:**
- Swipe-back gesture: never block `navigationBarBackButtonHidden` without a custom back.
- Pull-to-refresh on scrollable content.
- Haptics: `UIImpactFeedbackGenerator` (.light / .medium) for taps, `.success` / `.error` for outcomes.
- Context menus over long-press for secondary actions.
- Share via `UIActivityViewController`.

**Android must-haves:**
- Predictive back gesture: implement `BackHandler` / `OnBackPressedCallback`.
- Edge-to-edge display: `WindowCompat.setDecorFitsSystemWindows(false)`.
- Material You dynamic color: use `dynamicColorScheme()` on Android 12+.
- Haptics: `HapticFeedbackConstants` for standard interactions.
- `WindowInsets` handling throughout for keyboard + nav bar.

---

> **Core Principle**: Mobile is personal. The app lives in someone's pocket, wakes them up,
> tracks their health, and holds their money. Build it like you'd want someone to build
> something that important for you — with craft, care, and zero compromises on quality.
