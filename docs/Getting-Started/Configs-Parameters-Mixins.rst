Configs, Parameters, Mix-ins, and Everything In Between
========================================================

A significant portion of generators in the REBAR framework use the Rocket Chip parameter system.
This parameter system enables for the flexible configuration of the SoC without invasive RTL changes.
In order to use the parameter system correctly, we will use several terms and conventions:

Parameters
--------------------

TODO: Need to explain up, site, field, and other stuff from Henry's thesis.

It is important to note that a significant challenge with the Rocket parameter system is being able to identify the correct parameter to use, and the impact that parameter has on the overall system.
We are still investigating methods to facilitate parameter exploration and discovery.

Configs
---------------------

A *Config* is a collection of multiple generator parameters being set to specific values.
Configs are additive, can override each other, and can be composed of other Configs.
The naming convention for an additive Config is ``With<YourConfigName>``, while the naming convention for a non-additive Config will be ``<YourConfig>``.
Configs can take arguments which will in-turn set parameters in the design or reference other parameters in the design (see :ref:`Parameters`).

:numref:`basic-config-example` shows a basic additive Config class that takes in zero arguments and instead uses hardcoded values to set the RTL design parameters.
In this example, ``MyAcceleratorConfig`` is a Scala case class that defines a set of variables that the generator can use when referencing the ``MyAcceleratorKey`` in the design.

.. _basic-config-example:
.. code-block:: scala

  class WithMyAcceleratorParams extends Config((site, here, up) => {
    case BusWidthBits => 128
    case MyAcceleratorKey =>
      MyAcceleratorConfig(
        rows = 2,
        rowBits = 64,
        columns = 16,
        hartId = 1,
        someLength = 256)
  })

This next example (:numref:`complex-config-example`) shows a "higher-level" additive Config that uses prior parameters that were set to derive other parameters.

.. _complex-config-example:
.. code-block:: scala

  class WithMyMoreComplexAcceleratorConfig extends Config((site, here, up) => {
    case BusWidthBits => 128
    case MyAcceleratorKey =>
      MyAcceleratorConfig(
        Rows = 2,
        rowBits = site(SystemBusKey).beatBits,
        hartId = up(RocketTilesKey, site).length)
  })

:numref:`top-level-config` shows a non-additive Config that combines the prior two additive Configs using ``++``.
The additive Configs are applied from the right to left in the list (or bottom to top in the example).
Thus, the order of the parameters being set will first start with the ``DefaultExampleConfig``, then ``WithMyAcceleratorParams``, then ``WithMyMoreComplexAcceleratorConfig``.

.. _top-level-config:
.. code-block:: scala

  class SomeAdditiveConfig extends Config(
    new WithMyMoreComplexAcceleratorConfig ++
    new WithMyAcceleratorParams ++
    new DefaultExampleConfig
  )

Cake Pattern
-------------------------

A cake pattern is a Scala programming pattern, which enable "mixing" of multiple traits or interface definitions (sometimes referred to as dependency injection).
It is used in the Rocket Chip SoC library and REBAR framework in merging multiple system components and IO interfaces into a large system component.

:numref:`cake-example` shows a Rocket Chip based SoC that merges multiple system components (BootROM, UART, etc) into a single top-level design.

.. _cake-example:
.. code-block:: scala

  class MySoC(implicit p: Parameters) extends RocketSubsystem
    with CanHaveMisalignedMasterAXI4MemPort
    with HasPeripheryBootROM
    with HasNoDebug
    with HasPeripherySerial
    with HasPeripheryUART
    with HasPeripheryIceNIC
  {
     //Additional top-level specific instantiations or wiring
  }

Mix-in
---------------------------

A mix-in is a Scala trait, which sets parameters for specific system components, as well as enabling instantiation and wiring of the relevant system components to system buses.
The naming convention for an additive mix-in is ``Has<YourMixin>``.
This is show in :numref:`cake-example` where things such as ``HasPeripherySerial`` connect a RTL component to a bus and expose signals to the top-level.

Additional References
---------------------------

A brief explanation of some of these topics is given in the following video: https://www.youtube.com/watch?v=Eko86PGEoDY.
