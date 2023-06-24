# LED Blinker: Full System Integration

Now it is time to add a GPIO driver to our system and attach it to the `led` component instance. We'll also connect the `led` component instance to a 1 Hz rate group. Finally, we'll configure the driver to manage the GPIO 13 pin on our hardware. Once this section is finished, the system is ready for running on hardware.

## Adding the GPIO Driver to the Topology

`fprime-arduino` provides a GPIO driver for Arduino microcontrollers called `Arduino.GpioDriver`. This should be added to both the instance definition list and the topology instance list just like we did for the `led` component. Since the GPIO driver is a passive component, its definition is a bit more simple.

Add to "Passive Component" section of `led-blinker/LedBLinker/Top/instance.fpp`:
```
  instance gpioDriver: Arduino.GpioDriver base id 0x4C00
```

Add to the instance list of `led-blinker/LedBLinker/Top/topology.fpp`:
```
    instance gpioDriver
```

> In `led-blinker` build the deployment and resolve any errors before continuing.

## Wiring The `led` Component Instance to the `gpioComponent` Component Instance and Rate Group

The `Led` component defines the `gpioSet` output port and the `Arduino.GpioDriver` defines the `gpioWrite` input port. These two ports need to be connected from output to input. The `ActiveRateGroup` component defines an array of ports called `RateGroupMemberOut` and one of these needs to be connected to `run` port defined on the `Led` component.

We can create a named connections block in the topology and connect the two port pairs. Remember to use the component instances and not the component definitions for each connection.

To do this, add the following lines to `led-blinker/LedBLinker/Top/topology.fpp`:
```
    # Named connection group
    connections LedConnections {
      # Rate Group 1 (1Hz cycle) ouput is connected to led's run input
      rateGroup1.RateGroupMemberOut[3] -> led.run
      # led's gpioSet output is connected to gpioDriver's gpioWrite input
      led.gpioSet -> gpioDriver.gpioWrite
    }
```

> `rateGroup1` is preconfigured to call all `RateGroupMemberOut` at a rate of 1 Hz. We use index `RateGroupMemberOut[3]` because `RateGroupMemberOut[0]` through `RateGroupMemberOut[2]` were used previously in the `RateGroups` connection block.

## Configuring The GPIO Driver

So far the GPIO driver has been instantiated and wired, but has not been told what GPIO pin to control. For this tutorial, the built-in LED will be used. To configure this, the `open` function needs to be called in the topology's C++ implementation and passed the pin's number and direction.

This is done by adding the following line to the `configureTopology` function defined in `led-blinker/LedBLinker/Top/LedBLinkerTopology.cpp`.

```
void configureTopology() {
    ...
    gpioDriver.open(Arduino::DEF_LED_BUILTIN, Arduino::GpioDriver::GpioDirection::OUT);
    ...
}
```

This code tells the GPIO driver to open pin LED_BUILTIN (usually pin 13) as an output pin. If your device does not have a built in LED, select a GPIO pin of your choice.

> In `led-blinker` build the deployment and resolve any errors before continuing.

## Conclusion

Congratulations!  You've wired your component to the rate group driver and GPIO driver components. It is time to try it on hardware.

### Next Step: [Running on Hardware](./running-on-hardware.md).
