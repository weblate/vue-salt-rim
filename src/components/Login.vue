<template>
    <div class="login-page">
        <Logo></Logo>
        <form @submit.prevent="login">
            <OverlayLoader v-if="isLoading"></OverlayLoader>
            <div class="form-group">
                <label class="form-label" for="email">{{ $t('email') }}:</label>
                <input class="form-input" type="email" id="email" v-model="email" required>
            </div>
            <div class="form-group">
                <label class="form-label" for="password">{{ $t('password') }}:</label>
                <input class="form-input" type="password" id="password" v-model="password" required>
            </div>
            <div class="server-status">
                <div class="server-status__title">Bar Assistant server:</div>
                <a :href="baServer" target="_blank" class="server-status__url">{{ baServer }}</a>
                <div class="server-status__status">
                    <template v-if="baServerAvailable">
                        {{ $t('status') }}: {{ $t('status-available') }} &middot; {{ server.version }}
                    </template>
                    <template v-else>
                        {{ $t('status') }}: {{ $t('status-not-available') }}
                    </template>
                </div>
            </div>
            <div style="text-align: right; margin-top: 20px;" v-if="baServerAvailable">
                <RouterLink class="button button--outline" :to="{ name: 'register' }">{{ $t('register') }}</RouterLink>
                <button type="submit" class="button button--dark" style="margin-left: 5px;" :disabled="!baServerAvailable">{{ $t('login') }}</button>
            </div>
        </form>
    </div>
</template>

<script>
import Auth from '../Auth'
import ApiRequests from '@/ApiRequests';
import OverlayLoader from '@/components/OverlayLoader.vue';
import Logo from '@/components/Logo.vue';

export default {
    data() {
        return {
            isLoading: false,
            email: null,
            password: null,
            baServer: window.srConfig.API_URL,
            meiliServer: window.srConfig.MEILISEARCH_URL,
            server: {},
            meiliServer: {}
        }
    },
    components: {
        OverlayLoader,
        Logo
    },
    created() {
        this.isLoading = true
        ApiRequests.fetchApiVersion().then(data => {
            this.server = data
            this.isLoading = false
        }).catch(() => {
            this.isLoading = false
        })
    },
    computed: {
        baServerAvailable() {
            return this.server.version != null;
        },
        meiliServerAvailable() {
            return this.meiliServer.status == 'available';
        }
    },
    methods: {
        login() {
            this.isLoading = true
            const redirectPath = this.$route.query.redirect || '/'

            ApiRequests.fetchLoginToken(this.email, this.password).then(token => {
                Auth.rememberToken(token)
                ApiRequests.fetchUser().then(userData => {
                    Auth.rememberUser(userData)

                    this.$router.push(redirectPath);
                })
            }).catch(e => {
                this.isLoading = false
                this.$toast.error(e.message)
            });
        }
    }
}
</script>

<style scoped>
.server-status {
    background: #fff;
    margin-bottom: 1rem;
    padding: 0.5rem;
    border-radius: 0.25rem;
    border: 2px solid var(--clr-gray-200);
}

.dark-theme .server-status {
    background-image: linear-gradient(160deg, var(--clr-accent-purple), var(--clr-dark-main-950) 70%);
    border: 0;
    border-top: 1px solid var(--clr-accent-purple);
}

.server-status__title {
    font-size: 0.75rem;
}

.server-status__url {
    font-weight: var(--fw-bold);
}

.server-status__status {
    font-size: 0.85rem;
}
</style>
