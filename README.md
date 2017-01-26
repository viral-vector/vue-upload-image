# vue-upload-image
Configurable image uploader with preview

 + drag and drop with input backup
 + image previews
 + events
 + 1 dependency [vue-resource]
 + minimal
 + configurable
 + batched or async

![example](example/images/example.png)

## Installation
#### NPM
```bash
npm install vue-upload-image --save
```

## Usage 
#### ES6
```js
    import UploadImage from 'vue-upload-image';

    new Vue({
        template: '<upload-image url=""></upload-image>',
        components: {
            UploadImage
        }
    })
```

```html
    <upload-image url="" name="" max_files=""></upload-image>
```

```css
    .vue_component__upload--image
        .upload_image_form__thumbnails
            .upload_image_form__thumbnail [&.bad-size, &.uploaded]
                .img [&.show, &:hover]
                span
```
## Events
    upload-image-attempt
    upload-image-success
    upload-image-failure

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

## License
[MIT](http://vjpr.mit-license.org)