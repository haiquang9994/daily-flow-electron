<template>
    <div class="container">
        <div class="container-input">
            <input
                type="text"
                v-model="input"
                ref="input"
                @keyup.enter="save"
                placeholder="What are you doing?"
            />
        </div>
        <div class="container-list">
            <div style="text-align: center;">
                <a-date-picker v-model="selected_date" />
            </div>
            <div class="items" ref="items">
                <ul>
                    <span v-if="loading" style="display: block; text-align: center; color: red;">Loading...</span>
                    <template v-else-if="items.length > 0">
                        <li v-for="item in items" :key="item.id">
                            <span>{{ item.content }}</span>
                            <span>{{ timeFormat(item.created_at) }}</span>
                        </li>
                    </template>
                    <span style="display: block; text-align: center; color: red;" v-else>Nothing!</span>
                </ul>
                <a-button class="button" type="danger" @click="logout">Logout</a-button>
            </div>
        </div>
    </div>
</template>

<script>
import moment from 'moment'

export default {
    data() {
        return {
            items: [],
            input: '',
            today: moment(),
            selected_date: moment(),
            error: false,
            loading: false,
            saving: false
        }
    },
    methods: {
        load() {
            this.loading = true
            let api = localStorage.getItem('api')
            let secret = localStorage.getItem('secret')
            this.$http.get(api + '/daily-flow?date=' + this.selected_date.format('YYYY-MM-DD'), {
                headers: {
                    Authorization: 'Bearer ' + secret
                }
            })
                .then(res => {
                    if (res.status === 200) {
                        let data = res.data
                        if (data.status) {
                            this.items = data.data
                        }
                    }
                    this.loading = false
                })
                .catch(e => {
                    this.loading = false
                    console.log(e)
                })
        },
        save(event) {
                event.preventDefault()
                let api = localStorage.getItem('api')
                let secret = localStorage.getItem('secret')
                let content = this.input.toString()
                this.input = ''
                this.$refs.items.scrollTo(0, 0)
                this.$http.post(api + '/daily-flow', JSON.stringify({
                    content
                }), {
                    headers: {
                        'Content-Type': 'application/json',
                        Authorization: 'Bearer ' + secret
                    }
                })
                    .then(res => {
                        if (res.status === 200) {
                            let data = res.data
                            if (data.status) {
                                this.items = [data.data, ...this.items]
                            }
                        }
                    })
                    .catch(e => {
                        console.log(e)
                    })
        },
        timeFormat(date) {
            let data = moment(date)
            return data.format('HH:mm:ss')
        },
        logout() {
            localStorage.removeItem('secret')
            this.$router.push('/login')
        }
    },
    watch: {
        selected_date(value) {
            if (value) {
                this.load()
            }
        }
    },
    created() {
        this.$http.interceptors.response.use(response => {
            return response
        }, error => {
            if (error.response.status === 403) {
                this.logout()
            }
            return Promise.reject(error)
        })
    },
    mounted() {
        let api = localStorage.getItem('api')
        let secret = localStorage.getItem('secret')
        if (!api || !secret) {
            this.$router.push('/login')
        } else {
            this.load()
            setTimeout(() => {
                this.$refs.input.focus()
            }, 0)
        }
    }
}
</script>

<style lang="scss" scoped>
.container {
    display: flex;
    flex-direction: column;
    height: 100vh;
}
.container-input {
    flex: 0 0 80px;
    max-height: 80px;
    display: table;
    width: 100%;
    input {
        height: 80px;
        width: 100%;
        text-align: center;
        font-size: 34px;
        outline: none;
        border: none;
    }
}
.container-list {
    flex: 0 0 calc(100% - 80px);
    max-height: calc(100% - 80px);
    font-size: 18px;
    line-height: 1.5;
    position: relative;
    .items {
        overflow: auto;
        position: absolute;
        top: 55px;
        left: 0;
        right: 0;
        bottom: 0;
        ul {
            max-width: 100%;
            width: 450px;
            margin: 0 auto;
            li {
                display: flex;
                padding: 5px;
                span {
                    &:nth-child(1) {
                        width: 100%;
                    }
                    &:nth-child(2) {
                        padding-left: 10px;
                    }
                }
                &:nth-child(odd) {
                    background-color: #f2f2f2;
                }
                &:nth-child(even) {
                    background-color: #ffffff;
                }
            }
        }
        .button {
            position: absolute;
            right: 10px;
            bottom: 10px;
        }
    }
}
</style>

<style>
</style>