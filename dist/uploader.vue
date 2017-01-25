<template>
    <div class="vue_component vue_component__imageupload" v-bind:class="{ 'dragover': onDragover }">
        <form v-bind:id="'image_upload_form--' + name" enctype="multipart/form-data">
            <div class="image_upload_form__thumbnails">
                <div v-for="(value, key) in files" class="image_upload_form__thumbnail" v-on:click="fileView($event, key)" 
                        v-bind:class="{ 'uploaded': value.uploaded, 'bad-size': value.bad_size }" >
                    <span v-on:click="fileDelete($event, key)">
                    &#x2716;
                    </span>
                    <img v-bind:src="image[key]" v-bind:class="{ 'show': image[key]}">
                </div>
            </div>
            <input type="file" v-bind:id="'image_upload_form__input--' + name" hidden multiple />
            <div>  
                <button type="submit" 
                    v-bind:class="button_class" 
                    v-on:click="submit" 
                    v-bind:disabled="onUploading" 
                    v-html="button_html"></button>
            </div>
        </form>
    </div>
</template>

<script>
    export default {
        name: 'ImageUpload',
        props: {
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
            },
        },
        data: function(){
            return {
                form: null,
                input: null,
                index: 0,
                total: 0,
                files: {}, 
                image: {},
                batch: {},
                onDragover: false,
                onUploading: false  
            }
        },
        mounted: function(){
            this.form = document.getElementById('image_upload_form--' + this.name);
            this.input = document.getElementById('image_upload_form__input--' + this.name);

            ['drag', 'dragstart', 'dragend', 'dragover', 'dragenter', 'dragleave', 'drop'].forEach(event => this.form.addEventListener(event, (e) => {
                e.preventDefault(); e.stopPropagation();
            }));

            ['dragover', 'dragenter']
                .forEach(event => this.form.addEventListener(event, this.dragEnter));

            ['dragleave', 'dragend', 'drop']
                .forEach(event => this.form.addEventListener(event, this.dragLeave));

            ['drop']
                .forEach(event => this.form.addEventListener(event, this.fileDrop));

            ['change']
                .forEach(event => this.input.addEventListener(event, this.fileDrop));

            this.form.addEventListener('click', (e) => {
                this.input.click();
            });
        },
        methods: {
            _can_xhr(){
                if(this.total >= this.max_files){
                    return false;
                }
                return true;
            },
            _can_upload_file(key){
                let file = this.files[key];

                if(file.attempted || file.bad_size){
                    return false;
                }
                return true;
            }, 
            _xhr: function(formData, keys, callback){
                this.onUploading = true;
                this.$emit('image-upload-attempt', formData);

                keys.forEach((key) => {
                    Vue.set(this.files[key], 'attempted', true);
                });

                this.$http.post(this.url, formData).then((response) => {
                    keys.forEach((key) => {
                        Vue.set(this.files[key], 'uploaded', true);

                        this.total++;
                    });

                    this.$emit('image-upload-success', [formData]);
                }, (response) => {
                    this.$emit('image-upload-failure', [response]);
                }).then((response) => {
                    this.onUploading = false;  
                    
                    callback(); 
                });
            },
            upload: function(){
                if(!this._can_xhr()) return false;

                for (let key in this.files) {
                    if(!this._can_upload_file(key)) continue;

                    let formData = new FormData();
                    formData.append(this.name, this.files[key]); 
                    
                    this._xhr(formData, [key], this.upload);
                    
                    return true;   
                }  
            },
            upload_batch: function(){
                if(!this._can_xhr()) return false;

                for (let key in this.batch) {                    
                    this._xhr(this.batch[key].form, this.batch[key].keys, this.upload_batch); 

                    delete this.batch[key];

                    return true;
                } 
            },
            create_batch: function(){
                let index = 0;
                let count = 0;

                this.batch = {};

                for (let key in this.files) {
                    if(!this._can_upload_file(key)) continue;

                    if(this.batch[index] == null || count == this.max_batch){
                        index++;
                        count = 0;
                        this.batch[index] = {form:new FormData(), keys:[]};
                    }

                    count++;
                    this.batch[index]['keys'].push(key);
                    this.batch[index]['form'].append(this.name, this.files[key]);
                }
            },
            submit: function(e){
                e.preventDefault(); e.stopPropagation();
                
                if(!this.onUploading){
                    if(this.max_batch > 1){
                               this.create_batch();
                        return this.upload_batch();                        
                    }
                    this.upload();
                }
            },
            dragEnter: function(e){
                e.preventDefault();
                this.onDragover = true;
            },
            dragLeave: function(e){
                e.preventDefault();
                this.onDragover = false;
            },
            fileDrop: function(e){
                e.preventDefault();

                let newFiles = e.target.files || e.dataTransfer.files;

                for(let i = 0; i < newFiles.length; i++){
                    Vue.set(this.files, this.index, newFiles[i]);
                    this.fileInit(this.index);
                    this.fileRead(this.index);

                    this.index++; 
                }

                e.target.value = '';
            },
            fileInit: function(key){
                let file = this.files[key];

                if((file.size * 0.001) > this.max_filesize){
                    Vue.set(this.files[key], 'bad_size', true);
                }
            },
            fileRead: function(key){
                let reader = new FileReader();

                reader.addEventListener("load", (e) => {
                    Vue.set(this.image, key, reader.result);
                });

                reader.readAsDataURL(this.files[key]);
            },
            fileDelete: function(e, key){
                Vue.delete(this.files, key);
                Vue.delete(this.image, key);

                this.$emit('image-upload-image-delete', [ this.files[key]]);
            },
            fileView: function(e, key){
                e.preventDefault(); e.stopPropagation();

                this.$emit('image-upload-image-view', [ this.files[key]]);
            }
        }
    }
</script>

<style lang="sass" scoped>
    .vue_component__imageupload{
        padding: 5px;
        cursor: pointer;
        min-height: 80px;
        border-radius: 5px;
        &.dragover{
        }
        form > div{
            text-align: center;
        }
        .image_upload_form__thumbnails{ 
            margin-bottom: 1em;
            .image_upload_form__thumbnail{
                border-radius: 2.5px;
                position:relative;
                width:20%;
                padding:20% 0 0;
                overflow: hidden;
                margin:10px;
                display:inline-block;
                img{
                    position: absolute;
                    top:50%;
                    left: 50%;
                    min-width: 100%;
                    min-height: 100%;
                    max-height: 150%;
                    opacity: 0;
                    transform: translateX(-50%) translateY(-50%);
                    transition: 1s opacity;
                    &.show{
                        opacity: 1;
                    }
                    &:hover{
                        filter: blur(2px);
                    }
                }
                span{
                    position: absolute;
                    top: -5px;
                    left: 0px;  
                    z-index: 100;
                    padding: 0px 1px;
                    border-radius: 2px;
                    background-color: grey;
                }
                &.bad-size{
                    img{
                        filter: grayscale(100%);  
                    }
                }
                &.uploaded{
                    img{
                        opacity: 0.1;
                    }
                }
            }
        }
    }
</style>