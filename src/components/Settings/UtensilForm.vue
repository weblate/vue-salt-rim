<template>
    <form @submit.prevent="submit">
        <OverlayLoader v-if="isLoading" />
        <div class="dialog-title">{{ dialogTitle }}</div>
        <div class="form-group">
            <label class="form-label form-label--required" for="name">{{ $t('name') }}:</label>
            <input class="form-input" type="text" id="name" v-model="utensil.name" required>
        </div>
        <div class="form-group">
            <label class="form-label" for="description">{{ $t('description') }}:</label>
            <textarea rows="5" class="form-input" id="description" v-model="utensil.description"></textarea>
        </div>
        <div class="dialog-actions">
            <button class="button button--outline" @click.prevent="$emit('utensilDialogClosed')">{{ $t('cancel') }}</button>
            <button class="button button--dark" type="submit">{{ $t('save') }}</button>
        </div>
    </form>
</template>

<script>
import ApiRequests from "./../../ApiRequests.js";
import OverlayLoader from './../OverlayLoader.vue'

export default {
    props: ['sourceData', 'dialogTitle'],
    data() {
        return {
            isLoading: false,
            categoryId: null,
            utensil: this.sourceData,
        };
    },
    components: {
        OverlayLoader
    },
    methods: {
        submit() {
            this.isLoading = true;

            const postData = {
                name: this.utensil.name,
                description: this.utensil.description,
            };

            if (this.utensil.id) {
                ApiRequests.updateUtensil(this.utensil.id, postData).then(() => {
                    this.isLoading = false;
                    this.$toast.default(this.$t('utensils.update-success'));
                    this.$emit('utensilDialogClosed')
                }).catch(e => {
                    this.$toast.error(e.message);
                    this.isLoading = false;
                })
            } else {
                ApiRequests.saveUtensil(postData).then(() => {
                    this.isLoading = false;
                    this.$toast.default(this.$t('utensils.add-success'));
                    this.$emit('utensilDialogClosed')
                }).catch(e => {
                    this.$toast.error(e.message);
                    this.isLoading = false;
                })
            }
        }
    }
}
</script>

<style scope>
</style>
