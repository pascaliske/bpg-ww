# bpg-ww
##### (forked from @m-2k)
BPG JavaScript Decoder on WebWorkers

Allows 8 bit depth BPG images and animations

## Usage

```javascript
const worker = new Worker('/static/bpgdec8a-ww.min.js');

worker.onmessage = (event) => {
    switch(event.data.type) {
        case 'log': console.log(`Worker log: ${event.data.message}`); break;
        case 'debug': console.log(`Worker debug: ${event.data.data}`); break;
        case 'res':
            let image = event.data.image;
            let frames = event.data.frames;
            let loop_count = event.data.loop_count;
            let canvas = document.getElementById(event.data.meta);
            canvas.width = image.width;
            canvas.height = image.height;
            
            let context = canvas.getContext('2d');

            (function() {
                function d() {
                    var a = image.n;
                    ++a >= frames.length && (0 == loop_count || image.q < loop_count ? (a = 0, image.q++) : a =- 1);
                    0 <= a && (image.n = a, context.putImageData(frames[a].image, 0, 0), setTimeout(d, frames[a].duration))
                };
                context.putImageData(image, 0, 0);
                frames.length > 1 && (image.n = 0, image.q = 0, setTimeout(d, frames[0].duration));
            }.bind(context, image, frames, loop_count))();

            console.log('Decode done');
            break;
        default: console.log(`Unknown event: ${event.data}`); break;
    };
};

/**
 * Decodes an bpg image
 *
 * @param  [ArrayBuffer] buffer
 * @param  [String] element
 * @return [void]
 */
function decodeBpg(buffer, element) {
    worker.postMessage({
        type: 'image',
        img: buffer,
        meta: element
    });
}
```

### More info
* http://bellard.org/bpg/
