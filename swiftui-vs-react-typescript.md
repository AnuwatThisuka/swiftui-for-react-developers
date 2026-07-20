# หลักการเขียน SwiftUI เทียบกับ React + TypeScript

> คู่มือนี้อธิบายแนวคิดของ SwiftUI สำหรับนักพัฒนาที่คุ้นเคยกับ React และ TypeScript พร้อมโครงสร้างโฟลเดอร์ที่แนะนำสำหรับโปรเจกต์ SwiftUI สมัยใหม่
>
> แนวทางหลักในเอกสารอิง SwiftUI แบบ Observation (`@Observable`) สำหรับ iOS 17+, iPadOS 17+, macOS 14+, watchOS 10+ และ tvOS 17+ หากต้องรองรับระบบเก่ากว่านี้ ให้ใช้ `ObservableObject`, `@StateObject` และ `@Published` ตามความเหมาะสม

---

## 1. ภาพรวมแนวคิด

SwiftUI และ React มีแนวคิดพื้นฐานคล้ายกันมาก คือเป็น **Declarative UI**

เราไม่ได้สั่ง UI ทีละขั้นว่า:

1. สร้างปุ่ม
2. เปลี่ยนข้อความ
3. ซ่อน Loading
4. แสดงผลลัพธ์

แต่จะเขียนว่า:

> เมื่อข้อมูลอยู่ในสถานะนี้ หน้าจอควรมีหน้าตาอย่างไร

เมื่อ State เปลี่ยน Framework จะคำนวณและอัปเดต UI ที่เกี่ยวข้องให้เอง

### React + TypeScript

```tsx
function Counter() {
  const [count, setCount] = useState<number>(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

### SwiftUI

```swift
import SwiftUI

struct CounterView: View {
    @State private var count = 0

    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
```

แนวคิดที่ตรงกันคือ:

| React + TypeScript | SwiftUI | ความหมาย |
|---|---|---|
| Functional Component | `struct` ที่ conform `View` | หน่วยประกอบ UI |
| JSX / TSX | `body` และ View Builder | ประกาศโครงสร้าง UI |
| Props | Stored properties ของ View | ข้อมูลจาก Parent |
| `useState` | `@State` | State ภายใน Component/View |
| Controlled props | `@Binding` | รับค่าและแก้ค่าของ Parent |
| Context | `@Environment` | Dependency หรือค่าที่แชร์ใน View tree |
| Context Provider | `.environment(...)` | ส่งค่าเข้า Environment |
| Custom Hook | Service, Store, ViewModel หรือ View Modifier | แยก Logic ที่นำกลับมาใช้ซ้ำ |
| `useEffect` | `.task`, `.onAppear`, `.onChange` | Side effect และการทำงานตาม Lifecycle |
| React Router | `NavigationStack` และ `NavigationPath` | Navigation |
| CSS / CSS Modules | View Modifier | Styling และ Layout |
| `map()` | `ForEach` | Render รายการ |
| `key` | `Identifiable` หรือ `id:` | Identity ของรายการ |

---

## 2. SwiftUI View เทียบกับ React Component

## 2.1 Component และ View ควรเป็นฟังก์ชันของ State

ใน React เรามักคิดว่า:

```text
UI = f(props, state)
```

ใน SwiftUI ก็คิดแบบเดียวกัน:

```text
View = f(input properties, state, environment)
```

SwiftUI View ส่วนใหญ่เป็น `struct` ซึ่งมีน้ำหนักเบา และ Framework สามารถสร้างค่า View ใหม่ได้บ่อย ดังนั้นไม่ควรคิดว่า View instance คือ Object ที่อยู่คงเดิมตลอดอายุหน้าจอ

```swift
struct ProfileHeaderView: View {
    let name: String
    let role: String

    var body: some View {
        VStack(alignment: .leading) {
            Text(name)
                .font(.title2.bold())

            Text(role)
                .foregroundStyle(.secondary)
        }
    }
}
```

เทียบกับ React:

```tsx
type ProfileHeaderProps = {
  name: string
  role: string
}

function ProfileHeader({ name, role }: ProfileHeaderProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{role}</p>
    </div>
  )
}
```

### หลักการสำคัญ

- View ควรเล็กและอ่านง่าย
- View ควรประกอบจาก View ย่อย
- ไม่ควรใส่ Network, Database หรือ Business Logic จำนวนมากไว้ใน `body`
- `body` ควรอธิบายหน้าตาของ UI มากกว่าขั้นตอนการทำงาน
- Computed property ที่สร้าง UI ควรแยกเมื่อช่วยให้อ่านง่าย แต่ View ที่ซับซ้อนควรแยกเป็น `struct` ใหม่

---

## 3. Props ใน React เทียบกับ Property ใน SwiftUI

React รับข้อมูลจาก Parent ผ่าน Props:

```tsx
function UserRow({ name }: { name: string }) {
  return <span>{name}</span>
}
```

SwiftUI รับผ่าน Property ธรรมดา:

```swift
struct UserRow: View {
    let name: String

    var body: some View {
        Text(name)
    }
}
```

เมื่อ Property เป็น `let` หมายถึง View ลูกอ่านได้อย่างเดียว คล้าย Props ปกติของ React

```swift
UserRow(name: "Anuwat")
```

### อย่าแก้ Input โดยตรง

React Props เป็น read-only ในเชิงแนวคิด ส่วน SwiftUI ก็ควรถือว่า Property ที่ Parent ส่งมาเป็น Input ที่ View ลูกไม่ควรแก้เอง

หาก View ลูกต้องแก้ State ของ Parent ให้ใช้ `@Binding`

---

## 4. `useState` เทียบกับ `@State`

## React

```tsx
const [isEnabled, setIsEnabled] = useState(false)
```

## SwiftUI

```swift
@State private var isEnabled = false
```

ตัวอย่าง:

```swift
struct SettingsView: View {
    @State private var isEnabled = false

