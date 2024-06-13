# Overlay Recorder

Records a webpage to transparent PNG files which create a MOV Movie with alpha transparency in it.

The way I use this is to create animated slideshows using wordpress and Slider Revolution plugin. That plugin allows you to export as an HTML file independent of wordpress so you can run a simple HTML server that displays the slider. 

I use a PHP server to host the slider and then initiate the `timecut` project that utilises `puppeteer` to record the screen. You can specify many settings including length of clip, fps, etc... 

[Slider Revolution](https://www.sliderrevolution.com/) is very powerful and allows you to easily create amazing animation that can be used to generate overlays for videos instead of web graphics.

As an addition, I have created a built-in function into the exported slider that enables you to supply a `inputs.json` file with overrides to the innerText and Style of any element on the page. This enables customisation without having to use Revolution Slider to re-render and export the changes. This way you can dynamically change the slider on the fly.

The output file can then be layered over a video for awesome effects.

## Timecut on a remote URL

```bash
alias timecut="node cli.js"

timecut http://localhost:8000/index.html  -S "svg" --transparent-background --viewport="1080,1920" --fps=30 --duration=3 --output-options="-c:v png" --pix-fmt=rgba --output=overlay_video.mov
```

or

## Timecut on a local file

```bash
alias timecut="node cli.js"

timecut index.html -S "svg" --transparent-background --viewport="1080,1920" --fps=30 --duration=3 --output-options="-c:v png" --pix-fmt=rgba --output=overlay_video.mov`
```


## Running on runner

Launch arguments are needed.

```bash
timecut "http://localhost:8080"  -S "sr7-module" --transparent-background --viewport="1080,1920" --fps=30 --duration=3 --output-options="-c:v png" --pix-fmt=rgba --output=overlay_video.mov --launch-arguments="--no-sandbox --disable-setuid-sandbox --allow-file-access-from-files"
```


## The Slider Revolution 

Slider Revolution Plugin in Wordpress is used to create the templates. You can create the animation with a transparent background and then export as HTML to a zip file. This allows you to run a simple PHP server (without wordpress) to run as a standalone HTML file. The slider has the embedded script (see below) to allow dynamic changing of the animation via a json file.


### Change via JSON files
The slides have the following function added so that you can control the innerText or Style via a JSON file called `inputs.json`

```js
function fetchFile(url1, url2) {
    return new Promise((resolve, reject) => {
        // Attempt to fetch from the first URL
        fetch(url1)
            .then(response => {
                if (response.ok) {
                    resolve(response);
                } else {
                    // If fetch from the first URL fails, attempt the second URL
                    fetch(url2)
                        .then(response => {
                            if (response.ok) {
                                resolve(response);
                            } else {
                                reject(new Error('File not found in both locations'));
                            }
                        })
                        .catch(error => reject(error));
                }
            })
            .catch(() => {
                // If fetch from the first URL fails, attempt the second URL
                fetch(url2)
                    .then(response => {
                        if (response.ok) {
                            resolve(response);
                        } else {
                            reject(new Error('File not found in both locations'));
                        }
                    })
                    .catch(error => reject(error));
            });
    });
}

// Check both of the locations for file
var url1 = './inputs.json';
var url2 = './wp-content/themes/slider/inputs.json';

fetchFile(url1, url2)
    .then(response => response.json())
    .then(data => {
        console.log(data);
        
        // Generate CSS rules with !important based on JSON data
        let cssRules = '';
        for (const key in data) {
            if (data.hasOwnProperty(key)) {
                if (typeof data[key] === 'object' && data[key].hasOwnProperty('style')) {
                    const styleObj = data[key].style;
                    cssRules += `#${key} {`;
                    for (const styleKey in styleObj) {
                        if (styleObj.hasOwnProperty(styleKey)) {
                            cssRules += `${styleKey}: ${styleObj[styleKey]} !important; `;
                        }
                    }
                    cssRules += `}\n`;
                }
            }
        }

        // Create <style> element and append to <head>
        const styleElement = document.createElement('style');
        styleElement.textContent = cssRules;
        document.head.appendChild(styleElement);
        
        // Continue with updating innerText if needed
        for (const key in data) {
            if (data.hasOwnProperty(key)) {
                const elementById = document.getElementById(key);
                if (elementById) {
                    if (typeof data[key] === 'object' && data[key].hasOwnProperty('innerText')) {
                        elementById.innerText = data[key].innerText;
                    }
                }
            }
        }
    })
    .catch(error => console.error('Error fetching or parsing JSON:', error));

```


