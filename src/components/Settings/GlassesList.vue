<template>
    <PageHeader>
        {{ $t('glass-types') }}
        <template #actions>
            <Dialog v-model="showDialog">
                <template #trigger>
                    <button type="button" class="button button--dark" @click.prevent="openDialog($t('glass-type.add'), {})">{{ $t('glass-type.add') }}</button>
                </template>
                <template #dialog>
                    <GlassForm :source-glass="editGlass" :dialog-title="dialogTitle" @glass-dialog-closed="refreshGlasses" />
                </template>
            </Dialog>
        </template>
    </PageHeader>
    <div class="settings-page">
        <div class="settings-page__nav">
            <Navigation />
        </div>
        <div class="settings-page__content">
            <OverlayLoader v-if="isLoading" />
            <div class="block-container block-container--padded">
                <table class="table">
                    <thead>
                        <tr>
                            <th>{{ $t('name') }} / {{ $t('description') }}</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="glass in glasses">
                            <td>
                                <a href="#" @click.prevent="openDialog($t('glass-type.edit'), glass)">{{ glass.name }}</a>
                                <br>
                                <small>{{ $t('cocktails') }}: {{ glass.cocktails_count }} &middot; {{ glass.description }}</small>
                            </td>
                            <td style="text-align: right;">
                                <a class="list-group__action" href="#" @click.prevent="deleteGlass(glass)">{{ $t('remove') }}</a>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script>
import ApiRequests from "@/ApiRequests";
import OverlayLoader from '@/components/OverlayLoader.vue'
import PageHeader from '@/components/PageHeader.vue'
import Navigation from '@/components/Settings/Navigation.vue'
import Dialog from '@/components/Dialog/Dialog.vue'
import GlassForm from '@/components/Settings/GlassForm.vue'

export default {
    components: {
        OverlayLoader,
        Navigation,
        PageHeader,
        GlassForm,
        Dialog
    },
    data() {
        return {
            isLoading: false,
            showDialog: false,
            dialogTitle: 'Glass data',
            editGlass: {},
            glasses: [],
        }
    },
    created() {
        document.title = `${this.$t('glass-types')} \u22C5 ${this.site_title}`

        this.refreshGlasses()
    },
    methods: {
        refreshGlasses() {
            this.showDialog = false;
            this.isLoading = true;
            ApiRequests.fetchGlasses().then(data => {
                this.glasses = data;
                this.isLoading = false;
            }).catch(e => {
                this.$toast.error(e.message);
            })
        },
        openDialog(title, obj) {
            this.dialogTitle = title
            this.editGlass = obj
            this.showDialog = true;
        },
        deleteGlass(glass) {
            this.$confirm(this.$t('glass-type.confirm-delete', {name: glass.name}), {
                onResolved: (dialog) => {
                    this.isLoading = true
                    dialog.close()
                    ApiRequests.deleteGlass(glass.id).then(() => {
                        this.isLoading = false;
                        this.$toast.default(this.$t('glass-type.delete-success'));
                        this.refreshGlasses()
                    }).catch(e => {
                        this.$toast.error(e.message);
                        this.isLoading = false;
                    })
                }
            });
        }
    }
}
</script>
