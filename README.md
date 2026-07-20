# SwiftUI for React Developers

คู่มือสำหรับนักพัฒนาที่ต้องการเรียนรู้และออกแบบแอปด้วย **SwiftUI สมัยใหม่** โดยอธิบายแนวคิดผ่านการเปรียบเทียบกับ **React + TypeScript**

Repository นี้เน้นเรื่องแนวคิดการเขียน UI แบบ Declarative, การจัดการ State, การออกแบบ Architecture, การแบ่ง Folder Structure และแนวทางเขียนโค้ด SwiftUI ที่อ่านง่าย ดูแลต่อได้ และเหมาะกับโปรเจกต์จริง

## เป้าหมายของ Repository

* อธิบายหลักการทำงานของ SwiftUI แบบเข้าใจง่าย
* เปรียบเทียบ SwiftUI กับ React + TypeScript
* ช่วยให้นักพัฒนา React เริ่มต้น SwiftUI ได้เร็วขึ้น
* แนะนำ Folder Structure สำหรับ SwiftUI สมัยใหม่
* รวบรวม SwiftUI Patterns และ Best Practices
* ใช้เป็นเอกสารอ้างอิงสำหรับพัฒนาแอป iOS, iPadOS และ macOS

## เหมาะสำหรับใคร

Repository นี้เหมาะสำหรับ:

* นักพัฒนา React หรือ React Native ที่ต้องการเรียน SwiftUI
* นักพัฒนา Web ที่เริ่มพัฒนาแอป iOS
* นักพัฒนา SwiftUI ระดับเริ่มต้นถึงระดับกลาง
* ทีมที่ต้องการกำหนดมาตรฐานโครงสร้างโปรเจกต์ SwiftUI
* ผู้ที่ต้องการตัวอย่าง Architecture สำหรับโปรเจกต์จริง

## เนื้อหาหลัก

### SwiftUI เทียบกับ React + TypeScript

อธิบายแนวคิดที่มีความใกล้เคียงกัน เช่น:

| React + TypeScript       | SwiftUI                           |
| ------------------------ | --------------------------------- |
| Component                | View                              |
| JSX                      | View Builder                      |
| Props                    | Stored Properties                 |
| `useState`               | `@State`                          |
| Controlled State         | `@Binding`                        |
| Context                  | `@Environment`                    |
| State Store              | `@Observable`                     |
| `useEffect`              | `.task`, `.onAppear`, `.onChange` |
| React Router             | `NavigationStack`                 |
| Promise / Async Function | Swift Concurrency                 |
| Interface / Type         | Protocol / Struct / Enum          |

### State Management

ครอบคลุมการใช้งาน:

* `@State`
* `@Binding`
* `@Observable`
* `@Environment`
* `@EnvironmentObject`
* `@StateObject`
* `@ObservedObject`
* Swift Observation Framework
* การเลือกเจ้าของ State ที่เหมาะสม
* การส่งข้อมูลระหว่าง Parent View และ Child View

### Architecture

แนวทาง Architecture ที่ใช้ในโปรเจกต์ SwiftUI เช่น:

* Feature-first Architecture
* MVVM
* Repository Pattern
* Service Layer
* Dependency Injection
* Protocol-based Abstraction
* State-driven Navigation
* Modular Architecture

### Folder Structure

ตัวอย่างโครงสร้างสำหรับโปรเจกต์ขนาดกลางถึงขนาดใหญ่:

```text
MyApp/
├── App/
│   ├── MyApp.swift
│   ├── AppEnvironment.swift
│   ├── AppRouter.swift
│   └── AppDependencies.swift
│
├── Core/
│   ├── DesignSystem/
│   ├── Networking/
│   ├── Persistence/
│   ├── Extensions/
│   ├── Utilities/
│   └── Common/
│
├── Features/
│   ├── Authentication/
│   │   ├── Models/
│   │   ├── Views/
│   │   ├── Components/
│   │   ├── ViewModels/
│   │   ├── Services/
│   │   └── Repositories/
│   │
│   ├── Home/
│   ├── Products/
│   ├── Settings/
│   └── Profile/
│
├── Shared/
│   ├── Models/
│   ├── Components/
│   ├── Services/
│   └── Resources/
│
├── Resources/
│   ├── Assets.xcassets
│   ├── Localizable.xcstrings
│   ├── Fonts/
│   └── Config/
│
├── PreviewContent/
│   ├── PreviewData/
│   └── PreviewMocks/
│
└── Tests/
    ├── UnitTests/
    ├── IntegrationTests/
    └── UITests/
```

