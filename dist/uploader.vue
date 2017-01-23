<template>
    <div class="container">
        <div class="vue_component vue_component__imageupload" v-bind:class="{ 'dragover': onDragover }">
            <form class="form" id="image_upload_form" action="" enctype="multipart/form-data">
                <div class="image_upload_form__thumbnails">
                    <div v-for="(value, key) in files" v-bind:class="{ 'uploaded': value.uploaded }">
                        <span v-on:click="fileRemove(key)">&#x2716;</span>
                        <img v-bind:src="image[key]" v-bind:class="{ 'show': image[key] }">
                    </div>
                </div>
                <div class="form__input hidden">
                    <input type="file" name="files[]" id="image_upload_form__input" multiple />
                </div>
                <div class="form__button padding-top-5">  
                    <button type="submit" class="btn btn-primary" v-on:click="submit" v-bind:disabled="onUploading">
                        <i class="fa fa-check fa-prefix"></i>{{ message_confirm }} {{ queue }} Images
                    </button>
                </div>
            </form>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'image_upload',
        props: {
            url: {
                type: String,
                required: true,
                default: null
            },
            max_files: {
                type: Number,
                required: false,
                default: -1
            },
            max_filesize: {
                type: Number,
                required: false,
                default: -1
            },
            confirm_text: {
                type: String,
                required: false,
                default: 'Upload!'
            },       
        },
        data: function(){
            return {
                form: null,
                input: null,
                click: null,
                index: 0,
                queue: 0,
                total: 0,
                files: {}, 
                image: {},
                onDragover: false,
                onUploading: false  
            }
        },
        mounted: function(){
            this.form = document.getElementById('image_upload_form');
            this.input = document.getElementById('image_upload_form__input');
            this.click = document.getElementById('image_upload_form__click');

            ['drag', 'dragstart', 'dragend', 'dragover', 'dragenter', 'dragleave', 'drop'].forEach(event => this.form.addEventListener(event, (e) => {
                e.preventDefault();
                e.stopPropagation();
            }));

            ['dragover', 'dragenter']
                .forEach(event => this.form.addEventListener(event, this.dragEnter));

            ['dragleave', 'dragend', 'drop']
                .forEach(event => this.form.addEventListener(event, this.dragLeave));

            ['drop']
                .forEach(event => this.form.addEventListener(event, this.fileDrop));

            ['change']
                .forEach(event => this.input.addEventListener(event, this.fileDrop));

            this.click.addEventListener('click', (e) => {
                this.input.click();
            });
        },
        methods: {
            submit: function(e){
                e.preventDefault();
                
                if(!this.onUploading){
                    this.upload();
                }
            },
            upload: function(){
                for (let key in this.files) {
                    if(this.files[key].uploaded || this.queue <= 0) continue;

                    this.onUploading = true;

                    let formData = new FormData();
                    formData.append(this.input.getAttribute('name'), this.files[key]); 
                    
                    this.$http.post(this.url, formData).then((response) => {
                        Vue.set(this.files[key], 'uploaded', true);

                        this.queue--;
                        this.total++;
                    }, (response) => {
                        
                    }).then((response) => {
                        this.upload();
                    });
                    
                    return false;   
                }

                this.onUploading = false;     
            },
            canDragDrop: function(){
                let div = document.createElement('div');
                return (('draggable' in div) || ('ondragstart' in div && 'ondrop' in div)) && 'FormData' in window && 'FileReader' in window;
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

                let newFiles = e.target.files || e.dataTransfer.files || e.originalEvent.dataTransfer.files;

                for(let i = 0; i < newFiles.length; i++){
                    this.files[this.index] = newFiles[i];

                    this.fileLoad(this.index);

                    this.index++;
                    this.queue++; 
                }

                e.target.value = ''; // clear file input for same file submission
            },
            fileLoad: function(index){
                let reader = new FileReader();

                reader.addEventListener("load", (e) => {
                    Vue.set(this.image, index, reader.result);
                });

                reader.readAsDataURL(this.files[index]);
            },
            fileRemove: function(key){    
                this.queue -= this.files[key].uploaded? 0:1;
                
                Vue.delete(this.files, key);
                Vue.delete(this.image, key);
            },
        }
    }
</script>
