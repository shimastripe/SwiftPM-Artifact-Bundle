# SwiftPM Artifact Bundle

- This is a SwiftPM artifact bundle repository.
  - Swift PM artifact is [SE-0305](https://github.com/apple/swift-evolution/blob/main/proposals/0305-swiftpm-binary-target-improvements.md) `Package Manager Binary Target Improvements` function.
  - Swift 5.6 supports extensible build tools in Swift Package Manager. [SE-0303](https://github.com/apple/swift-evolution/blob/main/proposals/0303-swiftpm-extensible-build-tools.md)
- This repository supports some swift major binaries.
  - [Mint](https://github.com/yonaskolb/Mint)
    - [LICENSE](https://github.com/yonaskolb/Mint/blob/master/LICENSE) 
  - [SwiftGen](https://github.com/SwiftGen/SwiftGen)
    - [LICENSE](https://github.com/SwiftGen/SwiftGen/blob/stable/LICENCE) 

## Example

- [SwiftGenPluginExample](https://github.com/shimastripe/SwiftGenPluginExample)

## Recuirements

- Swift 5.6 ( Xcode 13.3 )

## How to use ( SwiftGen example )

Use build tool plugin in `Package.swift`.

```swift
// swift-tools-version:5.6

import PackageDescription

let package = Package(
    name: "SwiftGenExample",
    products: [
        .library(
            name: "SwiftGenExample",
            targets: ["SwiftGenExample"]),
    ],
    targets: [
        .binaryTarget(
            name: "swift-cli-tools",
            url: "https://github.com/shimastripe/SwiftPM-Artifact-Bundle/releases/download/0.2.1/swift-cli-tools.artifactbundle.zip",
            checksum: "f8a9d286b891ba8981ddd9cb1a7ceaa45e9385976b310d49ef62bdc05a704e0c"),
        .plugin(
            name: "SwiftGenPlugin",
            capability: .buildTool(),
            dependencies: ["swift-cli-tools"]),
        .target(
            name: "SwiftGenExample",
            dependencies: [],
            resources: [.process("Resources")],
            plugins: [.plugin(name: "SwiftGenPlugin")]),
    ]
)
```

And make `SwiftGenExample/Plugins/SwiftGenPlugin/SwiftGenPlugin.swift`

```swift
import PackagePlugin

@main
struct SwiftGenPlugin: BuildToolPlugin {
    func createBuildCommands(context: PluginContext, target: Target) async throws -> [Command] {
        let outputFilesDirectory = context.pluginWorkDirectory

        let targetAssets = target.directory.appending("Resources/Color.xcassets")
        let outputFile = outputFilesDirectory.appending("Color.generated.swift")

        return [
            .prebuildCommand(
                displayName: "SwiftGen",
                executable: try context.tool(named: "swiftgen").path,
                arguments: [
                    "run", "xcassets",
                    targetAssets.string,
                    "--param", "publicAccess", "--templateName", "swift5", "--output",
                    outputFile.string,
                ],
                environment: [:],
                outputFilesDirectory: outputFilesDirectory)
        ]
    }
}
```

When every SwiftGenExample builds, generates a swift file in `$HOME/Library/Developer/Xcode/DerivedData/******/SourcePackages/swiftgenexample/SwiftGenExample/SwiftGenPlugin/Asset.generated.swift`!