## เอกสารภายใน Repository

| ไฟล์                                                                 | รายละเอียด                                                                 |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [`swiftui-vs-react-typescript.md`](./swiftui-vs-react-typescript.md) | เปรียบเทียบ SwiftUI กับ React + TypeScript พร้อมแนวทางจัดโครงสร้างโปรเจกต์ |
| [`swiftui-state-management.md`](./swiftui-vs-react-typescript.md)                                        | แนวทางจัดการ State ใน SwiftUI                                              |
| `swiftui-navigation.md`                                              | NavigationStack, Route และ Deep Link                                       |
| `swiftui-architecture.md`                                            | Architecture และ Dependency Injection                                      |
| `swiftui-folder-structure.md`                                        | แนวทางจัด Folder Structure ตามขนาดโปรเจกต์                                 |
| `swiftui-testing.md`                                                 | Unit Test, UI Test และการ Mock Dependency                                  |

> ปัจจุบันอาจมีเพียงบางไฟล์ และจะเพิ่มเนื้อหาเพิ่มเติมในอนาคต

## ตัวอย่างแนวคิดพื้นฐาน

### React

```tsx
type CounterProps = {
  initialValue?: number
}

export function Counter({ initialValue = 0 }: CounterProps) {
  const [count, setCount] = useState(initialValue)

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

### SwiftUI

```swift
import SwiftUI

struct CounterView: View {
    @State private var count: Int

    init(initialValue: Int = 0) {
        _count = State(initialValue: initialValue)
    }

    var body: some View {
        VStack(spacing: 16) {
            Text("Count: \(count)")

            Button("Increase") {
                count += 1
            }
        }
    }
}
```

แนวคิดสำคัญคือทั้ง React และ SwiftUI จะสร้าง UI จาก State เมื่อ State เปลี่ยน Framework จะคำนวณและอัปเดตส่วนที่เกี่ยวข้องของ UI

## หลักการที่ Repository นี้แนะนำ

### View ควรมีหน้าที่แสดงผล

หลีกเลี่ยงการใส่ Business Logic จำนวนมากไว้ภายใน `View`

```swift
struct ProductListView: View {
    @State private var store: ProductListStore

    var body: some View {
        List(store.products) { product in
            ProductRow(product: product)
        }
        .task {
            await store.loadProducts()
        }
    }
}
```

### แยก Dependency ออกจาก View

```swift
protocol ProductRepository {
    func fetchProducts() async throws -> [Product]
}
```

```swift
@Observable
final class ProductListStore {
    private let repository: ProductRepository

    var products: [Product] = []
    var isLoading = false
    var errorMessage: String?

    init(repository: ProductRepository) {
        self.repository = repository
    }

