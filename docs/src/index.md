# TensorBoardLogger.jl

*Log data to TensorBoard from Julia*

[TensorBoard](https://www.tensorflow.org/guide/summaries_and_tensorboard) is an
interactive dashboard that can be used to visualize data generated from your programs.
**TensorBoardLogger** is a [Julia](https://julialang.org) package that allows
you to log data to TensorBoard through the standard Julia Logging system.

## Installation

To install this Julia package run the following command in the julia REPL:
```
] add TensorBoardLogger
```

!!! note
    To log images to the *Image* backend of TensorBoard you must also install
    `ImageMagick`. You won't need to import it.
    ```
    ] add ImageMagick
    ```
!!! note
    *TensorBoardLogger* only contains code to serialize data to TensorBoard.
    To run the TensorBoard interface, you should install it and run it.
    The easiest way to install it is to [install TensorFlow](https://www.tensorflow.org/install).
    Instructions for running TensorBoard can be found [on the GitHub page](https://github.com/tensorflow/tensorboard).



## Basic Usage
```@meta
CurrentModule = TensorBoardLogger
```

The fundamental type defined in this package is a `TBLogger`, which behaves like
other standard loggers in Julia such as `ConsoleLogger` or `TextLogger`. You can
create one by passing it the path to the folder where you want to store the data.
You can also pass an optional second argument to specify the behaviour in case
there already exhist a document at the given path.

Once you have created a `TBLogger`, you can use it as you would use any other
logger in Julia:
    - You can set it to be your global logger with the function [`global_logger`](https://docs.julialang.org/en/v1/stdlib/Logging/index.html#Base.CoreLogging.global_logger)
    - You can set it to be the current logger in a scope with the function [`with_logger`](https://docs.julialang.org/en/v1/stdlib/Logging/index.html#Base.CoreLogging.with_logger)
    - You can combine it with other Loggers using [LoggingExtras.jl](https://github.com/oxinabox/LoggingExtras.jl), so that messages are logged to TensorBoard and to other backends at the same time.

```julia
using TensorBoardLogger, Logging, Random

lg=TBLogger("tensorboard_logs/run", min_level=Logging.Info)

struct sample_struct first_field; other_field; end

with_logger(lg) do
    for i=1:100
        x0          = 0.5+i/30; s0 = 0.5/(i/20);
        edges       = collect(-5:0.1:5)
        centers     = collect(edges[1:end-1] .+0.05)
        histvals    = [exp(-((c-x0)/s0)^2) for c=centers]
        data_tuple  = (edges, histvals)
        data_struct = sample_struct(i^2, i^1.5-0.3*i)


        @info "test" i=i j=i^2 dd=rand(10).+0.1*i hh=data_tuple
        @info "test_2" i=i j=2^i hh=data_tuple log_step_increment=0
        @info "" my_weird_struct=data_struct   log_step_increment=0
        @debug "debug_msg" this_wont_show_up=i
    end
end
```

## Backends

*TensorBoardLogger* supports five backends:
    - Scalars
    - Distributions
    - Images
    - Audio
    - Text/Markdown

When you log data to tensorboard, it is processed to convert it to the format
used by TensorBoard.
In particoular, fields in structures are logged individually individually and complex
numbers/arrays are split into their real and imaginary parts. Depending on it's type, an
object is serialized to the relevant backend, with the text backend being the
default fallback.

For more detailed informations on how data is dispatched to a certain backend, and
how to specify a desired backend refer to [Specifying a backend](@ref).

If you want to define a new default behaviour for a custom type refer to section
[Extending TensorBoardLogger](@ref).

## Explicit logging

In alternative, you can also log data to TensorBoard through it's functional interface,
by calling the relevant method with a tag string and the data. For information
on this interface refer to @ref()...