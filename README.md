# HTML5-banner-masking
Use this small library to create cross-browser compatible svg masks that can animate

## Dependencies
* Greensock (only for animation)

## The JS Magic
```
// USE CLIPPY (http://bennettfeely.com/clippy/) to get the polygon coordinates. These will be in percentages, convert them to pixels (based on height and width of banner), and then put them in the data-polygon attribute within the image you're clipping, and add the class clip-me
	function convertPolyAttrToPath(cb) {
		var polys = document.getElementsByClassName('clip-me');
		[].forEach.call(polys, convertPoly);

		function convertPoly(poly){
			var svgNS = "http://www.w3.org/2000/svg";
			var path = document.createElementNS(svgNS,'path');
			var points = poly.getAttribute('data-polygon').split(/\s+|,/);
			var x0 = points.shift(), y0 = points.shift();
			var pathdata = 'M' + x0 + ','+ y0 + 'L' + points.join(' ');
			if (poly.getAttribute('data-polygon')) pathdata += 'z';
			path.setAttribute('d',pathdata);
			poly.setAttribute('data-path', pathdata)
		}
		cb();
	}

	function createCrossBrowserImageMasks(cb) {
		var polys = document.getElementsByClassName('clip-me');
		var images = [];
		var count = 0;
		
		for (var i = 0; i <= polys.length; i++) {
			var image = polys[i];
			if (image !== undefined) {
				if (image.classList) {
					image.classList.add('transforming-to-svg-' + i);
				}	else {
					image.className += ' ' + className;
				}
				images.push('transforming-to-svg-' + i)
			}
		}

		images.forEach(function(imageClass) {
			var image = document.getElementsByClassName(imageClass)[0];
			var maskPath = image.getAttribute('data-path');
			var maskSrc = image.getAttribute('src');
			var maskWidth = image.getAttribute('width');
			var maskHeight = image.getAttribute('height');
			var maskAlt = image.getAttribute('alt');
			var uniqueID = count;
			var svg = '<svg version="1.1" xmlns="http://www.w3.org/2000/svg" class="svgMask" width="'
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
				+maskAlt+'" id="clipmask' +uniqueID+ '" ' + '/></svg>';
			image.outerHTML = svg;
			count++;
			delete maskPath, maskSrc, maskWidth, maskHeight, maskAlt, uniqueID, svg; //clean up
		});
		cb();
	}
```
## Usage
### Markup
```
<img src="../../../assets/images/save-300x600-bg1.jpg" alt="bg1-mask-top" width="300" height="600" class="clip-me" data-polygon="0 197, 0 0, 300 0, 300 163">
// OR
<div id="mask-wrapper-1" style="overflow: hidden;">
	<img src="../../../assets/images/save-300x600-bg1.jpg" alt="bg1-mask-top" width="300" height="600" class="clip-me" data-polygon="0 197, 0 0, 300 0, 300 163">
</div>
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
				// Path corners, BLX (Bottom Left X coord) BLY TLX TLY TRX TRY BRX BRY 
				// OR
				.to("#mask-wrapper-1", 0.5, {width: 0}, "+=1")
		})
})
```
