There are three different flavors of gibberwocky, with each targeting a different application. Although there are minor
application-specific differences between these versions the majority of the applications work fairly similarly.
The primary difference is the targets for sequencing. In gibberwocky.Live we primarily sequence `tracks`; in 
gibberwocky.Max we primarily sequence `namespaces`; and in gibberwocky.MIDI we primarily sequence `channels`.

These differences aside, the API / syntax for sequencing and modulation is basically identical, with just a few different
keywords. There are also many common objects that are used identically across all three systems.

In this reference, we start by documenting the objects that are unique to each system, and then document ones
that are shared. Note that for almost every object in gibberwocky, each method can be *sequenced* by attaching a call
to `.seq()` to it. This is covered in the sequencing tutorials included in each envrionment.

#Gibberwocky.Live
In gibberwocky.live, the Live set is primarily manipulated through the global `tracks` and `returns` array, as well as the `master` object. Each track and return (as well as the master) has an array of `devices` that can be controlled; in turn each device has individual parameters that can be sequenced and modulated.

### Device
Devices are stored in the `devices` property of each individual track. They primarily represent softsynths and audio effects. In addition to a few convenience methods, devices in gibberwocky provide programmatic access to all of their parameters, so that they can all be sequecned and modulated.

```javascript
// toggle an effect on/off every measure
tracks[0].devices[1].toggle.seq( 1, 1 )

// turn the third device in the track's device chain off
tracks[0].devices[2].off()

// set value of parameter named Volume on device
tracks[0].devices[0].Volume( .5 )
```

#### Properties
* _parameters_: Object. Read-only. An array of all the parameters on a device. It is important to note that all parameters are also exposed on the device by name, which is an easier way to access them e.g. `tracks[0].devices[0][ 'Filter Cutoff' ]`. In practice the `parameters` array is not typically used.

#### Methods
* _on_: Turn the device on.
* _off_: Turn the device off.
* _toggle_: Toggle the on / off state of the device.
* _galumph_: Randomly pick a parameter of the device and assign a random number to it.

### Master
Provides access to the master track through the global `master` object and any effect devices placed in its device chain. 

#### Properties
* _devices_: Array. Read-only. An array of all the devices in the track's device chain, *with the exception of the gibberwocky plugin itself*. See the [Device description](#Gibberwocky.Live-Device) for more information.

#### Methods
* _on_: Turn the device on.
* _volume_: Float. Set the volume of the track.
* _pan_: Float. Set the panning of the track. 0 = left, .5 = center, 1 = right.
* _start_: Starts any sequences running on the track that may have been previously halted.
* _stop_: Stops all sequences currently running on the track.

### Return
The global `returns` array provides access to the return objects, which are tracks that instrument tracks can route audio to using their `sends`. 

#### Properties
* _sends_: Array. Read-only. An array of all the sends available for the retrun. For example, to ramp the value of send #1 over the course of two beats:
```javascript
returns[ 0 ].sends[ 1 ]( beats( 2 ) )
```

* _devices_: Array. Read-only. An array of all the devices in the return's device chain, *with the exception of the gibberwocky plugin itself*. See the [Device description](#Gibberwocky.Live-Device) for more information.

#### Methods
* _volume_: Float. Set the volume of the track.
* _pan_: Float. Set the panning of the track. 0 = left, .5 = center, 1 = right.
* _mute_: Integer. A value of `0` mutes the track, while `1` unmutes it.
* _solo_: Integer. A value of `1` solos the track, while `0` unsolos it. Note that soloing a track will *not* unsolo other tracks that have been previously soloed; you must do this manually.
* _start_: Starts any sequences running on the track that may have been previously halted.
* _stop_: Stops all sequences currently running on the track.
* _select_: Select the track in Live's GUI window and focus it.

### Track
All instrument tracks in a Live set are stored in the global `tracks` array. To access an individual track, you can either
use an array index or use the track's name. For example, assuming your first track was named `1-Impulse 606` you could access
it using `tracks[0]` or `tracks['1-Impulse 606']`.

Track objects in gibberwocky accept a variety of messages for triggering notes on any instrument on a track. They also provide access
to every control on every device, enabling you to set values and/or modulate them. However, in order to control an instrument or device
on a track, the track in question *must have a gibberwocky.midi plugin as the first device in the track's device chain*.

