# Swift
## The Basics
### Hello World
```swift
class Hello {
    init() {
    }

    func world() {
        return "Hello World!"
    }
}

print(Hello().world())
```

### Converting String to Double (using numberFormatter, Swift v5.8) - [REF](https://stackoverflow.com/questions/73962667/convert-string-number-with-exponent-to-double-value-in-swift)
```swift
var numberFormatter = NumberFormatter()
numberFormatter.numberStyle = .scientific
if let value = numberFormatter.number(from: "-121.03777") {
    let dVal = Double(exactly: value)!
    print(dVal)
}
```

## Core Data
### Generating Core Data class code. - [REF](https://developer.apple.com/documentation/coredata/modeling_data/generating_code)
Don't forget to change the persistence store name... - [REF](https://developer.apple.com/forums/thread/658033)
```swift
//Must be accessed while viewing persistent store file with the entity you want to generate selected. (It has screens to walk you through generating all/some entities too.)
Editor > Create NSManagedObject
```

### Stored data trumps in-memory data
```swift
import CoreData

let content = Persistence Controller.shared.container
context.mergePolicy = NSMergeByPropertyStoreTrumpMergePolicy
```

Note: Cannot fetch for entities manually, you MUST make a bulk request and create some checks locally before then being able to push through with building any URL Requests that update the table (requested Core Data content).

### PersistenceController (Autogenerated from Xcode for iOS)
```swift
import CoreData

struct PersistenceController {
    static let shared = PersistenceController()

    static var preview: PersistenceController = {
        let result = PersistenceController(inMemory: true)
        let viewContext = result.container.viewContext
        for _ in 0..<10 {
            let newItem = Item(context: viewContext)
            newItem.timestamp = Date()
        }
        do {
            try viewContext.save()
        } catch {
            // Replace this code to handle this error properly
            // fatalError will cause the app to crash
            let nsError = error as NSError
            fatalError("Unresolved error \(nsError), \(nsError.userInfo)")
        }
        return result
    }()

    let container: NSPersistentContainer

    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "CoreDataSample")
        if inMemory {
            container.persistentStoreDescription.first!.url = URL(fileURLWithPath: "/dev/null")
        }
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        container.viewContext.automaticallyMergesChangesFromParent = true
    }
}
```

## MapKit
### CLLocationCoordinate2D - [REF](https://developer.apple.com/documentation/corelocation/cllocationcoordinate2d/1423703-init)

### Creating a MapView - [REF](https://www.hackingwithswift.com/quick-start/swiftui/how-to-show-a-map-view)
```swift
struct ContentView: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(
            latitude: 51.507222,
            longitude: -0.1275),
        span: MKCoordinateSpan(
            latitudeDelta: 0.5,
            longitudeDelta: 0.5))

    var body: some View {
        Map(coordinateRegion: $region)
            .frame(width: 400, height: 300)
    }
}
```
Alternative way to build a MapView..
```swift
import SwiftUI
import MapKit

struct MapView: View {
    var coordinate:CLLocationCoordinate2D
    @State private var region = MKCoordinateRegion()

    var body: some View {
        Map(coordinateRegion: $region)
            .onAppear {
                setRegion(coordinate)
            }
    }

    private func setRegion(_ coordinate: CLLocationCoordinate2D) {
        region = MKCoordinateRegion(center: coordinate, span: MKCoordinateSpan(latitudeDelta: 0.2, longitudeDelta: 0.2))
    }
}

struct MapView_Previews: PreviewProvider {
    static var previews: some View {
        MapView(coordinate: CLLocationCoordinate2D(latitude: 34.011_286, longitude: -116.166_868))
    }
}
```

