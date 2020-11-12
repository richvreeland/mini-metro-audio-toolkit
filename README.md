# Mini Metro : Guidelines for Building a City Soundscape
Last Update : October 18th, 2020

Each city in Mini Metro has its own text file which specifies how the music and sound for that city should be. For instance, London has a file `london.txt`. All of these text files inherit from a master text file, `city.txt`. This file contains the default settings for all cities. Any of these settings can be overridden by individual cities.

## Instruments
It's important to first know both literally and symbolically which game objects are responsible for making the sound in Mini Metro.

`Lines`
Each metro line in Mini Metro has a corresponding `LineLoop` module responsible for generating a musical pulse. Each pulsing line has a rhythm and a pitch. These pulsing notes are the foundation of the musical experience of the game. Most levels contain 7 lines, represented by the number range `0..6`. Some contain less than 7, such as Osaka. There are additional musical UI modules associated with lines, such as `LineCreated`  and `AssetPanelSelectLine` which will automatically inherit the correct musical information, so you generally don't need to worry about these.

`Trains`
Each train has a looping engine sound, whose pitch is inherited from the corresponding `LineLoop` for the metro line it's on. There are 7 engine sounds to choose from. Four have a rhythmic feel to them, noted in their names:  `SixEight`, `TwelveSixteen`, `Three`, and  `Four`. The rest are arrhythmic or more droning in nature:  `Scooter`, `Orange`, and finally `Shinkansen`, which was especially designed for the bullet trains in the Osaka level, but can be used anywhere. The timing of certain train events, such as trains arriving can be controlled. 

`Peeps`
This is the fun name we use to take about the little passenger shapes. You can control the timing of peep events such as appearing (spawning), and embarking (getting on trains). Mini Metro is unique in that these timing decisions actually effect the gameplay, so tweak these values with the utmost care !

`Stations`
Most of the station sounds are immediate and thus locked in, however it's worth noting that station events such as spawning and mutating (one station type morphing into another) will trigger new  `bass` notes.

## Harmony Progression System

The selection of musical notes is handled within the `harmonies` section of a city's txt file. It's important to note that the `HARMONY TEMPLATE` section of `city.txt` is not the parent of `harmonies`, it's merely an example. In other words, this section has no inheritance scheme. Below are its associated properties.

Note: Enharmonic pitches (ie. `Bb` or `A#`) are always represented with a `#`, so if you desired a `Bb4`, use `A#4`.

`weeks`  (required)
Each entry represents a week. By default this list of entries loops, so if you've reached week 4 & you only have 3 entries, it will loop around and use the 1st entry.
	
* `sequences`  (required)
	* A 2D list of pitch sequences, where pitches can range from `C2` to `B5`. 
	* These sequences can be assigned to different lines using the `assignments` property (see below).
	* If multiple lines are assigned to the same sequence, they are assigned notes in a linear fashion.
		* ie. if lines 0, 1 & 2 are all assigned the same sequence of `["C3", "E3", "G3"]` , line 0 will get `C3`,  1 gets `E3`, and so forth.
	* All sequences will progress forward by one entry when the least recently edited line gets edited by the player. 
		* in the previous example, this would result in line 0 getting `E3`, line 1 getting `G3`, and line 2 looping back around to receive `C3`.

