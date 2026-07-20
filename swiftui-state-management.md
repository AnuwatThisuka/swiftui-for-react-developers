# SwiftUI State Management

คู่มือการจัดการ State ใน SwiftUI สมัยใหม่ โดยเน้นแนวทางสำหรับโปรเจกต์ที่ใช้ **Swift 5.9+**, **Xcode 15+** และมี Deployment Target ตั้งแต่ **iOS 17 / macOS 14** ขึ้นไป

เอกสารนี้อธิบายแนวคิด State Management ตั้งแต่ระดับพื้นฐานจนถึงการจัดการ State สำหรับ Feature จริง พร้อมเปรียบเทียบแนวคิดกับ React + TypeScript

---

## Table of Contents

1. [State คืออะไร](#state-คืออะไร)
2. [หลักการสำคัญ](#หลักการสำคัญ)
3. [ภาพรวม Property Wrapper](#ภาพรวม-property-wrapper)
4. [Stored Property หรือ Input](#stored-property-หรือ-input)
5. [@State](#state)
6. [@Binding](#binding)
7. [@Observable](#observable)
8. [@Bindable](#bindable)
9. [@Environment](#environment)
10. [State Management สำหรับระบบเก่า](#state-management-สำหรับระบบเก่า)
11. [Loading, Error และ Empty State](#loading-error-และ-empty-state)
12. [Async/Await และ Task](#asyncawait-และ-task)
13. [State Ownership](#state-ownership)
14. [Derived State](#derived-state)
15. [Navigation State](#navigation-state)
16. [Form State](#form-state)
17. [Dependency Injection](#dependency-injection)
18. [Feature Store Pattern](#feature-store-pattern)
19. [การทดสอบ State](#การทดสอบ-state)
20. [Anti-patterns](#anti-patterns)
21. [Decision Guide](#decision-guide)
22. [Project Structure](#project-structure)
23. [Checklist](#checklist)

---

## State คืออะไร

State คือข้อมูลที่สามารถเปลี่ยนแปลงระหว่างการทำงานของแอป และมีผลต่อสิ่งที่แสดงบนหน้าจอ

ตัวอย่าง State:

- ข้อความที่ผู้ใช้กรอก
- สถานะเปิดหรือปิดของ Sheet
- รายการสินค้าที่โหลดจาก API
- สถานะกำลังโหลด
- ข้อความ Error
- รายการที่ผู้ใช้เลือก
- เส้นทาง Navigation
- ข้อมูล Session ของผู้ใช้

SwiftUI เป็น Declarative UI Framework หมายความว่าเรากำหนดว่า UI ควรมีหน้าตาอย่างไรสำหรับ State ปัจจุบัน แทนการสั่งแก้ UI ทีละจุด

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("Count: \(count)")

            Button("Increase") {
                count += 1
            }
        }
    }
}
```

เมื่อ `count` เปลี่ยน SwiftUI จะประเมิน `body` ใหม่และอัปเดตเฉพาะส่วนของ UI ที่ได้รับผลกระทบ

### เปรียบเทียบกับ React

```tsx
function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increase
      </button>
    </div>
  )
}
```

แนวคิดหลักเหมือนกัน:

```text
State เปลี่ยน
    ↓
Framework ประเมิน UI ใหม่
    ↓
หน้าจอแสดงผลตาม State ล่าสุด
```

---

## หลักการสำคัญ

### 1. มี Single Source of Truth

ข้อมูลหนึ่งชุดควรมีเจ้าของหลักเพียงจุดเดียว

```swift
struct ParentView: View {
    @State private var username = ""

    var body: some View {
        UsernameField(username: $username)
    }
}
```

`ParentView` เป็นเจ้าของ `username` ส่วน `UsernameField` ได้รับสิทธิ์อ่านและแก้ไขผ่าน `Binding`

ไม่ควรสร้าง State ซ้ำใน Child View เพราะอาจทำให้ข้อมูลสองจุดไม่ตรงกัน

### 2. State ควรอยู่ใกล้จุดใช้งานมากที่สุด

ถ้า State ใช้เพียง View เดียว ให้เก็บไว้ใน View นั้น

ถ้า State ใช้ร่วมกันหลาย Child View ให้ยก State ขึ้นไปยัง Parent ที่ใกล้ที่สุด

ถ้า State ใช้ทั้ง Feature ให้เก็บใน Feature Store หรือ Model

ถ้า State ใช้ทั่วทั้งแอป เช่น Session หรือ Theme ให้ส่งผ่าน Environment

### 3. View ควรเป็นผลลัพธ์ของ State

คิดในรูปแบบ:

```text
UI = function(State)
```

แทนการเขียนโค้ดแบบ Imperative เช่น:

```text
ถ้าโหลดเสร็จ ให้ซ่อน Spinner
จากนั้นแสดง List
จากนั้นเปลี่ยน Label
```

ควรเขียนเป็น:

```swift
switch state {
case .loading:
    ProgressView()

case .loaded(let products):
    ProductList(products: products)

case .failed(let message):
    ErrorView(message: message)
}
```

### 4. อย่าเก็บข้อมูลที่คำนวณได้เป็น State โดยไม่จำเป็น

ไม่ควร:

```swift
@State private var firstName = ""
@State private var lastName = ""
@State private var fullName = ""
```

ควร:

```swift
@State private var firstName = ""
@State private var lastName = ""

private var fullName: String {
    "\(firstName) \(lastName)"
}
```

`fullName` เป็น Derived State เพราะสามารถคำนวณจาก State อื่นได้

---

## ภาพรวม Property Wrapper

| รูปแบบ | ใช้เมื่อ | เจ้าของข้อมูล |
|---|---|---|
| Stored Property | รับข้อมูลเพื่อแสดงผลอย่างเดียว | Parent |
| `@State` | Local State หรือ Observable Model ที่ View เป็นเจ้าของ | View |
| `@Binding` | Child ต้องอ่านและแก้ State ของ Parent | Parent |
| `@Observable` | สร้าง Model หรือ Store ที่มี State หลายค่า | Model / Store |
| `@Bindable` | ต้องสร้าง Binding ไปยัง property ของ `@Observable` | เจ้าของ Observable Model |
| `@Environment` | Dependency หรือ Shared State จาก View Hierarchy | Root / Ancestor |
| `@StateObject` | View เป็นเจ้าของ `ObservableObject` รุ่นเก่า | View |
| `@ObservedObject` | View รับ `ObservableObject` รุ่นเก่าจากภายนอก | Parent |
| `@EnvironmentObject` | Shared `ObservableObject` รุ่นเก่า | Root / Ancestor |

### แนวทางสมัยใหม่

สำหรับ iOS 17+, macOS 14+ และระบบรุ่นเทียบเท่า แนะนำให้ใช้:

```text
@Observable + @State + @Bindable + @Environment
```

แทนระบบเดิม:

```text
ObservableObject + @StateObject + @ObservedObject + @EnvironmentObject
```

ระบบเดิมยังจำเป็นเมื่อแอปรองรับ OS รุ่นก่อน iOS 17 หรือใช้งานร่วมกับ API เดิม

---

## Stored Property หรือ Input

ถ้า View รับข้อมูลมาเพื่อแสดงผลและไม่ต้องเป็นเจ้าของ State ให้ใช้ Stored Property ธรรมดา

```swift
struct UserRow: View {
    let name: String
    let email: String

    var body: some View {
        VStack(alignment: .leading) {
            Text(name)
                .font(.headline)

            Text(email)
                .font(.subheadline)
                .foregroundStyle(.secondary)
        }
    }
}
```

เปรียบเทียบกับ React Props:

```tsx
type UserRowProps = {
  name: string
  email: string
}

function UserRow({ name, email }: UserRowProps) {
  return (
    <div>
      <strong>{name}</strong>
      <span>{email}</span>
    </div>
  )
}
```

ใช้ `let` เป็นค่าเริ่มต้น เพื่อแสดงให้เห็นว่า Child View ไม่ควรแก้ไขข้อมูลเอง

```swift
let title: String
```

ใช้ `var` เฉพาะเมื่อ API หรือการออกแบบจำเป็นต้องใช้

---

## @State

`@State` ใช้กับข้อมูลที่ View เป็นเจ้าของและต้องการให้ SwiftUI อัปเดต UI เมื่อข้อมูลเปลี่ยน

```swift
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""
    @State private var isPasswordVisible = false

    var body: some View {
        Form {
            TextField("Email", text: $email)

            if isPasswordVisible {
                TextField("Password", text: $password)
            } else {
                SecureField("Password", text: $password)
            }

            Toggle(
                "Show password",
                isOn: $isPasswordVisible
            )
        }
    }
}
```

### ควรประกาศเป็น private

```swift
@State private var isPresented = false
```

เพราะ State เป็น implementation detail ของ View และไม่ควรถูกแก้จากภายนอกโดยตรง

### เหมาะกับข้อมูลประเภทใด

- `Bool`
- `String`
- `Int`
- `Double`
- `Enum`
- `Struct`
- Collection ขนาดเหมาะสม
- Observable Model ที่ View เป็นเจ้าของในระบบ Observation

### ตัวอย่าง Enum State

```swift
enum SelectedTab {
    case home
    case products
    case settings
}

struct MainView: View {
    @State private var selectedTab: SelectedTab = .home

    var body: some View {
        TabView(selection: $selectedTab) {
            HomeView()
                .tag(SelectedTab.home)

            ProductListView()
                .tag(SelectedTab.products)

            SettingsView()
                .tag(SelectedTab.settings)
        }
    }
}
```

### การกำหนดค่าเริ่มต้นผ่าน init

```swift
struct CounterView: View {
    @State private var count: Int

    init(initialValue: Int = 0) {
        _count = State(initialValue: initialValue)
    }

    var body: some View {
        Text("Count: \(count)")
    }
}
```

ใช้เมื่อค่าเริ่มต้นมาจาก Parameter แต่หลังจาก View สร้าง State แล้ว State จะมี lifecycle ของตัวเอง

อย่าใช้รูปแบบนี้เพื่อพยายาม Sync State กับ Parameter ที่เปลี่ยนตลอดเวลา

---

## @Binding

`@Binding` ใช้เมื่อ Child View ต้องอ่านและแก้ไข State ที่ Parent เป็นเจ้าของ

### Parent View

```swift
struct SettingsView: View {
    @State private var notificationsEnabled = true

    var body: some View {
        NotificationToggle(
            isEnabled: $notificationsEnabled
        )
    }
}
```

### Child View

```swift
struct NotificationToggle: View {
    @Binding var isEnabled: Bool

    var body: some View {
        Toggle(
            "Enable notifications",
            isOn: $isEnabled
        )
    }
}
```

เครื่องหมาย `$` ใช้ส่ง Projected Value ซึ่งในกรณีนี้คือ `Binding<Bool>`

### Binding ไม่ได้เป็นเจ้าของข้อมูล

```text
Parent
  @State var value
       │
       └── $value
             │
             ▼
Child
  @Binding var value
```

### Binding ที่มี Custom Getter และ Setter

```swift
struct VolumeView: View {
    @State private var volume = 50

    private var normalizedVolume: Binding<Double> {
        Binding(
            get: {
                Double(volume) / 100
            },
            set: { newValue in
                volume = Int(newValue * 100)
            }
        )
    }

    var body: some View {
        Slider(value: normalizedVolume, in: 0...1)
    }
}
```

### Optional Binding

บางครั้ง Child View ต้องรับ Binding แบบ Optional

```swift
struct ProductEditor: View {
    @Binding var selectedProduct: Product?

    var body: some View {
        Button("Clear selection") {
            selectedProduct = nil
        }
    }
}
```

### อย่าส่ง Binding โดยไม่จำเป็น

ถ้า Child ต้องการเพียงแจ้ง Event กลับ Parent การใช้ Closure มักชัดเจนกว่า

```swift
struct ProductRow: View {
    let product: Product
    let onDelete: () -> Void

    var body: some View {
        HStack {
            Text(product.name)

            Spacer()

            Button(role: .destructive, action: onDelete) {
                Image(systemName: "trash")
            }
        }
    }
}
```

Binding เหมาะกับ Two-way Data Flow ส่วน Closure เหมาะกับ Event

---

## @Observable

`@Observable` ใช้สร้าง Reference Type ที่ SwiftUI สามารถติดตามการเปลี่ยนแปลงของ Property ได้

```swift
import Observation

@Observable
final class CounterStore {
    var count = 0

    func increase() {
        count += 1
    }

    func decrease() {
        count -= 1
    }
}
```

นำไปใช้ใน View:

```swift
struct CounterView: View {
    @State private var store = CounterStore()

    var body: some View {
        VStack {
            Text("Count: \(store.count)")

            HStack {
                Button("Decrease") {
                    store.decrease()
                }

                Button("Increase") {
                    store.increase()
                }
            }
        }
    }
}
```

### ทำไมใช้ @State กับ @Observable

`@State` ทำให้ SwiftUI ดูแล lifecycle ของ instance ที่ View เป็นเจ้าของ

```swift
@State private var store = CounterStore()
```

ส่วน `@Observable` ทำให้ SwiftUI ติดตาม Property ที่ `body` อ่านและอัปเดตเฉพาะ View ที่พึ่งพา Property นั้น

### ตัวอย่าง Feature Store

```swift
import Observation

@Observable
@MainActor
final class ProductListStore {
    private let repository: ProductRepository

    var products: [Product] = []
    var isLoading = false
    var errorMessage: String?

    init(repository: ProductRepository) {
        self.repository = repository
    }

    func loadProducts() async {
        guard !isLoading else {
            return
        }

        isLoading = true
        errorMessage = nil

        defer {
            isLoading = false
        }

        do {
            products = try await repository.fetchProducts()
        } catch {
            errorMessage = error.localizedDescription
        }
    }

    func retry() async {
        await loadProducts()
    }
}
```

View:

```swift
struct ProductListView: View {
    @State private var store: ProductListStore

    init(repository: ProductRepository) {
        _store = State(
            initialValue: ProductListStore(
                repository: repository
            )
        )
    }

    var body: some View {
        Group {
            if store.isLoading && store.products.isEmpty {
                ProgressView("Loading products...")
            } else if let errorMessage = store.errorMessage,
                      store.products.isEmpty {
                ContentUnavailableView(
                    "Unable to load products",
                    systemImage: "wifi.exclamationmark",
                    description: Text(errorMessage)
                )
            } else {
                List(store.products) { product in
                    ProductRow(product: product)
                }
            }
        }
        .task {
            await store.loadProducts()
        }
    }
}
```

### ใช้ @MainActor เมื่อ Store แก้ UI State

```swift
@Observable
@MainActor
final class ProfileStore {
    var profile: UserProfile?
    var isLoading = false
}
```

ทำให้การเปลี่ยน State ที่ UI ใช้งานเกิดบน Main Actor

### Property ที่ไม่ต้องการให้ Observation ติดตาม

```swift
import Observation

@Observable
final class SearchStore {
    var query = ""
    var results: [SearchResult] = []

    @ObservationIgnored
    private var currentTask: Task<Void, Never>?
}
```

ใช้กับ Dependency, Cache, Task หรือข้อมูลภายในที่ไม่ต้องทำให้ UI Refresh

---

## @Bindable

ใช้ `@Bindable` เมื่อ View ต้องสร้าง Binding ไปยัง Property ของ `@Observable`

Model:

```swift
@Observable
final class ProfileEditorModel {
    var displayName = ""
    var biography = ""
    var isPublic = true
}
```

View:

```swift
struct ProfileEditorView: View {
    @Bindable var model: ProfileEditorModel

    var body: some View {
        Form {
            TextField(
                "Display name",
                text: $model.displayName
            )

            TextField(
                "Biography",
                text: $model.biography,
                axis: .vertical
            )

            Toggle(
                "Public profile",
                isOn: $model.isPublic
            )
        }
    }
}
```

ถ้า View รับ Model มาเป็น Stored Property ธรรมดาและต้องใช้ `$model.property` ภายใน `body` สามารถสร้าง Local Bindable ได้

```swift
struct ProfileEditorView: View {
    let model: ProfileEditorModel

    var body: some View {
        @Bindable var model = model

        Form {
            TextField(
                "Display name",
                text: $model.displayName
            )

            Toggle(
                "Public profile",
                isOn: $model.isPublic
            )
        }
    }
}
```

### ความแตกต่างระหว่าง @Binding และ @Bindable

| Wrapper | ใช้กับ |
|---|---|
| `@Binding` | ค่า State หนึ่งค่า เช่น `String`, `Bool`, `Product` |
| `@Bindable` | Observable Model ที่ต้องสร้าง Binding ให้ Property ภายใน |

ตัวอย่าง:

```swift
@Binding var username: String
```

เทียบกับ:

```swift
@Bindable var model: ProfileEditorModel
```

---

## @Environment

`@Environment` ใช้รับค่า Dependency หรือ Shared State ที่ถูกส่งผ่าน View Hierarchy

เหมาะกับ:

- App Session
- Authentication State
- Theme
- Router
- API Client
- Repository
- Locale
- Dismiss Action
- Open URL Action
- Model Context

### ใช้กับค่าที่ SwiftUI มีให้

```swift
struct EditProfileView: View {
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        Button("Close") {
            dismiss()
        }
    }
}
```

### ส่ง Observable Model ผ่าน Environment

```swift
@Observable
@MainActor
final class SessionStore {
    var currentUser: User?
    var isAuthenticated: Bool {
        currentUser != nil
    }

    func signOut() {
        currentUser = nil
    }
}
```

App Root:

```swift
@main
struct ModernApp: App {
    @State private var session = SessionStore()

    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(session)
        }
    }
}
```

Child View:

```swift
struct AccountView: View {
    @Environment(SessionStore.self) private var session

    var body: some View {
        VStack {
            if let user = session.currentUser {
                Text(user.name)
            }

            Button("Sign out") {
                session.signOut()
            }
        }
    }
}
```

### Environment ไม่ควรถูกใช้แทน Parameter ทุกอย่าง

ไม่ควรนำข้อมูลเฉพาะ Row หรือเฉพาะ Screen ใส่ Environment

Environment เหมาะกับข้อมูลหรือ Dependency ที่หลาย View ใน Subtree ต้องใช้ และการส่งผ่าน Parameter หลายชั้นทำให้โค้ดซับซ้อนเกินไป

### Environment Key สำหรับ Service

```swift
protocol AnalyticsService {
    func track(event: AnalyticsEvent)
}

struct NoOpAnalyticsService: AnalyticsService {
    func track(event: AnalyticsEvent) {}
}
```

```swift
private struct AnalyticsServiceKey: EnvironmentKey {
    static let defaultValue: any AnalyticsService =
        NoOpAnalyticsService()
}

extension EnvironmentValues {
    var analyticsService: any AnalyticsService {
        get { self[AnalyticsServiceKey.self] }
        set { self[AnalyticsServiceKey.self] = newValue }
    }
}
```

ใช้งาน:

```swift
struct ProductDetailView: View {
    @Environment(\.analyticsService)
    private var analyticsService

    let product: Product

    var body: some View {
        ProductDetailContent(product: product)
            .onAppear {
                analyticsService.track(
                    event: .viewProduct(product.id)
                )
            }
    }
}
```

---

## State Management สำหรับระบบเก่า

ถ้า Deployment Target ต่ำกว่า iOS 17 หรือ macOS 14 ให้ใช้ `ObservableObject`

```swift
import Combine

@MainActor
final class ProductListViewModel: ObservableObject {
    @Published var products: [Product] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
}
```

### @StateObject

ใช้เมื่อ View เป็นผู้สร้างและเป็นเจ้าของ ObservableObject

```swift
struct ProductListView: View {
    @StateObject private var viewModel =
        ProductListViewModel()

    var body: some View {
        ProductListContent(viewModel: viewModel)
    }
}
```

### @ObservedObject

ใช้เมื่อ View รับ ObservableObject ที่สร้างจากภายนอก

```swift
struct ProductListContent: View {
    @ObservedObject var viewModel:
        ProductListViewModel

    var body: some View {
        List(viewModel.products) { product in
            ProductRow(product: product)
        }
    }
}
```

### @EnvironmentObject

Root:

```swift
@main
struct LegacyApp: App {
    @StateObject private var session =
        SessionViewModel()

    var body: some Scene {
        WindowGroup {
            RootView()
                .environmentObject(session)
        }
    }
}
```

Child:

```swift
struct AccountView: View {
    @EnvironmentObject private var session:
        SessionViewModel
}
```

### ตารางเทียบระบบใหม่และระบบเดิม

| Observation รุ่นใหม่ | ObservableObject รุ่นเดิม |
|---|---|
| `@Observable` | `ObservableObject` |
| Property ธรรมดา | `@Published` |
| `@State` | `@StateObject` |
| Stored Property / `@Bindable` | `@ObservedObject` |
| `.environment(model)` | `.environmentObject(model)` |
| `@Environment(Model.self)` | `@EnvironmentObject` |

อย่าผสม Wrapper โดยไม่มีเหตุผล เช่น ไม่ควรใช้ `@ObservedObject` กับ Class ที่ประกาศด้วย `@Observable`

---

## Loading, Error และ Empty State

Feature ที่โหลดข้อมูลควรแยกสถานะอย่างชัดเจน

### แบบหลาย Property

```swift
@Observable
@MainActor
final class ProductStore {
    var products: [Product] = []
    var isLoading = false
    var errorMessage: String?
}
```

เหมาะกับหน้าที่สามารถแสดงข้อมูลเดิมพร้อม Loading หรือ Error Banner ได้

### แบบ State Machine

```swift
enum LoadState<Value> {
    case idle
    case loading
    case loaded(Value)
    case empty
    case failed(message: String)
}
```

Store:

```swift
@Observable
@MainActor
final class ProductStore {
    private let repository: ProductRepository

    var state: LoadState<[Product]> = .idle

    init(repository: ProductRepository) {
        self.repository = repository
    }

    func load() async {
        state = .loading

        do {
            let products =
                try await repository.fetchProducts()

            state = products.isEmpty
                ? .empty
                : .loaded(products)
        } catch {
            state = .failed(
                message: error.localizedDescription
            )
        }
    }
}
```

View:

```swift
struct ProductListView: View {
    @State private var store: ProductStore

    var body: some View {
        content
            .task {
                await store.load()
            }
    }

    @ViewBuilder
    private var content: some View {
        switch store.state {
        case .idle:
            Color.clear

        case .loading:
            ProgressView("Loading products...")

        case .loaded(let products):
            List(products) { product in
                ProductRow(product: product)
            }

        case .empty:
            ContentUnavailableView(
                "No products",
                systemImage: "shippingbox"
            )

        case .failed(let message):
            ContentUnavailableView(
                "Unable to load products",
                systemImage: "exclamationmark.triangle",
                description: Text(message)
            )
        }
    }
}
```

### ป้องกัน Impossible State

หลาย Property อาจสร้างสถานะที่ขัดแย้งกันได้ เช่น:

```text
isLoading = true
products มีข้อมูล
errorMessage ไม่เป็น nil
```

บางหน้ารองรับสถานะแบบนี้ แต่บางหน้าไม่ควรเกิดขึ้น

ถ้าหน้ามีลำดับ State ที่ชัดเจน ให้ใช้ Enum State Machine

---

## Async/Await และ Task

### ใช้ .task สำหรับงานตาม Lifecycle ของ View

```swift
struct ProductListView: View {
    @State private var store: ProductStore

    var body: some View {
        ProductListContent(store: store)
            .task {
                await store.load()
            }
    }
}
```

`.task` จะเชื่อม Task เข้ากับ Lifecycle ของ View และสามารถถูก Cancel เมื่อ View หายไป

### ใช้ task(id:) เมื่อ Dependency เปลี่ยน

```swift
struct SearchResultView: View {
    let query: String

    @State private var store: SearchStore

    var body: some View {
        SearchResultContent(store: store)
            .task(id: query) {
                await store.search(query: query)
            }
    }
}
```

เมื่อ `query` เปลี่ยน Task เดิมจะถูกยกเลิกและเริ่ม Task ใหม่

### ตรวจสอบ Cancellation

```swift
func search(query: String) async {
    do {
        try await Task.sleep(
            for: .milliseconds(300)
        )

        try Task.checkCancellation()

        results = try await repository.search(
            query: query
        )
    } catch is CancellationError {
        return
    } catch {
        errorMessage = error.localizedDescription
    }
}
```

### Button กับ Async Function

```swift
Button("Save") {
    Task {
        await store.save()
    }
}
.disabled(store.isSaving)
```

### ป้องกันการกดซ้ำ

```swift
func save() async {
    guard !isSaving else {
        return
    }

    isSaving = true
    defer { isSaving = false }

    do {
        try await repository.save(draft)
        saveSucceeded = true
    } catch {
        errorMessage = error.localizedDescription
    }
}
```

---

## State Ownership

ก่อนเลือก Property Wrapper ให้ถามว่า:

```text
ใครเป็นเจ้าของข้อมูลนี้?
```

### View เป็นเจ้าของ

```swift
@State private var isSheetPresented = false
```

### Parent เป็นเจ้าของ Child แก้ไขได้

```swift
@Binding var isEnabled: Bool
```

### Feature Store เป็นเจ้าของ

```swift
@State private var store = ProductStore()
```

### App Root เป็นเจ้าของ

```swift
@State private var session = SessionStore()
```

และส่งลง Environment:

```swift
.environment(session)
```

### Server เป็นเจ้าของข้อมูลจริง

แม้ View หรือ Store จะเก็บข้อมูลชั่วคราว แต่ Source of Truth ระยะยาวอาจอยู่บน Server

ตัวอย่าง:

```text
API / Database
      ↓
Repository
      ↓
Feature Store
      ↓
SwiftUI View
```

Store ไม่ควรทำ Networking โดยตรงทุกกรณี ควรเรียกผ่าน Repository หรือ Service เพื่อให้ทดสอบได้ง่าย

---

## Derived State

Derived State คือค่าที่คำนวณจาก State อื่น

```swift
@Observable
final class CheckoutStore {
    var items: [CartItem] = []
    var discount = 0.0

    var subtotal: Double {
        items.reduce(0) {
            $0 + ($1.price * Double($1.quantity))
        }
    }

    var total: Double {
        max(0, subtotal - discount)
    }

    var canCheckout: Bool {
        !items.isEmpty && total > 0
    }
}
```

ไม่ควรเก็บ `subtotal`, `total` และ `canCheckout` เป็น Stored State ถ้าสามารถคำนวณได้อย่างรวดเร็ว

### เมื่อ Derived State มีค่าใช้จ่ายสูง

อาจคำนวณเมื่อข้อมูลต้นทางเปลี่ยน หรือสร้าง Cache ภายใน Model

อย่างไรก็ตามควรวัด Performance ก่อนเพิ่มความซับซ้อน

---

## Navigation State

Navigation ควรถูกขับเคลื่อนด้วย State เช่นเดียวกับ UI อื่น

```swift
enum AppRoute: Hashable {
    case productDetail(Product.ID)
    case profile
    case settings
}
```

```swift
@Observable
@MainActor
final class AppRouter {
    var path: [AppRoute] = []

    func showProduct(id: Product.ID) {
        path.append(.productDetail(id))
    }

    func showSettings() {
        path.append(.settings)
    }

    func popToRoot() {
        path.removeAll()
    }
}
```

Root View:

```swift
struct RootView: View {
    @State private var router = AppRouter()

    var body: some View {
        NavigationStack(path: $router.path) {
            HomeView()
                .navigationDestination(
                    for: AppRoute.self
                ) { route in
                    destination(for: route)
                }
        }
        .environment(router)
    }

    @ViewBuilder
    private func destination(
        for route: AppRoute
    ) -> some View {
        switch route {
        case .productDetail(let id):
            ProductDetailView(productID: id)

        case .profile:
            ProfileView()

        case .settings:
            SettingsView()
        }
    }
}
```

เนื่องจาก `router` เป็น `@Observable` และต้องสร้าง Binding ให้ `path` อาจใช้ Local `@Bindable` ในบางโครงสร้าง:

```swift
var body: some View {
    @Bindable var router = router

    NavigationStack(path: $router.path) {
        HomeView()
    }
}
```

### Sheet State

แบบ Bool:

```swift
@State private var isEditorPresented = false
```

เหมาะกับ Sheet แบบเดียวที่ไม่ต้องส่งข้อมูล

แบบ Item:

```swift
@State private var selectedProduct: Product?
```

```swift
.sheet(item: $selectedProduct) { product in
    ProductEditorView(product: product)
}
```

แบบ Item ช่วยรวมข้อมูลที่เลือกและสถานะแสดง Sheet เป็น Source of Truth เดียว

---

## Form State

สำหรับ Form ขนาดเล็ก สามารถใช้ `@State` หลายตัวได้

```swift
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""
}
```

สำหรับ Form ขนาดใหญ่ ควรรวม State ใน Model

```swift
@Observable
final class RegistrationForm {
    var firstName = ""
    var lastName = ""
    var email = ""
    var password = ""
    var acceptedTerms = false

    var fullName: String {
        "\(firstName) \(lastName)"
            .trimmingCharacters(
                in: .whitespaces
            )
    }

    var isEmailValid: Bool {
        email.contains("@")
    }

    var isPasswordValid: Bool {
        password.count >= 8
    }

    var canSubmit: Bool {
        !firstName.isEmpty &&
        !lastName.isEmpty &&
        isEmailValid &&
        isPasswordValid &&
        acceptedTerms
    }
}
```

View:

```swift
struct RegistrationView: View {
    @State private var form =
        RegistrationForm()

    var body: some View {
        @Bindable var form = form

        Form {
            TextField(
                "First name",
                text: $form.firstName
            )

            TextField(
                "Last name",
                text: $form.lastName
            )

            TextField(
                "Email",
                text: $form.email
            )
            .textInputAutocapitalization(.never)
            .keyboardType(.emailAddress)

            SecureField(
                "Password",
                text: $form.password
            )

            Toggle(
                "Accept terms",
                isOn: $form.acceptedTerms
            )

            Button("Create account") {
                // Submit form
            }
            .disabled(!form.canSubmit)
        }
    }
}
```

### Draft State กับ Saved State

อย่าแก้ Model จริงทันทีถ้าหน้ามี Cancel

```swift
struct ProfileDraft {
    var displayName: String
    var biography: String
}
```

เมื่อเปิด Editor ให้สร้าง Draft จาก Model จริง และ Save กลับเมื่อผู้ใช้ยืนยัน

---

## Dependency Injection

State Store ควรรับ Dependency ผ่าน Initializer

```swift
protocol ProductRepository {
    func fetchProducts() async throws -> [Product]
    func deleteProduct(id: Product.ID) async throws
}
```

```swift
@Observable
@MainActor
final class ProductStore {
    private let repository:
        any ProductRepository

    var products: [Product] = []

    init(repository: any ProductRepository) {
        self.repository = repository
    }
}
```

Production:

```swift
ProductListView(
    repository: APIProductRepository(
        client: URLSessionAPIClient()
    )
)
```

Preview:

```swift
#Preview {
    ProductListView(
        repository: PreviewProductRepository()
    )
}
```

Test:

```swift
let repository = MockProductRepository()
let store = ProductStore(repository: repository)
```

### อย่าสร้าง Dependency ภายใน Store โดยตรง

ไม่ควร:

```swift
final class ProductStore {
    private let repository =
        APIProductRepository()
}
```

เพราะเปลี่ยน Implementation และ Mock ได้ยาก

---

## Feature Store Pattern

ตัวอย่าง Feature ที่รวม State, Action และ Side Effect ไว้ใน Store

```swift
@Observable
@MainActor
final class ProductListStore {
    enum ViewState {
        case idle
        case loading
        case content
        case empty
        case error(String)
    }

    private let repository:
        any ProductRepository

    private(set) var products: [Product] = []
    private(set) var viewState: ViewState = .idle

    var searchText = ""
    var selectedProduct: Product?
    var productPendingDeletion: Product?

    init(repository: any ProductRepository) {
        self.repository = repository
    }

    var filteredProducts: [Product] {
        guard !searchText.isEmpty else {
            return products
        }

        return products.filter {
            $0.name.localizedCaseInsensitiveContains(
                searchText
            )
        }
    }

    func load() async {
        viewState = .loading

        do {
            products =
                try await repository.fetchProducts()

            viewState = products.isEmpty
                ? .empty
                : .content
        } catch {
            viewState = .error(
                error.localizedDescription
            )
        }
    }

    func requestDelete(_ product: Product) {
        productPendingDeletion = product
    }

    func cancelDelete() {
        productPendingDeletion = nil
    }

    func confirmDelete() async {
        guard let product =
                productPendingDeletion else {
            return
        }

        do {
            try await repository.deleteProduct(
                id: product.id
            )

            products.removeAll {
                $0.id == product.id
            }

            productPendingDeletion = nil

            if products.isEmpty {
                viewState = .empty
            }
        } catch {
            viewState = .error(
                error.localizedDescription
            )
        }
    }
}
```

View:

```swift
struct ProductListView: View {
    @State private var store:
        ProductListStore

    init(repository: any ProductRepository) {
        _store = State(
            initialValue: ProductListStore(
                repository: repository
            )
        )
    }

    var body: some View {
        @Bindable var store = store

        NavigationStack {
            content
                .navigationTitle("Products")
                .searchable(
                    text: $store.searchText
                )
                .task {
                    await store.load()
                }
                .sheet(
                    item: $store.selectedProduct
                ) { product in
                    ProductDetailView(
                        product: product
                    )
                }
                .alert(
                    "Delete product?",
                    isPresented: deleteAlertBinding,
                    presenting:
                        store.productPendingDeletion
                ) { product in
                    Button(
                        "Delete",
                        role: .destructive
                    ) {
                        Task {
                            await store.confirmDelete()
                        }
                    }

                    Button(
                        "Cancel",
                        role: .cancel
                    ) {
                        store.cancelDelete()
                    }
                } message: { product in
                    Text(
                        "Delete \(product.name)?"
                    )
                }
        }
    }

    private var deleteAlertBinding:
        Binding<Bool> {
        Binding(
            get: {
                store.productPendingDeletion != nil
            },
            set: { isPresented in
                if !isPresented {
                    store.cancelDelete()
                }
            }
        )
    }

    @ViewBuilder
    private var content: some View {
        switch store.viewState {
        case .idle:
            Color.clear

        case .loading:
            ProgressView("Loading products...")

        case .content:
            List(store.filteredProducts) {
                product in

                Button {
                    store.selectedProduct = product
                } label: {
                    ProductRow(product: product)
                }
                .swipeActions {
                    Button(
                        "Delete",
                        role: .destructive
                    ) {
                        store.requestDelete(product)
                    }
                }
            }

        case .empty:
            ContentUnavailableView(
                "No products",
                systemImage: "shippingbox"
            )

        case .error(let message):
            ContentUnavailableView(
                "Unable to load products",
                systemImage:
                    "exclamationmark.triangle",
                description: Text(message)
            )
        }
    }
}
```

---

## การทดสอบ State

การแยก State และ Logic ออกจาก View ทำให้ Unit Test ง่ายขึ้น

```swift
import Testing

@MainActor
struct ProductListStoreTests {
    @Test
    func loadProductsSuccess() async throws {
        let repository =
            MockProductRepository(
                products: [
                    Product(
                        id: UUID(),
                        name: "Product A"
                    )
                ]
            )

        let store = ProductListStore(
            repository: repository
        )

        await store.load()

        #expect(store.products.count == 1)
    }
}
```

Mock Repository:

```swift
struct MockProductRepository:
    ProductRepository {

    var products: [Product] = []
    var error: Error?

    func fetchProducts() async throws
        -> [Product] {

        if let error {
            throw error
        }

        return products
    }

    func deleteProduct(
        id: Product.ID
    ) async throws {}
}
```

### สิ่งที่ควรทดสอบ

- Initial State
- Loading State
- Success State
- Empty State
- Error State
- Validation
- Derived State
- Action ที่เปลี่ยน State
- Cancellation
- การป้องกัน Request ซ้ำ
- Navigation State

ไม่จำเป็นต้องทดสอบ Property Wrapper เอง แต่ควรทดสอบ Behavior ของ Store

---

## Anti-patterns

### 1. คัดลอก Props ไปเป็น State โดยไม่จำเป็น

ไม่ควร:

```swift
struct UserView: View {
    let user: User

    @State private var localUser: User

    init(user: User) {
        self.user = user
        _localUser = State(initialValue: user)
    }
}
```

เพราะเมื่อ `user` จาก Parent เปลี่ยน `localUser` อาจไม่เปลี่ยนตาม

ใช้ `let user` โดยตรง หรือใช้ Draft Model ถ้าต้องแก้ไขแบบ Save/Cancel

### 2. สร้าง Store ใน body

ไม่ควร:

```swift
var body: some View {
    let store = ProductStore(
        repository: repository
    )

    ProductListContent(store: store)
}
```

เพราะ `body` สามารถถูกประเมินซ้ำและทำให้สร้าง instance ใหม่

ควรเก็บใน `@State` หรือให้ Parent สร้างและส่งเข้ามา

### 3. ใช้ Environment กับทุก Dependency

ทำให้ Dependency ไม่ชัดเจนและตามหา Source ยาก

ใช้ Initializer Injection เป็นค่าเริ่มต้น และใช้ Environment กับ Shared Dependency ที่เหมาะสม

### 4. เก็บ Derived State ซ้ำ

ไม่ควรเก็บ `isFormValid` แล้วพยายามอัปเดตทุกครั้งที่ Field เปลี่ยน

ควรสร้าง Computed Property

### 5. Boolean จำนวนมากแทน State Machine

ไม่ควร:

```swift
var isLoading = false
var isEmpty = false
var hasError = false
var isLoaded = false
```

ควร:

```swift
enum ViewState {
    case idle
    case loading
    case content
    case empty
    case error(String)
}
```

### 6. Business Logic อยู่ใน View มากเกินไป

ไม่ควร:

```swift
Button("Checkout") {
    let subtotal = items.reduce(0) { ... }
    let tax = subtotal * 0.07
    let total = subtotal + tax
    // Validate, call API, save, navigate...
}
```

ย้าย Logic ไป Store, Use Case หรือ Domain Model

### 7. ใช้ onAppear สำหรับทุก Async Work

`onAppear` อาจถูกเรียกหลายครั้งและไม่มี Structured Cancellation แบบ `.task`

ควรใช้:

```swift
.task {
    await store.load()
}
```

### 8. ส่ง Binding ลึกเกินไป

ถ้า Binding ถูกส่งผ่านหลายชั้นและหลาย Component อาจเป็นสัญญาณว่าควรใช้ Feature Store หรือ Environment

### 9. View Model ขนาดใหญ่มาก

View Model ที่ดูแลหลาย Screen, Networking, Cache, Navigation และ Analytics พร้อมกันจะทดสอบและแก้ไขยาก

แยกตาม Feature และ Responsibility

### 10. เปิดสิทธิ์แก้ State ทั้งหมด

ใช้ `private(set)` เมื่อภายนอกควรอ่านได้แต่แก้ผ่าน Action เท่านั้น

```swift
private(set) var products: [Product] = []
```

ช่วยควบคุม State Transition

---

## Decision Guide

### ใช้ Stored Property เมื่อ

```text
View รับค่าเพื่อแสดงผลอย่างเดียว
```

```swift
let product: Product
```

### ใช้ @State เมื่อ

```text
View เป็นเจ้าของ Local State
```

```swift
@State private var isPresented = false
```

หรือ View เป็นเจ้าของ Observable Store

```swift
@State private var store = ProductStore()
```

### ใช้ @Binding เมื่อ

```text
Parent เป็นเจ้าของค่า
Child ต้องอ่านและแก้ไข
```

```swift
@Binding var text: String
```

### ใช้ @Observable เมื่อ

```text
มี State และ Logic หลายค่า
ต้องการ Reference Semantics
ต้องการแชร์ Instance ระหว่างหลาย View
```

```swift
@Observable
final class ProductStore {}
```

### ใช้ @Bindable เมื่อ

```text
ต้องสร้าง Binding ไปยัง Property ภายใน @Observable
```

```swift
@Bindable var store: ProductStore
```

### ใช้ @Environment เมื่อ

```text
Dependency หรือ Shared State ถูกใช้งานหลายระดับใน View Tree
```

```swift
@Environment(SessionStore.self)
private var session
```

### ใช้ ObservableObject เมื่อ

```text
รองรับ OS ต่ำกว่า iOS 17 / macOS 14
หรือทำงานร่วมกับระบบ Combine เดิม
```

---

## Project Structure

โครงสร้างที่แนะนำสำหรับ State Management แบบ Feature-first:

```text
MyApp/
├── App/
│   ├── MyApp.swift
│   ├── AppDependencies.swift
│   ├── AppRouter.swift
│   └── SessionStore.swift
│
├── Core/
│   ├── Networking/
│   ├── Persistence/
│   ├── Analytics/
│   └── Common/
│
├── Features/
│   ├── Products/
│   │   ├── Models/
│   │   │   └── Product.swift
│   │   │
│   │   ├── Views/
│   │   │   ├── ProductListView.swift
│   │   │   └── ProductDetailView.swift
│   │   │
│   │   ├── Components/
│   │   │   └── ProductRow.swift
│   │   │
│   │   ├── State/
│   │   │   └── ProductListStore.swift
│   │   │
│   │   ├── Repositories/
│   │   │   ├── ProductRepository.swift
│   │   │   └── APIProductRepository.swift
│   │   │
│   │   └── Tests/
│   │       └── ProductListStoreTests.swift
│   │
│   └── Authentication/
│       ├── Models/
│       ├── Views/
│       ├── State/
│       ├── Repositories/
│       └── Tests/
│
└── Shared/
    ├── Models/
    ├── Components/
    └── State/
```

ใช้ชื่อ Folder เป็น `State`, `Stores` หรือ `ViewModels` ก็ได้ แต่ควรใช้ให้สม่ำเสมอทั้งโปรเจกต์

สำหรับโปรเจกต์ใหม่ที่ใช้ Observation และ Store Pattern ชื่อ `State` หรือ `Stores` มักสื่อความหมายได้ตรงกว่า `ViewModels`

---

## Checklist

ก่อนสร้าง State ใหม่ ให้ตรวจสอบ:

- [ ] ข้อมูลนี้จำเป็นต้องเป็น State หรือคำนวณได้
- [ ] มี Source of Truth เพียงจุดเดียว
- [ ] State อยู่ใกล้ View ที่ใช้งานมากที่สุด
- [ ] View ที่เป็นเจ้าของ State ใช้ `@State`
- [ ] Child ที่แก้ State ของ Parent ใช้ `@Binding`
- [ ] Observable Model ใช้ `@Observable`
- [ ] Binding ของ Observable Property ใช้ `@Bindable`
- [ ] Shared State หรือ Dependency ที่เหมาะสมใช้ Environment
- [ ] Async UI State เปลี่ยนบน Main Actor
- [ ] Loading, Empty และ Error State แยกชัดเจน
- [ ] ไม่มี Derived State ที่เก็บซ้ำโดยไม่จำเป็น
- [ ] Store รับ Dependency ผ่าน Initializer
- [ ] Store สามารถทดสอบแยกจาก View
- [ ] ไม่มี Networking หรือ Business Logic จำนวนมากใน View
- [ ] Task รองรับ Cancellation
- [ ] ป้องกันการกดหรือโหลดซ้ำเมื่อจำเป็น
- [ ] Navigation ถูกขับเคลื่อนด้วย State
- [ ] ใช้ `private(set)` กับ State ที่ควรถูกแก้ผ่าน Action เท่านั้น

---

## สรุป

แนวทาง State Management ที่แนะนำสำหรับ SwiftUI สมัยใหม่คือ:

```text
Local UI State
    → @State

Parent-owned Editable State
    → @Binding

Feature State และ Business Logic
    → @Observable Store

Binding ไปยัง Observable Property
    → @Bindable

Shared State และ Shared Dependency
    → @Environment

รองรับ OS รุ่นเก่า
    → ObservableObject
      + @StateObject
      + @ObservedObject
      + @EnvironmentObject
```

หัวใจสำคัญไม่ใช่การเลือก Property Wrapper ที่ซับซ้อนที่สุด แต่คือ:

1. กำหนดเจ้าของข้อมูลให้ชัดเจน
2. รักษา Single Source of Truth
3. ให้ UI เป็นผลลัพธ์ของ State
4. แยก Side Effect และ Business Logic ออกจาก View
5. ออกแบบ State Transition ให้เข้าใจและทดสอบได้

---

## References

- [SwiftUI: State](https://developer.apple.com/documentation/swiftui/state)
- [SwiftUI: Binding](https://developer.apple.com/documentation/swiftui/binding)
- [Managing model data in your app](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app)
- [Migrating from ObservableObject to Observable](https://developer.apple.com/documentation/swiftui/migrating-from-the-observable-object-protocol-to-the-observable-macro)
- [Observation Framework](https://developer.apple.com/documentation/observation)
- [Driving changes in your UI with state and bindings](https://developer.apple.com/tutorials/swiftui-concepts/driving-changes-in-your-ui-with-state-and-bindings)
- [Monitoring model data changes in your app](https://developer.apple.com/documentation/swiftui/monitoring-model-data-changes-in-your-app)
