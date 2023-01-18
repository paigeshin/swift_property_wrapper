# swift_property_wrapper

```swift
import SwiftUI

@propertyWrapper struct Document: DynamicProperty {
    @State private var value = ""
    private let url: URL
    
    var wrappedValue: String {
        get {
            return self.value
        }
        nonmutating set {
            do {
                try newValue.write(to: url, atomically: true, encoding: .utf8)
                self.value = newValue
            } catch {
                print("Failed to write output")
            }
        }
    }
    
    init(_ filename: String) {
        let paths: [URL] = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)
        self.url = paths[0].appendingPathComponent(filename)
        let initialText = (try? String(contentsOf: self.url)) ?? ""
        self._value = State(wrappedValue: initialText)
    }
    
    var projectedValue: Binding<String> {
        Binding(
            get: { self.wrappedValue },
            set: { self.wrappedValue = $0 }
        )
    }
    
}

struct ContentView: View {
    
    @Document("test.txt") var document: String
    
    var body: some View {
        NavigationView {
            VStack {
                TextEditor(text: self.$document)
                
                Button("Change document") {
                    self.document = String(Int.random(in: 0...1000))
                }
            }
            .navigationTitle("SimpleText")
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

```
