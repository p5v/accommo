<template>
    <div class="uploader"
        @dragenter="onDragEnter"
        @dragleave="onDragLeave"
         @dragover.prevent
         @drop="onDrop"
         :class="{ dragging: isDragging }">
        <div class="upload-control" v-show="images.length">
            <label for="file">Select a file</label>
            <button @click="upload">Upload</button>
        </div>

        <div v-show="!images.length">
            <i class="fas fa-cloud-upload-alt"></i>
            <p>Drag your images here</p>
            <div>OR</div>
            <div class="file-input">
                <label for="file">Select a file</label>
                <input type="file" id="file" @change="onInputChange" multiple>
            </div>
        </div>

        <div class="images-preview" v-show="images.length">
            <div class="img-wrapper" v-for="(image, index) in images" :key="index">
                <img :src="image" :alt="'Image Uploader ${index}'">
                <div class="details">
                    <span class="name" v-text="files[index].name"></span>
                    <span class="size" v-text="getFileSize(files[index].size)"></span>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data: () => ({
            isDragging: false,
            dragCount: 0,
            files: [],
            images: []
        }),
        methods: {
            onDragEnter(e) {
                e.preventDefault();

                this.dragCount++;
                this.isDragging = true;
            },
            onDragLeave(e) {
                e.preventDefault();
                this.dragCount--;

                if (this.dragCount <= 0)
                    this.isDragging = false;
            },
            onInputChange(e) {
                const files = e.target.files;

                Array.from(files).forEach(file => this.addImage(file));
            },
            onDrop(e) {
              e.preventDefault();
              e.stopPropagation();

              this.isDragging = false;

              const files = e.dataTransfer.files;

              Array.from(files).forEach(file => this.addImage(file));
            },
            addImage(file) {
                if(!file.type.match('image.*')) {
                    // console.log('${file.name} is not an image');
                    this.$toastr.e(`${file.name} is not an image`);
                    return;
                }

                this.files.push(file);

                const img = new Image(),
                    reader = new FileReader();

                reader.onload = (e) => this.images.push(e.target.result);

                reader.readAsDataURL(file);
            },
            getFileSize(size) {
              const fSExt = ['Bytes', 'KB', 'MB', 'GB'];
              let i = 0;

              while (size > 900) {
                  size /= 1024;
                  i++;
              }

              return `${(Math.round(size * 100) / 100)} ${fSExt[i]}`;
            },
            upload() {
                const formData = new FormData();

                this.files.forEach(file => {
                    formData.append('images[]', file, file.name);
                });

                axios.post('/management/galleries', formData)
                    .then(response => {
                       // alert('images have been uploaded successfully');
                       this.$toastr.s('All images uploaded successfully');
                       this.images = [];
                       this.files = [];
                    });
            }
        }
    }
</script>

<style lang="scss">
    @import '~vue-toastr/src/vue-toastr.scss';

    .uploader {
        width: 100%;
        background: #3490dc;
        color: white;
        padding: 40px 15px;
        text-align: center;
        border-top: 3px dashed #59b4ff;
        border-bottom: 3px dashed #59b4ff;
        font-size: 1.3rem;
        position: relative;

        &.dragging {
            background: white;
            color: #3490dc;
            border-top: 3px dashed #3490dc;
            border-bottom: 3px dashed #3490dc;

            .file-input label {
                background: #3490dc;
                color: white;
            }
        }

        i {
            font-size: 6rem;
        }

        .file-input {
            width: 200px;
            margin: auto;
            height: 68px;
            position: relative;

            label,
            input {
                background: white;
                color: #3490dc;
                width: 100%;
                position: absolute;
                left: 0;
                top: 0;
                padding: 10px;
                border-radius: 4px;
                margin-top: 10px;
                cursor: pointer;
            }

            input {
                opacity: 0;
                z-index: -1;
            }
        }
        .images-preview {
            display: flex;
            flex-wrap: wrap;
            margin-top: 20px;

            .img-wrapper {
                width: 160px;
                display: flex;
                flex-direction: column;
                margin: 10px;
                height: 150px;
                justify-content: space-between;
                background: white;
                box-shadow: 5px 5px 20px #343434;

                img {
                    max-height: 105px;
                }
            }

            .details {
                font-size: 12px;
                background: white;
                color: black;
                display: flex;
                flex-direction: column;
                align-items: self-start;
                padding: 3px 6px;

                .name {
                    overflow: hidden;
                    height: 18px;
                }
            }
        }

        .upload-control {
            position: absolute;
            width: 100%;
            background: white;
            top: 0;
            left: 0;
            padding: 10px;
            padding-bottom: 4px;
            text-align: right;

            button, label {
                background: #3490dc;
                border: 2px solid #3ac1ff;
                border-radius: 3px;
                color: white;
                font-size: 15px;
                cursor: pointer;
            }

            label {
                padding: 2px 5px;
                margin-right: 10px;
            }
        }
    }
</style>