    var body: some View {
        Form {
            Toggle("Enable notification", isOn: $isEnabled)

            Text(isEnabled ? "Enabled" : "Disabled")
        }
    }
}
```

เครื่องหมาย `$isEnabled` คือการส่ง **Binding** แทนการส่งค่าธรรมดา

### ใช้ `@State` เมื่อใด

ใช้กับ State ที่:

- View นี้เป็นเจ้าของ
- เป็น UI State ชั่วคราว
- ไม่จำเป็นต้องแชร์ทั่วทั้งแอป
- เช่น การเปิด Sheet, Tab ที่เลือก, ข้อความในช่องค้นหา, Loading เฉพาะหน้า

```swift
@State private var searchText = ""
@State private var isShowingSheet = false
@State private var selectedTab: Tab = .home
```

### ไม่ควรใช้ `@State` กับอะไร

- API Client
- Database Repository
- Object ที่มี Side Effect หนักใน initializer
- State ระดับแอปที่ต้องใช้หลาย Feature
- ข้อมูลที่ควรมี Source of Truth อยู่ใน Store หรือ Model ชั้นบน

---

## 5. Controlled Component เทียบกับ `@Binding`

ใน React Controlled Component มักรับทั้งค่าและ callback:

```tsx
type SearchBoxProps = {
  value: string
  onChange: (value: string) => void
}
```

SwiftUI รวมแนวคิดนี้ไว้ใน `Binding`

```swift
struct SearchBox: View {
    @Binding var text: String

    var body: some View {
        TextField("Search", text: $text)
            .textFieldStyle(.roundedBorder)
    }
}
```

Parent:

```swift
struct ProductListView: View {
    @State private var searchText = ""

    var body: some View {
        VStack {
            SearchBox(text: $searchText)
            Text("Searching: \(searchText)")
        }
    }
}
```

### หลักการ

- `@State` = เจ้าของข้อมูล
- `@Binding` = ยืมสิทธิ์อ่านและแก้ข้อมูลจากเจ้าของ
- อย่าสร้างสำเนา State ซ้ำโดยไม่จำเป็น
- เก็บ Source of Truth ไว้จุดเดียว

---

## 6. Object State สมัยใหม่ด้วย `@Observable`

สำหรับ Deployment Target รุ่นใหม่ Apple แนะนำ Observation framework และ `@Observable`

```swift
import Observation

@Observable
final class LoginStore {
    var email = ""
    var password = ""
    var isLoading = false
    var errorMessage: String?

    var canSubmit: Bool {
        !email.isEmpty && !password.isEmpty && !isLoading
    }
}
```

นำมาใช้ใน View:

```swift
import SwiftUI

struct LoginView: View {
    @State private var store = LoginStore()

    var body: some View {
        Form {
            TextField("Email", text: $store.email)
                .textInputAutocapitalization(.never)

            SecureField("Password", text: $store.password)

            Button("Sign in") {
                Task {
                    await signIn()
                }
            }
            .disabled(!store.canSubmit)
        }
    }

    private func signIn() async {
        store.isLoading = true
        defer { store.isLoading = false }

        // เรียก Service ที่นี่ หรือส่ง Action ไปยัง Store
    }
}
```

ในบางกรณีที่ต้องสร้าง Binding จาก Observable model แบบชัดเจน สามารถใช้ `@Bindable`

```swift
struct LoginForm: View {
    @Bindable var store: LoginStore

    var body: some View {
        TextField("Email", text: $store.email)
    }
}
```

### เทียบกับ React

| React | SwiftUI Observation |
|---|---|
| `useState` หลายตัว | Properties ภายใน `@Observable` Store |
| Reducer/Store | `@Observable` class หรือ Actor-backed Store |
| Context Store | Observable Store ผ่าน Environment |
| Selector | View อ่านเฉพาะ Property ที่ต้องใช้ |
| State setter | แก้ Property หรือเรียก Method/Action |

### รองรับ iOS ก่อน 17

โค้ดแบบเดิม:

```swift
final class LoginViewModel: ObservableObject {
    @Published var email = ""
    @Published var isLoading = false
}

struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()

    var body: some View {
        TextField("Email", text: $viewModel.email)
    }
}
```

ใช้แบบเดิมเมื่อ Deployment Target ต่ำกว่า iOS 17 หรือเมื่อต้องทำงานร่วมกับ API เก่าที่ต้องการ `ObservableObject`

---

## 7. Context เทียบกับ Environment

React ใช้ Context เพื่อหลีกเลี่ยง Prop Drilling:

```tsx
<AuthContext.Provider value={authStore}>
  <App />
</AuthContext.Provider>
```

SwiftUI ใช้ Environment:

```swift
@main
struct ExampleApp: App {
    @State private var session = SessionStore()

    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(session)
        }
    }
}
```

อ่านจาก View ลูก:

```swift
struct ProfileView: View {
    @Environment(SessionStore.self) private var session

    var body: some View {
        Text(session.currentUser?.name ?? "Guest")
    }
}
```

### ใช้ Environment กับอะไร

เหมาะกับ Dependency ที่หลายหน้าต้องใช้ เช่น:

- Session / Authentication
- App Router
- App Settings
- Theme
- Analytics interface
- Dependency Container

### สิ่งที่ควรระวัง

อย่าโยนทุกอย่างเข้า Environment เพราะจะทำให้ Dependency ของ View ไม่ชัดเจน

แนวทางที่ดีคือ:

- Dependency ที่ใช้เฉพาะ Feature ให้ส่งผ่าน initializer
- Dependency ระดับแอปจึงค่อยใช้ Environment
- Protocol-based Service ช่วยให้ Preview และ Unit Test ง่ายขึ้น

---

## 8. Event Handler

React:

```tsx
<button onClick={handleSave}>Save</button>
```

SwiftUI:

```swift
Button("Save", action: handleSave)
```

หรือ:

```swift
Button("Save") {
    handleSave()
}
```

ตัวอย่างส่ง Action จาก Child ไป Parent:

```swift
struct ProductRow: View {
    let product: Product
    let onDelete: () -> Void

