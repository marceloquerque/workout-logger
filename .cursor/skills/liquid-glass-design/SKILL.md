---
name: liquid-glass-design
description: Implements Liquid Glass design system in SwiftUI applications. Use when creating UI components, buttons, or visual effects that should follow Apple's Liquid Glass design patterns. Reference documentation at /Applications/Xcode.app/Contents/PlugIns/IDEIntelligenceChat.framework/Versions/A/Resources/AdditionalDocumentation/SwiftUI-Implementing-Liquid-Glass-Design.md
---

# Liquid Glass Design Implementation

Liquid Glass is a dynamic material introduced in iOS that combines the optical properties of glass with a sense of fluidity. It blurs content behind it, reflects color and light from surrounding content, and reacts to touch and pointer interactions in real time.

## Quick Reference

### Basic Usage

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect()
```

### Custom Shapes

```swift
.glassEffect(in: .rect(cornerRadius: 16.0))
.glassEffect(in: .capsule)
.glassEffect(in: .circle)
```

### Interactive Glass

```swift
.glassEffect(.regular.interactive())
.glassEffect(.regular.tint(.orange).interactive())
```

### Multiple Glass Effects

Always use `GlassEffectContainer` when applying Liquid Glass to multiple views:

```swift
GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .frame(width: 80.0, height: 80.0)
            .glassEffect()

        Image(systemName: "eraser.fill")
            .frame(width: 80.0, height: 80.0)
            .glassEffect()
    }
}
```

### Button Styles

```swift
Button("Click Me") {
    // Action
}
.buttonStyle(.glass)

Button("Important Action") {
    // Action
}
.buttonStyle(.glassProminent)
```

## Best Practices

1. Always use `GlassEffectContainer` when applying Liquid Glass to multiple views for better performance and morphing effects.
2. Apply the `.glassEffect()` modifier after other modifiers that affect the appearance of the view.
3. Use animations when changing view hierarchies to enable smooth morphing transitions.
4. Add `.interactive()` to glass effects that should respond to user interaction.

## Complete Documentation

For complete implementation details, morphing effects, and advanced techniques, see:
`/Applications/Xcode.app/Contents/PlugIns/IDEIntelligenceChat.framework/Versions/A/Resources/AdditionalDocumentation/SwiftUI-Implementing-Liquid-Glass-Design.md`
