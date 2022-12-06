<template>
    <div v-if="progress > 0" :class="className('fileItem')">
        <div :class="className('removeFile')" @click="onDeleteItem">
            <trash-icon :class="className('removeIcon')" />
        </div>
        <div :class="className('progressUpload')">
            <div :class="className('progressBar')" :style="{ 'width': `${progress}%` }">
                <div :class="className('progressStatus')">
                    <div v-if="status === 'pending'">
                        {{ progress | localize }}
                    </div>
                    <tick-icon v-else />
                </div>
            </div>
            <div :class="className('progressName')">
                {{ fileName }}
            </div>
        </div>
    </div>
</template>

<script>


export default {
    name: 'FileItem',
    components: {
        TickIcon,
        TrashIcon,
    },
    mixins: [className],
    props: {
        file: File,
        id: stringFactory(true),
        status: stringFactory(true, 'pending'),
    },
    data() {
        return {
            progress: 0,
            isUploading: true,
            cancelToken: null,
            fileName: '',
        };
    },
    mounted() {
        this.uploadAttachment(this.file);
    },
    methods: {
        onCancel() {
            this.$emit('cancel', this.id);
            this.cancelToken();
        },
        onDelete() {
            this.$emit('cancel', this.id);
        },
        onDone(token) {
            this.$emit('done', this.id, token);
        },
        uploadAttachment(file) {
            const formData = new FormData();
            const fileName = file.name.replace(/\s/gi, '_');
            this.fileName = fileName;
            this.isUploading = true;

            formData.append('file', file);
            formData.append('filename', fileName);

            upload(formData, this.updateProgress, this.cancelExecuter)
                .then(({ data }) => {
                    const { token } = data;
                    this.onDone(token);
                    this.isUploading = false;
                });
        },
        updateProgress(progressEvent) {
            const uploadPercentage = parseInt(
                Math.round((progressEvent.loaded * 100) / progressEvent.total),
                10,
            );
            this.progress = uploadPercentage;
        },
        cancelExecuter(token) {
            this.cancelToken = token;
        },
        onDeleteItem() {
            if (this.file.status === 'done') {
                return this.onDelete();
            }

            return this.onCancel();
        },
    },
};
</script>