See tutorials #1, #2, and #5 for more information.

```javascript
// send notes to track 1
tracks[0].note.seq( 0, 1/4 )

// set the value of send #2 on track 2
tracks[1].sends[ 1 ]( .5 )

// modulate the Transpose property of an Impulse device on track 1
tracks[0].devices['Impulse 606']['Global Transpose']( lfo( .2 ) )
```

#### Properties
* _sends_: Array. Read-only. An array of all the sends available for the track. For example, to ramp the value of send #1 over the course of two beats:
```javascript
tracks[0].sends[0]( beats( 2 ) )
```

* _devices_: Array. Read-only. An array of all the devices in the track's device chain, *with the exception of the gibberwocky plugin itself*. See the [Device description](#Gibberwocky.Live-Device) for more information.

#### Methods
* _note_: Integer or String. Play a note using gibberwocky's built-in theory system. See the harmony tutorial for more information. Strings take the form of note name + octave, such as `c4` or `eb2`.
* _midinote_:  Integer. Triggers a note using MIDI note numbers.
* _chord_: String or Array. Play a chord using gibberwocky's built-in theory system; see the harmony tutorial for more information. Strings take the form of note name + octave + quality + extension, for example `c4maj7` or `bb3dim9`. Alternatively an array of scale indices can be passed.
* _midichord_:  Array. Trigger a list of MIDI note values simultaneously.
* _duration_: Integer (ms). Set the length of subsequent notes and chords that are played on this track in milliseconds.
* _velocity_: Integer {0,127}. Set the MIDI velocity value of subsequent cnotes and chords that are played on this track.
* _volume_: Float. Set the volume of the track.
* _pan_: Float. Set the panning of the track. 0 = left, .5 = center, 1 = right.
* _mute_: Integer. A value of `0` mutes the track, while `1` unmutes it.
* _solo_: Integer. A value of `1` solos the track, while `0` unsolos it. Note that soloing a track will *not* unsolo other tracks that have been previously soloed; you must do this manually.
* _start_: Starts any sequences running on the track that may have been previously halted.
* _stop_: Stops all sequences currently running on the track.
* _select_: Select the track in Live's GUI window and focus it.


