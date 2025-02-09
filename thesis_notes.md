# Basilisk Thesis Notes
This repo is a direct FORK from the original [Basilisk repo](https://github.com/AVSLab/basilisk). Run `git fetch upstream` regularly to catch latest updates from the original repo. Simulations and configuration files created for this project are located inside the `dev/` file, including chosen example simulation scripts inside `dev/template-examples` copied from scripts in `examples/`.

## Building the Project:
Run the following for clean build, this is required when a new C/C++ message/module type is newly-implemented/modified. For more information see the Basilisk online resources on [Building the Software Framework](http://hanspeterschaub.info/basilisk/Install/installBuild.html).
```
python3 conanfile.py --clean
```

## Run basic simulations by:
```
source .venv/bin/activate $activate python virtual env
python3 scenarioBasicOrbitStream.py $run basic simulations in terminal 
```

## Viz-support:
Viz Socket: [tcp://localhost:5556](tcp://localhost:5556)

### Loading other CAD files for visualization:
[scenarioDataToViz.py](examples/scenarioDataToViz.py) shows how (line 178 / https://hanspeterschaub.info/basilisk/Vizard/VizardGUI.html#import-a-custom-shape-model)

Example: 
```
viz = vizSupport.enableUnityVisualization(scSim, simTaskName, scList
                                                  , saveFile=fileName
                                                  , liveStream=True
                                                  )
# Enable "liveStream=True" for Viz tool to work!
# Enable "saveFile=fileName"
```

## Basilisk video takeaways:
- `scSim, Process, scObject` as SC, add the essential lines as covered in the Basic orbit case + specify moment of inertias (as Astrobee)
- Gravity factory `gravFactory` -> create Earth or other planets, map these using the line to attach Gravity model to SC (photo underlined)
- From video Basilisk: attitude feedback with RW devices: RW factory -> can specify RW spec here, see [scenarioAttitudeFeedbackRW.py](examples/scenarioAttitudeFeedbackRW.py) (probably same for thruster?); create 3 wheels with same/different initial speed and direction; seems also can map with VoltageIO, check if can related to power consumption/EPS things
- Orbital motions module [orbitalMotion.py](dist3/Basilisk/utilities/orbitalMotion.py) for state vectors r,v & rotational + attitude sigma & omega (?)
- Always use line: scSim.AddModelToTask() and add all used objects, gravity bodies, RWEffectors, etc for the the logging to work


## Useful Basilisk modules (from `Basilisk.utilities` or `Basilisk.architecture`):
- [macros.py](dist3/Basilisk/utilities/macros.py): time (e.g. day, hour, second conversions to nanosec), degree-to-radian, RPM conversions
- [unitTestSupport.py](dist3/Basilisk/utilities/unitTestSupport.py): unit test supports including Vector/Value comparisons, pass-fail and `np2EigenMatrix3D()` used in spacecraft hub for moment of inertia. (see [scConfig.py](dev/scConfig.py) at `.createSC()`: `scObject.hub.IHubPntBc_B = unitTestSupport.np2EigenMatrix3d(sc.IHub)`)
- [RigidBodyKinematics.py](dist3/Basilisk/utilities/RigidBodyKinematics.py): EP, Gibbs vector (or Classical Rodrigues Parameters), MRP, RV, Euler angles conversions***
- [orbitalMotion.py](dist3/Basilisk/utilities/orbitalMotion.py): planetary constants, state r,v to element & vice versa, Atmospheric Drag functions (need c_D, area, mass inputs), J-Perturbations of Earth & diff. planets, Solar Radiation pressure {***Mind the units when use them, some uses km/s^2 !}, Orbital Element conversions (classicial <=> equinoctial <=> hill frame <=> RV): ![rv2hill](./ref-images/rv2hill_hill.gpg.jpeg)
- [astroFunctions.py](dist3/Basilisk/utilities/astroFunctions.py): 
*To create thrusters/RWs:*
- [simIncludeThruster.py](dist3/Basilisk/utilities/simIncludeThruster.py) & [simIncludeRW.py](dist3/Basilisk/utilities/simIncludeRW.py), see examples in [scenarioAttitudeFeedbackRW.py line 441](examples/scenarioAttitudeFeedbackRW.py) & [scenarioFormationReconfig.py line 148](examples/scenarioFormationReconfig.py) for RWs/thrusters creation & interfacing. Also see [fswSetupThrusters.py](dist3/Basilisk/utilities/fswSetupThrusters.py) & [fswSetupRW.py](dist3/Basilisk/utilities/fswSetupRW.py) & the `messaging` module in `Basilisk.architecture` to understand the messaging and data accessing after the simulation ends -> how to get useful data (state vectors, orbits, thruster/RW performances) & visualization params out of the simulation {also check [`this messaging related module`](dist3/Basilisk/architecture/messaging/__init__.py) or [web](https://hanspeterschaub.info/basilisk/Documentation/architecture/msgPayloadDefC/index.html) for exhaustive available Basilisk messages}.
- [inertial3D.py](src/fswAlgorithms/attGuidance/inertial3D): Computes the reference attitude trajectory for a general 3D inertial pointing. A corrected body frame will align with the desired reference frame.
- [attTrackingError.py](src/fswAlgorithms/attGuidance/attTrackingError/attTrackingError.c): determines S/C attitude error/difference from the reference attitude in MRP.
- [mrpRotation.py](src/fswAlgorithms/attGuidance/mrpRotation/mrpRotation.c) or [web](https://hanspeterschaub.info/basilisk/Documentation/fswAlgorithms/attGuidance/mrpRotation/mrpRotation.html): held constant angular velocity vector constant, gives MRP input/output, may need to used with [initial3DSpin.py](https://hanspeterschaub.info/basilisk/Documentation/fswAlgorithms/attGuidance/inertial3DSpin/inertial3DSpin.html)?
- [simpleNav.py](https://hanspeterschaub.info/basilisk/Documentation/simulation/navigation/simpleNav/simpleNav.html?highlight=simplenav): navigation message (attitude sigma + position x-y-z) passing, reading from S/C states

## Useful simulation examples:
- [scenarioRendezVous.py](dev/template-examples/scenarioRendezVous.py): The script illustrates how the simulation can be run for fixed periods of time after which
some flight software modules change their input subscript to switch between the three possible
attitude pointing modes 1) Hill pointing, 2) spacecraft point at the debris object and 3) sun pointing of the solar panels.
- [scenarioFormationBasic.py](dev/template-examples/scenarioFormationBasic.py) & [web](https://hanspeterschaub.info/basilisk/examples/scenarioFormationBasic.html): Demonstrates a basic method to simulate 3 satellites with 6-DOF motion and how to visualize the simulation
data in :ref:`Vizard <vizard>`.  One satellite is a 3-axis attitude controlled
satellite, while the second satellite is a tumbling space debris object.  The controlled satellite simulation components
are taken from [scenarioAttitudeFeedbackRW.py](examples/scenarioAttitudeFeedbackRW.py). The purpose of this script is to show an explicit method to
setup multiple satellites, and also show how to store the Basilisk simulation data to be able to visualize
both satellite's motions within the :ref:`Vizard <vizard>` application. Note, this scenario also illustrates how to ensure that the differential equations of motion of
the servicer and debris object are integrated at the same time (see [Advanced: Using `DynamicObject` Basilisk Modules](https://hanspeterschaub.info/basilisk/Learn/bskPrinciples/bskPrinciples-9.html)).  This is not required in this scenario
as there are no direct satellite-to-satellite dynamic interactions.; ALSO see line 248 which sync the integration of both S/Cs + demo of changing integrator from default RK4.
- [scenarioHohmann.py](dev/template-examples/scenarioHohmann.py): Hohnmann transfer for delta-V examples & Hill-pointing
- [scenarioAttLocPoint.py](examples/scenarioAttLocPoint.py): Ground location / Ground station creation + Vizard visualisation style (`stationName` & ground location vector `r_GP_P`).
- [scenarioFormationReconfig.py](examples/scenarioFormationReconfig.py) & [spacecraftReconfig.c](src/fswAlgorithms/formationFlying/spacecraftReconfig/spacecraftReconfig.c) & [web](https://hanspeterschaub.info/basilisk/Documentation/fswAlgorithms/formationFlying/spacecraftReconfig/spacecraftReconfig.html): formation flight spacecraft reconfiguring module (Via _Scheduled delta-V_ at Apogee/Perigee) with potetial useful references on messaging. (also see [web for formationBarycenter](https://hanspeterschaub.info/basilisk/Documentation/fswAlgorithms/formationFlying/formationBarycenter/formationBarycenter.html), determinethe barycenter of a swarm of satellites). This module (line 335) computes the difference in _orbital elements_ instead of cartesian x-y-z.
- [scenario_BasicOrbitFormation.py](https://hanspeterschaub.info/basilisk/examples/BskSim/scenarios/scenario_BasicOrbitFormation.html): could utilise this pre-defined structure of simulation/modules
![BSK_MultiSat_Sim](./ref-images/test_scenario_BasicOrbitFormation.svg)

## Making Python Modules:
- NEW: see [scenarioAttitudePointingPy.py](examples/scenarioAttitudePointingPy.py) line 286 for `PythonMRPPD` implementations
- See [making-pyModules.py](docs/source/codeSamples/making-pyModules.py) & [this url](https://hanspeterschaub.info/basilisk/Learn/makingModules/pyModules.html) for basic Python module creation, should be useful for creating a new controller! 
- Also see how the [mrpFeedback.py](dist3/Basilisk/fswAlgorithms/mrpFeedback.py) for controller module demo ( & if necessary [mrpFeedback.c](src/fswAlgorithms/attControl/mrpFeedback/mrpFeedback.c) for actual control laws)  (`mrpSteering` module is not useful as it relates to servo control) ; ![mrpFeedback](./ref-images/moduleIOMrpFeedback.svg)
- Flow chart from [scenarioFormationBasic from Basilisk web](https://hanspeterschaub.info/basilisk/examples/scenarioFormationBasic.html): ![basicFormationFlowChart](./ref-images/basicFormation-FlowChart.png)
- Message `rwMotorTorque`: see [web](https://hanspeterschaub.info/basilisk/Documentation/fswAlgorithms/effectorInterfaces/rwMotorTorque/rwMotorTorque.html?highlight=rwmotortorque)to map RW thrust, relates with the above flow chart; check this [link](https://hanspeterschaub.info/basilisk/Documentation/fswAlgorithms/effectorInterfaces/index.html) as well.


## Messaging between modules:
- Message subscription notes/example **I/O**:
- Control torque Output (e.g. `mrpControl.cmdTorqueOutMsg`) -> `rwMotorTorque.rwMotorTorque()` -> `reactionWheelStateEffector.ReactionWheelStateEffector()` 
- `extForceTorque.ExtForceTorque()` can be used to replace the `rwMotorTorque` & `rwStateEffector` when no actuators (RW) for command torque is defined
- To create new message, visit this [link](https://hanspeterschaub.info/basilisk/Learn/makingModules/makingModules-2.html?highlight=new%20message)
- [Message recorder](https://hanspeterschaub.info/basilisk/Learn/bskPrinciples/bskPrinciples-4.html): use this
```
someMsgRec = module.someOutMsg.recorder() # or recorder(minUpdateTime) to reduce the record time, must be lower than the recorded module update time!
scSim.AddModelToTask("taskName", someMsgRec) # need to add recorder model to sim!
```
Use `someMsgRec.times()` to get the time array of recorder, e.g. MRP against time (also explore what us `someMsgRec.timesWritten()`)

Call `someMsgRec.clear()` to clear recorder data and only add new data

Use `msgCopy = msg.read()` to create an initial copy of message/payload

To [record Module variables](https://hanspeterschaub.info/basilisk/Learn/bskPrinciples/bskPrinciples-6.html) instead of simple module I/O: `moduleLogger = module.logger(variableName, recordingTime)`, where `variableName` must be either a string or a list of strings like: 
`mod2.dummy = 1; mod2.dumVector = [1., 2., 3.]; variableName = ["dummy", "dumVector"]`; again it has `moduleLogger.times() & .clear()` functions

[](): `messaging.SomeMsg_C_addAuthor(someCModule.dataOutMsg, cStandAloneMsg)`

![addAuthor](./ref-images/message_addAuthor_C.svg)

Updated Controller Flow to State Effectors affecting EOM:
- Concerning `rwMotorTorque()` & `thrForceMapping()`/`torqueForceThrustMapping()`:  they take `CmdTorqueBodyMsg` / `CmdForceBodyMsg` IN, `ArrayMotorTorqueMsgPayload` / `THRArrayCmdForceMsg` OUT; New flowchart included below:
![Message Flow Chart from controller to EOM state effector](./ref-images/bsk_module_control_flow_controller_to_EOM_state_effectors.jpeg)


## BSKLogger & BSK Log levels:
- Consult [bskLogging.py](dist3/Basilisk/architecture/bskLogging.py), [attTrackingError.c](src/fswAlgorithms/attGuidance/attTrackingError/attTrackingError.c) and [transError.py](dev/transError.py) for implementations of BSKLogging in the terminal!
This is for aliging the `Reset()` function action of our created Python Modules with the C modules
![BSKLogging](/ref-images/bskLogger_Python_from_att_transError.png) 
- __TODO__: add back `Reset()` actions for [PIDController.py](/dev/PIDController.py) or others controllers in accordance to [mrpFeedback.c](src/fswAlgorithms/attControl/mrpFeedback/mrpFeedback.c)

## Building the simulation framework using the BSK_MultiSat structure
- [BSK_MultiSatMasters.py](dev/MultiSatBskSim/BSK_MultiSatMasters.py): User defined scenarios should inherit the classes `BSKSim` & `BSKScenario` in `BSK_MultiSatMasters`. For __general usage__: User shall build scenario files inheritting `BSK_MultiSatMasters.BSKSim` (get/set Environment, Dynamics & FSW models corresponding to `BSK_Environment_{}`, `BSK_MultiSatDynamics` & `BSK_MultiSatFsw`) & `.BSKScenario` (override its inheritted `configure_initial_conditions(), log_outputs() & pull_outputs(showPlots, SCIndex)`); Consult example [scenario_StationKeepingMultiSat.py](dev/MultiSatBskSim/scenariosMultiSat/scenario_StationKeepingMultiSat.py) for details.


## Quick Notes:
- [scenarioFormationBasic.py](dev/template-examples/scenarioFormationBasic.py) does not use `inertial3D` module for the attitude reference signal as it is already using the `hillPoint` module which also gives a constant attitude reference signal (line 335) to feed into the `attError` module.
- `thrusterStateEffector` (newer, contains time integration & can turn on in ratio of 0-100% / 0-1) v.s. `thrusterDynamicEffector` (older, without time integration, only on/off or 0/1)
- [BSK_MultiSatDynamics.py](examples/MultiSatBskSim/modelsMultiSat/BSK_MultiSatDynamics.py) & [BSK_MultiSatFsw.py](examples/MultiSatBskSim/modelsMultiSat/BSK_MultiSatFsw.py): good example module setup for gathering S/C creation, gravity body creation, thruster/RW creations & other instances (GS, batteries, power, solar panel, etc.), as well as setting up FSW related modules like `inertial3D`, `attTrackingError` & `mrpFeedback`
- [Simulation Update Debugging](https://hanspeterschaub.info/basilisk/Learn/bskPrinciples/bskPrinciples-2a.html): For testing purposes, to execute the code, this script doesn’t run the simulation for a period of time. Rather, the simulation is executed for a single time step. This is convenient in particular when testing the module input-output behavior. The command to execute Basilisk for one time step is: 
``` 
# perform a single Update on all modules
scSim.TotalSim.SingleStepProcesses() 
```
- [Task Priority & Simulation Documentation Figures](https://hanspeterschaub.info/basilisk/Learn/bskPrinciples/bskPrinciples-2b.html): Print task priority in terminal & save priority figures via:
```
# print to the terminal window the execution order of the processes, task lists and modules
scSim.ShowExecutionOrder()
# uncomment this code to show the execution order figure and save it off
# fig = scSim.ShowExecutionFigure(False)
# fig.savefig("qs-bsk-2b-order.svg", transparent=True, bbox_inches = 'tight', pad_inches = 0)
```
![taskPriority](./ref-images/task_priority_fig.svg)
- __Unit Test__: see [Unit Test for `ThrusterStateEffector`](src/simulation/dynamics/Thrusters/thrusterStateEffector/_UnitTest/test_ThrusterStateEffectorUnit.py) for how to mimic a unit test case
- Updates to create RW: in [simIncludeRW.py](dist3/Basilisk/utilities/simIncludeRW.py) & example specified in [scenarioAttitudeConstrainedManeuver.py](examples/scenarioAttitudeConstrainedManeuver.py) line 211, RW should be initialised with spin axis `gsHat_B` and position vector `rwB_B` or `varrWB_B` (if any); example of creating RWs are:

__Case 1__ - Simple 3-axis aligned:

```
#
# add RW devices
#
# Make RW factory instance
rwFactory = simIncludeRW.rwFactory()

# store the RW dynamical model type
varRWModel = messaging.BalancedWheels

# create each RW by specifying the RW type, the spin axis gsHat, plus optional arguments
maxMomentum = 0.01
maxSpeed = 6000 * macros.RPM
RW1 = rwFactory.create('custom', [1, 0, 0], Omega=0.  # RPM
                        , Omega_max=maxSpeed
                        , maxMomentum=maxMomentum
                        , u_max=0.001
                        , RWModel=varRWModel)
RW2 = rwFactory.create('custom', [0, 1, 0], Omega=0.  # RPM
                        , Omega_max=maxSpeed
                        , maxMomentum=maxMomentum
                        , u_max=0.001
                        , RWModel=varRWModel)
RW3 = rwFactory.create('custom', [0, 0, 1], Omega=0.  # RPM
                        , Omega_max=maxSpeed
                        , maxMomentum=maxMomentum
                        , u_max=0.001
                        , RWModel=varRWModel)

numRW = rwFactory.getNumOfDevices()

# create RW object container and tie to spacecraft object
rwStateEffector = reactionWheelStateEffector.ReactionWheelStateEffector()
rwStateEffector.ModelTag = "RW_cluster"
rwFactory.addToSpacecraft(scObject.ModelTag, rwStateEffector, scObject)

# add RW object array to the simulation process
scSim.AddModelToTask(simTaskName, rwStateEffector, 2)
```

__Case 2__ - Tetrahedron structure (avoid saturation?) (in original [BSK_MultiSatDynamics.py](dev/MultiSatBskSim/modelsMultiSat/BSK_MultiSatDynamicsOld.py)):
```
# We should redefine the axis of the RWs!
def SetReactionWheelDynEffector(self):
    """
    Defines the RW state effector.
    """
    # specify RW momentum capacity
    maxRWMomentum = 50.  # Nms

    # Define orthogonal RW pyramid
    # -- Pointing directions
    rwElAngle = np.array([40.0, 40.0, 40.0, 40.0]) * mc.D2R
    rwAzimuthAngle = np.array([45.0, 135.0, 225.0, 315.0]) * mc.D2R
    rwPosVector = [[0.8, 0.8, 1.79070],
                    [0.8, -0.8, 1.79070],
                    [-0.8, -0.8, 1.79070],
                    [-0.8, 0.8, 1.79070]]

    for elAngle, azAngle, posVector in zip(rwElAngle, rwAzimuthAngle, rwPosVector):
        gsHat = (rbk.Mi(-azAngle, 3).dot(rbk.Mi(elAngle, 2))).dot(np.array([1, 0, 0]))
        self.rwFactory.create('Honeywell_HR16',
                                gsHat,
                                maxMomentum=maxRWMomentum,
                                rWB_B=posVector)

    self.numRW = self.rwFactory.getNumOfDevices()
    self.rwFactory.addToSpacecraft("RWArray" + str(self.spacecraftIndex), self.rwStateEffector, self.scObject)
```

![RW Settings](./ref-images/rwSettings_simIncludeRW.png)

- Updates to create Thrusters: in [simIncludeThruster.py](dist3/Basilisk/utilities/simIncludeThruster.py) & example specified in [scenarioAttitudeConstrainedManeuver.py](examples/scenarioAttitudeConstrainedManeuver.py) line 211, RW should be initialised with `location` and `direction` vector; an example of creating thrusters is:
```
def SetThrusterDynEffector(self): 
        """
        Defines the thruster state effector.
        """
        location = [[0.0, 0.0, 0.0], 
                    [0.0, 0.0, 0.0], 
                    [0.0, 0.0, 0.0], 
                    [0.0, 0.0, 0.0], 
                    [0.0, 0.0, 0.0], 
                    [0.0, 0.0, 0.0]] # Setting thrusters positions ALL at S/C body centre for now
        direction = [[1.0, 0.0, 0.0], 
                     [-1.0, 0.0, 0.0], 
                     [0.0, 1.0, 0.0], 
                     [0.0, -1.0, 0.0], 
                     [0.0, 0.0, 1.0], 
                     [0.0, 0.0, -1.0], ] # Setting thrusters direction at +/- x,y,z direction of S/C.

        # create the thruster devices by specifying the thruster type and its location and direction
        for pos_B, dir_B in zip(location, direction):
            self.thrusterFactory.create('TEST_Thruster', pos_B, dir_B, useMinPulseTime=False)

        self.numThr = self.thrusterFactory.getNumOfDevices()

        # create thruster object container and tie to spacecraft object
        self.thrusterFactory.addToSpacecraft("thrusterFactory", self.thrusterDynamicEffector, self.scObject)
        
```

# Resolved issues:
- Force mapping issues with the `forceTorqueThrForceMapping` module, see [test_forceTorqueThrForceMapping.py](src/fswAlgorithms/effectorInterfaces/forceTorqueThrForceMapping/_UnitTest/test_forceTorqueThrForceMapping.py) & standalone test [try_forceTorqueThrForceMapping.py](src/fswAlgorithms/effectorInterfaces/forceTorqueThrForceMapping/_UnitTest/try_forceTorqueThrForceMapping.py); link to module from [BSK net](https://hanspeterschaub.info/basilisk/Documentation/fswAlgorithms/effectorInterfaces/forceTorqueThrForceMapping/forceTorqueThrForceMapping.html?highlight=thrarraycmdforce); some thruster configurations are based on general location/unit vector definitions or scenarios like [scenarioFormationReconfig.py](dev/template-examples/scenarioFormationReconfig.py) (but without the `forceTorqueThrForceMapping` module used here) 
- __z-component command force__ from hill frame - inertial frame conversion in the DCM to investigate, see line 146-148 in [PIDController.py](dev/PIDController.py)
- __SRL Robot initial condition configs__ should be with inclination = 0.0 deg so that the z-components are neglects fully. (fixed in [SRL_config.json](dev/MultiSatBskSim/scenariosMultiSat/simInitConfig/SRL_config.json))

# Current issues:
- __Thruster log__ giving zero values despite connecting to seemingly correct messaging structures, see line 334 in [MultiSat_test_scenario.py](dev/MultiSatBskSim/scenariosMultiSat/MultiSat_test_scenario.py), __36 thruster arrays__ created and passed on messaging `THRArrayCmdForceMsg` instead of available number of thrusters (6 or 8 thrusters); Comparing[forceTorqueThrForceMapping.c](src/fswAlgorithms/effectorInterfaces/forceTorqueThrForceMapping/forceTorqueThrForceMapping.c) & [thrForceMapping.c](src/fswAlgorithms/effectorInterfaces/thrForceMapping/thrForceMapping.c) might give insights
- __PID Tuning__ at function `SetTransController()` (line 329) of [BSK_MultiSatFsw.py](dev/MultiSatBskSim/modelsMultiSat/BSK_MultiSatFsw.py) 
- __Problematic configuration for spacecraft initial condition__ using cartesians inertial coordinates which gives strange visualization in terms of initial spacecraft position in Vizard (check indexing in line 114 of `setInitialCondition()` in [scConfig.py](dev/scConfig.py))
- __Include a Disturbance Module__: New Python module for white noise generation of thruster actuation errors

You can run the following command to test the very first few time steps:
```
python dev/MultiSatBskSim/scenariosMultiSat/MultiSat_test_scenario.py "dev/MultiSatBskSim/scenariosMultiSat/simInitConfig/init_config.json" .0001
```