    var body: some View {
        HStack {
            Text(product.name)
            Spacer()
            Button("Delete", role: .destructive, action: onDelete)
        }
    }
}
```

คล้าย Callback Props ใน React

```tsx
type ProductRowProps = {
  product: Product
  onDelete: () => void
}
```

### หลักการ

- View ย่อยควรส่ง “เหตุการณ์” ออกไป มากกว่ารู้วิธีทำงานทั้งระบบ
- ชื่อ callback ควรสื่อเจตนา เช่น `onSave`, `onRetry`, `onProductSelected`
- Parent หรือ Store เป็นผู้ตัดสินใจว่าจะทำอะไรต่อ

---

## 9. Conditional Rendering

React:

```tsx
return isLoading ? <Spinner /> : <Content />
```

SwiftUI:

```swift
if isLoading {
    ProgressView()
} else {
    ContentView()
}
```

ตัวอย่างหลายสถานะ:

```swift
switch state {
case .idle:
    EmptyView()

case .loading:
    ProgressView("Loading...")

case .loaded(let products):
    ProductList(products: products)

case .failed(let message):
    ErrorView(message: message)
}
```

แนะนำให้สร้าง State เป็น `enum` เมื่อหน้าจอมีสถานะที่ไม่ควรเกิดพร้อมกัน

```swift
enum Loadable<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(String)
}
```

ดีกว่าการมี Boolean หลายตัว เช่น:

```swift
var isLoading: Bool
var isError: Bool
var isSuccess: Bool
```

เพราะ Boolean หลายตัวอาจสร้าง State ที่ขัดแย้งกัน เช่น Loading และ Success เป็น `true` พร้อมกัน

---

## 10. Render List: `map` เทียบกับ `ForEach`

React:

```tsx
{products.map(product => (
  <ProductRow key={product.id} product={product} />
))}
```

SwiftUI:

```swift
List(products) { product in
    ProductRow(product: product)
}
```

Model:

```swift
struct Product: Identifiable, Codable, Hashable {
    let id: UUID
    let name: String
    let price: Decimal
}
```

หรือ:

```swift
ForEach(products, id: \.id) { product in
    ProductRow(product: product)
}
```

### Identity สำคัญมาก

อย่าใช้ Index เป็น ID หากรายการสามารถเพิ่ม ลบ หรือเรียงใหม่ได้

ไม่แนะนำ:

```swift
ForEach(products.indices, id: \.self) { index in
    ProductRow(product: products[index])
}
```

แนะนำ:

```swift
ForEach(products) { product in
    ProductRow(product: product)
}
```

Identity ที่เสถียรช่วยให้ Animation, State ของแถว และ Diffing ทำงานถูกต้อง

---

## 11. CSS เทียบกับ View Modifier

React:

```tsx
<button className="primary-button">Save</button>
```

SwiftUI:

```swift
Button("Save") {
    save()
}
.buttonStyle(.borderedProminent)
.controlSize(.large)
```

View Modifier ทำงานแบบ Chain:

```swift
Text("Dashboard")
    .font(.largeTitle.bold())
    .foregroundStyle(.primary)
    .frame(maxWidth: .infinity, alignment: .leading)
    .padding()
    .background(.thinMaterial)
    .clipShape(RoundedRectangle(cornerRadius: 16))
```

### ลำดับ Modifier มีผล

```swift
Text("Hello")
    .padding()
    .background(.blue)
```

ต่างจาก:

```swift
Text("Hello")
    .background(.blue)
    .padding()
```

เพราะ Modifier แต่ละตัวสร้าง View ใหม่ครอบ View เดิม คล้ายการซ้อน Wrapper

### สร้าง Style ที่นำกลับมาใช้ซ้ำ

```swift
struct CardModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding(16)
            .background(.background)
            .clipShape(RoundedRectangle(cornerRadius: 16))
            .shadow(radius: 4, y: 2)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardModifier())
    }
}
```

ใช้งาน:

```swift
VStack {
    Text("Machine Status")
    Text("Running")
}
.cardStyle()
```

แนวคิดใกล้เคียง CSS class, styled component หรือ reusable style function

---

## 12. `useEffect` เทียบกับ `.task`, `.onAppear`, `.onChange`

React `useEffect` ใช้เพื่อ Synchronize กับระบบภายนอก ไม่ควรใช้เพื่อคำนวณค่าที่หาได้จาก Props หรือ State โดยตรง

SwiftUI มีหลาย API ที่ควรเลือกตามงาน

## 12.1 โหลดข้อมูลแบบ Async ด้วย `.task`

```swift
struct ProductListView: View {
    @State private var store: ProductListStore

    init(service: ProductServiceProtocol) {
        _store = State(initialValue: ProductListStore(service: service))
    }

    var body: some View {
        ProductListContent(state: store.state)
            .task {
                await store.loadProducts()
            }
    }
}
```

ข้อดีของ `.task` คือ SwiftUI ผูก Task กับอายุของ View และสามารถยกเลิก Task เมื่อ View หายไป

## 12.2 ทำงานเมื่อค่าเปลี่ยนด้วย `.onChange`

```swift
.onChange(of: searchText) { _, newValue in
    store.search(newValue)
}
```

## 12.3 ทำงานเมื่อ View ปรากฏด้วย `.onAppear`

```swift
.onAppear {
    analytics.trackScreen("product_list")
}
```

### เลือกใช้แบบใด

| งาน | API ที่เหมาะสม |
|---|---|
| Async load เมื่อ View แสดง | `.task` |
| Async load ตาม ID หรือ Query | `.task(id:)` |
| ติดตามค่าที่เปลี่ยน | `.onChange(of:)` |
| Event เมื่อ View ปรากฏ | `.onAppear` |
| Event เมื่อ View หาย | `.onDisappear` |
| ผู้ใช้กดปุ่ม | `Button` action |

ตัวอย่าง `.task(id:)` สำหรับค้นหา:

```swift
.task(id: searchText) {
    guard !searchText.isEmpty else {
        store.clearSearch()
        return
    }

    try? await Task.sleep(for: .milliseconds(300))
    guard !Task.isCancelled else { return }

    await store.search(searchText)
}
```

ใช้แทน Debounced Effect ได้ในหลายกรณี

---

## 13. Derived State

React ไม่ควรใช้ Effect เพื่อเก็บ State ที่คำนวณได้จาก State อื่น

SwiftUI ก็เหมือนกัน

ไม่แนะนำ:

```swift
@State private var firstName = ""
@State private var lastName = ""
@State private var fullName = ""
```

แล้วคอย Sync `fullName`

แนะนำ:

```swift
@State private var firstName = ""
@State private var lastName = ""

private var fullName: String {
    "\(firstName) \(lastName)"
        .trimmingCharacters(in: .whitespaces)
}
```

### หลักการ

- เก็บเฉพาะ Source of Truth
- ค่าอื่นให้คำนวณเป็น Computed Property
- ลดโอกาสข้อมูลไม่ตรงกัน
- ลด Side Effect ที่ไม่จำเป็น

---

## 14. Navigation เทียบกับ React Router

SwiftUI สมัยใหม่ใช้ `NavigationStack`

```swift
struct AppRouterView: View {
    @State private var path: [Route] = []