#Gibberwocky.Max
Two global arrays and one constructor are the primary ways of interacting with gibberwocky.max. To use gibberwocky.max, you must first instantitate the gibberwocky.max object inside of your Max patch. This object has a single inlet, which is used to open the gibberwocky live-coding interface, and a number of outlets. The first outlet ouputs messages sent from objects created using the [Namespace](#gibberwocky.max-namespace) constructor. The remaining outlets output gen~ graphs and are accessed via the global `signals` array. Finally, any Max object in a patch that also contains a gibberwocky object and has a scripting name assinged to it can be accessed through the global `params` array.

The combination of these three entry points provides a great deal of flexibility when integrating gibberwocky with your Max patch. You can create complex routing schemes and send messages as needed, you can create complex signal graphs using gen~ expressions, and you can easily target any object with a scripting name and send it messages.

### Namespace
The `namespace` constructor lets you create a messages that will be outputted from the first outlet of the gibberwocky.max object, with a prefix of your choice. For example, given:

```javascript
foo = namespace('foo')
foo.bar.seq( [0,1], 1 )
foo.baz.seq( -1, 1/2 )
```

... the following messages would be alternately sent out the first outlet of the gibberwocky Max object:

```
foo bar 0
foo baz -1
foo baz -1
foo bar 1
foo baz -1
foo baz -1
foo bar 0
...etc.
```

In effect, the sole argument to the `namespace` constructor determines the first word in the message it sends to Max, while the next property access adds another word to the message.  `namespace('foo').bar( 1000 )` would send the message 'foo bar 1000' immediately.

In addition to using your own words for namespace routing, each namespace has several methods that are used to output MIDI noteon / noteoff messages.

#### Methods
* _note_: Integer or String. Play a note using gibberwocky's built-in theory system. See the harmony tutorial for more information. Strings take the form of note name + octave, such as `c4` or `eb2`.
* _midinote_:  Integer. Triggers a note using MIDI note numbers.
* _chord_: String or Array. Play a chord using gibberwocky's built-in theory system; see the harmony tutorial for more information. Strings take the form of note name + octave + quality + extension, for example `c4maj7` or `bb3dim9`. Alternatively an array of scale indices can be passed.
* _midichord_:  Array. Trigger a list of MIDI note values simultaneously.
* _duration_: Integer (ms). Set the length of subsequent notes and chords that are played on this track in milliseconds.
* _velocity_: Integer {0,127}. Set the MIDI velocity value of subsequent cnotes and chords that are played on this track.


### Signals
There is a global `signals` array that provides access to all the outlets in the gibberwocky.max object except for the first one (which is used for messaging). As the name suggests, these outlets are all used to output audio signals created using gen~ graphs, as opposed to irregular messages. For a detailed reference of ugens available, please see the [gen~ reference](https://docs.cycling74.com/max7/vignettes/gen~_operators). gen~ graphs can also be sequenced; see the gen~ modulation tutorial for more information.

```javascript
// send a sine oscillator out the second outlet
signals[0]( cycle( .5 ) )

// send a sawtooth to the third outlet and sequence
// its frequency
mysin = phasor( 2 )
signals[1]( mysin )

mysin[0].seq( [.5,2,4,8], 1 )
```

### Params
Any object can be 'named' in Max/MSP by selecting it, opening up its info panel, and providing it with a scripting name. In addition, `receive` objects are named automatically when instantiated. gibberwocky provides a dictionary of these named parameters and `receive` objects, enabling easy messaging to them. For example, the gibberwocky.max help patch includes knobs with the scripting names 'White_Queen' and 'Red_Queen'. We can sequence messages to them as follows:

```javascript
params['White_Queen'].seq( [0,1,2,3], 1/4 )

params['Red_Queen'].seq( 1, Euclid(5,8) )
```

#Gibberwocky.MIDI
Programming in gibberwocky.midi is primarily done by sequencing MIDI `channel` objects, which are accessible through a global `channels` array.

### Channel
Each `channel` object represents a MIDI channel on the MIDI port selected for output (selection is made in gibberwocky's sidebar, under the MIDI tab).

```javascript
// play midinote 60 every quarter note
channels[0].midinote.seq( 60, 1/4 )

// change the velocity every measure
channels[0].velocity.seq( Rndi(0,127), 1 )

// sequence changes to CC 0
channels[0].cc0.seq( [0,32,64,127], 1/2 )

// assign a modulation graph to CC 1
channels[0].cc1( lfo( 4 ) )
```

#### Methods
* _note_: Integer or String. Play a note using gibberwocky's built-in theory system. See the harmony tutorial for more information. Strings take the form of note name + octave, such as `c4` or `eb2`.
* _midinote_:  Integer. Triggers a note using MIDI note numbers, where 60 represents middle C.
* _chord_: String or Array. Play a chord using gibberwocky's built-in theory system; see the harmony tutorial for more information. Strings take the form of note name + octave + quality + extension, for example `c4maj7` or `bb3dim9`. Alternatively an array of scale indices can be passed.
* _midichord_:  Array. Trigger a list of MIDI note values simultaneously.
* _duration_: Integer (ms). Set the length of subsequent notes and chords that are played on this channel in milliseconds.
* _velocity_: Integer {0,127}. Set the MIDI velocity value of subsequent notes and chords that are played on this channel.
* _start_: Starts any sequences running on the channel that may have been previously halted.
* _stop_: Stops all sequences currently running on the channel.

### Clock
Unlike the other gibberwocky systems, gibberwocky.midi can either be controlled by an external clock source or by its own internal clock. The global `Clock` object
represents this internal clock. It has a single property, `bpm`, that can be set to determine the tempo used by gibberwocky.

```javascript
channels[0].midinote.seq( 60, 1/4 )

Clock.bpm = 240 // change from 120 (default) to 240 bpm

Clock.bpm = 60 // change to 60 bpm
```


# Common Objects
###Arp

The Arp object is an arpeggiator providing a variety of controls. See the Arpeggio tutorial for detailed information.

Example:
```javascript
// 1,4,6, and 7th scale degrees,
// traveling up and down over 2 octaves
a = Arp( [0,3,5,6], 2, 'updown' )

// Live: play arpeggio on track 0 with 1/16th notes
tracks[0].note.seq( a, 1/16 )

// Max: play arpeggio through namespace 'foo' with 1/32 notes
namespace( 'foo' ).note.seq( a, 1/32 )

// c-diminished-7 chord in the third octave, 
// traveling up over 4 octaves
a2 = Arp( 'c3dim7', 4, 'up' )

// MIDI: play argpeggio on MIDI channel 1 using Euclidean Rhythm
channels[0].note.seq( a2, Euclid( 5,8,1/16 ) )
```

#### Constructor
* _values_ : Array. The base values outputted by the sequencer. These values will be duplicated and transposed over a provided `octaves` argument.
* _octaves_ : Literal. The number of octaves to play the Arpeggio over.
* _pattern_ : String. The playback pattern used by the Arpeggiator.

#### Properties

* _pattern_ : String. Default value is 'up'. Controls the direction of notes. Possible values are 'up', 'down', 'updown' and 'updown2'. For 'updown2', the top and bottom notes of the arpeggio won't be repeated as it changes directions.
* _octaves_ : Number. Default is 1. The number of octaves that the arpeggio will span.
* _notes_: A Gibber Pattern object that holds all the notes the arpeggiator sequences, as determined by the chord passed to it, the `mult` property and the arpeggiation `pattern` used. 
 
#### Methods
The Arp object uses a [Pattern](#common-objects-pattern) object as a mixin; this means that you can call most methods for
pattern manipulation (such as reverse, shuffle, rotate, transpose etc.) on an object created by the Arp
constructor. See [Pattern](#common-objects-pattern) for more information.

###Euclid

The Euclid constructor creates a Pattern object that outputs [Euclidean Rhythms](http://archive.bridgesmathart.org/2005/bridges2005-47.pdf).
The algorithm for these rhythms specifies that a certain number of pulses will occur over a certain number of time slots. For example, the
specification {5,8} will result in `x.xx.xx.` where an `x` represents a pulse and a `.` represents a rest. See the Euclidean tutorial for more
inforamation.

Example:
```javascript
// Live: play midinote 60 on track 0 with Euclidean rhythm
tracks[0].midinote.seq( 60, Euclid( 3,9 ) )

// Max: play notes through namespace 'foo' with Euclidean rhythm
namespace( 'foo' ).note.seq( [0,7], Euclid(5,12,1/8) )

p = Euclid(5,16)

// MIDI: play midinote 60 on MIDI channel 1 using Euclidean rhythm
channels[0].note.seq( 60, p )

// rotate Euclidean rhythm one position to the right each measure
p.rotate.seq( 1,1 )
```

#### Constructor
* _pulses_ : Int. The number of pulses (aka 'hits' or 'notes') in the generated pattern.
* _slots_ : Int. The number of slots in the pattern. If no third argument is passed, this also determines the duration
of each slot. For example, `Euclid(3,8)` means three pulses spread over eight slots, with each slot lasting an eighth note in duration.
* _slotDuration_ : Optional. Float. When passed to the constructor, this value determines the duration of each slot. For example, 
`Euclid(3,8,1/16)` means that there will be three pulses distributed among eight slots, with each slot lasting a 1/16th note in duration.

#### Methods
The Euclid object uses a [Pattern](#common-objects-pattern) object as a mixin; this means that you can call most methods for
pattern manipulation (such as reverse, rotate etc.) on an object created by the Euclid.
constructor. See [Pattern](#common-objects-pattern) for more information.

###Pattern

Patterns are functions that output values from an internal list that is typically passed as an argument when the pattern is first created. These lists can be manipulated in various ways, influencing the output of the patterns. Alternatively, `filters` placed on the pattern (each filter is simply a function expected to return an array of values) can also change the output of the pattern dynamically, without affecting its underlying list.

Whenever you sequence any method or property in Gibber, Pattern(s) are created behind the scenes to handle sequencer output. All methods of the pattern object can be additionally sequenced; you can also sequence the `start`, `end`, and `stepSize` properties. 


Example:
```javascript
// two patterns are created when notes are sequenced. 
// The `values` pattern determines the output,
// while the `timings` pattern determines timing.
// In gibberwocky.MIDI:
channels[0].note.seq( [0,1,2,3], [1/4,1/8], 0 )

channels[0].note[0].values.reverse()
// a.note.values now equals [3,2,1,0]

channels[0].note[0].values.scale( 2 )
// a.note.values now equals [6,4,2,0]

// You can also pass Patterns directly to sequencing calls
// In gibberwocky.Max:
p = Pattern( 4,5,6,7 )

namespace('bar').note.seq( p, 1/4 )

// rotate the pattern -> 5,6,7,4 -> 6,7,4,5 -> 7,4,5,6 etc.
p.rotate.seq( 1,1 )
```

The gibberwocky Pattern tutorial has more information and examples of the Pattern object in use.

#### Constructor
* _values_ : ...arg list. The Pattern constructor takes a flexible number of arguments; each argument is added to an internal
list of possible output values for the pattern.


#### Properties

* _start_ : Int. The first index of the underlying list that will be used for output. For examples, if a pattern has values `[0,1,2]` and the `start` property is 1 then the pattern will output 1,2,1,2,1,2,1,2... skipping the 0-index item in the list.
* _end_ : Int. The last index of the underlying list that will be used for output. For examples, if a pattern has values `[0,1,2]` and the `end` property is 1 then the pattern will output 0,1,0,1,0,1... skipping the 2-index item in the list.
* _phase_ : Int. This is the next index in the underlying list that the pattern will output.
* _values_ : Array. The underlying list used by the pattern to determine output.
* _original_ : Array. A copy of the original values used to instantiate the pattern. These values can be used to restore the pattern to its original state after any transformations have been made using the `pattern.reset()` method.
* _storage_ : Array. The current permutation of a pattern can be stored at any time with a call to the `pattern.store()` method. Every stored pattern is placed in the `pattern.storage` array. The `pattern.switch` method can be used to switch between stored permutations.
* _stepSize_ : Float. Default 1. The amount that the phase is increased / decreased by each time the pattern outputs. For example, a `pattern.stepSize` of -1 means that the pattern will play in reverse. A `stepSize` of .5 means that each value in the list will be repeated before advancing to the next index.
* _integersOnly_ : Boolean. Default false. In certain cases (for example, scale degrees) we can ensure that any transformations applied to the pattern only result in integer values by setting this property to be true. For example, if a pattern has the original values of [2,3,4,5] and `pattern.scale(.5)` is applied, the values would normally then become [1,1.5,2,2.5]. By setting the value of `pattern.integersOnly` to be true, the values instead become [1,2,2,3] as the floats are rounded up.

 
#### Methods

* _reverse_: Reverse the current ordering of the pattern's `values` array.
* _range_( Array, or Int,Int ): Set both the start and end properties with a single method call. `pattern.range` can be called in two forms. In the first, both the start and the end values are passed as separate arguments. In the second, a single array is passed containing the start and the end values. This allows the range to be easily sequenced with calls to Rndi, for example: `pattern.range.seq( Rndi( 0,5,2 ), 1/2 )`. Note that if the value for start is lower than the value for end, this method will automatically switch the values.
* _set_( Array or list of values): Replace the current `pattern.values` array with new members.
* _repeat_( List of values ): Repeat certain values (not indices) in an array a certain number of times. When passing arguments, the even numbered arguments are the values you want to have repeated, while the odd values represent how many times each one should be played. For example, given the pattern [0,2,4] and the line `pattern.repeat( 0,2,4,2)` the pattern output would be 0,0,2,4,4 through one cycle (both 0 and 4 repeated ). This is particularly useful to line up odd time values; for example, if a pattern has 1/6 notes you can specify a repeat of 3 to make sure it lines up with a 1/2 note grid.
* _reset_: Return values to their original state at pattern creation.
* _store_: Push the current permutation of the pattern to the `pattern.storage` array.
* _switch_( Int ): Switch the values outputted by the pattern to a stored index in the `pattern.storage` array
* _transpose_ ( Number ): Add argument to all numerical values in the pattern. If a pattern consists of an array of chords, each member of each chord will be modified. For example [[0,2,4], [1,3,5]] transposed by 1 becomes [[1,3,5], [2,4,6]]
* _scale_ ( Number ): Multiply all numerical values in the pattern by argument. If a pattern consists of an array of chords, each member of each chord will be modified. For example [[0,2,4], [1,3,5]] scaled by 2 becomes [[0,4,8], [2,6,10]]
* _shuffle_: Randomize the order of values found in the `pattern.values` array.
* _flip_: Change the positions of pattern members so that the position of the highest member becomes the position of the lowest member, the position of the second highest becomes the position of the second lowest etc.
* _invert: Similar to the technique from serialist compositions. Assume that the zero-index member of the pattern is our axis. Flip all other pattern members to the other side of the access. Thus, a member that is two higher than the zero-index member now becomes two lower.
* _rotate_( Int ): Shift the positions of all pattern members by the argument. For example, a pattern of `[0,1,2,3]` that is rotated by 1 becomes `[3,0,1,2]`.

###Scale
The `Scale` object is a global singleton that lets you control musical harmony in gibberwocky. By default, notes created using the `note` and `chord` methods in
gibberwocky use C minor, starting in the fourth octave, as their basis for generating notes. So, for example, a call to xxx.note( 0 ) will generate note `c4` while
a call to xxx.note(2) will generate `eb4`. The `Scale` object has `root` and `mode` functions that can be sequenced over time. You can also easily define your own modes; see the gibberwocky harmony tutorial for more information.

```javascript
// MIDI sequence:
channels[0].note( [0,1,2,3,4,5,6,7], 1/8 )

Scale.root.seq( ['c4','eb4'], 2 )
Scale.mode.seq( ['aeolian', 'lydian', 'locrian'], 2 )
```

#### Methods
* _root_: String. Change the base note for the current scale. The argument is the note name followed by an octave; for example: `c4`, `fb2`, or `g#1`.
* _mode_: String. Set the mode of the global scale. Possible values include: `major, minor, ionian, dorian, phrygian, lydian, mixolydian, aeolian, locrian, chromatic, wholeHalf, halfWhole`.

###Score

The Score object lets you define a timeline of events to be executed. Each event is a function that is called; these functions can play individual notes, create and start sequences playing, and even create other Score objects and start them. In addition to a list of events, time values are provided that tell the Score object how long to wait until it triggers its next event. These events and timestamps are listed in alternating order and passed to the Score constructor.

See the gibberwocky Score tutorial for more information.

```javascript
// MIDI score:
myscore = Score([
  0, ()=> channels[0].midinote( 60 ),
  1, ()=> channels[0].midinote( 72 ),
  1, ()=> channels[0].midinote( 48 ),
  1, ()=> channels[0].midinote.seq( [48,60,72], 1/8 ),  
])
myscore.start()
```

#### Constructor
* _score_ : Array. The `Score` constructor accepts a single argument, an array storing timestamps associated with events. Each event is a function, while each timestamp provides an amount of time to wait before launching an event, relative to the last event that was played. In the above example, events are triggered on subsequent measures. The special value `Score.wait` can be passed as a timestamp to tell the score to stop playback until the `.next()` method is called on the score object.

#### Properties
* _wait_: Number. Read-only. A special value that can be included in scores to stop playback until the `.next()` method is called.

#### Methods
* _start_: If score playback is stopped, start it.
* _stop_:  If the score is running, stop it.
* _next_: This method stops scores that have been halted after receiving a `Score.wait` message in their timeline.
* _combine_: ... argument list of Score objects. This method lets you combine multiple scores together. For example, if you use different scores to arrange different parts of a song, the combine method can be used to put them together in the end.

```javascript
verse  = Score([ 1, channels[0].midinote.seq( 60, 1/8 ) ] )
chorus = Score([ 1, channels[0].midinote.seq( 72, 1/8 ) ] )
bridge = Score([ 1, channels[0].midinote.seq( 48, 1/8 ) ] )

song = Score.combine( verse, chorus, verse, chorus, bridge, chorus )
song.start()
```
###Seq

The Seq object is a standalone sequencer that can schedule calls to methods and properties on a target object. Alternatively, it can be used to quickly sequence repeated calls to an anonymous function. Scheduling occurs at audio-rate and may be modulated by audio sources.

Sequencers can work in one of two ways. In the first, anonymous functions can easily be scheduled to execute repeatedly. To create a `Seq` that accomplishes this, simply pass the constructor two values: the function to be executing and an array of time values to use for scheduling. In the second mode, four values are expected: a `values` array or literal, a `timings` array or literal, a `key` specifying the method to be sequenced, and the `target` object that owns the method.

```javascript
// MIDI: sequence four notes with alternating rhythms
a = Seq( [0,1,2,3], [1/4,1/2], 'note', channels[0] ).start()

// Live: sequence four notes with alternating rhythms
b = Seq( [0,1,2,3], [1/4,1/2], 'note', tracks[0] ).start()

// Max: sequence four notes with alternating rhythms
c = Seq( [0,1,2,3], [1/4,1/2], 'note', namespace('foo') ).start()

// ALL: sequence anonymous function that repeats every two measures
let count = 0
d = Seq( ()=> { Gibber.log( count++ ) }, 2 ).start()
```

#### Constructor
The Seq object has an overloaded constructor that accepts either four or two values. The two value option is only
for scheduling the repeated execution of functions.

* _values_ : Array, Literal, or Function. The value(s) outputted by the sequencer. This can either be an array of values, a single literal,
or a function that will be executed to provide a value... such functions should always return a valid value. Returning a value of `null` means
that the sequencer will not generate output for a particular execution.
* _timings_ : Array, Literal, or Function. The scheduling for the sequencer. This can either be an array of values, a single literal,
or a function that will be executed to provide a value.
* _key_ : String. The method or property that the sequencer will use on the `target` object.
* _target_ : Object. This determines the object that the sequencer will either execute a method on or set a property value on, depending on
the value of `key`.


* _values_ : Array or Function. Function(s) that will be called by the sequencer.
* _timings_ : Array, Literal, or Function. The scheduling for the sequencer. This can either be an array of values, a single literal,
or a function that will be executed to provide a value.

#### Properties
* _values_: Array, Function, or Number. The values used by the sequencer. This is a Gibber [Pattern][pattern] object.
* _timings_: Array, Function, or Number. The scheduling used by the sequencer. This is a Gibber [Pattern][pattern] object.
* _key_: String. The name of the method the sequencer calls on the `target` object.
* _target_ : Object. When the Seq object has a target, it can be used to change properties and call methods on that `target` object.
 
#### Methods
* _delay_: Delay the start of the sequencer by an argument value. For example, to start a pattern with a quarter note offset:
```javascript
a = Seq( ()=> Gibber.log('hi'), 1/2 )
  .delay( 1/4 )
  .start()
```
* _start_: If the sequencer is stopped, start it.
* _stop_:  If the sequencer is running, stop it.
* _clear_: Stops sequencer and (hopefully) removes associated annotations.

# Common Functions

### rndi
Returns a single random integer.

#### Arguments
* _min_ : Integer, default = 0. The lower bound of the range of random numbers that can be generated.
* _max_ : Integer, default = 1. The upper bound of the range of random numbers that can be generated.

### rndf
Returns a single random floating point number.

#### Arguments
* _min_ : Float, default = 0. The lower bound of the range of random numbers that can be generated.
* _max_ : Float, default = 1. The upper bound of the range of random numbers that can be generated.

### Rndi
Returns a function that, when executed, returns a random integer within a set of bounds. This is useful
for sequencing, so that every time the function is called a new random value is generated.

#### Arguments
* _min_ : Integer, default = 0. The lower bound of the range of random numbers that can be generated.
* _max_ : Integer, default = 1. The upper bound of the range of random numbers that can be generated.
* _size_: Integer, default = 1. The number of random integers to generate. If `size` is set to more than one the generated function will return an array.

### Rndf
Returns a function that, when executed, returns a random float within a set of bounds. This is useful
for sequencing, so that every time the function is called a new random value is generated.
#### Arguments
* _min_ : Float, default = 0. The lower bound of the range of random numbers that can be generated.
* _max_ : Float, default = 1. The upper bound of the range of random numbers that can be generated.
* _size_: Float, default = 1. The number of random integers to generate. If `size` is set to more than one the generated function will return an array.

### log
Logs a message to the console in the gibberwocky sidebar, in addition to the developer's console of the browser.

#### Arguments
* _text_ : String or Number. The text that should be printed to gibberwocky's console.