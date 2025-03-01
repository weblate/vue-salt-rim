<template>
    <PageHeader>
        {{ $t('profile') }}
    </PageHeader>
    <div class="settings-page">
        <div class="settings-page__nav">
            <Navigation />
        </div>
        <div class="settings-page__content">
            <OverlayLoader v-if="isLoading" />
            <form @submit.prevent="submit">
                <div class="form-group">
                    <label class="form-label form-label--required" for="name">{{ $t('user.name') }}:</label>
                    <input class="form-input" type="text" id="name" v-model="user.name" required>
                </div>
                <div class="form-group">
                    <label class="form-label form-label--required" for="email">{{ $t('email') }}:</label>
                    <input class="form-input" type="email" id="email" v-model="user.email" required>
                </div>
                <div class="form-group">
                    <label class="form-label" for="new-password">{{ $t('new-password') }}:</label>
                    <input class="form-input" type="password" id="new-password" v-model="user.password">
                </div>
                <div class="form-group">
                    <label class="form-label" for="repeat-new-password">{{ $t('repeat-password') }}:</label>
                    <input class="form-input" type="password" id="repeat-new-password" v-model="user.repeatPassword">
                </div>
                <div class="form-group">
                    <label class="form-label" for="repeat-new-password">{{ $t('ui-language') }}:</label>
                    <select class="form-select" v-model="currentLocale">
                        <option :value="locale" v-for="locale in $i18n.availableLocales">{{ $t('locales.' + locale) }}</option>
                    </select>
                </div>
                <div class="form-actions">
                    <button class="button button--dark" type="submit">{{ $t('save') }}</button>
                </div>
            </form>
        </div>
    </div>
</template>

<script>
import ApiRequests from "@/ApiRequests";
import Auth from "@/Auth";
import OverlayLoader from '@/components/OverlayLoader.vue'
import PageHeader from '@/components/PageHeader.vue'
import Navigation from '@/components/Settings/Navigation.vue'

export default {
    data() {
        return {
            isLoading: false,
            user: {},
            currentLocale: this.$i18n.locale
        };
    },
    components: {
        OverlayLoader,
        Navigation,
        PageHeader
    },
    created() {
        document.title = `${this.$t('profile')} \u22C5 ${this.site_title}`

        this.isLoading = true;

        ApiRequests.fetchUser().then(data => {
            this.user = data
            this.isLoading = false;
        }).catch(e => {
            this.$toast.error(e.message);
            this.isLoading = false;
        })
    },
    methods: {
        submit() {
            this.isLoading = true;

            const postData = {
                email: this.user.email,
                name: this.user.name,
                password: this.user.password,
                password_confirmation: this.user.repeatPassword,
            };

            if (this.currentLocale) {
                window.localStorage.setItem('ui-language', this.currentLocale);
                this.$i18n.locale = this.currentLocale;
            }

            ApiRequests.updateUser(postData).then(data => {
                Auth.rememberUser(data);
                this.isLoading = false;
                this.$toast.default(this.$t('profile-updated'));
                this.user.password = null;
                this.user.repeatPassword = null;
            }).catch(e => {
                this.isLoading = false;
                this.$toast.error(e.message);
            })
        }
    }
}
</script>