    @MainActor
    func loadProducts() async {
        isLoading = true
        defer { isLoading = false }

        do {
            products = try await repository.fetchProducts()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

### แบ่งโค้ดตาม Feature

แนะนำให้เก็บไฟล์ที่เกี่ยวข้องกับ Feature เดียวกันไว้ด้วยกัน แทนการแยกทั้งโปรเจกต์ตามชนิดไฟล์เพียงอย่างเดียว

```text
Features/
└── Products/
    ├── Models/
    ├── Views/
    ├── Components/
    ├── ViewModels/
    ├── Services/
    └── Repositories/
```

วิธีนี้ช่วยให้:

* ค้นหาไฟล์ได้ง่าย
* ลดการเชื่อมโยงข้าม Feature
* ย้ายหรือลบ Feature ได้สะดวก
* รองรับการแยกเป็น Swift Package ในอนาคต
* ช่วยให้หลายทีมทำงานร่วมกันได้ง่ายขึ้น

## Requirements

ตัวอย่างและแนวทางใน Repository นี้เน้นเทคโนโลยีสมัยใหม่ ได้แก่:

* Swift 5.9 หรือใหม่กว่า
* SwiftUI
* Xcode 15 หรือใหม่กว่า
* iOS 17 หรือใหม่กว่า สำหรับ Observation Framework บางส่วน
* Swift Concurrency
* Async/Await
* Observation Framework

บางแนวคิดสามารถนำไปปรับใช้กับ iOS เวอร์ชันเก่าได้ โดยเปลี่ยนจาก `@Observable` ไปใช้ `ObservableObject`, `@StateObject` และ `@ObservedObject`

## วิธีใช้งาน Repository

Clone Repository:

```bash
git clone https://github.com/AnuwatThisuka/swiftui-for-react-developers.git
```

เข้าสู่โฟลเดอร์:

```bash
cd swiftui-for-react-developers
```

เริ่มอ่านจาก:

```text
swiftui-vs-react-typescript.md
```

หรือเปิดจากหน้าเอกสาร:

* [SwiftUI vs React + TypeScript](./swiftui-vs-react-typescript.md)

## Roadmap

เนื้อหาที่วางแผนจะเพิ่ม:

* SwiftUI State Management แบบละเอียด
* NavigationStack และ Router Pattern
* Dependency Injection
* Networking ด้วย URLSession
* Error Handling
* SwiftData และ Persistence
* Authentication Flow
* Design System
* Reusable Components
* Unit Testing
* UI Testing
* Preview และ Mock Data
* Modularization ด้วย Swift Package Manager
* ตัวอย่างโปรเจกต์จริงแบบ Feature-first
* แนวทาง Migration จาก React หรือ React Native

## แนวทางการมีส่วนร่วม

สามารถสร้าง Issue หรือ Pull Request เพื่อ:

* แก้ไขเนื้อหาที่ไม่ถูกต้อง
* เพิ่มตัวอย่างโค้ด
* เพิ่ม SwiftUI Pattern ใหม่
* ปรับปรุง Folder Structure
* เพิ่มคำอธิบายเปรียบเทียบกับ React
* เพิ่มตัวอย่างสำหรับโปรเจกต์จริง

ก่อนส่ง Pull Request กรุณาตรวจสอบว่า:

* ตัวอย่างโค้ดสามารถอ่านและเข้าใจได้ง่าย
* ใช้ Naming Convention ที่สม่ำเสมอ
* ไม่มี Business Logic จำนวนมากอยู่ใน View
* Dependency สามารถ Mock สำหรับการทดสอบได้
* เนื้อหาสอดคล้องกับ SwiftUI และ Swift รุ่นปัจจุบัน

## Disclaimer

Repository นี้เป็นแนวทางและตัวอย่างสำหรับการเรียนรู้ ไม่มี Architecture หรือ Folder Structure รูปแบบเดียวที่เหมาะกับทุกโปรเจกต์

ควรเลือกใช้ Pattern ตาม:

* ขนาดของโปรเจกต์
* จำนวนสมาชิกในทีม
* อายุของโปรเจกต์
* ความซับซ้อนของ Business Logic
* Deployment Target
* ความต้องการด้าน Testing
* แผนการ Modularize ในอนาคต

โปรเจกต์ขนาดเล็กไม่จำเป็นต้องใช้ Architecture ที่ซับซ้อน แต่ควรมีการแบ่งความรับผิดชอบของแต่ละส่วนอย่างชัดเจน

## License

สามารถกำหนด License ตามวัตถุประสงค์ของ Repository เช่น:

* MIT License สำหรับ Repository แบบ Open Source
* Private Repository สำหรับใช้งานภายในทีม
* Creative Commons สำหรับเนื้อหาเอกสาร

## Author

**Anuwat Thisuka**

- GitHub: [AnuwatThisuka](https://github.com/AnuwatThisuka)

จัดทำขึ้นเพื่อใช้เป็นคู่มือเรียนรู้และอ้างอิงในการพัฒนาแอปด้วย SwiftUI สมัยใหม่
