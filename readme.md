# Motor.js
Motor is a multitrack step sequencer for the browser. It is inspired by MIDI sequencers in modern music software, but is flexible enough to control any audio or visual elements in a webpage. Play back sequences of numbers, strings, or objects at a given tempo, and use them to control anything you want. Send strings to a text-to-speech library, while sending MIDI notes to a synthesizer library, while adding DOM elements to the page, all in sync.

## Examples
**[Realtime Bling](http://urmston.xyz/realtimebling)**
- strings -> text-to-speech ([responsivevoice.js](link))
- strings -> display text in the DOM
- note numbers -> synthesizer and drum samples ([gibberish.js](https://github.com/charlieroberts/Gibberish))

**[Youtube Remix](http://urmston.xyz/trackYoutubeRemix)**
- Create a new song by jump around in videos with the YouTube API
- note numbers -> drum samples [gibberish.js](https://github.com/charlieroberts/Gibberish)

## Getting Started
#### Simple Example: Add text to two separate divs
```javascript
<!doctype HTML>
<html>
<head>
<!-- link to motor.js -->
<script type=“text/javascript” src=“motor.js”></script>
</head>
<body>
<div id="lyrics"></div>
<div id="drums"></div>
<script type="text/javascript">

// Instantiate a new Motor
sillySequencer = new Motor()

// Create a new sequence with two tracks, “lyrics” and “drums”
sillySequencer.newSeq(‘intro’, {
	lyrics:[[“hey”,30],,,,,,,,[“ho”,50],,,,,,,,,,,,[”let’s”, 20],,,,[“go”,50],,,,,,,,],
	drums:[["boom",20],,,,["bap",10],,,,["boom",20],,,,["bap",10],,,,]
})

// Define some functions to do something with the data
// The track name and the value of each step is sent to the output functions
var setTextOfDiv = function(value, trackName) {
   	var divToFill = document.getElementById(trackName)
   	//set the text of the div
    	divToFill.innerHTML = value[0]
    	//set the fontSize of the text
    	divToFill.style.fontSize = value[1]
}

var appendTextToBody = function(trackName,value) {
    	//append text to body
    	document.body.innerHTML += value
}

// Tell Motor what function(s) to call with the data from each track
sillySequencer.setOutputs({
	lyrics:[setTextOfDiv,console],
	drums:[setTextOfDiv,console],
	backgroundText:[appendTextToBody]
)}

// Play the sequence. If no argument is supplied, the last sequence will play.
sillySequencer.bpm = 120
sillySequencer.play()
</script>
</body>
```

## Documentation
#### Motor {}
**Properties**
* **Motor.bpm** *[number]*     
Motor will play sequences at this tempo. 120 by default.
* **Motor.swing** *[number 0-1]*     
The amount of swing applied to sequences. A value of 0.5 produces a “straight” sequence with no swing.
* **Motor.currentStep** *[number, read-only]*     
The step number since the last sequence was launched.
* **Motor.currentSeq** *[object, read-only]*      
Always contains the current playing sequence. Don’t modify this.
* **Motor.seqs** *[object, read-only]*     
Contains all Sequence objects

**Methods**     
* **Motor.newSeq**( name *[string]*, sequence *[object]*)
Creates a new sequence. Use the following syntax:
```javascript
sillySequencer.newSeq(‘intro’, {  
	kick: 		[1,,,,,,,, 1,,,,,,,,],  
	hatOpen: 	[,,1,,1,,,,],  
	background:	[,,,,"blue",,,"red",]
})
```
* **Motor.play**( sequenceToPlay *[string, optional]* )
Starts playing a sequence. If no argument is supplied, the last-created sequence will play.

#### Sequence {}
**Methods**     
* Motor.seqs[ sequence name ].**onLaunch**( function *[function]* )     
Define a function that is called whenever the sequence is started.
* Motor.seqs[ sequence name ].**onStep**( function *[function]* )    
Define a function that is called on each step of a sequence.
* Motor.seqs[ sequence name ].**playNext**( sequences *[object]* )     
Using playNext(), you can chain together sequences to form longer compositions. In the object, the key is a step value that when reached, will trigger the next sequence. If you include more than one sequence name in the array, the chance of playing each sequence is divided amongst them. To make one sequence more likely to play next, simply add more copies of its name to the array. Using this method, it is easy to create complex, chance based logic to control the flow of a composition.
```javascript
sillySequencer.newSeq(‘intro’, {  
	kick: 		[1,,,,,,,, 1,,,,,,,,],  
	hatOpen: 	[,,1,,1,,,,],  
	background:	[,,,,"blue",,,,]
}).playNext({
	32: ['chorus']
})

sillySequencer.newSeq(‘chorus’, {  
	kick: 		[1,,,,,,,, 1,,,,,,,,],  
	hatOpen: 	[,,1,,1,,,,],  
	background:	[,,,,"blue",,,,]
}).playNext({
	32: ['intro','verse']
})
````

**Method Chaining**     
The Motor.Sequence object supports method chaining for quick sequence creation:
````javascript
sillySequencer.newSeq(‘intro’, {  
	kick: 		[1,,,,,,,, 1,,,,,,,,],  
	hatOpen: 	[,,1,,1,,,,],  
	background:	[,,,,"blue",,,,]
}).onLaunch( function() {
	//do something
}).onStep( function() {
	//do something
}).playNext({
	64:['chorus','bridge']
})
```

**Track Layers**     
Motor includes specially-named tracks that modify existing tracks.
A track can be layered upon by using the same root name and appending a keyword.
* **_transpose**     
This will modulate the track's midiTransposeAmount. By taking an existing melody and transposing it with the _transpose track layer you can achieve some interesting melodic effects, especially when the two layers are different lengths. Example:
```javascript
sillySequencer.newSeq(‘breakdown’, {  
	bassline: 		[40,,,40,,,,40,,,40,,,,],  
	bassline_transpose: 	[0,,,5,,,]
})
````
* **_2**, **_3**, **_4**, etc.    
This small yet powerful feature allows you to generate [polyrhythms](https://en.wikipedia.org/wiki/Polyrhythm) by creating several track layers of different lengths. Example:
```javascript
sillySequencer.newSeq(‘breakdown’, {  
	bassline: 	[40,,,,,,,,40,,,,,,,],  
	bassline_2: 	[42,,,48,,,],
	bassline_3: 	[38,,,,,,,,,,,,,,,,,,,,,,,42,,,,,,,]
})
````
**Poly**     
By default, Motor outputs whatever is at the current step of each track the current sequence. There are some instances (for example when playing a chord in a polyphonic synth) where it is useful to be able to loop through several values in one step and send them out individually. p() returns a special object that Motor parses in that way. p()'s arguments can be any type. Use it like this:
```javascript
Motor.newSeq(‘outro’, {  
	synth: [,,,,,,,, p(40,43,45),,,,,,,,]  
})
````
## Tips
Use [HackTimer.js](https://github.com/turuslan/HackTimer) to prevent timers from slowing down when a tab is idle.
## License
