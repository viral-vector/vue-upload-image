# vue-image-uploader
Configurable image uploader with preview

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
    ### events
    image-upload-attempt
    image-upload-success
    image-upload-failure

    ### props
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
    max_batch: { // # of files to upload within one request
        type: Number,
        required: false,
        default: 0
    },
    max_files: { // total # of files allowed to be uploaded
        type: Number,
        required: false,
        default: 10
    },
    max_filesize: { // max files size in KB
        type: Number,
        required: false,
        default: 5000
    },
    button_html: { // text/htm for button
        type: String,
        required: false,
        default: 'Upload Images'
    },
    button_class: { // classes for button
        type: String,
        required: false,
        default: 'btn btn-primary'
    }