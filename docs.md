There are three different flavors of gibberwocky, with each targeting a different application. Although there are minor
application-specific differences between these versions the majority of the applications work fairly similarly.
The primary difference is the targets for sequencing. In gibberwocky.Live we primarily sequence `tracks`; in 
gibberwocky.Max we primarily sequence `namespaces`; and in gibberwocky.MIDI we primarily sequence `channels`.

These differences aside, the API / syntax for sequencing and modulation is basically identical, with just a few different
keywords. There are also many common objects that are used identically across all three systems.

In this reference, we start by documenting the objects that are unique to each system, and then document ones
that are shared.

#Gibberwocky.Live

#Gibberwocky.MIDI

#Gibberwocky.Max

# Common Objects
###Arp

The Arp object is an arpeggiator providing a variety of controls. See the Chords and Arpeggios tutorial for detailed information.

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

#### Properties

* _pattern_ : String. Default value is 'up'. Controls the direction of notes. Possible values are 'up', 'down', 'updown' and 'updown2'. For 'updown2', the top and bottom notes of the arpeggio won't be repeated as it changes directions.
* _octaves_ : Number. Default is 1. The number of octaves that the arpeggio will span.
* _notes_: A Gibber Pattern object that holds all the notes the arpeggiator sequences, as determined by the chord passed to it, the `mult` property and the arpeggiation `pattern` used. 
 
#### Methods
The Arp object uses a [Pattern](#common-objects-pattern) object as a mixin; this means that you can call most methods for
pattern manipulation (such as reverse, shuffle, rotate, transpose etc.) on an object created by the Arp
constructor. See [Pattern](#common-objects-pattern) for more information.

###Pattern

Patterns are functions that output values from an internal list that is typically passed as an argument when the pattern is first created. These lists can be manipulated in various ways, influencing the output of the patterns. Alternatively, `filters` placed on the pattern (each filter is simply a function expected to return an array of values) can also change the output of the pattern dynamically, without affecting its underlying list.

Whenever you sequence any method or property in Gibber, Pattern(s) are created behind the scenes to handle sequencer output. All methods of the pattern object can be additionally sequenced; you can also sequence the `start`, `end`, and `stepSize` properties.

Example:
```javascript
// two patterns are created when notes are sequenced. 
// The `values` pattern determines the output,
// while the `durations` pattern determines timing.
// In gibberwocky.MIDI:
channels[0].note.seq( [0,1,2,3], [1/4,1/8] )

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