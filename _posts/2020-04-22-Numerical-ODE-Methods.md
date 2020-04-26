---
title: Numerical ODE Methods
description: Learn how to implement numerical methods for solving well posed ordinary differential equations
keywords: Swift, SwiftUI, differential equations, euler, runge-kutta, ODE
logo: /assets/postLogos/Numerical-Schemes.png
mathjax: true
---

<br>
<br>
<br>

## Mathematical Background

Euler’s method is a simple but effective strategy for numerically approximating ODE’s. The method takes advantage of the formal definition of the derivative and Taylor series.

$$\begin{aligned}
f'(x) &= \lim_{h\rightarrow 0} \frac{f(x+h) - f(x)}{h} \\ \\
f'(x) &\approx  \frac{f(x+h) - f(x)}{h} \\ \\
hf'(x) &\approx {f(x+h) - f(x)} \\
f(x+h) &\approx f(x) + h f'(x)
\end{aligned}$$

This is called the Forward difference approximation. Changing the value from ￼$x+h$ to ￼$x-h$ creates what’s called the backward difference approximation

$$f(x-h) \approx f(x) - h f'(x) $$

This method is easily applied to systems of ODE’s that frequently occur in classical dynamics.

## How To Use

1. Starting with some differential equation algebraically solve for $f'(x)$￼ so that computing the RHS gives you a numerical value $f'(x)$￼.
2. Using a known starting value of ￼$f(x_0) = f_0$, compute the next value using some step size ￼ $h$. $f(x+h) \approx f(x) + h f'(x)$
3. Repeat successively using the last ￼$f(x+h)$ and the next value for $f(x)$￼.
4. Continue until all needed values are obtained.

**Important Notes**

* ODE’s of order higher than 1 need to be reduced to a system of $n$ first order differential equations.
* The step size has an impact not only on the accuracy of the calculated values but also the convergent behavior of the solutions.


## What This Means For A Programmer

In programming terminology every single expression for a first order derivative must be programmed in as a function. So after reducing your differential equations and solving for the first order derivatives, you must create a function for each expression.  That function will look something like this:

```swift
func someDerivative(independentValue: Double, otherDependencies: [Double]) -> Double {
    // Your expression wont just be some simple multiplication
   return independentValue*otherDependencies.first!
}
```

Since the expression for the derivative likely depends not only on the dependent variable but possibly also upon value of the unknown function itself or some other dynamic values.

Using this format for expressing first order derivatives as functions, The amount of code needed to be written for solving larger systems can be significantly reduced. This means that an Euler solver can be written just once instead of for each derivative function.

<br>
## Simple Euler
<br>

Uses simple finite difference with no trial steps
Each call to this function performs one step. This implies that the `simpleEuler` function is meant to be used as part of a larger solve.
- **parameters**:
  - `stepSize`: The increment to increase the independent variable by each iteration
  - `independentVariable`: This is the current value the independent variable
  - `dependentVariables`: These are the current values of all dependencies of the derivative functions
  - `functions`: An array of references to derivative functions.
- **returns**
   The new values of the `dependentVariables` computed at the next step.
- **important**: The `dependentVariables` array and `functions` array must have the same number of elements and be in corresponding order.

```swift
func simpleEuler(_ stepSize: Double,
                 _ independentVariable: Double,
                 _ dependentVariables: [Double],
                 functions: [(Double, [Double]) -> Double]) -> [Double] {
    var newValues = [Double]()
    functions.enumerated().forEach { (i, f) in
        newValues.append( dependentVariables[i] + f(independentVariable, dependentVariables)*stepSize)
    }
    return newValues
}
```

This version of Euler’s method is considered to be the simplest and least effective approximation scheme in terms of convergence. Many other schemes make use of “trial steps”, such as a modified version of Euler’s method or the popular Runge-Kutta 4th Order scheme.

<br>
## Improved Euler
<br>

Makes use of a trial step and then averages the trial and real step values.
- **parameters**:
  - `stepSize`: The increment to increase the independent variable by each iteration
  - `independentVariable`: This is the current value the independent variable
  - `dependentVariables`: These are the current values of all dependencies of the derivative functions
  - `functions`: An array of references to derivative functions.

- **returns**
     The new values of the `dependentVariables` computed at the next step.
- **important**: The `dependentVariables` array and `functions` array must have the same number of elements and be in corresponding order.