    var body: some View {
        NavigationStack(path: $path) {
            HomeView(
                onOpenProduct: { productID in
                    path.append(.productDetail(productID))
                }
            )
            .navigationDestination(for: Route.self) { route in
                destination(for: route)
            }
        }
    }

    @ViewBuilder
    private func destination(for route: Route) -> some View {
        switch route {
        case .productDetail(let id):
            ProductDetailView(productID: id)

        case .settings:
            SettingsView()
        }
    }
}

enum Route: Hashable {
    case productDetail(UUID)
    case settings
}
```

### เทียบแนวคิด

| React Router | SwiftUI |
|---|---|
| Route definition | `Route` enum |
| URL/path state | `NavigationPath` หรือ Array ของ Route |
| Navigate | `path.append(...)` |
| Back | `path.removeLast()` หรือ Environment dismiss |
| Route component | `.navigationDestination(for:)` |
| Modal route | `.sheet`, `.fullScreenCover` |

### แนะนำ

- ใช้ `enum Route` เพื่อให้ Navigation type-safe
- อย่าให้ทุก View แก้ NavigationPath โดยตรงหากระบบซับซ้อน
- โปรเจกต์ใหญ่ควรมี Router หรือ Coordinator ระดับ App/Feature
- แยก Push Navigation ออกจาก Modal Presentation

---

## 15. Async/Await เทียบกับ Promise

React + TypeScript:

```tsx
async function loadProducts() {
  const response = await fetch('/api/products')
  return response.json()
}
```

Swift:

```swift
func loadProducts() async throws -> [Product] {
    let url = URL(string: "https://example.com/api/products")!
    let (data, response) = try await URLSession.shared.data(from: url)

    guard let httpResponse = response as? HTTPURLResponse,
          200..<300 ~= httpResponse.statusCode else {
        throw APIError.invalidResponse
    }

    return try JSONDecoder().decode([Product].self, from: data)
}
```

### Store ที่เรียก Service

```swift
import Observation

@MainActor
@Observable
final class ProductListStore {
    enum State {
        case idle
        case loading
        case loaded([Product])
        case failed(String)
    }

    private(set) var state: State = .idle

    private let service: ProductServiceProtocol

    init(service: ProductServiceProtocol) {
        self.service = service
    }

    func loadProducts() async {
        state = .loading

        do {
            let products = try await service.fetchProducts()
            state = .loaded(products)
        } catch is CancellationError {
            // Task ถูกยกเลิก ไม่จำเป็นต้องแสดง Error
        } catch {
            state = .failed(error.localizedDescription)
        }
    }
}
```

### ทำไมใช้ `@MainActor`

UI State ควรถูกแก้จาก Main Actor เพื่อป้องกัน Data Race และทำให้ Intent ของโค้ดชัดเจน

Service ที่ทำ Network ไม่จำเป็นต้องเป็น `@MainActor` เพราะ `URLSession` รองรับ Async และไม่ Block Main Thread

---

## 16. Service และ Dependency Injection

กำหนด Protocol:

```swift
protocol ProductServiceProtocol: Sendable {
    func fetchProducts() async throws -> [Product]
    func fetchProduct(id: UUID) async throws -> Product
}
```

Implementation จริง:

```swift
struct ProductService: ProductServiceProtocol {
    let apiClient: APIClient

    func fetchProducts() async throws -> [Product] {
        try await apiClient.send(.products)
    }

    func fetchProduct(id: UUID) async throws -> Product {
        try await apiClient.send(.product(id: id))
    }
}
```

Mock สำหรับ Preview/Test:

```swift
struct MockProductService: ProductServiceProtocol {
    var products: [Product] = Product.samples

    func fetchProducts() async throws -> [Product] {
        products
    }

    func fetchProduct(id: UUID) async throws -> Product {
        guard let product = products.first(where: { $0.id == id }) else {
            throw APIError.notFound
        }

        return product
    }
}
```

### เทียบกับ React

| React / TypeScript | SwiftUI / Swift |
|---|---|
| Interface/type contract | `protocol` |
| API module | Service |
| Dependency provider | App Container / Environment |
| Mock API | Mock Service conform Protocol |
| Hook using API | Store/ViewModel using Service |

### หลักการ

- View ไม่ควรรู้รายละเอียด URLSession
- Store ไม่ควรรู้ URL string โดยตรง
- Service ทำหน้าที่เชื่อม Domain กับ Data Source
- Protocol ทำให้ Test และ Preview ง่าย
- Dependency ควรถูกสร้างจาก Composition Root เช่น `App` หรือ Feature Factory

---

## 17. Architecture ที่แนะนำ

ไม่มี Architecture เดียวที่เหมาะกับทุกแอป แต่สำหรับ SwiftUI สมัยใหม่ แนะนำแนวทาง:

> Feature-first + unidirectional data flow + protocol-based dependencies

โครงสร้างภายในแต่ละ Feature อาจใช้รูปแบบคล้าย MVVM แต่ไม่จำเป็นต้องสร้าง ViewModel ให้ทุก View

### หน้าที่แต่ละส่วน

- **View**: แสดง UI และส่ง User Action
- **Store/ViewModel**: เก็บ Presentation State และประสาน Use Case
- **Domain Model**: นิยามข้อมูลและกฎทางธุรกิจ
- **Service/Repository**: ติดต่อ API, Database หรือระบบภายนอก
- **Router**: จัดการ Navigation เมื่อ Flow ซับซ้อน

### ไม่จำเป็นต้องมี ViewModel ทุกหน้า

View ที่เรียบง่ายสามารถใช้ `@State` ได้โดยตรง

```swift
struct QuantityPicker: View {
    @State private var quantity = 1

