# FMOD Studio integration for Godot

A Godot C++ module that provides an integration and wrapper around the FMOD Studio API.

FMOD is an audio engine and middleware solution for interactive audio in games. It has been the audio engine behind many titles such as Transistor, Pyre and Celeste. [More on FMOD's website](https://www.fmod.com/).

This module exposes most of the Studio API functions to Godot's GDScript and also provides helpers for performing common functions like attaching Studio events to Godot nodes and playing 3D/positional audio. _It is still very much a work in progress and some API functions are not yet exposed._ Feel free to tweak/extend it based on your project's needs.

## Installing the module

1. Clone the repository.
2. [Download the FMOD Studio API](https://www.fmod.com/download) (You need to create an account) and install it on your system.
3. Copy the contents of the api directory of the FMOD API into the repository's api directory. On Windows this is (usually) found at `C:/Program Files (x86)/FMOD SoundSystem/FMOD Studio API Windows/api`
4. Clone the latest version of Godot from the [master branch](https://github.com/godotengine/godot). At the time of writing this was Godot 3.1 beta.
5. `cd` into the source directory and create a new directory named `fmod` inside modules folder `godot/modules/fmod`
6. Copy the contents of the Godot FMOD repository (with the FMOD API now included) into this directory.
7. Recompile the engine. For more information on compiling the engine, refer to the [Godot documentation](https://docs.godotengine.org/en/latest/development/compiling/index.html).
8. Place the FMOD library files within the `bin` directory for Godot to start. Eg. on Windows these would be `fmod64.dll` and `fmodstudio64.dll` for a 64-bit build of Godot.

FMOD should now be integrated. If you are tweaking/extending the module it is faster to compile it as a dynamic library instead of a static library. Refer to the Godot documentation for more details on this.

## Using the module

### Basic usage

```gdscript
extends Node

# create an instance of the module
# ideally this has to be done in an AutoLoad script
# as that way you'll be able to call FMOD functions from any script
# refer to the demo project provided
var FMOD = Fmod.new()

func _ready():
	# set up FMOD
	FMOD.system_set_software_format(0, Fmod.FMOD_SPEAKERMODE_STEREO, 0)
	# initializing with the LIVE_UPDATE flag lets you
	# connect to Godot from the FMOD Studio editor
	# and author events in realtime
	FMOD.system_init(1024, Fmod.FMOD_STUDIO_INIT_LIVEUPDATE, Fmod.FMOD_INIT_NORMAL)

	# load banks
	# place your banks inside the project directory
	FMOD.bank_load("./Banks/Desktop/Master Bank.bank", Fmod.FMOD_STUDIO_LOAD_BANK_NORMAL)
	FMOD.bank_load("./Banks/Desktop/Master Bank.strings.bank", Fmod.FMOD_STUDIO_LOAD_BANK_NORMAL)

	# register listener
	FMOD.system_add_listener($Listener)

	# play some events
	FMOD.play_one_shot("event:/Car engine", $SoundSource1)
	FMOD.play_one_shot("event:/Waterfall", $SoundSource2)
```

### Calling Studio events

Following is an example of an event instance called manually (ie. not directly managed by the integration). These instances are identified by a unique ID in the form of a string that you must generate in script ideally through a UUID generator. You could write one yourself or use something like [UUID for Godot](https://github.com/binogure-studio/godot-uuid). Note that the string could be anything as long as it is unique within the current instance pool. Remember to release the instance once you're done with it.

```gdscript
# unique identifier used to id your event instance
# throughout its lifecycle
var my_music_event = UUID.new()

# create an event instance associated with the unique identifier
# this is a music event that has been authored in the Studio editor
FMOD.event_create_instance(my_music_event, "event:/Waveshaper - Wisdom of Rage")

# start the event
FMOD.event_start(my_music_event)

# wait a bit
yield(music_state_timer, "timeout")

# setting an event parameter
# in this case causes the music to transition to the next phase
FMOD.event_set_parameter(my_music_event, "State", 2.0)

# wait a bit
yield(music_timer, "timeout")

# stop the event
FMOD.event_stop(my_music_event, Fmod.FMOD_STUDIO_STOP_ALLOWFADEOUT)
# release the event
FMOD.event_release(my_music_event)
```

### Using the integration helpers

These are helper functions provided by the integration for attaching event instances to Godot Nodes for 3D/positional audio. The listener position and 3D attributes of any attached instances are automatically updated every time you call `FMOD.system_update()`. Instances are also automatically cleaned up once finished so you don't have to manually call `event_release()`.

```gdscript
# play an event at this Node's position
# 3D attributes are only set ONCE
# parameters cannot be set
FMOD.play_one_shot("event:/Footstep", self)

# same as play_one_shot but lets you set initial parameters
# subsequent parameters cannot be set
FMOD.play_one_shot_with_params("event:/Footstep", self, { "Surface": 1.0, "Speed": 2.0 })

# play an event attached to this Node
# 3D attributes are automatically set every frame (when update is called)
# parameters cannot be set
FMOD.play_one_shot_attached("event:/Footstep", self)

# same as play_one_shot_attached but lets you set initial parameters
# subsequent parameters cannot be set
FMOD.play_one_shot_attached_with_params("event:/Footstep", self, { "Surface": 1.0, "Speed": 2.0 })

# attaches a manually called instance to a Node
# once attached 3D attributes are automatically set every frame (when update is called)
FMOD.attach_instance_to_node(uuid, self)

# detaches the instance from its Node
FMOD.detach_instance_from_node(uuid)
```

## Issues

This project is still a work in progress and is probably not yet ready for full-blown production. If you run into issues (crashes, memory leaks, broken 3D sound etc.) open a issue or submit a PR with your fix.