# vue-image-uploader
Configurable image preview and ajax up-uploader

 + drag and drop with input backup
 + image previews
 + events
 + 1 dependency [vue-resource]
 + minimal
 + configurable
 + batched or async

## Installation

### NPM
```
npm install vue-image-uploader --save
```

## Utilization 
### ES6
```js
    import ImageUpload from 'vue-image-uploader';

    new Vue({
        template: '<image-upload url=""></image-upload>',
        components: {
            ImageUpload
        }
    })
```
```html
    <image-upload url="" name="" max_files=""></image-upload>
```

## Configuration

    url: {
        type: String,
        required: true,
        default: null
    },
    name: {
        type: String,
        required: false,
        default: 'images[]'
    },
    max_batch: {
        type: Number,
        required: false,
        default: 0
    },
    max_files: {
        type: Number,
        required: false,
        default: 10
    },
    max_filesize: {
        type: Number,
        required: false,
        default: 5000
    },
    button_html: {
        type: String,
        required: false,
        default: 'Upload Images'
    },
    button_class: {
        type: String,
        required: false,
        default: 'btn btn-primary'
    }