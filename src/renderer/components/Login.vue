<template>
    <div class="box">
        <a-input class="input" placeholder="Your API!" v-model="api" @keyup.enter="login" />
        <a-input-password class="input" placeholder="Your secret code!" v-model="secret" @keyup.enter="login" />
        <a-button class="button" type="primary" @click="login" v-if="api && secret">Login</a-button>
    </div>
</template>

<script>
import md5 from 'md5'

export default {
    data() {
        return {
            api: '',
            secret: '',
            http_api: ''
        }
    },
    methods: {
        login() {
            this.$http.post(this.http_api + '/daily-flow-login', JSON.stringify({
                secret: md5(this.secret)
            }), {
                headers: {
                    'Content-Type': 'application/json'
                }
            })
                .then(res => {
                    if (res.status === 200 && res.data.status === true) {
                        localStorage.setItem('secret', md5(this.secret))
                        localStorage.setItem('api', this.http_api)
                        localStorage.removeItem('src_secret')
                        this.$router.push('/landing-page')
                    }
                })
        }
    },
    watch: {
        api(value) {
            let http_api = ''
            if (value) {
                if (value.startsWith('http')) {
                    http_api = value.toString()
                } else {
                    http_api = 'https://' + value.toString()
                }
                if (!http_api.endsWith('/')) {
                    http_api += '/'
                }
                this.http_api = http_api + 'api/v1'
            } else {
                this.http_api = ''
            }
            localStorage.setItem('src_api', value || '')
        },
        secret(value) {
            localStorage.setItem('src_secret', value || '')
        }
    },
    mounted() {
        this.api = localStorage.getItem('src_api')
        this.secret = localStorage.getItem('src_secret')
    }
}
</script>

<style lang="scss" scoped>
.box {
    max-width: 100%;
    width: 320px;
    margin: 0 auto;
    text-align: center;
    padding-top: 100px;
}
.input {
    text-align: center;
    margin-bottom: 12px;
}
</style>