* `bass`
	* A linear list of pitch sequences, where pitches can range from `C` to `B`.  Octave range is handled automatically (basically, it's low!)
	* Bass notes progress forward with the standard sequences mentioned above.

* `screens` 
	* These are screens where a chord can optionally be triggered. These definitions are relative to the week in which they are defined.
	* Each entry requires a sequence where the 1st array member is a bass note, ie. `D`, followed by chord tones, ie. `F3`.  Screen Options include :
		* `NewAsset`  : the week end screen where you can acquire new assets, such as tunnels and carriages.
		* `GameOver` : the game over screen. Pretty self-explanatory !


`randomize`
Whether or not to randomize the harmonic week order. `false` by default.

`pedalingBass`
When set to false, bass sequence will progress to the next note every time a bass note is fired. Otherwise, bass pedals (doesn't always change), and only updates when the least recently updated line is updated, and is using `sequence` 0.

`assignments` (required)
Which sequence index to use for each line. Cities typically have 7 lines, so this array is typically 7 entries long. For instance, if you wanted all 7 lines to use the same harmonic sequence, and you only had one sequence, the assignments would be: `[0, 0, 0, 0, 0, 0, 0]`. 

`order`
The linear, repeating order of `weeks` to use week over week in game. This is an optional parameter, as the default behavior is to use harmonic weeks in the order they are specified.

- - - -

## Glossary

`MasterPulse`
The metronome of the game,  which is 72bpm at `TimeScale.Single`. All modules use subdivisions of this pulse (including a subdivision of 1, or the same length as the master pulse, if desired).

`Pulse`
The number of subdivisions of the `MasterPulse` to be used by a module, or more specifically, the maximum frequency at which the associated module can trigger sounds.

`Arpeggio`
A module type that given a group of incoming game events, fans out playback over subsequent pulses.

`LineLoop`
The per metro line module responsible for generating the pulsing pitches that make up the foundation of the audio experience.

`Standard`
Peeps or Stations of the following shapes: triangle, circle, square. These are the most common to appear.

`Special`
Peeps or Stations of shapes not covered by `Standard`. ie: star, cross, diamond, egg, etc. These appear later in the progression of a city and are generally more rare.

`TimeScale`
The speed of the game. There are four scales:
* `SingleSlow`  : 75% of normal game speed. This is when you pause the game from normal game speed.
* `Single`  :  100% . This is normal game speed.
* `DoubleSlow` : 150%, or 75% of fast game speed. This is when you pause the game from fast game speed.
* `Double` : 200% of normal game speed. This is fast forward mode.

- - - -

## Overridable Constants

### LineLoop Specific

`LineLoop_Pulse`
This is the number of subdivisions of the `MasterPulse` to use for `LineLoop` modules in this level, or more specifically, the maximum frequency at which these sounds can be triggered. By default this number is `12`, which afford you all manner of duplet and triplet rhythms, such as `2, 3, 4, 6, 8, 9, 12, 18, 24`.  For other types of pulses, such as quintuplets and septuplets, you will need to use a higher subdivision in order to get the intended effect. For instance, `paris.txt` contains `"LineLoop_Pulse" : 420`, which allows for rhythms such as `120 (2/7)`,  `140 (1/3)`, and  `168 (2/5)`. In other words, that city contains duplet, triplet, quintuplet AND septuplet rhythms.

Important to note: the smallest supported rhythm is `1/4` . You can use smaller rhythms in theory, but keep in mind that Mini Metro has a fast forward mode, so it will probably sound really insane. Also the `offset` system will not support those rhythms.

ALSO important to note: If you choose to use a pulse other than `12`, you will likely have to override all of the pulse constants so that they get the correct value. ie. the default value of `TrainArrives_Pulse` is 2, which is equivalent to `1/6`. That value would not mean the same thing if you change `LineLoop_Pulse` to a greater value, like 24.


`Rhythms`
An array of pulse length options for `LineLoop` in this level. This list is generally non-repeating, in descending numerical order from largest to smallest. Worth noting: this rhythm list gets turned into a palindrome (ie. `[12, 6, 4, 3] -> [12, 6, 4, 3, 4, 6]`) during gameplay, so that the rhythms listed cycle through in a natural manner.


`Steps_***`
These array constants represent the rhythmic `offsets` available for each `LineLoop` rhythm type. These offsets are options within the pulse where new pulsing elements can enter. The more rhythmic offset options you make available for a given rhythm, the more quickly the music system can react to new gameplay changes. For instance, if supply an array of `[0]`, a new `LineLoop` will wait until the next `MasterPulse` (about 0.8 secs) to enter.

A glossary is shown below, where `LineLoop_Pulse` is represented by the letter `p`, and a `Bar` represents one `MasterPulse` (think of it as a really fast bar).

* `Steps_Double			= p * 2    `
* `Steps_5BarTriplet		= p * (5/3)`
* `Steps_DottedWhole		= p * (3/2)`
* `Steps_4BarTriplet		= p * (4/3)` 	
* `Steps_Whole				= p * 1    `	
* `Steps_DottedHalf		= p * (3/4)`
* `Steps_Half				= p * (1/2)`
* `Steps_2BarTriplet		= p * (2/3)`
* `Steps_1BarTriplet		= p * (1/3)`
* `Steps_Quarter			= p * (1/4)`
* `Steps_4BarQuintuplet	= p * (4/5)`
* `Steps_3BarQuintuplet	= p * (3/5)`
* `Steps_2BarQuintuplet	= p * (2/5)`
* `Steps_4BarSeptuplet	    = p * (4/7)`
* `Steps_3BarSeptuplet	    = p * (3/7)`
* `Steps_2BarSeptuplet	    = p * (2/7)`


`Swing`
This system is a little bit jank, but does the job. Swing is a value from `0..1` , where `0` is no swing, and `1`  will swing the note to an extreme degree. Also worth noting: given a triplet style (1 bar, 2 bar or 4 bar) rhythm, the rhythm will be swung every _third_ note instead of every second note.

`Swing_Range`
An array of two values representing the range of rhythms that get swung.
For instance : given `"LineLoop_Pulse" : 12`  and `"Rhythms": 12, 8, 6, 4, 3`, a range of `[0,4]` will swing `3` and `4`.


`Octave_Chance`
A `0..1`  value representing the probability that `LineLoop` notes are triggered up an octave.

`VoiceLimit_FadeTime`
By default, the total polyphony of `LineLoop` voices playing at any one time is limited to the number of lines currently in the level. When that limit is reached, the oldest voice is faded out using the duration provided by this constant.

`VoiceLimit_Static`
Alternatively, you can specify a hard limit on the number of voices `LineLoop` uses by specifying it with this constant. Values less than `1` are ignored.

`FadeIn_Times`
A four number array representing the fade in time for all `LineLoop` samples at different `TimeScale`  (game speed) settings. A fade time of  `0` means the samples are not faded in. A fade time of `1.5` would mean each sample takes 1.5 seconds to fade in.

Here are the `TimeScale` settings as they correspond to the array: `[Single Slow, Single, Double Slow, Double]`.
See the glossary for more info on what these represent.

- - - -

###  Arpeggio Specific

`Flam_Limit`
The maximum number of sounds to arpeggiate for events that use the `Arpeggio` module. This includes peeps appearing, embarking and transferring. This module takes gameplay events that come in at the same time, and spreads them out in a rhythmically pleasing manner.

`PeepEmbarks_Flam_Pulse`
The pulse, or maximum frequency of sound triggering for Peep Embarks sounds.  These sounds are all subject to flamming (aka arpeggiation). This value represents a subdivision of the `MasterPulse`.

`PeepEmbarks_Rhythm`
This represents the reaction time for arpeggiating peep embarkation events. For instance, given  a `PeepEmbarks_Flam_Pulse` of 12, a value of `3` would mean a new arpeggiation could start once every 3 pulses.

`PeepAppears_Flam_Pulse`
This has the same function as `PeepEmbarks_Flam_Pulse`, but for Peep appearance events.

`PeepAppears_Step_Standard`
This controls which step in the pulse is reserved for `Standard`  peeps. Peep appearance events always begin on the hour (on the downbeat of the `MasterPulse`).

`PeepAppears_Step_Special`
This controls which step in the pulse is reserved for `Special`  peeps. 

- - - -

### Relevant to Other Modules

`TrainArrives_Pulse`
The frequency with which the Train Arrival sound fires.

`Train_Engines`
An array of engine types to use for this city, chosen at random.

`Train_AssignEngineByLine`
Whether or not to use the the `Train_Engines` array as an ordered assignment of which engine to use for which line.

`ChordTone_Pulse`
When the game plays chords (generally on week end and game over screens), the pulse length of each chord tone.

`Chord_Playback`
Whether or not to play chords alongside bass notes, ie. when stations spawn and mutate. By default this is set to `false`.

`Percussion_PitchVar`
Controls the amount of pitch variation for Peep Appear and Transfer sounds by adding a random deviation, +/- within the value specified. ie. `0.2` means the pitch will be anywhere in the range `0.8..1.2`. For reference: `0.5` is down an octave, and `2` is up an octave.

 `Transpositions`
An array of key choices for the level to start in, given as relative values. ie. `-3` is down three semitones from C, or A#. Options range from `-12..12`. Note: if using values close to the edges of the range, many of the notes will be transposed up/down an octave in order to fit within the range of available pitches.

`***_Gain` 
These constants are for mixing volume levels of the various audio modules. Generally not recommended to change, but available should you choose.

- - - -

## Optional Modules

`Beat`  
This is a special module that is only used in a few cities, such as Berlin, Stockholm, Singapore, and Washington DC.  It plays a kick drum at a pre-defined rhythm, but only when the game is not paused. and has some customizable properties as listed below. It will also update at the same time as the LineLoop `Rhythms`. For now, to get this to work properly, simply copy the module over from one of these other cities.  Only properties that can be altered safely are listed below :  
   
`Rhythms`  
This is a list that operates more or less in the same fashion as `LineLoop_Pulse`.  
   
`Rhythms_Double`  
By default, all rhythms are doubled in speed when in `TimeScale.Double`. This allows you to override that. For instance, if you have a rhythm that is swift sounding in `TimeScale.Single`, it may now be too fast to sound good at the Double scale. This alleviates that problem.  
   
`StartingWeekIndex`  
This is an integer value that represents how many weeks it will take before you start hearing the beat. `0` means play from the start.  
   
`gain`, `pulse`   
These operate as all other modules do.  

- - - -

## Additional Overrides

You will see that some cities override the step values for PeepAppears shapes directly and individually, as opposed to using the constants `PeepAppears_Step_Standard` and `PeepAppears_Step_Special`.

The methodology is as follows :
`{ "id"	: "PeepAppears: ShapeNameGoesHere", "step" : int }`, where `int` is an integer of your choosing.

Any overrides done in this manner will supersede any value you supply to `PeepAppears_Step_Standard`  or `PeepAppears_Step_Special`.

- - - -

## Debug Key Commands

```
H : Increase Score
I : Invincible
L : Kill Peeps
P : Add Random Peep
X : Die
S : Add Assets
M : Mutate a Random Station
F : Fast Forward to end game
Q : Time Scale Increase
A : Time Scale Decrease
N : Audio Debugging
```

- - - -

## Going Beyond
Of course, there are more tweaks and overrides you can make to your city, if you're feeling adventurous. These are simply the standard, most commonly used applications. Feel free to go beyond this guide, at your own peril !

#project/done/mini metro#
