---
title: Acceleration configuration
permalink: configuration-acceleration.html
keywords: configuration, acceleration, under-relaxation, Aitken, Quasi-Newton
summary: "Mathematically, implicit coupling schemes lead to fixed-point equations at the coupling interface. A pure implicit coupling without acceleration corresponds to a simple fixed-point iteration, which still has the same stability issues as an explicit coupling. We need acceleration techniques to stabilize and accelerate the fixed-point iteration."
---

To find out more about the mathematical background, please refer, for example, to [this paper](https://www.sciencedirect.com/science/article/pii/S0898122115005933).

In preCICE, three different types of <tt>acceleration</tt> can be configured: `constant` (constant under-relaxation), `aitken` (adaptive under-relaxation), and various quasi-Newton variants (`IQN-ILS` aka. Anderson acceleration, `IQN-IMVJ` aka. generalized Broyden). Before looking at the details, we need to understand which data gets modified when.

## Coupling data and primary data

All data communicated within a coupling scheme needs to be configured through `exchange` tags. Let's call these data fields **coupling data**.

```xml
<coupling-scheme:serial-implicit>
  <participants first="FluidSolver" second="StuctureSolver"/>
  <exchange data="Displacements" mesh="StructureMesh" from="StuctureSolver" to="FluidSolver"/>  
  <exchange data="Velocities"    mesh="StructureMesh" from="StuctureSolver" to="FluidSolver"/>  
  <exchange data="Forces"        mesh="StructureMesh" from="FluidSolver"    to="StuctureSolver"/>        
  ...
  <acceleration:...>
    <data name="Displacements" mesh="StructureMesh"/>
    ...
  </acceleration:...>
</coupling-scheme:serial-implicit>
```

The acceleration modifies coupling data in `advance()`, meaning, written values by the one participant are not the same when read by another participant. The values are stabilized (or "accelerated") based on a linear combination of previous iterations.

* For a **parallel coupling**, all coupling data is accelerated the same way. This means all coupling data use the same coefficients for the linear combination.
* For a **serial coupling**, only coupling that is exchanged from the `second` to the `first` participant is accelerated. Coupling data exchanged in the other direction (from `first` to `second`) is not modified.

**Example:** For fluid-structure interaction ignoring the velocities, if we first execute the fluid solver with given interface displacements followed by the structure solver taking forces from the fluid solver and computing new interface displacements, (only) the displacements are accelerated in case of serial coupling. For parallel coupling, both displacements and forces are accelerated.

Different acceleration schemes compute these coefficients in different ways.
They are generally based on the values of previous iterations.
A notable exception is the constant under-relaxation, which uses fixed coefficients.

Such value-dependent acceleration schemes need to select which data to compute these coefficients from by listing them as `data` tags inside the `acceleration` tag.
We call data which influences the coefficients **primary data** and data which is accelerated without influencing the coefficients **secondary data**.
The former is explicitly listed in the `acceleration` using `data` tags, while the latter is implied by the primary data and the configured coupling scheme.
Which data may be configured depends on the coupling scheme:

* For **serial coupling**, you can only configure primary data from coupling data which is exchanged from the `second` to the `first` participant. In the FSI example, the `Displacements`.
* For **parallel coupling**, all coupling data is available as primary data. For numerical performance reasons, you should define at least one coupling data field of each direction (one from `second` to `first`, one from `first` to `second`). In the FSI example, configure `Displacements` and `Forces`.

In the code example above, `Displacements` is primary data and `Velocities` is secondary data. `Forces` is not accelerated as the case uses a serial coupling scheme.
Changing the `serial-implicit` to a `parallel-implicit` scheme would turn `Forces` into secondary data.
As mentioned above, this is a theoretical setup as one primary data per direction is highly encouraged.
Hence, `Forces` should be marked primary as well.

Now, we know the difference between coupling data and primary data. Next, we have a look on how we actually configure the type of acceleration.

## Constant under-relaxation

```xml
<acceleration:constant>
  <relaxation value="0.5"/>
</acceleration:constant>
```

The configuration for constant under-relaxation is straight-forward. The only parameter to be configured is the under-relaxation factor <tt>relaxation</tt>. In particular, the configuration of primary data is not necessary as the relaxation parameter stays constant, i.e. the linear combination has fixed coefficients (`relaxation` for the current iteration, `1-relaxation` for the previous iteration).

Constant under-relaxation with a factor of 0.5 can be a good choice for e.g. turbulent FSI at a high Reynolds number.

## Dynamic Aitken under-relaxation

```xml
<acceleration:aitken>
  <data name="Displacements" mesh="StructureMesh"/>
  <initial-relaxation value="0.1"/>
</acceleration:aitken>
```

Aitken under-relaxation adapts the under-relaxation factor in every iteration based on current residuals of the defined primary data and we only need to define an initial relaxation factor `initial-relaxation`. An initial relaxation factor of 0.1 usually leads to a robust simulation.

Aitken under-relaxation can be a good choice for strong interaction with a fluid solver that does not fully converge in every iteration, or for compressible fluid solvers. For most cases, however, it is beneficial to change to a quasi-Newton scheme. Using Aitken under-relaxation is generally not recommended in combination with a parallel coupling scheme (refer to table 1 in [this paper](https://doi.org/10.1016/j.camwa.2015.12.025), where *Vec-Aitken* refers to our implementation of Aitken under-relaxation for parallel coupling schemes).

## Quasi-Newton schemes

```xml
<acceleration:IQN-ILS>
  <data name="Displacements" mesh="StructureMesh"/>
  <data name="Forces" mesh="StructureMesh"/>
  <preconditioner type="residual-sum"/>
  <filter type="QR2" limit="1e-3"/>
  <initial-relaxation value="0.1"/>
  <max-used-iterations value="100"/>
  <time-windows-reused value="20"/>
</acceleration:IQN-ILS>
```

For quasi-Newton, the configuration is more involved. We provide robust default values for all options, however. In the following, we list all (optional) options and parameters and give hints on good combinations of choices:

* Pick a quasi-Newton variant from the following choices: <tt>IQN-ILS</tt> (aka. Anderson acceleration), <tt>IQN-IMVJ</tt> (aka. generalized Broyden). `IQN-ILS` is simpler to start with.
* If parallel coupling is used and, thus, several primary data fields are configured, an equal weighting between them has to be ensured. This is done by defining a <tt>preconditioner</tt>. As <tt>type</tt>, we recommend to use `"residual-sum"`.
* To ensure linear independence of columns in the multi-secant system for Jacobian estimation, a <tt>filter</tt> can be used. The <tt>type</tt> can be chosen as <tt>QR1</tt> or <tt>QR2</tt>. In addition to <tt>type</tt>, a threshold for linear dependency, <tt>limit</tt> has to be defined. In most cases, the filter efficiency is not very sensitive with respect to the <tt>limit</tt>. We recommend to start with a `limit` of 1e-3 or 1e-2 and <tt>QR2</tt> (For `QR1` 1e-6 or 1e-5 is a good choice). A filter should be used with all quasi-Newton variants. If the respective line of the configuration file is omitted, no filter is applied. To find our more, you can have a look at [this paper](https://www.sciencedirect.com/science/article/pii/S004579491630164X).
* In the first iteration, quasi-Newton methods don’t provide an estimate for the Jacobian yet. Thus, the first iteration is an under-relaxed fixed-point iteration, for which we have to define the parameter <tt>initial-relaxation</tt>. 0.1 is a robust choice. Too small values can render the information from the first iteration too coarse for the calculation of a good Jacobian estimate. Too large values might lead to stability problems.
* The parameter <tt>max-used-iterations</tt> specifies the maximum number of previous iterations used to generate the data basis for Jacobian estimation. In particular for small simulations with only few degrees for freedom, this is an important parameter. It should be chosen to be smaller than half of the total number of degrees of freedom at the interface. For large-scale simulations 100 is a robust choice.
* The parameter `time-windows-reused` also limits the number of previous iterations, but in a per-time-window fashion. Iterations from older time windows than `time-windows-reused` are dropped. Note that, as we don’t know the number of iterations per time window a priori, this is not equivalent to setting max-used-iterations. For `IQN-IMVJ`, this parameter can be set to 0 as information from past time windows is implicitly used in the modified Jacobian norm minimization. For `IQN-ILS`, this parameter is an important tuning parameter, in particular if no filtering or filtering with a very low threshold is used. The optimum highly depends on the application, the used solvers and also the grid resolution. We recommend to choose a rather large value (10-30) and combine it with effective filtering (e.g., `QR2` with limit 1e-2) as a starting point for further optimization. With increasing degree of non-linearity of the considered application, the optimal value for `time-windows-reused` is expected to decrease.
* While the `IQN-IMVJ` method exhibits excellent convergence properties as a result of implicit reuse of prior information, it also causes quadratic storage and runtime complexity because of the explicit computation of Jacobian matrix. To make this method practical for large-scale simulations, we introduced a set of `imvj-restart-mode` variants, which reduce the method's complexity to a linear one. In particular, the `RS-SVD` mode gives good performance comparing to other options while retaining the original multi-vector convergence properties, although its performance slightly depends on the choice of the truncation parameter.

Quasi-Newton acceleration is a good choice for strong interactions. Please note that a necessary prerequisite for convergence of the implicit coupling loop is the proper convergence of each participant internally. Inner convergence measure (e.g. of the fluid solver) should be two orders of magnitude stricter than the coupling convergence-measure to achieve good performance with quasi-Newton.