    var body: some View {
        Stepper("Quantity: \(quantity)", value: $quantity, in: 1...99)
    }
}
```

สร้าง Store/ViewModel เมื่อมี:

- Async workflow
- Business rule
- State หลายสถานะ
- Dependency ภายนอก
- Logic ที่ต้อง Unit Test
- Logic ที่ใช้ร่วมกันหลาย View

---

## 18. Folder Structure ที่แนะนำสำหรับ SwiftUI สมัยใหม่

ตัวอย่างนี้ใช้ **Feature-first structure** ซึ่งเหมาะกับโปรเจกต์ขนาดกลางถึงใหญ่

```text
MyApp/
├── App/
│   ├── MyApp.swift
│   ├── AppContainer.swift
│   ├── AppRouter.swift
│   └── RootView.swift
│
├── Features/
│   ├── Authentication/
│   │   ├── Models/
│   │   │   └── UserSession.swift
│   │   ├── Views/
│   │   │   ├── LoginView.swift
│   │   │   └── LoginForm.swift
│   │   ├── Stores/
│   │   │   └── LoginStore.swift
│   │   ├── Services/
│   │   │   ├── AuthServiceProtocol.swift
│   │   │   └── AuthService.swift
│   │   └── Routing/
│   │       └── AuthRoute.swift
│   │
│   ├── Products/
│   │   ├── Models/
│   │   │   └── Product.swift
│   │   ├── Views/
│   │   │   ├── ProductListView.swift
│   │   │   ├── ProductRow.swift
│   │   │   └── ProductDetailView.swift
│   │   ├── Stores/
│   │   │   ├── ProductListStore.swift
│   │   │   └── ProductDetailStore.swift
│   │   ├── Services/
│   │   │   ├── ProductServiceProtocol.swift
│   │   │   └── ProductService.swift
│   │   └── Routing/
│   │       └── ProductRoute.swift
│   │
│   └── Settings/
│       ├── Views/
│       ├── Stores/
│       └── Models/
│
├── Core/
│   ├── Networking/
│   │   ├── APIClient.swift
│   │   ├── APIEndpoint.swift
│   │   ├── APIError.swift
│   │   └── HTTPMethod.swift
│   ├── Persistence/
│   │   ├── PersistenceController.swift
│   │   └── KeyValueStore.swift
│   ├── Security/
│   │   └── KeychainStore.swift
│   ├── Analytics/
│   │   ├── AnalyticsClient.swift
│   │   └── AnalyticsEvent.swift
│   └── Extensions/
│       ├── Date+Formatting.swift
│       └── View+Conditional.swift
│
├── DesignSystem/
│   ├── Components/
│   │   ├── PrimaryButton.swift
│   │   ├── AppTextField.swift
│   │   ├── LoadingView.swift
│   │   └── ErrorView.swift
│   ├── Styles/
│   │   ├── CardModifier.swift
│   │   └── PrimaryButtonStyle.swift
│   ├── Tokens/
│   │   ├── AppSpacing.swift
│   │   ├── AppRadius.swift
│   │   └── AppTypography.swift
│   └── Resources/
│       └── Assets.xcassets
│
├── Shared/
│   ├── Models/
│   ├── Utilities/
│   ├── Protocols/
│   └── Views/
│
├── Resources/
│   ├── Localizable.xcstrings
│   ├── Configuration.xcconfig
│   └── Preview Content/
│
├── MyAppTests/
│   ├── Features/
│   ├── Core/
│   └── TestDoubles/
│
└── MyAppUITests/
    ├── AuthenticationUITests.swift
    └── ProductFlowUITests.swift
```

---

## 19. คำอธิบายแต่ละ Folder

## 19.1 `App/`

เป็น Composition Root และจุดเริ่มต้นของแอป

ประกอบด้วย:

- `MyApp.swift`: Entry point ที่มี `@main`
- `AppContainer.swift`: สร้างและถือ Dependency ระดับแอป
- `AppRouter.swift`: Navigation ระดับแอป
- `RootView.swift`: เลือก Flow เช่น Login หรือ Main App

ตัวอย่าง:

```swift
@main
struct MyApp: App {
    @State private var container = AppContainer.live

    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(container.session)
                .environment(container.router)
        }
    }
}
```

ไม่ควรใส่ Business Logic จำนวนมากใน `MyApp.swift`

---

## 19.2 `Features/`

จัดโค้ดตามความสามารถทางธุรกิจ เช่น:

- Authentication
- Products
- Orders
- Dashboard
- Settings

ข้อดี:

- เปิด Folder เดียวแล้วเห็นโค้ดของ Feature เกือบทั้งหมด
- Feature แยกออกเป็น Module ในอนาคตได้ง่าย
- ลดการกระจายไฟล์ไปทั่ว `Views`, `ViewModels`, `Models` ระดับ Root
- ทีมหลายคนทำงานคนละ Feature ได้ง่ายขึ้น

### เปรียบเทียบกับ Layer-first

โครงสร้างที่มักเริ่มต้น:

```text
Views/
ViewModels/
Models/
Services/
```

เมื่อแอปใหญ่ จะเกิดปัญหา:

- Folder มีไฟล์จำนวนมาก
- ไฟล์ของ Feature เดียวกันอยู่คนละที่
- ย้ายหรือลบ Feature ยาก

Feature-first จึงเหมาะกว่าในระยะยาว

---

## 19.3 `Core/`

เก็บ Infrastructure ที่ใช้ร่วมกันหลาย Feature เช่น:

- Networking
- Persistence
- Keychain
- Logging
- Analytics
- Foundation extensions

กฎสำคัญ:

- `Core` ไม่ควร Import Feature
- Feature สามารถใช้ Core ได้
- หลีกเลี่ยงการทำ `Core` ให้กลายเป็นถังรวมไฟล์ทุกชนิด
- สิ่งที่อยู่ Core ควรเป็นของที่ใช้ข้าม Feature จริง

---

## 19.4 `DesignSystem/`

เก็บองค์ประกอบ UI ที่เป็นมาตรฐานของผลิตภัณฑ์

ตัวอย่าง:

- Button style
- Text field
- Card
- Empty state
- Loading state
- Typography token
- Spacing token
- Corner radius
- Icons และ Assets

ควรแยก **Visual component** ออกจาก **Feature component**

ตัวอย่าง:

- `PrimaryButton` อยู่ `DesignSystem`
- `AddProductButton` อยู่ Feature Products เพราะมีความหมายทางธุรกิจเฉพาะ

---

## 19.5 `Shared/`

ใช้กับโค้ดที่แชร์หลาย Feature แต่ไม่ใช่ Infrastructure และไม่ใช่ Design System

ตัวอย่าง:

- Shared domain model
- Reusable validation
- Common protocol
- View ที่ใช้หลาย Feature แต่มีความหมายระดับแอป

ใช้ Folder นี้อย่างระมัดระวัง เพราะอาจกลายเป็น Folder รวมของที่ไม่รู้จะวางตรงไหน

ก่อนวางไฟล์ใน `Shared` ให้ถามว่า:

1. ใช้มากกว่าหนึ่ง Feature จริงหรือไม่
2. เป็น Core infrastructure หรือไม่
3. เป็น Design System หรือไม่
4. ควรอยู่ใน Feature เจ้าของหลักหรือไม่

---

## 19.6 `Resources/`

เก็บ Resource และ Configuration เช่น:

- Localization
- Asset catalog
- JSON fixture
- Configuration
- Font registration
- Privacy manifest
- Preview data

ข้อมูลลับ เช่น API Secret ไม่ควรฝังใน Repository หรือ App Bundle เพราะผู้ใช้สามารถ Extract ได้

---

## 19.7 Tests

จัด Test ให้สะท้อนโครงสร้าง Production

```text
MyAppTests/
├── Features/
│   ├── Products/
│   │   └── ProductListStoreTests.swift
│   └── Authentication/
│       └── LoginStoreTests.swift
├── Core/
│   └── Networking/
│       └── APIClientTests.swift
└── TestDoubles/
    ├── MockProductService.swift
    └── URLProtocolStub.swift