```swift
func improvedEuler(_ stepSize: Double,
                   _ independentVariable: Double,
                   _ dependentVariables: [Double],
                   functions: [(Double, [Double]) -> Double]) -> [Double] {
    var trialValues = [Double]()
    functions.enumerated().forEach { (i, f) in
        trialValues.append( dependentVariables[i] + f(independentVariable, dependentVariables)*stepSize)
    }
    var newValues = [Double]()
    functions.enumerated().forEach { (i, f) in
        newValues.append(dependentVariables[i] + (f(independentVariable, trialValues) + f(independentVariable, dependentVariables))*stepSize/2)
    }
    return newValues
}
```

<br>
## Runge-Kutta 4th Order
<br>

By far the most overused albeit useful numerical scheme is the Runge-Kutta 4th Order scheme.

Approximates the solutions to differential equations using the 4th order Runge-Kutta numerical scheme.

$$\begin{aligned}
\text{let } f(t, x) & \text{ be any differential equation depending on } t \text{ and } x \\
\text{Then:} &\\
 k_1 &= Δt*f(t, x) \\
 k_2 &= Δt*f(t + \frac{Δt}{2}, x + \frac{k_1}{2}) \\
 k_3 &= Δt*f(t + \frac{Δt}{2}, x + \frac{k_2}{2}) \\
 k_4 &= Δt*f(t + Δt, x + k_3) \\
 Δf &= \frac{(k1 + 2k_2 + 2k_3 + k_4)}{6} \\
 f_{\text{new}}  &= f + Δf
 \end{aligned}$$

- **parameters**:
  - `stepSize`: The increment to increase the independent variable by each iteration
  - `independentVariable`: This is the current value the independent variable
  - `dependentVariables`: These are the current values of all dependencies of the derivative functions
  - `functions`: An array of references to derivative functions.
- **returns**
   The new values of the `dependentVariables` computed at the next step.
- **important**: The `dependentVariables` array and `functions` array must have the same number of elements
            and be in corresponding order.

```swift
func rK4(_ stepSize: Double,
         _ independentVariable: Double,
         _ dependentVariables: [Double],
         functions: [(Double, [Double]) -> Double]) -> [Double] {
    var k1: [Double] = []
    functions.forEach { (f) in
        k1.append(stepSize*f(independentVariable, dependentVariables))
    }
    var k2: [Double] = []
    functions.forEach { (f) in
        let newD = dependentVariables.enumerated().map { (index, nd)  in
            nd + k1[index]/2
        }
        k2.append(stepSize*f(independentVariable + stepSize/2, newD))
    }
    var k3: [Double] = []
    functions.forEach { (f) in
        let newD = dependentVariables.enumerated().map { (index, nd)  in
            nd + k2[index]/2
        }
        k3.append(stepSize*f(independentVariable + stepSize/2, newD))
    }
    var k4: [Double] = []
    functions.forEach { (f) in
        let newD = dependentVariables.enumerated().map { (index, nd)  in
            nd + k3[index]
        }
        k4.append(stepSize*f(independentVariable + stepSize, newD))
    }

    var kTot: [Double] = []
    for index in 0..<dependentVariables.count {
        kTot.append((k1[index] + 2*k2[index] + 2*k3[index] + k4[index])/6)
    }

    var newValues: [Double] = []

    for index in 0..<dependentVariables.count {
        newValues.append(dependentVariables[index] + kTot[index])
    }

    return newValues
}
```

## Wrapping Up

To string all of our schemes together we can make an enumeration to make picking between the easier

```swift
enum ODEScheme: Int, CaseIterable, Identifiable, Hashable {
    case rungeKutta
    case euler
    case improvedEuler

    var scheme: (Double, Double, [Double], [(Double, [Double]) -> Double]) -> [Double] {
        switch self {
        case .rungeKutta:
            return rK4(_:_:_:functions:)
        case .euler:
            return simpleEuler(_:_:_:functions:)
        case .improvedEuler:
            return improvedEuler(_:_:_:functions:)
        }
    }
    var name: String {
        switch self {
        case .rungeKutta:
            return "Runge-Kutta"
        case .euler:
            return "Euler"
        case .improvedEuler:
            return "Improved Euler"
        }
    }
    var id: Int { rawValue }
}
```

## Link

[Numerical ODE Shemes](https://gist.github.com/kieranb662/5d9d787e52bc1518d22fcb6f8c3fc4d6)
