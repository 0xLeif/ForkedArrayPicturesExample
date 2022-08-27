# Fork ForkedArray Pictures Example

```swift
import Fork
import SwiftUI

class PicsumImageStore: ObservableObject {
    @Published var images: [UIImage] = []
    
    // TaskGroup
    func fetch(urls: [URL]) async throws {
        await withThrowingTaskGroup(of: Void.self) { taskGroup in
            for url in urls {
                taskGroup.addTask {
                    try await self.fetch(url: url)
                }
            }
        }
    }
    
    // ForkedArray
    func forkFetch(urls: [URL]) async throws {
        try await urls.asyncForEach(fetch(url:))
    }
    
    private func fetch(url: URL) async throws {
        let (data, _) = try await URLSession.shared.data(from: url)
        
        guard let image = UIImage(data: data) else { return }
        
        DispatchQueue.main.async {
            self.images.append(image)
        }
    }
}

struct ContentView: View {
    private let imageURLs: [URL] = (1 ... 100).compactMap { URL(string: "https://picsum.photos/id/\($0)/200/300") }
    
    @StateObject var imageStore = PicsumImageStore()
    
    var body: some View {
        if imageStore.images.isEmpty {
            Button("Fetch Images") {
                Task {
                    /*
                     // Using Swift TaskGroup
                     try! await imageStore.fetch(urls: imageURLs)
                     */
                     
                     // Using ForkedArray
                     try! await imageStore.forkFetch(urls: imageURLs)
                }
            }
        } else {
            List(imageStore.images, id: \.self) { image in
                Image(uiImage: image)
            }
        }
    }
}
```
