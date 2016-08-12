# ebu-ttd-parser

A player agnostic ebu-tt-d subtitles library (and example) written in javascript.

## Documentation

### Running the example
Clone this repository and execute npm install to download all dependencies

```javascript
npm install
```

Once it is finished, you can serve it with your favourite HTTP Server and open the index.html on browser.

### Using the library
This library is composed by two JS files -subtitles.js and cuepoints.js- and a CSS -subtitles.css- which have to be included on HTML together with some vendor dependencies.

These are all the files that must be included on head:

* subtitles.css: Its purpose is load, parse and render ebu-tt-d subtitles
* [jquery](https://github.com/jquery/jquery)
* [underscore.js](https://github.com/jashkenas/underscore)
* [require.js](https://github.com/requirejs/requirejs)
* subtitles.js: Library to parse and render EBU-TT-D subtitles
* cuepoints.js: Tiny library to bind subtitles to video playing time.


```html
<link rel="stylesheet" href="css/subtitles.css">
<script src="node_modules/jquery/dist/jquery.min.js"></script>
<script src="node_modules/underscore/underscore-min.js"></script>
<script src="node_modules/requirejs/require.js"></script>
<script src="js/subtitles.js"></script>
<script src="js/cuepoints.js"></script>
```

This library is player agnostic, it is responsibility of developer keep the subtitles plugin informed about the current time of the video and place the subtitles over the video, but in order to simplify the understanding we will show the minimum HTML structure necessary for HTML5 video.

```html
<!-- Container for video and subtitles -->
<div class="subtitles-container" style="width:640px;height:360px;background-color:black;position:relative;">
	<!-- Video -->
	<video width="100%" height="100%" controls></video>
	<!-- End video -->
	<!-- Subtitles layer -->
	<div id="subtitles" class="subtitles"></div>
	<!-- End subtitles layer -->
</div>
<!-- End container for video and subtitles -->
```
And this is the minimum JS code to play a video with subtitles on HTML5 video.

```javascript
require(['SubtitlesPlugin', 'Cuepoints'], function (SubtitlesPlugin, Cuepoints) {
	//Constructs cuepoints
	cuepoints = new Cuepoints();
	//Constructs subtitles assigning the HTML element where it will render
	subtitles = new SubtitlesPlugin($('#subtitles').get(0));
	//Loads subtitles from url and inserts in Cuepoints
	subtitles.load(cuepoints, subtitlesSrc);
	//Changes video source and registers to some video events
	$('video')
		.attr('src', videoSrc)
		.on('timeupdate', function (e) {
			//Check if exists some cuepoint in the current time
			cuepoints.checkCuepoints(e.currentTarget.currentTime * 1000);
		})
		.on('seeking seeked', function () {
			//Clears subtitles and recalculate sizes
			subtitles.clearAndRefresh();
		});
});
```

Check index.html file to see the complete example.

## SubtitlesPlugin class

### Initialization
Subtitles constructor needs an HTMLDomElement where the subtitles will be rendered
```javascript
var subtitle = new SubtitlesPlugin(el);
```

### Methods

#### load (Cuepoints cue, String url)
Loads subtitles from an URL and assign it to a timecode where it should show and hide.
```javascript
subtitles.load(cuepoints, url);
```

#### unload ()
Unloads subtitles, this method should be called when destroy.
```javascript
subtitles.unload();
```

#### clearAndRefresh ()
Cleans subtitles from layer and recalculate sizes.
```javascript
subtitles.clearAndRefresh();
```

## Cuepoints class
Cuepoints is a tiny library that let us schedule functions inside a timeline.

### Initialization
Cuepoints constructor doesn't have any dependencies but it must be imported as require.js module.
```javascript
require(['Cuepoints'], function (Cuepoints) {
	var cuepoints = new Cuepoints();
	...
```

### Methods

#### addCuepoint(Object config)
With the following configuration this cuepoint will shown a console.log message if time is between 800ms and 1200ms
```javascript
var cue = cuepoints.addCuepoint({
	ms: 1000, //Time where function will be called
	callback: function(ms, config){ //Time where function will be called
				console.log('received at ' + ms + 'ms');
			  }, 
	negativemargin: 200, //Negative threshold when subtitle should be shown (e.g. in this case >800ms)
	positivemargin: 200 //Positive threshold when subtitle should be shown (e.g. in this case <1200ms)
})
```

#### removeCuepoint(Array cues) 
Removes all cuepoints passed as an array
```javascript
cuepoints.removeCuepoints([cue])
```

#### checkCuepoints(int ms)
Triggers all functions that are binded to the milisecond passed

# License
Apache License 2.0