```

ทำให้ค้นหา Test ของแต่ละ Feature ได้ง่าย

---

## 20. Folder Structure สำหรับโปรเจกต์ขนาดเล็ก

ไม่จำเป็นต้องสร้าง Folder จำนวนมากตั้งแต่วันแรก

```text
MyApp/
├── App/
│   ├── MyApp.swift
│   └── RootView.swift
├── Features/
│   ├── Home/
│   │   ├── HomeView.swift
│   │   └── HomeStore.swift
│   └── Settings/
│       └── SettingsView.swift
├── Services/
│   └── APIClient.swift
├── Models/
│   └── User.swift
├── Components/
│   └── PrimaryButton.swift
└── Resources/
    └── Assets.xcassets
```

หลักการคือ:

- เริ่มเรียบง่าย
- แยกเมื่อมีเหตุผล
- อย่าสร้างไฟล์เปล่าตาม Architecture
- เมื่อ Feature โต ค่อยแยก Models, Views, Stores และ Services ภายใน Feature

---

## 21. ตัวอย่าง Feature แบบครบวงจร

```text
Features/
└── Products/
    ├── Models/
    │   └── Product.swift
    ├── Views/
    │   ├── ProductListView.swift
    │   ├── ProductListContent.swift
    │   └── ProductRow.swift
    ├── Stores/
    │   └── ProductListStore.swift
    └── Services/
        ├── ProductServiceProtocol.swift
        └── ProductService.swift
```

### Model

```swift
struct Product: Identifiable, Codable, Hashable, Sendable {
    let id: UUID
    let name: String
    let price: Decimal
}
```

### Service Protocol

```swift
protocol ProductServiceProtocol: Sendable {
    func fetchProducts() async throws -> [Product]
}
```

### Store

```swift
import Observation

@MainActor
@Observable
final class ProductListStore {
    enum ViewState: Equatable {
        case idle
        case loading
        case loaded([Product])
        case empty
        case failed(message: String)
    }

    private(set) var state: ViewState = .idle
    var searchText = ""

    private let service: ProductServiceProtocol

    init(service: ProductServiceProtocol) {
        self.service = service
    }

    func load() async {
        state = .loading

        do {
            let products = try await service.fetchProducts()
            state = products.isEmpty ? .empty : .loaded(products)
        } catch is CancellationError {
            return
        } catch {
            state = .failed(message: error.localizedDescription)
        }
    }

    func retry() async {
        await load()
    }
}
```

### View

```swift
struct ProductListView: View {
    @State private var store: ProductListStore

    init(service: ProductServiceProtocol) {
        _store = State(initialValue: ProductListStore(service: service))
    }

    var body: some View {
        ProductListContent(
            state: store.state,
            searchText: $store.searchText,
            onRetry: {
                Task { await store.retry() }
            }
        )
        .navigationTitle("Products")
        .task {
            await store.load()
        }
    }
}
```

### Presentational View

```swift
struct ProductListContent: View {
    let state: ProductListStore.ViewState
    @Binding var searchText: String
    let onRetry: () -> Void

    var body: some View {
        VStack {
            TextField("Search", text: $searchText)
                .textFieldStyle(.roundedBorder)
                .padding(.horizontal)

            switch state {
            case .idle, .loading:
                ProgressView()
                    .frame(maxHeight: .infinity)

            case .loaded(let products):
                List(products) { product in
                    ProductRow(product: product)
                }

            case .empty:
                ContentUnavailableView(
                    "No Products",
                    systemImage: "shippingbox"
                )

            case .failed(let message):
                ContentUnavailableView {
                    Label("Unable to Load", systemImage: "exclamationmark.triangle")
                } description: {
                    Text(message)
                } actions: {
                    Button("Retry", action: onRetry)
                }
            }
        }
    }
}
```

การแยก Container View และ Presentational View ทำให้:

- Preview UI state ต่าง ๆ ได้ง่าย
- Test Logic แยกจาก Layout
- View หลักทำหน้าที่ประกอบ Dependency และ Lifecycle
- View ลูกทำหน้าที่ Render State

---

## 22. React Custom Hook ควรแปลงเป็นอะไรใน SwiftUI

Custom Hook มีหลายหน้าที่ จึงไม่มีตัวแทนเดียวใน SwiftUI

| Custom Hook ทำหน้าที่ | SwiftUI ที่เหมาะสม |
|---|---|
| เก็บ UI State | `@State` หรือ `@Observable` Store |
| เรียก API | Service + Store |
| อ่าน Dependency | `@Environment` |
| Reusable styling | `ViewModifier` หรือ `ButtonStyle` |
| Reusable UI behavior | Custom View / ViewModifier |
| Navigation logic | Router |
| Utility ที่ไม่เกี่ยว UI | Function, struct หรือ extension |

ตัวอย่าง React Hook:

```tsx
function useProducts() {
  const [products, setProducts] = useState<Product[]>([])
  const [loading, setLoading] = useState(false)

  async function load() {
    setLoading(true)
    try {
      setProducts(await productService.fetchProducts())
    } finally {
      setLoading(false)
    }
  }

  return { products, loading, load }
}
```

SwiftUI Store:

```swift
@MainActor
@Observable
final class ProductStore {
    private(set) var products: [Product] = []
    private(set) var isLoading = false

    private let service: ProductServiceProtocol

    init(service: ProductServiceProtocol) {
        self.service = service
    }

