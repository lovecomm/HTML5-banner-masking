# HTML5-banner-masking
Use this small library to create cross-browser compatible svg masks that can animate

## Dependencies
* jQuery
* Greensock

## The JS Magic
```
function convertPolyAttrToPath(cb) {
		var polys = $('.clip-me');
		[].forEach.call($(polys), convertPoly);

		function convertPoly(poly){
			var svgNS = "http://www.w3.org/2000/svg";
			var path = document.createElementNS(svgNS,'path');
			var points = $(poly).attr('data-polygon').split(/\s+|,/);
			var x0=points.shift(), y0=points.shift();
			var pathdata = 'M'+x0+','+y0+'L'+points.join(' ');
			if ($(poly).attr('data-polygon')) pathdata+='z';
			path.setAttribute('d',pathdata);
			$(poly).attr('data-path', pathdata)
		}
		cb();
	}

	function createCrossBrowserImageMasks(cb) {
		$('.clip-me').each(function(i) {
			var maskPath = $(this).attr('data-path');
			var maskSrc = $(this).attr('src');
			var maskWidth = $(this).attr('width');
			var maskHeight = $(this).attr('height');
			var maskAlt = $(this).attr('alt');
			var uniqueID = i;
			//build SVG from our IMG attributes & path coords.
			var svg = $('<svg version="1.1" xmlns="http://www.w3.org/2000/svg" class="svgMask" width="'
				+maskWidth+'" height="'
				+maskHeight+'" viewBox="0 0 '
				+maskWidth+' '
				+maskHeight+'"><defs><clipPath id="maskID'
				+uniqueID+'"><path d="'
				+maskPath+'"' + 'id="clip-' + uniqueID + '"/></clipPath>  </defs><image clip-path="url(#maskID'
				+uniqueID+')" width="'
				+maskWidth+'" height="'
				+maskHeight+'" xlink:href="'
				+maskSrc+'"' + ' alt="'
				+maskAlt+' id="clipmask' +uniqueID+ '" ' + '/></svg>');
			$(this).replaceWith(svg); //swap original IMG with SVG
			delete maskPath, maskSrc, maskWidth, maskHeight, maskAlt, uniqueID, svg; //clean up
		});
		cb()
	}
```
## Usage
### Markup
```
<img src="../../../assets/images/save-300x600-bg1.jpg" alt="bg1-mask-top" width="300" height="600" class="clip-me" data-polygon="0 197, 0 0, 300 0, 300 163">
```
USE CLIPPY (http://bennettfeely.com/clippy/) to get the polygon coordinates. These will be in percentages, convert them to pixels (based on height and width of banner), and then put them in the data-polygon attribute within the image you're clipping, and add the class clip-me
### JS
For each `<img class='clip-me'...`, the JS functions will add an ID of *clip-$* with the *$* being replaced with a zero-based incrementer. As such, call the *convertPolyAttrToPath* and *createCrossBrowserImageMasks* functions, then animate the path's *d* attribute as follows... 
```
convertPolyAttrToPath(function() {
	createCrossBrowserImageMasks(function() {
		console.log('done with convertPolyAttrToPath')
				tl
				.to("#clip-0", 0.5, {attr: {d: 'M300,197L 300 0  300 0  300 163z'}, ease: Expo.easeOut}, "+=1")
		})
})
```
