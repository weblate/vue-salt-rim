<template>
    <div class="note">
        <h5 class="note-title">{{ createdDate }} &middot; <a href="#" @click.prevent="deleteNote">{{ $t('remove') }}</a></h5>
        <div class="note-content">{{ note.note }}</div>
    </div>
</template>

<script>
import dayjs from 'dayjs';
import ApiRequests from './../../ApiRequests.js';

export default {
    props: ['note'],
    computed: {
        createdDate() {
            const date = dayjs(this.note.created_at).toDate();

            return this.$d(date, 'long');
        }
    },
    methods: {
        deleteNote() {
            this.$confirm(this.$t('note.confirm-delete'), {
                onResolved: (dialog) => {
                    dialog.close();
                    ApiRequests.deleteNote(this.note.id).then(() => {
                        this.$toast.default(this.$t('note.delete-success'));
                        this.$emit('noteDeleted');
                    }).catch(e => {
                        this.$toast.error(e.message);
                    })
                }
            });
        }
    }
}
</script>

<style scoped>
.note {
    margin-bottom: 1rem;
    background-color: rgba(255, 255, 255, .3);
    padding: 1rem;
    border-radius: 3px;
}

.dark-theme .note {
    background-color: rgba(0, 0, 0, .2);
}

.note:last-child {
    margin: 0;
}

.note-title {
    font-size: 0.75rem;
    opacity: 0.65;
}

.note-content {
    font-size: 1rem;
}
</style>