    func load() async {
        isLoading = true
        defer { isLoading = false }

        do {
            products = try await service.fetchProducts()
        } catch {
            // Handle error state
        }
    }
}
```

---

## 23. Naming Convention ที่แนะนำ

### View

```text
ProductListView
ProductRow
LoginForm
MachineStatusCard
```

ไม่จำเป็นต้องลงท้าย `View` ทุกไฟล์ แต่ทีมควรกำหนดแนวทางให้สม่ำเสมอ

### Store / ViewModel

```text
ProductListStore
LoginStore
DashboardViewModel
```

เลือกคำเดียวให้สม่ำเสมอในโปรเจกต์

- `Store` เหมาะเมื่อเน้น State + Action
- `ViewModel` เหมาะกับทีมที่ใช้ MVVM ชัดเจน

### Service

```text
ProductServiceProtocol
ProductService
AuthServiceProtocol
LiveAuthService
MockAuthService
```

ใน Swift บางทีมไม่นิยมเติม `Protocol` และใช้ชื่อ:

```swift
protocol ProductService { }
struct LiveProductService: ProductService { }
```

ทั้งสองแนวทางใช้ได้ ควรเลือกแบบเดียวทั้งทีม

### Method

ใช้คำกริยาที่สื่อ Intent:

```swift
loadProducts()
submitOrder()
signIn()
retry()
selectProduct(_ product: Product)
```

หลีกเลี่ยงชื่อทั่วไปเกินไป:

```swift
doAction()
processData()
handleThing()
```

---

## 24. แนวทางเขียน View ที่อ่านง่าย

ลำดับสมาชิกใน View ที่แนะนำ:

```swift
struct ProductDetailView: View {
    // 1. Environment
    @Environment(\.dismiss) private var dismiss

    // 2. Input
    let productID: UUID

    // 3. Local state
    @State private var store: ProductDetailStore

    // 4. Initialization
    init(productID: UUID, service: ProductServiceProtocol) {
        self.productID = productID
        _store = State(
            initialValue: ProductDetailStore(
                productID: productID,
                service: service
            )
        )
    }

    // 5. Body
    var body: some View {
        content
            .navigationTitle("Product")
            .toolbar { toolbarContent }
            .task { await store.load() }
    }

    // 6. Subviews / ViewBuilder
    @ViewBuilder
    private var content: some View {
        // ...
    }

    @ToolbarContentBuilder
    private var toolbarContent: some ToolbarContent {
        ToolbarItem(placement: .topBarTrailing) {
            Button("Close") {
                dismiss()
            }
        }
    }

    // 7. Event methods
    private func handleDelete() {
        // ...
    }
}
```

### เมื่อไรควรแยก View ใหม่

แยกเมื่อ:

- มีชื่อเชิงความหมายทาง UI หรือ Domain
- มี State ของตัวเอง
- ใช้ซ้ำ
- ต้อง Preview แยก
- `body` ยาวจนอ่าน Flow ไม่ออก
- มี Branch ของ UI ที่ซับซ้อน

ไม่ควรแยกทุก `HStack` เป็นไฟล์ใหม่โดยไม่มีเหตุผล

---

## 25. Error Handling

อย่าเก็บเพียง `error.localizedDescription` หากต้องแยกประเภท Error เพื่อแสดง UI หรือ Retry ต่างกัน

```swift
enum ProductError: LocalizedError, Equatable {
    case offline
    case unauthorized
    case server
    case invalidData

    var errorDescription: String? {
        switch self {
        case .offline:
            "No internet connection"
        case .unauthorized:
            "Please sign in again"
        case .server:
            "Server is unavailable"
        case .invalidData:
            "Received invalid data"
        }
    }
}
```

แปลง Infrastructure Error เป็น Domain Error ที่ Service หรือ Repository boundary

```swift
func fetchProducts() async throws -> [Product] {
    do {
        return try await apiClient.send(.products)
    } catch URLError.notConnectedToInternet {
        throw ProductError.offline
    } catch APIError.unauthorized {
        throw ProductError.unauthorized
    } catch {
        throw ProductError.server
    }
}
```

---

## 26. Swift Concurrency ที่ควรระวัง

### ใช้ Structured Concurrency

แนะนำ:

```swift
.task {
    await store.load()
}
```

หรือเมื่อเกิดจาก Event:

```swift
Button("Refresh") {
    Task {
        await store.refresh()
    }
}
```

หลีกเลี่ยงการสร้าง Detached Task โดยไม่จำเป็น:

```swift
Task.detached {
    // อาจสูญเสีย Actor context และ Cancellation relationship
}
```

### รองรับ Cancellation

```swift
func load() async {
    do {
        let products = try await service.fetchProducts()
        try Task.checkCancellation()
        state = .loaded(products)
    } catch is CancellationError {
        return
    } catch {
        state = .failed(error.localizedDescription)
    }
}
```

### ใช้ `Sendable`

Model ที่ส่งข้าม Concurrency boundary ควร conform `Sendable` เมื่อทำได้

```swift
struct Product: Codable, Identifiable, Sendable {
    let id: UUID
    let name: String
}
```

---

## 27. Preview เทียบกับ Storybook

SwiftUI Preview ทำหน้าที่คล้าย Storybook หรือ Component Preview

```swift
#Preview("Loaded") {
    NavigationStack {
        ProductListContent(
            state: .loaded(Product.samples),
            searchText: .constant(""),
            onRetry: {}
        )
    }
}

#Preview("Error") {
    ProductListContent(
        state: .failed(message: "Network unavailable"),
        searchText: .constant(""),
        onRetry: {}
    )
}
```

### Preview ที่ดีควรครอบคลุม

- Loading
- Empty
- Loaded
- Error
- Dark mode
- Dynamic Type ขนาดใหญ่
- Localization ที่ข้อความยาว
- iPhone และ iPad เมื่อ Layout ต่างกัน

Preview ไม่แทน Unit Test หรือ UI Test แต่ช่วยตรวจ Visual state ได้เร็ว

---

## 28. Testing Strategy

### Unit Test Store

```swift
import Testing

