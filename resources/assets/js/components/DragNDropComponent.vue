<style>
    .dropzone {
        border-radius: 2px;
        border: 2px dashed #212529;
        min-height: 15rem;
        font-size: 3rem;
        cursor: default;
        text-align: center;
    }
</style>

<template>
    <div class="row">
        <div id="dropzone" class="dropzone d-flex justify-content-center align-items-center offset-1 col-10"></div>
    </div>
</template>

<script>
    import Dropzone from '../../../../vendor/enyo/dropzone/dist/dropzone';

    Dropzone.autoDiscover = false;

    function clearFlashMessages() {
        const flashWrapper = document.getElementsByClassName('flash__wrapper');
        const countBlocks = flashWrapper.length;

        for (let i = 0; i < countBlocks; i++) {
            while (flashWrapper[i].firstChild) {
                flashWrapper[i].removeChild(flashWrapper[i].firstChild);
            }
        }
    }

    export default {
        mounted() {
            let thumbnailDataUrl = '';

            const dropzone = new Dropzone('div#dropzone', {
                url: 'load',
                headers: {
                    'X-CSRF-TOKEN': document.head.querySelector('meta[name="csrf-token"]').content
                },
                dictDefaultMessage: 'Drop Gif Here',
                dictUploadCanceled: 'Upload was failed, try again',
                acceptedFiles: 'image/gif',
                maxFilesize: 36,
                maxThumbnailFilesize: 36,
                maxFiles: 1,
                clickable: true,
                previewTemplate: '',

                addedfile: function (file) {},

                thumbnail: function (file, dataUrl) {
                    thumbnailDataUrl = dataUrl;
                }
            });

            dropzone.on('success', (file, response) => {
                if (!response.url) {
                    dropzone.removeFile(file);
                    this.flashError(response);
                    return false;
                }

                this.$store.commit('setUrl', response.url);
                this.$store.commit('setFilename', response.filename);
                this.$store.commit('setPreview', thumbnailDataUrl);

                this.$router.push(response.redirectUrl);
            });

            dropzone.on('sending', (file, xhr) => {
                xhr.ontimeout = () => {
                    dropzone.removeFile(file);
                };
            });

            dropzone.on('error', (file, response) => {
                dropzone.removeFile(file);
                this.flashError(response);
            });
        },
        beforeDestroy() {
            clearFlashMessages();
        }
    };
</script>