## SwiftUI
[Publicly available Apple Developer Reference](https://developer.apple.com/tutorials/swiftui/building-lists-and-navigation)
### SwiftUI App Launch
```swift
import SwiftUI

@main
struct NavigationListAndExamples: App {
    @StateObject private var object = ModelData()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(modelData)
        }
    }
}
```

### Default ContentView with Preview Provider
```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Test")
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
            .environmentObject(ModelData())
    }
}
```

### NavigationView with multiple devices for Previews
All device previews, must be available in Simulator
```swift
import SwiftUI

struct LandmarkList: View {
    @EnvironmentObject var modelData: ModelData
    @State private var showFavoritesOnly = false

    var filteredLandmarks: [Landmark] {
        modelData.landmarks.filter { landmark in
            (!showFavoritesOnly || landmark.isFavorite)
        }
    }

    var body: some View {
        NavigationView {
            List {
                Toggle(isOn: $showFavoritesOnly) {
                    Text("Favorites only")
                }
                ForEach(filteredLandmarks) { landmark in
                    NavigationLink {
                        LandmarkDetail(landmark: landmark)
                    } label {
                        LandmarkRow(landmark: landmark)
                    }
                }
            }
            .navigationTitle("Landmarks")
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        ForEach(["iPhone SE (3rd generation)", "iPhone 14 Pro Max"], id:\.self) { deviceName in
            LandmarkList()
                .previewDevice(PreviewDevice(rawValue: deviceName))
                .previewDisplayName(deviceName)
        }
    }
}
```

### Loading From JSON
```swift
import Foundation
import Combine

final class ModelData: ObservableObject {
    @Published var landmarks:[Landmark] = load("landmarkData.json")
}

func load<T:Decodable>(_ filename:String) -> T {
    let data:Data
    guard let file = Bundle.main.url(forResources: filename, withExtension: nil)
    else {
        fatalError("Couldn't find \(filename) in main bundle.")
    }

    do {
        data = try Data(contentsOf: file)
    } catch {
        fatalError("Couldn't load \(filename) from main bundle: \n\(error)")
    }

    do {
        let decoder = JSONDecoder()
        return try decoder.decode(T.self, from: data)
    } catch {
        fatalError("Couldn't parse \(filename) as \(T.self):\n\(error)")
    }
}
```

## URLSessionCodable Example
```swift
import SwiftUI

struct ContentView: View {
    @State private var results:[Result] = []

    var body: some View {
        List(results, id: \.team_number) { item in
            VStack(alignment: .leading) {
                Text(item.key)
                    .font(.headline)
                Text(item.nickname ?? (item.school_name ?? "FRC Team \(item.team_number)"))
            }
        }
        .task {
            results = []
            await loadData() //Fetches data asynchronously
        }
    }

    func loadData() async {
        for page in 0..<20 {
            print(page)
            guard let url = URL(string: "https://thebluealliance.com/api/v3/team/\(page)") else {
                print("Invalid URL")
                return
            }
            var req = URLRequest(url: url)
            req.addValue("[token]", forHTTPHeaderField: "X-TBA-Auth-Key")

            let task = URLSession.shared.dataTask(with: req) { data, response, error in
                if let d = data {
                    if let decodedResponse = try? JSONDecoder().decode([Result].self, from: d) {
                        //If results is empty, add decodedResponse
                        if results.count == 0 {
                            results += decodedResponse
                        } else {
                            //If the start of results has a higher team_number, then add decodedResponse before the existing results
                            if results[0].team_number > decodedResponse[0].team_number {
                                results = decodedResponse + results
                            //Else if the end of the array is less than decodedResponse, add decodedResponse to the end of the array
                            } else if results[results.count - 1].team_number < decodedResponse[0].team_number {
                                results += decodedResponse
                            //else if it is somewhere in the middle of the array
                            } else if results.count > 499 {
                                for i in 499..<results.count {
                                    //Find the index in the array where decodedResponse fits correctly
                                    if results[i].team_number > decodedResponse[0].team_number {
                                        //Confirm that the index before the current position is less than that value (should be)
                                        if results[i-1].team_number < decodedResponse[0].team_number {
                                            //Add decodedResponse between position i-1 and i
                                            DispatchQueue.main.async {
                                                results = results[0..<i] + decodedResponse + results[i..<results.count]
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
            task.resume()
        }
    }
}
```

## SwiftUI Design reference (Child views cannot be the ones to update a parent view's array)
### Issue Solution (below)
In this example, you cannot use the DeleteButton (View) to update the numbers variable passed in from DemoCustomDelete. I discovered this while trying to develop a custom row view that you can swipe to remove for [Fetchr](https://github.com/riigess/Fetchr).
```swift
struct DemoCustomDelete: View {
    @State private var numbers = Array(0...9) //Inclusive [0, ..., 9]

    var body: some View {
        NavigationView {
            VStack(spacing: 10) {
            ForEach(numbers, id: \.self) { number in
                VStack {
                    Text("\(number)")
                }
                .frame(width: 50, height: 50)
                .background(Color.red)
                .overlay(DeleteButton(number: number, number: $numbers, onDelete: removeRows), alignment: .topTrailing)
            }
            .onDelete(perform: removeRows)
        }
        .navigationTitle("Trying")
        .navigationBarItems(trailing:EditButton())
    }

    func removeRows(at offsets: IndexSet) {
        withAnimation {
            numbers.remove(atOffsets:offsets)
        }
    }
}

struct DeleteButton:View {
    @Environment(\.editMode) var editMode

    let number:Int
    @Binding var numbers:[Int]
    let onDelete:(IndexSet) -> ()

    var body: some View {
        VStack {
            if self.editMode?.wrappedValue == .active {
                Button(action: {
                    if let index = numbers.firstIndex(of: number) {
                        self.onDelete(IndexSet(integer: index))
                    }
                }) {
                    Image(systemName: "minus.circle")
                }
                .offset(x: 10, y: -10)
            }
        }
    }
}
```