@MainActor
struct ProductListStoreTests {
    @Test
    func load_success_setsLoadedState() async throws {
        let expected = [
            Product(id: UUID(), name: "Motor", price: 1_500)
        ]

        let service = MockProductService(products: expected)
        let store = ProductListStore(service: service)

        await store.load()

        #expect(store.state == .loaded(expected))
    }
}
```

### สิ่งที่ควร Test

- State transition
- Validation
- Error mapping
- Sorting และ Filtering
- Navigation decision ที่มีเงื่อนไข
- Data transformation

### สิ่งที่ไม่จำเป็นต้อง Unit Test มากเกินไป

- SwiftUI modifier ธรรมดา
- Getter ที่ตรงไปตรงมา
- Framework behavior ของ Apple

UI flow สำคัญให้ใช้ UI Test ส่วน Visual regression อาจใช้ Snapshot Testing Library เพิ่มตามความเหมาะสม

---

## 29. Anti-pattern ที่พบบ่อย

## 29.1 View ขนาดใหญ่เกินไป

อาการ:

- `body` หลายร้อยบรรทัด
- Network call อยู่ใน View
- Logic ซ้อนหลายชั้น
- Preview ยาก

แก้โดยแยก:

- Presentational View
- Store
- Service
- Reusable component

## 29.2 ViewModel ทุก View

ไม่จำเป็นต้องสร้าง ViewModel สำหรับ Text, Row หรือ Card ที่ไม่มี Logic

## 29.3 State ซ้ำหลายที่

เช่น Parent มี `selectedProduct` และ Child สร้างสำเนาอีกชุดโดยไม่จำเป็น ทำให้ข้อมูลไม่ Sync

ใช้ Value input หรือ Binding ตาม Ownership ที่แท้จริง

## 29.4 ใช้ Environment กับทุก Dependency

ทำให้ View ดูเหมือนไม่มี Dependency แต่จริง ๆ ต้องพึ่งหลาย Object และ Test ยาก

## 29.5 `onAppear` เรียก API ซ้ำโดยไม่ตั้งใจ

`onAppear` สามารถเกิดมากกว่าหนึ่งครั้ง ใช้ `.task` และให้ Store รองรับ Idempotency เมื่อจำเป็น

```swift
func loadIfNeeded() async {
    guard case .idle = state else { return }
    await load()
}
```

## 29.6 เก็บ Derived State

ค่าที่คำนวณได้ควรเป็น Computed Property ไม่ใช่ State อีกชุด

## 29.7 ใช้ Singleton ทุก Service

Singleton ทำให้ Test ยากและ Dependency ซ่อนอยู่ ควร Inject ผ่าน initializer หรือ Container

## 29.8 Folder เยอะเกินจริง

Folder Structure ควรสะท้อนโค้ดที่มี ไม่ใช่สร้างเพื่อให้ดูเป็น Enterprise

---

## 30. Checklist สำหรับผู้ที่ย้ายจาก React TypeScript มา SwiftUI

### Component และ View

- React Component → SwiftUI `View`
- Props → `let` property
- Callback prop → Closure property
- JSX → `body`
- Fragment → `Group` หรือ View Builder

### State

- `useState` → `@State`
- Controlled value → `@Binding`
- Store state → `@Observable`
- Context → Environment
- Derived state → Computed property

### Lifecycle และ Side Effect

- Data fetch → `.task`
- Effect ตาม Dependency → `.task(id:)` หรือ `.onChange`
- Mount event → `.onAppear`
- Unmount cleanup → `.onDisappear` หรือ Task cancellation

### Navigation

- Router → `NavigationStack`
- Route union type → Swift `enum Route`
- Modal → `.sheet`
- Full-screen modal → `.fullScreenCover`

### Styling

- CSS class → View Modifier
- Design token → Swift constants หรือ Environment values
- Styled component → Custom View, `ViewModifier`, `ButtonStyle`

### Data และ API

- TypeScript interface → Swift `struct` / `protocol`
- Promise → `async throws`
- API module → Service
- Runtime schema validation → Explicit decoding/error handling หรือ validation layer

---

## 31. แนวทางแนะนำโดยสรุป

1. มอง SwiftUI เป็น Declarative UI เหมือน React ไม่ใช่ UIKit แบบย่อ
2. ให้ State มีเจ้าของเพียงจุดเดียว
3. ใช้ `@State` กับ Local UI State
4. ใช้ `@Binding` เมื่อ Child ต้องแก้ State ของ Parent
5. ใช้ `@Observable` กับ Store/Model สำหรับ Deployment Target รุ่นใหม่
6. ใช้ `.task` สำหรับ Async work ที่ผูกกับอายุ View
7. หลีกเลี่ยง Side Effect เพื่อ Sync Derived State
8. แยก Service ด้วย Protocol เพื่อ Test และ Preview
9. จัด Folder แบบ Feature-first เมื่อโปรเจกต์เริ่มโต
10. ไม่ต้องสร้าง ViewModel ให้ทุก View
11. แยก Design System ออกจาก Feature-specific UI
12. ใช้ Typed Route และ `NavigationStack` สำหรับ Navigation สมัยใหม่
13. วาง Composition Root และ Dependency creation ไว้ที่ `App/`
14. ให้ View แสดง State และส่ง Action ส่วน Store จัดการ Workflow
15. เริ่มจากโครงสร้างที่เรียบง่าย แล้วขยายตามความซับซ้อนจริง

---

## 32. แหล่งอ้างอิงหลัก

- Apple Developer — SwiftUI: https://developer.apple.com/documentation/swiftui
- Apple Developer — App organization: https://developer.apple.com/documentation/swiftui/app-organization
- Apple Developer — Managing model data in your app: https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app
- Apple Developer — Observation: https://developer.apple.com/documentation/observation
- Apple Developer — Migrating to the Observable macro: https://developer.apple.com/documentation/swiftui/migrating-from-the-observable-object-protocol-to-the-observable-macro
- Apple Developer — State: https://developer.apple.com/documentation/swiftui/state
- React — Using TypeScript: https://react.dev/learn/typescript
- React — Managing State: https://react.dev/learn/managing-state
- React — Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React — Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects
- React — You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect

---

## 33. หมายเหตุเรื่องเวอร์ชัน

- `@Observable`, `@Bindable` และ Observation integration ใน SwiftUI ใช้ได้ตั้งแต่ iOS 17, iPadOS 17, macOS 14, tvOS 17 และ watchOS 10
- โปรเจกต์ที่รองรับ OS ต่ำกว่านี้ยังต้องใช้ `ObservableObject`, `@Published`, `@StateObject`, `@ObservedObject` หรือออกแบบ Compatibility layer
- API ของ SwiftUI เปลี่ยนแปลงตาม Xcode และ SDK ควรตรวจ Availability ก่อนใช้ API ใหม่
- Folder Structure ในเอกสารเป็นคำแนะนำ ไม่ใช่ข้อบังคับของ Apple ควรปรับตามขนาดทีม ขอบเขตแอป และ Deployment Target
