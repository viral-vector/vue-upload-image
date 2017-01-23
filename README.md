# vue-image-uploader [pre-alpha]
Configurable image preview and ajax up-uploader

## Mission Goals
 + drag and drop with input backup
 + non blokcing 
 + image previews
 - events
 - 0 dependencies..except vuejs
    - 1 dependency [vue-resource]
 * minimal
 * configurable

## TODO
 * trim component html
 * move response handling to events
 * dispatch events
 * remove vue-resource dependency
 * batched & parallel uploading
 * auto submission
 * more configurations 

## Installation

### NPM
```
npm install vue-image-upload --save
```

## Utilization 
### ES6
```js
    import ImageUpload from 'vue-image-upload';

    new Vue({
        template: '<image-upload url=""></image-upload>',
        components: {
            ImageUpload
        }
    })

```

## Configuration