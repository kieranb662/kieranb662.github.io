---
title: Pulse and Sheen
description: Learn how to make your SwiftUI buttons irresistible with the pulse and sheen effects.
keywords: Swift,SwiftUI,UX,Design,buttons,pulse,sheen
logo: /assets/PulseSheen/PulseSheen.gif
mathjax: true
---

<div class="icon-container">
<img class="post-icon" src="/assets/PulseSheen/PulseSheen.gif">
</div>

Recently for a project I needed to make some buttons that alerted the user that various features could be toggled on/off. So I thought it would be a great opportunity to create some passive animations.

## Pulse

The first thing I wanted was, given any shape create a pulsing effect. One of the issues I ran into was that `Shape` is a protocol with associated values. This means that you cannot just swap out different types of `Shapes` anytime you want. Luckily Swift has a built in way for dealing with such inconveniences, generics.

Once I had that down the next problem to solve was how to make the animation play. The solution to this was to make a `ViewModifier` with an `isOn` state variable, and then switch `isOn`'s value from false to true when the view appeared. From there I just had to make the actual pulse, which simple is two effects:

1. Scaling the shape from 1 -> 2
2. Changing the opacity from 1 -> 0

So what I finally came up with is the following:

```swift
struct PulseEffect<S: Shape>: ViewModifier {
    var shape: S
    @State var isOn: Bool = false
    var animation: Animation {
        Animation
            .easeIn(duration: 1)
            .repeatCount(8, autoreverses: false)
    }

    func body(content: Content) -> some View {
        content
            .background(
                ZStack {
                    shape
                        .stroke(Color.yellow, lineWidth: 1)
                        .scaleEffect(self.isOn ? 2 : 1)
                        .opacity(self.isOn ? 0 : 1)
                    shape
                        .stroke(Color.blue)
            })
            .onAppear {
                withAnimation(self.animation) {
                    self.isOn = true
                }
        }
    }
}

public extension View {
    func pulse<S: Shape>(_ shape: S) -> some View  {
        self.modifier(PulseEffect(shape: shape))
    }
}
```

Now whenever I want to make a pulsing shape all I need to do is call the .pulse(_ shape: ) method with my shape of choice.

**Example**
```swift
Image(systemName: "mic.slash")
            .foregroundColor(.blue)
            .padding()
            .pulse(Circle())
```

which looks like this:

<div class="icon-container">
<img class="post-icon" src="/assets/PulseSheen/CirclePulse.gif">
</div>

## Sheen

The sheen effect also has a simple solution, but it took a little math to get there. I originally was going to make this super dynamic and robust rhombus `Shape`. Then after much deliberation from the numerous ways one could define the rhombus shape, I decided to use a shear transform instead. Applying the `transformEffect` modifier onto a rounded rectangle did the trick. `transformEffect` takes a CGAffine transform as its parameter. Essentially an affine transform is a neat algebraic trick that makes operations easier by representing an $N$-dimensional vector as an $N+1$-dimensional vector. That explanation may not really help so here is a cheatsheet of the various things affine transforms can do.

<p align="center">
<img width="100%" max-width="600px" src="/assets/Resources/AffineTransforms.png">
</p>

We want to use the shear in the x direction version of affine transform. Now that we have a plan for making a rhombus, making the animation comes down to:

1. Overlaying a rounded rectangle nested within a GeometryReader on our content
2. Transforming the rounded rectangle into a rounded rhombus
3. Having the x offset start at (-width) when isOn = false and (Width) when isOn = true

```swift
struct SheenEffect: ViewModifier {
    @State var isOn: Bool = false
    var color = Color.white.opacity(0.6)
    var width: CGFloat = 50
    let animation = {
        Animation
            .easeInOut(duration: 1)
            .delay(0.8)
            .repeatCount(8, autoreverses: false)
    }()

    func body(content: Content) -> some View {
      // 1
        content.overlay(
            GeometryReader { proxy in
                RoundedRectangle(cornerRadius: 5)
                    .foregroundColor(self.color)
      // 2
                    .transformEffect(CGAffineTransform(a: 1, b: 0, c: tan(-.pi/6), d: 1, tx: 0, ty: 0))
      // 3
                    .offset(x: self.isOn ? proxy.size.width : -proxy.size.width, y: 0)
                    .frame(width: self.width)
                    .blur(radius: 7)
            }.mask(content)
        )
            .onAppear {
                withAnimation(self.animation) {
                    self.isOn = true
                }

        }
    }
}
public extension View {
    func sheen() -> some View {
        self.modifier(SheenEffect())
    }
}
```

I added some blur and and masked the rhombus with the content to make a better look. Now whenever you want some sheen just append `.sheen()` to any View!


## Bonus

Heres how to make the example button

<div class="icon-container">
<img class="post-icon" src="/assets/PulseSheen/PulseSheen.gif">
</div>

```swift
struct SheenButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
        .foregroundColor(configuration.isPressed ? Color.blue: Color.white)
        .padding()
            .background(RoundedRectangle(cornerRadius: 5)
                .fill(configuration.isPressed ? Color.white: Color.blue))
            .overlay(RoundedRectangle(cornerRadius: 5)
                .stroke(configuration.isPressed ? Color.blue: Color.white))
        .sheen()
    }
}

struct SheenPulseExample: View {
    var body: some View {
      Button(action: {print("pressed")}) {
          Text("Press Me")
      }.buttonStyle(SheenButtonStyle())
      .pulse(RoundedRectangle(cornerRadius: 5))  
    }
}
```
