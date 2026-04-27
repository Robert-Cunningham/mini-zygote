# mini-zygote

`mini-zygote` is a small source-to-source reverse-mode automatic differentiation system in Julia that began as a rough clone of [`Zygote.jl`](https://github.com/FluxML/Zygote.jl). I made this while taking 18.S191 at MIT to better understand how Zygote-style automatic differentiation works.

The project manipulates Julia SSA-form IR using `IRTools.jl`, then inserts adjoint computations to compute gradients.

This is a learning project, not a production autodiff library. Many things are rough or outright broken.

## Structure

This repo walks through a few increasingly capable versions of a tiny autodiff system:

- `autodiff.jl`: basic reverse-mode AD over simple expressions using `+`, `*`, and `sin`.
- `autodiff_v2.jl`: factors primitive derivative logic into extensible `derivative_rule` methods.
- `autodiff_v3.jl`: experiments with recursive descent into user-defined functions.
- `autodiff_v4.jl`: experiments with conditional control flow by tracing executed basic blocks.

The main writeup is in `Julia Autodiff Final.ipynb`. The other notebooks are scratch work from exploring Julia, IRTools, recursive descent, and conditionals.

## Example

  ```julia
  include("autodiff_v4.jl")

  function manipulate(a, b)
      c = a * b * sin(b) * sin(a)

      if a >= b * 3
          return a * a
      elseif a >= b
          c = a * b
          return c * sin(c)
      elseif a > sin(b)
          return sin(b)
      else
          return sin(sin(sin(a)) * b) * a
      end
  end

  using Zygote

  @assert gradient_4(manipulate, 5, 1) == Zygote.gradient(manipulate, 5, 1)
  @assert gradient_4(manipulate, 5, 3) == Zygote.gradient(manipulate, 5, 3)
  @assert gradient_4(manipulate, 5, 10π) == Zygote.gradient(manipulate, 5, 10π)
  @assert gradient_4(manipulate, -3, 3) == Zygote.gradient(manipulate, -3, 3)
```

## References

- Zygote paper (https://arxiv.org/pdf/1810.07951.pdf)
- Zygote internals documentation (https://fluxml.ai/Zygote.jl/latest/internals/)
- Reverse-mode automatic differentiation explainer (https://rufflewind.com/2016-12-30/reverse-mode-automatic-differentiation)
